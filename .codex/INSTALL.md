# 为 Codex 安装 Superpowers

通过原生 skill discovery 在 Codex 中启用 superpowers skills。只需要 clone 仓库并创建符号链接。

## 前置要求

- Git

## 安装

1. **克隆 superpowers 仓库：**
   ```bash
   git clone https://github.com/obra/superpowers.git ~/.codex/superpowers
   ```

2. **创建 skills 符号链接：**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/superpowers/skills ~/.agents/skills/superpowers
   ```

   **Windows（PowerShell）：**
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\superpowers" "$env:USERPROFILE\.codex\superpowers\skills"
   ```

3. **重启 Codex**（退出并重新启动 CLI），让它重新发现这些 skills。

## 从旧 bootstrap 迁移

如果你在原生 skill discovery 上线前安装过 superpowers，需要执行以下步骤：

1. **更新仓库：**
   ```bash
   cd ~/.codex/superpowers && git pull
   ```

2. **创建 skills 符号链接**（见上面的第 2 步），这是新的发现机制。

3. **从 `~/.codex/AGENTS.md` 删除旧 bootstrap 配置块**，任何引用 `superpowers-codex bootstrap` 的内容都已经不再需要。

4. **重启 Codex。**

## 验证

```bash
ls -la ~/.agents/skills/superpowers
```

你应该能看到一个指向 superpowers skills 目录的符号链接（Windows 上则是 junction）。

## 更新

```bash
cd ~/.codex/superpowers && git pull
```

通过符号链接，skills 会立即更新。

## 卸载

```bash
rm ~/.agents/skills/superpowers
```

如有需要，也可以删除 clone 下来的仓库：`rm -rf ~/.codex/superpowers`。
