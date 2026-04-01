---
name: finishing-a-development-branch
description: "当实现已经完成、测试全部通过，并且需要决定如何集成这部分工作时使用；它会用结构化选项引导你完成 merge、PR 或收尾清理。"
---

# 收尾开发分支

## 概览

通过清晰选项来完成开发收尾，并按用户选择执行后续流程。

**核心原则：** 先验证测试 → 再给选项 → 再执行选择 → 最后清理。

**开始时要说明：** “我正在使用 finishing-a-development-branch skill 来完成这项工作。”

## 流程

### 步骤 1：验证测试

**在给任何选项前，先确认测试通过：**

```bash
# 运行项目测试套件
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with merge/PR until tests pass.
```

到此停止。不要进入 Step 2。

**如果测试通过：** 继续 Step 2。

### 步骤 2：确定基准分支

```bash
# 尝试常见基准分支
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

也可以直接问用户：“这个分支是从 `main` 切出来的，对吗？”

### 步骤 3：给出选项

必须原样给出这 4 个选项：

```
Implementation complete. What would you like to do?

1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)
4. Discard this work

Which option?
```

**不要额外解释**，保持简洁。

### 步骤 4：执行选择

#### 选项 1：本地合并

```bash
# 切回基准分支
git checkout <base-branch>

# 拉最新
git pull

# 合并功能分支
git merge <feature-branch>

# 对合并结果重新验证测试
<test command>

# 如果测试通过
git branch -d <feature-branch>
```

然后进入：清理 worktree（Step 5）

#### 选项 2：推送并创建 PR

```bash
# 推送分支
git push -u origin <feature-branch>

# 创建 PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## 摘要
<2-3 bullets of what changed>

## 测试 计划
- [ ] <verification steps>
EOF
)"
```

然后进入：清理 worktree（Step 5）

#### 选项 3：保持现状

汇报：
`Keeping branch <name>. Worktree preserved at <path>.`

**不要清理 worktree。**

#### 选项 4：丢弃这部分工作

**必须先确认：**
```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>
- Worktree at <path>

Type 'discard' to confirm.
```

等待精确输入确认。

确认后执行：

```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

然后进入：清理 worktree（Step 5）

### Step 5：清理 Worktree

**适用于选项 1、2、4：**

先检查当前是否在 worktree 中：

```bash
git worktree list | grep $(git branch --show-current)
```

如果是：

```bash
git worktree remove <worktree-path>
```

**选项 3：** 保留 worktree，不做清理。

## 快速参考

| 选项 | 合并 | Push | 保留 Worktree | 清理分支 |
|------|------|------|---------------|----------|
| 1. 本地合并 | ✓ | - | - | ✓ |
| 2. 创建 PR | - | ✓ | ✓ | - |
| 3. 保持现状 | - | - | ✓ | - |
| 4. 丢弃 | - | - | - | ✓（强制） |

## 常见错误

**跳过测试验证**
- **问题：** 把坏代码合进去，或者创建一个注定失败的 PR
- **修正：** 在给选项前必须先验证测试

**提开放式问题**
- **问题：** “你想接下来怎么做？” 太模糊
- **修正：** 必须给出明确的 4 个结构化选项

**自动清理 worktree**
- **问题：** 在其实可能还要用到它时就删掉了（尤其是选项 2、3）
- **修正：** 只对需要清理的选项执行清理

**丢弃前不确认**
- **问题：** 一不小心把工作删了
- **修正：** 必须要求用户输入 `discard`

## 红旗信号

**绝不要：**
- 测试失败还继续往下走
- 合并结果不验证测试
- 未确认就删除工作
- 没有明确要求就 force-push

**永远都要：**
- 在给选项前先验证测试
- 只给出这 4 个选项
- 选项 4 必须要求输入确认
- 对需要清理的选项执行 worktree 清理

## 集成关系

**被以下 skill 调用：**
- **subagent-driven-development**（Step 7）- 所有任务完成后
- **executing-plans**（Step 5）- 所有 batch 完成后

**与以下 skill 配合：**
- **using-git-worktrees** - 用于清理由该 skill 创建的 worktree
