# Gemini CLI 工具映射

skills 使用的是 Claude Code 的工具名。当你在 skill 里遇到这些名字时，请改用当前平台的等价工具：

| Skill 中引用的工具 | Gemini CLI 等价工具 |
|-------------------|----------------------|
| `Read`（读文件） | `read_file` |
| `Write`（创建文件） | `write_file` |
| `Edit`（编辑文件） | `replace` |
| `Bash`（运行命令） | `run_shell_command` |
| `Grep`（搜索文件内容） | `grep_search` |
| `Glob`（按名称找文件） | `glob` |
| `TodoWrite`（任务跟踪） | `write_todos` |
| `Skill` tool（调用 skill） | `activate_skill` |
| `WebSearch` | `google_web_search` |
| `WebFetch` | `web_fetch` |
| `Task` tool（派发 subagent） | 无对应项，Gemini CLI 不支持 subagent |

## 不支持 subagent

Gemini CLI 没有 Claude Code `Task` tool 的对应能力。依赖 subagent 派发的 skills（例如 `subagent-driven-development`、`dispatching-parallel-agents`）会退化为通过 `executing-plans` 在单会话中执行。

## Gemini CLI 额外提供的工具

这些工具在 Gemini CLI 中可用，但 Claude Code 没有直接对应项：

| 工具 | 用途 |
|------|------|
| `list_directory` | 列出文件和子目录 |
| `save_memory` | 跨会话把事实持久化到 `GEMINI.md` |
| `ask_user` | 向用户请求结构化输入 |
| `tracker_create_task` | 更丰富的任务管理（创建、更新、列出、可视化） |
| `enter_plan_mode` / `exit_plan_mode` | 在修改前切换到只读研究模式 |
