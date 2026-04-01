# 零依赖 Brainstorm Server

用一个只依赖 Node.js 内建模块的 `server.js` 替换 brainstorm companion server 当前 vendored 的 `node_modules`（`express`、`ws`、`chokidar`，共 714 个被跟踪文件）。

## 动机

把 `node_modules` 直接 vendoring 进 git 仓库，会带来供应链风险：冻结的依赖不会自动拿到安全补丁，714 个第三方文件在没有审计的前提下进入仓库，而且对 vendored 代码的修改在提交历史里看起来就像正常业务代码。虽然实际风险不高（因为它只是 localhost-only 的开发服务器），但移除这些依赖其实很直接。

## 架构

一个单独的 `server.js` 文件（约 250 到 300 行），只使用 `http`、`crypto`、`fs` 和 `path`。它承担两个角色：

- **直接运行时**（`node server.js`）：启动 HTTP / WebSocket 服务器
- **被 `require` 时**（`require('./server.js')`）：导出 WebSocket 协议函数，供单元测试使用

### WebSocket 协议

只实现 RFC 6455 的文本帧：

**握手：** 使用客户端的 `Sec-WebSocket-Key` 加上 RFC 6455 规定的 magic GUID 做 SHA-1，计算出 `Sec-WebSocket-Accept`。返回 `101 Switching Protocols`。

**帧解码（客户端 → 服务端）：** 处理三种被 mask 的长度编码：
- 小帧：payload < 126 字节
- 中帧：126-65535 字节（16 位扩展长度）
- 大帧：> 65535 字节（64 位扩展长度）

用 4 字节 mask key 对 payload 做 XOR 解码。返回 `{ opcode, payload, bytesConsumed }`，如果缓冲区还不完整则返回 `null`。拒绝未加 mask 的帧。

**帧编码（服务端 → 客户端）：** 使用未加 mask 的帧，长度编码方式与上面一致。

**支持的 opcode：** TEXT（0x01）、CLOSE（0x08）、PING（0x09）、PONG（0x0A）。对未知 opcode 返回状态码 1003（Unsupported Data）的 close frame。

**明确不做：** 二进制帧、分片消息、扩展（如 permessage-deflate）、subprotocol。这些对 localhost 间的小型 JSON 文本消息没有必要。扩展和 subprotocol 都是在握手阶段协商的，只要不声明支持，它们就永远不会启用。

**缓冲累积：** 每个连接维护自己的 buffer。收到 `data` 时，把数据 append 进去，并循环调用 `decodeFrame`，直到它返回 `null` 或 buffer 被清空。

### HTTP 服务器

三个路由：

1. **`GET /`** - 按 mtime 返回 screen 目录里最新的 `.html`。如果是完整 HTML 文档就直接返回；如果只是 fragment，就用 frame template 包起来，并注入 `helper.js`。返回 `text/html`。如果当前没有任何 `.html` 文件，就返回一个硬编码等待页（“Waiting for Claude to push a screen...”），同样注入 `helper.js`。
2. **`GET /files/*`** - 从 screen 目录中静态提供文件，MIME type 通过一个硬编码扩展名表决定（html、css、js、png、jpg、gif、svg、json）。找不到返回 404。
3. **其他所有请求** - 404。

WebSocket upgrade 通过 HTTP server 的 `'upgrade'` 事件处理，和普通 request handler 分开。

### 配置

环境变量（全部可选）：

- `BRAINSTORM_PORT` - 绑定端口（默认：随机高位端口 49152-65535）
- `BRAINSTORM_HOST` - 绑定地址（默认：`127.0.0.1`）
- `BRAINSTORM_URL_HOST` - 启动 JSON 中用于生成 URL 的主机名（默认：当 host 是 `127.0.0.1` 时使用 `localhost`，否则与 host 相同）
- `BRAINSTORM_DIR` - screen 目录路径（默认：`/tmp/brainstorm`）

### 启动顺序

1. 如果 `SCREEN_DIR` 不存在，就用递归 `mkdirSync` 创建
2. 从 `__dirname` 加载 frame template 和 `helper.js`
3. 在配置好的 host / port 上启动 HTTP server
4. 对 `SCREEN_DIR` 启动 `fs.watch`
5. 成功监听后，把 `server-started` JSON 打到 stdout：`{ type, port, host, url_host, url, screen_dir }`
6. 同时把这份 JSON 写入 `SCREEN_DIR/.server-info`，这样在后台执行、stdout 不可见时，agent 仍能找到连接信息

### 应用层 WebSocket 消息

当收到客户端发送的 TEXT frame 时：

1. 解析成 JSON。如果解析失败，写到 stderr，然后继续。
2. 把它作为 `{ source: 'user-event', ...event }` 记录到 stdout。
3. 如果事件里带有 `choice` 字段，就把 JSON append 到 `SCREEN_DIR/.events`（每行一个事件）。

### 文件监听

用 `fs.watch(SCREEN_DIR)` 替代 chokidar。对于 HTML 文件事件：

- 新文件（`rename` 且文件存在）：如果 `.events` 文件存在就删掉（`unlinkSync`），并把 `screen-added` 作为 JSON 打到 stdout
- 文件变更（`change`）：把 `screen-updated` 作为 JSON 打到 stdout（**不要**清空 `.events`）
- 两类事件都要向所有已连接的 WebSocket 客户端广播 `{ type: 'reload' }`

按文件名做约 100ms 的 debounce，避免 macOS / Linux 上常见的重复事件。

### 错误处理

- 来自 WebSocket 客户端的非法 JSON：写 stderr，继续
- 未支持的 opcode：以状态码 1003 关闭
- 客户端断开：从广播集合中移除
- `fs.watch` 出错：写 stderr，继续
- 不做优雅退出逻辑，进程生命周期由 shell 脚本通过 SIGTERM 管理

## 改动内容

| 之前 | 之后 |
|---|---|
| `index.js` + `package.json` + `package-lock.json` + 714 个 `node_modules` 文件 | `server.js`（单文件） |
| 依赖 express、ws、chokidar | 无依赖 |
| 没有静态文件服务 | `/files/*` 可直接从 screen 目录提供文件 |

## 保持不变的部分

- `helper.js` - 不改
- `frame-template.html` - 不改
- `start-server.sh` - 只做一行更新：把 `index.js` 改成 `server.js`
- `stop-server.sh` - 不改
- `visual-companion.md` - 不改
- 所有现有对外行为和外部契约保持一致

## 平台兼容性

- `server.js` 只依赖跨平台的 Node.js built-ins
- 对于单层目录监听，`fs.watch` 在 macOS、Linux 和 Windows 上都足够可靠
- shell 脚本需要 bash（Windows 上依赖 Git Bash，而这本来就是 Claude Code 的要求）

## 测试

**单元测试**（`ws-protocol.test.js`）：通过 `require server.js` 导出的函数，直接测试 WebSocket 帧编码 / 解码、握手计算以及协议边界情况。

**集成测试**（`server.test.js`）：测试完整服务行为，包括 HTTP 返回、WebSocket 通信、文件监听以及 brainstorming 工作流。使用 `ws` npm 包作为仅测试期客户端依赖（不会分发给最终用户）。
