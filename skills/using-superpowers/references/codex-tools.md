# Codex 工具映射

skills 使用的是 Claude Code 的工具名。当你在 skill 中遇到这些名字时，请改用当前平台的对应能力：

| Skill 中引用的工具 | Codex 对应能力 |
|-------------------|------------------|
| `Task` tool（派发 subagent） | `spawn_agent`（见下方 [命名 agent 派发](#命名-agent-派发)） |
| 多次 `Task` 调用（并行） | 多次 `spawn_agent` 调用 |
| Task 返回结果 | `wait` |
| Task 自动结束 | 用 `close_agent` 释放槽位 |
| `TodoWrite`（任务跟踪） | `update_plan` |
| `Skill` tool（调用 skill） | skills 原生加载，直接遵循说明即可 |
| `Read`、`Write`、`Edit`（文件） | 使用当前平台的原生文件工具 |
| `Bash`（运行命令） | 使用当前平台的原生 shell 工具 |

## Subagent 派发需要 multi-agent 支持

把下面配置加入 Codex 配置文件 `~/.codex/config.toml`：

```toml
[features]
multi_agent = true
```

这样就能在 `dispatching-parallel-agents`、`subagent-driven-development` 这类 skill 中使用 `spawn_agent`、`wait` 和 `close_agent`。

## 命名 agent 派发

Claude Code 的 skills 会引用类似 `superpowers:code-reviewer` 这样的命名 agent 类型。
Codex 没有命名 agent 注册表，`spawn_agent` 只会基于内置角色（`default`、`explorer`、`worker`）创建通用 agent。

当某个 skill 要求派发命名 agent 类型时：

1. 找到该 agent 的 prompt 文件（例如 `agents/code-reviewer.md`，或 skill 本地模板如 `code-quality-reviewer-prompt.md`）
2. 读取 prompt 内容
3. 填充模板占位符（例如 `{BASE_SHA}`、`{WHAT_WAS_IMPLEMENTED}` 等）
4. 用填充后的内容作为 `message`，派发一个 `worker` agent

| Skill 中的指令 | Codex 等价写法 |
|-----------------|------------------|
| `Task tool (superpowers:code-reviewer)` | 用 `code-reviewer.md` 的内容执行 `spawn_agent(agent_type="worker", message=...)` |
| 带内联 prompt 的 `Task tool (general-purpose)` | 用同样的 prompt 执行 `spawn_agent(message=...)` |

### Message 包装方式

`message` 参数属于用户级输入，不是 system prompt。为了最大化遵循度，建议这样组织：

```
Your task is to perform the following. Follow the instructions below exactly.

<agent-instructions>
[filled prompt content from the agent's .md file]
</agent-instructions>

Execute this now. Output ONLY the structured response following the format
specified in the instructions above.
```

- 使用任务委派式 framing（“Your task is...”），而不是人格设定式 framing（“You are...”）
- 用 XML 标签包裹指令，模型通常会把带标签的内容视为更高优先级
- 最后加一个明确的执行指令，避免模型只是在概括这些说明

### 什么时候可以移除这个变通方案

这个方案是在弥补 Codex 插件系统目前还不支持 `plugin.json` 中 `agents` 字段的缺口。等 `RawPluginManifest` 支持 `agents` 字段后，插件就可以像现有 `skills/` 符号链接那样，把 `agents/` 也链接进去，skills 也就能直接派发命名 agent 了。

## 环境检测

那些会创建 worktree 或收尾分支的 skills，在继续之前应该用只读 git 命令检测当前环境：

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` → 已经在 linked worktree 中（跳过创建）
- `BRANCH` 为空 → detached HEAD（无法在 sandbox 内建分支 / push / 提 PR）

`using-git-worktrees` 的 Step 0 和 `finishing-a-development-branch` 的 Step 1 展示了各个 skill 如何使用这些信号。

## Codex App 收尾方式

当 sandbox 阻止建分支或 push 操作时（例如在外部管理的 worktree 里处于 detached HEAD），agent 应完成全部提交，并告知用户改用 App 原生控件：

- **“Create branch”**：先命名分支，然后通过 App UI 完成 commit / push / PR
- **“Hand off to local”**：把工作移交给用户本地 checkout

agent 仍然可以运行测试、暂存文件，并给出建议的分支名、提交信息和 PR 描述，供用户直接使用。
