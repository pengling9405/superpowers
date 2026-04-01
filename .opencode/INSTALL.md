# 为 OpenCode 安装 Superpowers

## 前置要求

- 已安装 [OpenCode.ai](https://opencode.ai)

## 安装

把 superpowers 加到 `opencode.json` 的 `plugin` 数组里（全局或项目级都可以）：

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git"]
}
```

重启 OpenCode。就这些，插件会自动安装并注册所有 skills。

可以通过提问来验证，例如：“Tell me about your superpowers”

## 从旧的 symlink 安装方式迁移

如果你之前是通过 `git clone` 加符号链接的方式安装 superpowers，请先移除旧配置：

```bash
# 删除旧的符号链接
rm -f ~/.config/opencode/plugins/superpowers.js
rm -rf ~/.config/opencode/skills/superpowers

# 如有需要，删除克隆下来的仓库
rm -rf ~/.config/opencode/superpowers

# 如果你曾为 superpowers 添加过 技能.paths，也要从 opencode.json 里删掉
```

然后按上面的安装步骤重新配置。

## 使用方式

使用 OpenCode 原生的 `skill` 工具：

```
use skill tool to list skills
use skill tool to load superpowers/brainstorming
```

## 更新

每次重启 OpenCode 时，Superpowers 都会自动更新。

如果要固定某个版本：

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git#v5.0.3"]
}
```

## 故障排查

### 插件没有加载

1. 检查日志：`opencode run --print-logs "hello" 2>&1 | grep -i superpowers`
2. 确认 `opencode.json` 里的插件配置行正确
3. 确保你使用的是较新的 OpenCode 版本

### 找不到 技能

1. 用 `skill` 工具列出当前已发现的 skills
2. 确认插件确实已加载（见上面）

### 工具映射

当 skill 中引用 Claude Code 的工具名时：
- `TodoWrite` → `todowrite`
- 带 subagent 的 `Task` → `@mention` 语法
- `Skill` tool → OpenCode 原生 `skill` 工具
- 文件操作 → 使用你当前平台的原生工具

## 获取帮助

- 问题反馈：https://github.com/obra/superpowers/issues
- 完整文档：https://github.com/obra/superpowers/blob/main/docs/README.opencode.md
