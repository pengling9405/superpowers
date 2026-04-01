---
name: using-git-worktrees
description: 当你开始需要与当前工作区隔离的功能开发，或在执行实现计划前使用；它会通过智能目录选择与安全校验来创建隔离 git worktree
---

# 使用 Git Worktrees

## 概览

Git worktree 能为同一个仓库创建多个隔离工作区，让你在不切换当前分支的前提下，同时处理多个分支上的工作。

**核心原则：** 系统化的目录选择 + 安全校验 = 可靠隔离。

**开始时要说明：** “我正在使用 using-git-worktrees skill 来创建隔离工作区。”

## 目录选择流程

按下面这个优先级顺序处理：

### 1. 检查现有目录

```bash
# 按优先级检查
ls -d .worktrees 2>/dev/null     # 首选（隐藏目录）
ls -d worktrees 2>/dev/null      # 备选
```

**如果找到了：** 就使用它。如果两个都存在，`.worktrees` 优先。

### 2. 检查 `CLAUDE.md`

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**如果其中指定了偏好目录：** 直接使用，不需要再问用户。

### 3. 询问用户

如果既没有现有目录，也没有 `CLAUDE.md` 偏好：

```
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (project-local, hidden)
2. ~/.config/superpowers/worktrees/<project-name>/ (global location)

Which would you prefer?
```

## 安全校验

### 对项目内目录（`.worktrees` 或 `worktrees`）

**在创建 worktree 前，必须验证目录已被 git ignore：**

```bash
# 检查目录是否被 ignore（会同时考虑 local / global / system gitignore）
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**如果没有被 ignore：**

按照 Jesse 的规则 “坏掉的东西要立刻修”：
1. 把合适的规则加进 `.gitignore`
2. 提交这个改动
3. 然后再继续创建 worktree

**为什么这一步关键：** 可以避免把 worktree 内容误提交进仓库。

### 对全局目录（`~/.config/superpowers/worktrees`）

不需要做 `.gitignore` 校验，因为它本来就在项目外面。

## 创建步骤

### 1. 检测项目名

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. 创建 Worktree

```bash
# 计算完整路径
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# 用新分支创建 worktree
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. 运行项目初始化

自动检测并执行对应 setup：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. 验证干净基线

运行测试，确保 worktree 起点是干净的：

```bash
# 例子，实际请使用项目对应命令
npm test
cargo test
pytest
go test ./...
```

**如果测试失败：** 汇报失败情况，并询问是继续还是先调查。

**如果测试通过：** 汇报可继续。

### 5. 报告位置

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## 快速参考

| 场景 | 动作 |
|------|------|
| `.worktrees/` 存在 | 使用它（并验证已 ignore） |
| `worktrees/` 存在 | 使用它（并验证已 ignore） |
| 两者都存在 | 使用 `.worktrees/` |
| 两者都不存在 | 先看 `CLAUDE.md` → 再问用户 |
| 目录未被 ignore | 加入 `.gitignore` 并提交 |
| 基线测试失败 | 汇报失败并询问 |
| 没有 package.json / Cargo.toml | 跳过依赖安装 |

## 常见错误

### 跳过 ignore 校验

- **问题：** worktree 内容被跟踪，污染 git status
- **修正：** 对项目内目录，创建前必须跑 `git check-ignore`

### 想当然地决定目录位置

- **问题：** 造成混乱，违背项目约定
- **修正：** 必须遵守优先级：现有目录 > `CLAUDE.md` > 询问用户

### 基线测试失败还继续推进

- **问题：** 无法区分是旧问题还是你新引入的问题
- **修正：** 先汇报，再拿到明确许可

### 硬编码 setup 命令

- **问题：** 在不同项目工具链下直接失效
- **修正：** 根据项目文件自动检测（例如 `package.json`）

## 示例流程

```
You: I'm using the using-git-worktrees skill to set up an isolated workspace.

[Check .worktrees/ - exists]
[Verify ignored - git check-ignore confirms .worktrees/ is ignored]
[Create worktree: git worktree add .worktrees/auth -b feature/auth]
[Run npm install]
[Run npm test - 47 passing]

Worktree ready at /Users/jesse/myproject/.worktrees/auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## 红旗信号

**绝不要：**
- 对项目内目录不做 ignore 校验就创建 worktree
- 跳过基线测试验证
- 测试失败却不问就继续
- 在目录位置有歧义时擅自决定
- 跳过 `CLAUDE.md` 检查

**永远都要：**
- 遵循优先级：现有目录 > `CLAUDE.md` > 询问用户
- 对项目内目录验证已被 ignore
- 自动检测并执行项目初始化
- 验证测试基线是干净的

## 集成关系

**被以下 skill 调用：**
- **brainstorming**（Phase 4）- 设计获批并进入实现时必须调用
- **subagent-driven-development** - 执行任何 task 前必须调用
- **executing-plans** - 执行任何 task 前必须调用
- 任何需要隔离工作区的 skill

**与以下 skill 配合：**
- **finishing-a-development-branch** - 工作完成后负责清理
