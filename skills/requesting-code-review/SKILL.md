---
name: requesting-code-review
description: "当任务完成、重要功能落地，或准备合并前需要验证工作是否符合要求时使用。"
---

# 请求代码审查

派发 `superpowers:code-reviewer` subagent，在问题扩散之前把它们抓出来。审查者拿到的是为评估精心整理过的上下文，而不是你整段会话历史。这能让审查聚焦在产出物本身，而不是你的思考过程，同时也保留你自己的上下文，便于继续工作。

**核心原则：** 尽早 review，经常 review。

## 什么时候请求 审查

**必须：**
- 在 subagent-driven development 中，每完成一个 task 就 review 一次
- 完成重大功能后 review
- 合并到 `main` 之前 review

**可选但很有价值：**
- 被卡住时（换一个新视角）
- 重构前（先做 baseline 检查）
- 修复复杂 bug 之后

## 如何发起 审查

**1. 先拿到 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派发 code-reviewer subagent：**

使用 `Task` tool，并指定 `superpowers:code-reviewer` 类型，按 `code-reviewer.md` 模板填充内容。

**占位符：**
- `{WHAT_WAS_IMPLEMENTED}` - 你刚刚实现了什么
- `{PLAN_OR_REQUIREMENTS}` - 它本来应该做什么
- `{BASE_SHA}` - 起始提交
- `{HEAD_SHA}` - 结束提交
- `{DESCRIPTION}` - 简短摘要

**3. 根据反馈行动：**
- 立刻修复 Critical 问题
- 在继续之前修复 Important 问题
- Minor 问题记下来，后续处理
- 如果 reviewer 判断错了，要基于技术理由明确反驳

## 示例

```
[刚完成 Task 2：添加 verification function]

你：我先请求一次代码审查，再继续下一步。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[派发 superpowers:code-reviewer subagent]
  WHAT_WAS_IMPLEMENTED: Verification and repair functions for conversation index
  PLAN_OR_REQUIREMENTS: Task 2 from docs/superpowers/plans/deployment-plan.md
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: Added verifyIndex() and repairIndex() with 4 issue types

[Subagent 返回]：
  Strengths: 架构清晰，测试真实
  Issues:
    Important: 缺少进度指示
    Minor: 报告间隔里用了 magic number（100）
  Assessment: 可以继续

你：[修复进度指示]
[继续 Task 3]
```

## 与工作流的集成

**Subagent-Driven Development：**
- 每个 task 完成后都 review
- 在问题叠加前先抓出来
- 修完再进入下一个 task

**Executing Plans：**
- 每一批（3 个 task）之后 review
- 拿到反馈，应用修复，再继续

**Ad-Hoc Development：**
- 合并前 review
- 被卡住时 review

## 红旗信号

**绝不要：**
- 因为“这很简单”就跳过 review
- 忽略 Critical 问题
- 带着未修复的 Important 问题继续往前走
- 对有效的技术反馈强行争辩

**如果 reviewer 错了：**
- 用技术理由反驳
- 给出能证明实现正确的代码 / 测试
- 要求对方进一步澄清

模板见：`requesting-code-review/code-reviewer.md`
