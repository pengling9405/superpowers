# Superpowers 中文整理版

> 说明：这是 `superpowers` 的中文整理版入口文档，用于帮助中文读者快速理解这套技能驱动的软件开发工作流。英文原文保持不变。

## 这是什么

`Superpowers` 是一套面向编码代理的软件开发工作流，建立在一组可组合的 `skills` 之上，并配合一套初始说明，确保代理在合适的时候自动调用这些技能。

它的核心思想是：

- 不要一上来就写代码
- 先把需求和规格讲清楚
- 再形成可执行计划
- 再通过技能驱动的开发流程实施
- 全程坚持 TDD、审查、验证和收尾

## 工作原理

它从你打开编码代理的那一刻开始介入。

当代理识别到你在 build 某个东西时，它不会立刻写代码，而是先退一步，搞清楚你真正要解决的问题。

典型路径是：

1. 从对话中提炼设计规格
2. 分段展示设计，让你能真正审阅并确认
3. 在设计确认后生成实现计划
4. 按计划调度子代理或按批次执行
5. 在实现中强制 RED → GREEN → REFACTOR
6. 通过代码审查与最终验证收尾

## 安装方式

### Claude Code 官方 marketplace

```bash
/plugin install superpowers@claude-plugins-official
```

### Claude Code（自定义 marketplace）

```bash
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

### Cursor

在 Cursor Agent chat 中：

```text
/add-plugin superpowers
```

### Codex

对 Codex 说：

```text
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md
```

详细说明见：[docs/README.codex.md](/Users/zhanyu/projects/superpowers/docs/README.codex.md)

### OpenCode

对 OpenCode 说：

```text
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

### Gemini CLI

```bash
gemini extensions install https://github.com/obra/superpowers
```

## 基本工作流

1. **brainstorming**
   - 在写代码前触发
   - 通过追问、比较方案、分段展示设计来收敛问题
2. **using-git-worktrees**
   - 在设计确认后创建隔离工作区和独立分支
3. **writing-plans**
   - 把工作拆成足够小、可验证、可审查的任务
4. **subagent-driven-development / executing-plans**
   - 进入实施阶段，使用 fresh subagent 推进任务
5. **test-driven-development**
   - 强制 RED / GREEN / REFACTOR
6. **requesting-code-review**
   - 在任务间插入代码审查
7. **finishing-a-development-branch**
   - 在任务完成时统一做收尾、验证与合并决策

## 仓库里有什么

### 测试

- `test-driven-development`

### 调试

- `systematic-debugging`
- `verification-before-completion`

### 协作与实施

- `brainstorming`
- `writing-plans`
- `executing-plans`
- `dispatching-parallel-agents`
- `requesting-code-review`
- `receiving-code-review`
- `using-git-worktrees`
- `finishing-a-development-branch`
- `subagent-driven-development`

### 元技能

- `writing-skills`
- `using-superpowers`

## 方法论

`Superpowers` 反复强调几件事：

- **测试先行**
- **系统化优于拍脑袋**
- **优先降低复杂度**
- **没有证据就不要宣布完成**

## 适合谁

- 想把编码代理从“即时补全器”升级成“系统化执行者”的开发者
- 需要严格工程流程的 solo builder
- 需要子代理、worktree、TDD 和审查协同的人

## 推荐阅读顺序

1. `README.md`
2. `docs/README.codex.md`
3. `skills/using-superpowers/SKILL.md`
4. `skills/brainstorming/SKILL.md`
5. `skills/writing-plans/SKILL.md`
6. `skills/test-driven-development/SKILL.md`
7. `skills/subagent-driven-development/SKILL.md`

## 相关文件

- 英文原文：[README.md](/Users/zhanyu/projects/superpowers/README.md)
- Codex 指南：[docs/README.codex.md](/Users/zhanyu/projects/superpowers/docs/README.codex.md)

