# Superpowers for Codex 中文整理版

> 说明：这是 `Superpowers` 在 Codex 上使用方式的中文整理版。英文原文保持不变。

## 这是什么

这是 `Superpowers` 在 OpenAI Codex 上的使用指南。核心思路是利用 Codex 的原生 skill discovery，从指定目录中自动发现并按需加载 skills。

## 快速安装

直接对 Codex 说：

```text
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md
```

## 手动安装

### 前提条件

- 已安装 OpenAI Codex CLI
- 已安装 Git

### 步骤

1. 克隆仓库：

   ```bash
   git clone https://github.com/obra/superpowers.git ~/.codex/superpowers
   ```

2. 创建 skills 软链接：

   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/superpowers/skills ~/.agents/skills/superpowers
   ```

3. 重启 Codex。

4. 如果要使用依赖多代理能力的技能（可选），在 Codex 配置中加入：

   ```toml
   [features]
   multi_agent = true
   ```

### Windows

Windows 下可以使用 junction：

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
cmd /c mklink /J "$env:USERPROFILE\.agents\skills\superpowers" "$env:USERPROFILE\.codex\superpowers\skills"
```

## 它是怎么工作的

Codex 会在启动时扫描 `~/.agents/skills/`，解析其中 `SKILL.md` 的 frontmatter，并在相关任务出现时自动加载技能。

在 `Superpowers` 中，核心做法是通过一条软链接把整个技能库暴露给 Codex：

```text
~/.agents/skills/superpowers/ → ~/.codex/superpowers/skills/
```

`using-superpowers` 这个 skill 会自动被发现，并在后续使用中强化“该用 skill 时必须用 skill”的纪律。

## 使用方式

通常无需手工激活，Codex 会在这些情况下自动调用：

- 你直接提到某个 skill 的名字
- 当前任务与某个 skill 的描述匹配
- `using-superpowers` skill 判断应当进一步调用某个 skill

## 自定义个人技能

你也可以把自己的 skill 放到：

```bash
~/.agents/skills/
```

示例：

```bash
mkdir -p ~/.agents/skills/my-skill
```

然后创建 `SKILL.md`：

```markdown
---
name: my-skill
description: Use when [condition] - [what it does]
---

# My 技能

[Your skill content here]
```

其中 `description` 非常重要，因为 Codex 会根据它判断何时自动触发该 skill。

## 更新

```bash
cd ~/.codex/superpowers && git pull
```

因为 skills 是通过软链接暴露给 Codex 的，所以更新后通常无需额外同步。

## 卸载

```bash
rm ~/.agents/skills/superpowers
```

如果还要删掉源码仓库：

```bash
rm -rf ~/.codex/superpowers
```

Windows 下可用 PowerShell 对应命令删除。

## 常见问题

### 技能 没有显示出来

依次检查：

1. 软链接是否存在
2. `~/.codex/superpowers/skills` 下是否真的有 skill
3. 是否已经重启 Codex

### Windows 下 junction 创建失败

可尝试用管理员权限运行 PowerShell。

## 相关文件

- 英文原文：[README.codex.md](/Users/zhanyu/projects/superpowers/docs/README.codex.md)
- 仓库入口：[README.md](/Users/zhanyu/projects/superpowers/README.md)

