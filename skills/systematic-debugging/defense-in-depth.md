# 纵深防御验证

## 概览

当你修一个由无效数据引发的 bug 时，在某一个点加校验，看起来常常已经够了。但单点校验很容易被其他代码路径、重构或者 mocks 绕过去。

**核心原则：** 在数据流经的每一层都做验证，让这个 bug 从结构上变成“不可能发生”。

## 为什么要多层校验

单层校验：**“我们修掉了这个 bug。”**  
多层校验：**“我们让这个 bug 不可能再出现。”**

不同层能兜住不同情况：
- 入口校验能挡住大部分明显错误
- 业务逻辑校验能挡边界情况
- 环境护栏能避免特定上下文中的危险操作
- 调试日志能在其他层失效时留下取证信息

## 四层防御

### 第 1 层：入口校验
**目的：** 在 API 边界直接拒绝明显无效的输入

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory is not a directory: ${workingDirectory}`);
  }
  // ... proceed
}
```

### 第 2 层：业务逻辑校验
**目的：** 确保数据对这项业务操作来说是有意义的

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ... proceed
}
```

### 第 3 层：环境护栏
**目的：** 在特定上下文中阻止危险操作

```typescript
async function gitInit(directory: string) {
  // 在测试环境中，拒绝在临时目录之外执行 git init
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ... proceed
}
```

### 第 4 层：调试埋点
**目的：** 留下上下文，方便事后取证

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... proceed
}
```

## 如何应用这个模式

当你发现一个 bug 时：

1. **追踪数据流** - 无效值从哪来？最后在哪被使用？
2. **列出所有检查点** - 数据经过了哪些位置？
3. **每层都补校验** - 入口层、业务层、环境层、调试层
4. **逐层测试** - 尝试绕过第 1 层，确认第 2 层仍能接住

## 来自真实会话的示例

Bug：空的 `projectDir` 导致 `git init` 跑到了源代码目录里

**数据流：**
1. 测试初始化 → 空字符串
2. `Project.create(name, '')`
3. `WorkspaceManager.createWorkspace('')`
4. `git init` 最终在 `process.cwd()` 中执行

**补上的四层防御：**
- 第 1 层：`Project.create()` 校验非空 / 存在 / 可写
- 第 2 层：`WorkspaceManager` 校验 `projectDir` 非空
- 第 3 层：`WorktreeManager` 在测试环境下拒绝在 `tmpdir` 外执行 `git init`
- 第 4 层：在 `git init` 前记录 stack trace

**结果：** 1847 个测试全部通过，且该 bug 再也无法复现

## 关键洞察

这四层全都需要。测试过程中，每一层都抓到了其他层没抓住的问题：
- 不同代码路径会绕过入口校验
- mocks 会绕过业务逻辑校验
- 跨平台边界情况需要环境护栏
- 调试日志帮助定位结构性误用

**不要在某一个验证点就停下。** 让每一层都参与防御。
