# Visual Brainstorming 重构：浏览器负责展示，终端负责命令

**日期：** 2026-02-19  
**状态：** Approved  
**范围：** `lib/brainstorm-server/`、`skills/brainstorming/visual-companion.md`、`tests/brainstorm-server/`

## 问题

在 visual brainstorming 流程里，Claude 会把 `wait-for-feedback.sh` 作为后台任务运行，然后阻塞在 `TaskOutput(block=true, timeout=600s)`。这会直接占住整个 TUI，用户无法继续在终端里跟 Claude 交互，浏览器反而变成了唯一输入通道。

Claude Code 的执行模型本质上是 turn-based。单个 turn 内没有办法同时监听两个输入通道。阻塞式 `TaskOutput` 用错了原语，它试图模拟平台本身并不支持的事件驱动行为。

## 设计

### 核心模型

**Browser = 交互式展示层。** 用来展示 mockup，也让用户点击选择选项。选择结果由服务端记录。

**Terminal = 对话通道。** 永远不阻塞，永远可用。用户继续在这里和 Claude 对话。

### 循环

1. Claude 把 HTML 文件写到 session 目录
2. 服务端通过 chokidar 检测到变更，并向浏览器推送 WebSocket reload（这一点保持不变）
3. Claude 结束当前 turn，并提示用户去浏览器看结果，再回到终端反馈
4. 用户查看浏览器，可选地点击某个选项，然后在终端输入反馈
5. 下一轮 turn 中，Claude 读取 `$SCREEN_DIR/.events` 里的浏览器交互事件流（点击、选择等），再与终端文本合并
6. 继续迭代，或推进到下一步

没有后台任务。没有 `TaskOutput` 阻塞。没有轮询脚本。

### 关键删除项：`wait-for-feedback.sh`

整个删除。它原本的职责是把“服务端把事件打到 stdout”和“Claude 需要接收到这些事件”之间桥接起来。现在 `.events` 文件替代了它：服务端直接把用户交互事件写进去，而 Claude 用平台现有的文件读取机制去读它就行。

### 关键新增项：`.events` 文件（每个 screen 的事件流）

服务端会把所有用户交互事件写入 `$SCREEN_DIR/.events`，格式为 JSONL，每行一个 JSON 对象。这意味着 Claude 能拿到当前 screen 的完整交互流，而不只是最后一次选择。它可以看到用户的探索路径，比如先点 A，再点 B，最后停在 C。

示例：

```jsonl
{"type":"click","choice":"a","text":"Option A - Preset-First Wizard","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Manual Config","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid Approach","timestamp":1706000115}
```

- 在单个 screen 生命周期内，这个文件只追加，不覆盖
- 当 chokidar 检测到新的 HTML 文件（新 screen）时，就删除 `.events` 文件，避免旧事件串进来
- 如果 Claude 读取时文件不存在，说明用户没有发生浏览器交互，此时只使用终端文本
- 文件里只保留用户事件（如 `click`），不记录 `server-started`、`screen-added` 这类服务端生命周期事件，保持小而聚焦
- Claude 可以读取完整事件流来理解用户探索过程，也可以只看最后一次 `choice` 事件作为最终选择

## 各文件改动

### `index.js`（服务端）

**A. 把用户事件写入 `.events` 文件。**

在 WebSocket `message` handler 里，在把事件打到 stdout 之后，使用 `fs.appendFileSync` 把事件以 JSON 行形式追加到 `$SCREEN_DIR/.events`。只写用户交互事件（也就是 `source: 'user-event'`），不要写服务端生命周期事件。

**B. 在新 screen 出现时清空 `.events`。**

在 chokidar 的 `add` handler 里（检测到新的 `.html` 文件），如果 `$SCREEN_DIR/.events` 存在，就删掉它。新 screen 的出现才是清空事件流的准确信号，比在 GET `/` 时清空靠谱得多，因为 `/` 每次 reload 都会触发。

**C. 替换 `wrapInFrame` 的内容注入方式。**

现有逻辑是靠 `<div class="feedback-footer">` 做正则锚点，但这个 footer 将被移除。改为在 `#claude-content` 内原本默认内容的位置（`<h2>Visual Brainstorming</h2>` 和说明段落）放入单一占位符：`<!-- CONTENT -->`。然后通过 `frameTemplate.replace('<!-- CONTENT -->', content)` 注入内容。这样更简单，也不依赖模板格式细节。

### `frame-template.html`（UI 外壳）

**移除：**
- `feedback-footer` div（textarea、Send 按钮、label、`.feedback-row`）
- 对应 CSS（`.feedback-footer`、其 label、`.feedback-row` 以及内部 textarea / button 样式）

**新增：**
- 在 `#claude-content` 内部加入 `<!-- CONTENT -->` 占位符，替换现有默认文案
- 在原 footer 所在区域加入一个“选择状态条”，两种状态：
  - 默认：`Click an option above, then return to the terminal`
  - 选择后：`Option B selected — return to terminal to continue`
- 为状态条新增 CSS，视觉权重保持克制，接近现有 header

**保持不变：**
- 顶部 header（"Brainstorm Companion" 标题和连接状态）
- `.main` 包裹层和 `#claude-content` 容器
- 所有组件 CSS（`.options`、`.cards`、`.mockup`、`.split`、`.pros-cons`、placeholder、mock 元素）
- 深浅色主题变量与 media query

### `helper.js`（客户端脚本）

**移除：**
- `sendToClaude()` 函数，以及 “Sent to Claude” 的整页替换逻辑
- `window.send()`（它原本绑定到已删除的 Send 按钮）
- 表单提交 handler，没有 textarea 后已经没有意义，只会制造噪声
- input change handler，同理
- `pageshow` 事件监听（之前是为了解决 textarea 状态残留）

**保留：**
- WebSocket 连接、重连逻辑、事件队列
- reload handler（收到服务端推送后执行 `window.location.reload()`）
- `window.toggleSelect()`，负责选择高亮
- `window.selectedChoice` 状态追踪
- `window.brainstorm.send()` 和 `window.brainstorm.choice()`，它们和已删除的 `window.send()` 不是一回事。二者调用 `sendEvent`，把事件通过 WebSocket 写回服务端，对完整 HTML 页面仍然有用

**收窄：**
- click handler 只捕获 `[data-choice]` 点击，不再拦所有按钮 / 链接。旧版需要这样做，是因为浏览器还承担反馈输入；现在它只负责选择状态追踪

**新增：**
- 当用户点击 `data-choice` 元素时，同时更新状态条文案，展示当前选中的选项

**从 `window.brainstorm` API 中移除：**
- `brainstorm.sendToClaude`，因为它已经不存在

### `visual-companion.md`（技能 指令）

把 “The Loop” 一节改写成上面描述的非阻塞流程，并删除以下所有内容：
- `wait-for-feedback.sh`
- `TaskOutput` 阻塞
- timeout / retry 逻辑（600 秒 timeout、30 分钟上限）
- 描述 `send-to-claude` JSON 的 “User Feedback Format” 一节

**替换成：**
- 新循环（写 HTML → 结束 turn → 用户在终端反馈 → 读取 `.events` → 继续）
- `.events` 文件格式说明
- 明确终端消息是主反馈来源，而 `.events` 只是额外提供浏览器交互上下文

**保留：**
- 服务端启动 / 关闭说明
- fragment 与 full document 的使用说明
- CSS class 参考与可用组件
- 设计建议（根据问题复杂度调整保真度、每屏 2 到 4 个选项等）

### `wait-for-feedback.sh`

**完全删除。**

### `tests/brainstorm-server/server.test.js`

需要更新的测试：
- 之前断言 fragment 响应里有 `feedback-footer`，现在应改成断言有新的选择状态条，或 `<!-- CONTENT -->` 已被替换
- 之前断言 `helper.js` 中存在 `send`，现在应改为新的 API 形态
- 之前断言 `sendToClaude` 对 CSS 变量有依赖，这部分要删（函数已不存在）

## 平台兼容性

服务端代码（`index.js`、`helper.js`、`frame-template.html`）完全平台无关，只使用纯 Node.js 和浏览器 JavaScript。已经在 Codex 的后台终端交互场景下验证可用。

skill 指令（`visual-companion.md`）才是平台适配层。各个平台上的 Claude 用自己的工具去启动服务、读取 `.events` 等。由于新模型不依赖任何平台专属阻塞原语，所以天然更适合跨平台。

## 这次改动带来的能力

- **TUI 始终可用**，visual brainstorming 期间用户仍可继续在终端输入
- **混合输入**，浏览器点击 + 终端文字可以自然合并
- **优雅退化**，即使浏览器没打开或用户不使用它，终端工作流仍然完整
- **结构更简单**，不再有后台任务、轮询脚本和 timeout 管理
- **跨平台**，同一套服务端代码可跑在 Claude Code、Codex 和未来平台上

## 这次改动放弃了什么

- **纯浏览器反馈工作流**：用户必须回到终端继续，不能像旧版那样点击 Send 后原地等待。状态条会引导用户，但确实多了一步
- **浏览器内文本反馈**：textarea 被删除，所有文字反馈都回到终端。这是刻意的，因为终端比一个小 textarea 更适合长文本输入
- **点击后立即得到 Claude 响应**：旧系统用户点 Send 后 Claude 会立刻接上；新系统中用户需要切回终端再发消息。实际通常只多几秒，而且用户反而可以顺手补充更多上下文
