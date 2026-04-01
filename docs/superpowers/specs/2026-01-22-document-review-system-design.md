# 文档审查系统设计

## 概览

给 superpowers 工作流新增两个 review 阶段：

1. **Spec 文档审查** - 在 brainstorming 之后、writing-plans 之前
2. **Plan 文档审查** - 在 writing-plans 之后、实现开始之前

两者都遵循与实现阶段 review 相同的迭代闭环模式。

## Spec 文档审查者

**目的：** 验证 spec 是否完整、一致，并且已经可以进入实现规划阶段。

**位置：** `skills/brainstorming/spec-document-reviewer-prompt.md`

**检查内容：**

| 类别 | 关注点 |
|------|--------|
| 完整性 | TODO、占位符、"TBD"、未完成章节 |
| 覆盖度 | 是否遗漏错误处理、边界情况、集成点 |
| 一致性 | 内部矛盾、需求冲突 |
| 清晰度 | 是否存在歧义需求 |
| YAGNI | 未被请求的功能、过度设计 |

**输出格式：**
```
## Spec Review

**Status:** Approved | Issues Found

**Issues (if any):**
- [Section X]: [issue] - [why it matters]

**Recommendations (advisory):**
- [suggestions that don't block approval]
```

**Review 闭环：** 有问题 → brainstorming agent 修 → 再审 → 直到通过。

**派发机制：** 使用 `Task` tool，`subagent_type: general-purpose`。完整 reviewer prompt 由模板提供，由 brainstorming skill 的 controller 负责派发。

## Plan 文档审查者

**目的：** 验证 plan 是否完整、是否符合 spec，以及任务拆解是否合理。

**位置：** `skills/writing-plans/plan-document-reviewer-prompt.md`

**检查内容：**

| 类别 | 关注点 |
|------|--------|
| 完整性 | TODO、占位符、未完成任务 |
| 与 Spec 对齐 | Plan 是否覆盖 spec 需求，是否有 scope creep |
| 任务拆解 | 任务是否足够原子，边界是否清晰 |
| 任务语法 | 任务和步骤是否使用 checkbox 语法 |
| Chunk 大小 | 每个 chunk 是否都小于 1000 行 |

**Chunk 定义：** chunk 是 plan 文档中的逻辑任务组，由 `## Chunk N: <name>` 标题分隔。writing-plans skill 会按逻辑阶段（例如 “Foundation”“Core Features”“Integration”）来创建这些边界。每个 chunk 都应足够自洽，便于独立审查。

**Spec 对齐验证：** reviewer 同时拿到：
1. plan 文档（或当前 chunk）
2. 对应 spec 文档路径

reviewer 会同时读两者，并对照需求覆盖情况。

**输出格式：** 与 spec reviewer 相同，但作用范围限制在当前 chunk。

**审查流程（逐 chunk）：**
1. writing-plans 生成 chunk N
2. controller 把 chunk N 内容和 spec 路径派给 plan-document-reviewer
3. reviewer 读取 chunk 和 spec，返回结论
4. 如果有问题：writing-plans agent 修 chunk N，然后回到步骤 2
5. 如果通过：进入 chunk N+1
6. 所有 chunk 都通过后，进入实现阶段

**派发机制：** 与 spec reviewer 相同，都是 `Task` tool + `subagent_type: general-purpose`

## 更新后的工作流

```
brainstorming -> spec -> SPEC REVIEW LOOP -> writing-plans -> plan -> PLAN REVIEW LOOP -> implementation
```

**Spec Review Loop：**
1. Spec 完成
2. 派发 reviewer
3. 如果有问题：修复后回到 2
4. 如果通过：继续

**Plan Review Loop：**
1. Chunk N 完成
2. 派发针对 chunk N 的 reviewer
3. 如果有问题：修复后回到 2
4. 如果通过：下一个 chunk 或进入实现

## Markdown 任务语法

任务和步骤统一使用 checkbox 语法：

```markdown
- [ ] ### Task 1: Name

- [ ] **Step 1:** Description
  - File: path
  - Command: cmd
```

## 错误处理

**Review 闭环终止条件：**
- 不设硬性迭代次数上限，直到 reviewer 通过为止
- 如果超过 5 轮仍未通过，controller 应把情况上抛给人工
- 人类可以选择：继续迭代、带已知问题通过，或直接终止

**分歧处理：**
- reviewer 只是 advisory，不是硬阻塞器
- 如果 agent 认为 reviewer 的反馈不对，应在修复说明中解释原因
- 如果同一问题经过 3 轮仍然分歧不消，就上抛给人工

**reviewer 输出格式异常：**
- controller 应检查 reviewer 输出是否包含必要字段（Status，以及有问题时的 Issues）
- 如果格式不合法，就带上期望格式说明重新派发
- 连续 2 次输出异常，则交给人工

## 需要改动的文件

**新增文件：**
- `skills/brainstorming/spec-document-reviewer-prompt.md`
- `skills/writing-plans/plan-document-reviewer-prompt.md`

**修改文件：**
- `skills/brainstorming/SKILL.md` - 在 spec 完成后加 review loop
- `skills/writing-plans/SKILL.md` - 增加逐 chunk review loop，并更新任务语法示例
