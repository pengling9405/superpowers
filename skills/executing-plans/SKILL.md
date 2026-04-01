---
name: executing-plans
description: "当你已经有一份书面的实现计划，并准备在独立会话中按照审查检查点执行时使用。"
---

# 执行计划

## 概览

加载计划，先做批判性审查，再执行全部任务，完成后汇报。

**开始时要说明：** “我正在使用 executing-plans skill 来实现这份计划。”

**注意：** 告诉协作者，Superpowers 在能访问 subagents 的平台上效果会明显更好。如果运行环境支持 subagent（例如 Claude Code 或 Codex），工作质量会显著提高。若 subagent 可用，应优先使用 `superpowers:subagent-driven-development`，而不是这个 skill。

## 流程

### 第 1 步：加载并审查计划
1. 读取 plan 文件
2. 做批判性审查，找出任何疑问或担忧
3. 如果有顾虑，开始前先向协作者提出
4. 如果没有顾虑，创建 TodoWrite 并继续

### 第 2 步：执行任务

对于每个任务：
1. Mark as in_progress
2. 严格按每个步骤执行（plan 应该已经拆成可执行的小步骤）
3. 按要求运行验证
4. 标记为 completed

### 第 3 步：完成开发

当所有任务都完成并验证通过后：
- 说明：“我正在使用 finishing-a-development-branch skill 来完成这项工作。”
- **必需的子 skill：** 使用 `superpowers:finishing-a-development-branch`
- 按那个 skill 的流程验证测试、呈现选项并执行选择

## 何时停止并寻求帮助

**遇到以下情况时，立刻停止执行：**
- 遇到阻塞（缺依赖、测试失败、指令不清晰）
- 计划存在关键缺口，导致无法开始
- 你不理解某条指令
- 验证反复失败

**优先请求澄清，不要靠猜。**

## 何时回到前面的步骤

**遇到以下情况时，回到审查阶段（第 1 步）：**
- 协作者根据你的反馈更新了计划
- 核心方案需要重新思考

**不要硬顶着阻塞往前冲**，停下来并提问。

## 记住
- 先批判性审查计划
- 严格遵循计划步骤
- 不要跳过验证
- 计划要求引用 skill 时就照做
- 被卡住就停下来，不要猜
- 未经用户明确同意，绝不要在 `main` / `master` 分支上直接开始实现

## 集成关系

**必需的工作流 skills：**
- **superpowers:using-git-worktrees** - 必需：开始前先建立隔离工作区
- **superpowers:writing-plans** - 用来创建这个 skill 要执行的计划
- **superpowers:finishing-a-development-branch** - 在所有任务完成后收尾开发分支
