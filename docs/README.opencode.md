# 面向 OpenCode 的 Superpowers

这是在 [OpenCode.ai](https://opencode.ai) 中使用 Superpowers 的完整指南。

## 安装

把 superpowers 加到 `opencode.json` 的 `plugin` 数组里（全局或项目级都可以）：

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git"]
}
```

重启 OpenCode。插件会通过 Bun 自动安装，并自动注册全部 skills。

可以通过提问验证，例如：“Tell me about your superpowers”

### 从旧的 symlink 安装方式迁移

如果你之前通过 `git clone` 加符号链接的方式安装 superpowers，请先移除旧配置：

```bash
# 删除旧的符号链接
rm -f ~/.config/opencode/plugins/superpowers.js
rm -rf ~/.config/opencode/skills/superpowers

# 如有需要，删除克隆下来的仓库
rm -rf ~/.config/opencode/superpowers

# 如果曾为 superpowers 添加过 skills.paths，也要从 opencode.json 中删掉
```

然后按上面的安装步骤重新配置。

## 使用

### 查找 Skills

使用 OpenCode 原生的 `skill` 工具列出所有可用 skills：

```
use skill tool to list skills
```

### 加载 Skill

```
use skill tool to load superpowers/brainstorming
```

### 个人 Skills

在 `~/.config/opencode/skills/` 下创建你自己的 skills：

```bash
mkdir -p ~/.config/opencode/skills/my-skill
```

然后创建 `~/.config/opencode/skills/my-skill/SKILL.md`：

```markdown
---
name: my-skill
description: Use when [condition] - [what it does]
---

# 我的 Skill

[在这里写你的 skill 内容]
```

### 项目 Skills

在项目内部的 `.opencode/skills/` 下创建项目专属 skills。

**Skill 优先级：** 项目 skills > 个人 skills > Superpowers skills

## 更新

每次重启 OpenCode 时，Superpowers 都会自动更新。每次启动都会从 git 仓库重新安装插件。

如果要固定某个版本，可以使用分支或 tag：

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git#v5.0.3"]
}
```

## 工作原理

这个插件主要做两件事：

1. 通过 `experimental.chat.system.transform` hook **注入 bootstrap 上下文**，让每次对话都具备 superpowers 意识。
2. 通过 `config` hook **注册 skills 目录**，使 OpenCode 无需符号链接或手动配置就能发现全部 superpowers skills。

### 工具映射

为 Claude Code 编写的 skills 会自动适配到 OpenCode：

- `TodoWrite` → `todowrite`
- 带 subagent 的 `Task` → OpenCode 的 `@mention` 系统
- `Skill` tool → OpenCode 原生 `skill` 工具
- 文件操作 → OpenCode 原生工具

## 故障排查

### 插件没有加载

1. 检查 OpenCode 日志：`opencode run --print-logs "hello" 2>&1 | grep -i superpowers`
2. 确认 `opencode.json` 里的插件配置行正确
3. 确保你运行的是较新的 OpenCode 版本

### 找不到 Skills

1. 使用 OpenCode 的 `skill` 工具列出当前可用 skills
2. 确认插件已经加载（见上文）
3. 每个 skill 都需要带有效 YAML frontmatter 的 `SKILL.md` 文件

### Bootstrap 没有出现

1. 检查 OpenCode 版本是否支持 `experimental.chat.system.transform` hook
2. 配置变更后重启 OpenCode

## 获取帮助

- 问题反馈：https://github.com/obra/superpowers/issues
- 主文档：https://github.com/obra/superpowers
- OpenCode 文档：https://opencode.ai/docs/
