---
name: receiving-code-review
description: 当你收到代码审查反馈，并准备实现建议之前使用，尤其是在反馈含义不清或技术上可疑时；它要求技术严谨和验证，而不是表演式认同或盲目执行
---

# 接收代码审查反馈

## 概览

代码审查需要的是技术判断，不是情绪表演。

**核心原则：** 先验证，再实现。先问清，再假设。技术正确性高于社交舒适感。

## 响应模式

```
WHEN receiving code review feedback:

1. READ: Complete feedback without reacting
2. UNDERSTAND: Restate requirement in own words (or ask)
3. VERIFY: Check against codebase reality
4. EVALUATE: Technically sound for THIS codebase?
5. RESPOND: Technical acknowledgment or reasoned pushback
6. IMPLEMENT: One item at a time, test each
```

## 禁止响应

**绝不要：**
- “You're absolutely right!”（明确违反 CLAUDE.md）
- “Great point!” / “Excellent feedback!”（表演式认同）
- “Let me implement that now”（在验证之前）

**应该这样做：**
- 用自己的话复述这个技术要求
- 对不清楚的地方提问
- 如果对方错了，用技术理由反驳
- 直接开始做事（行动 > 客套）

## 处理不清晰反馈

```
IF any item is unclear:
  STOP - do not implement anything yet
  ASK for clarification on unclear items

WHY: Items may be related. Partial understanding = wrong implementation.
```

**示例：**
```
your human partner: "Fix 1-6"
You understand 1,2,3,6. Unclear on 4,5.

❌ WRONG: Implement 1,2,3,6 now, ask about 4,5 later
✅ RIGHT: "I understand items 1,2,3,6. Need clarification on 4 and 5 before proceeding."
```

## 不同来源的处理方式

### 来自你的协作者
- **默认可信**，理解清楚后就可以实现
- **范围不清仍然要问**
- **不要表演式认同**
- **直接进入行动**，或者做技术性确认

### 来自外部 Reviewer
```
BEFORE implementing:
  1. Check: Technically correct for THIS codebase?
  2. Check: Breaks existing functionality?
  3. Check: Reason for current implementation?
  4. Check: Works on all platforms/versions?
  5. Check: Does reviewer understand full context?

IF suggestion seems wrong:
  Push back with technical reasoning

IF can't easily verify:
  Say so: "I can't verify this without [X]. Should I [investigate/ask/proceed]?"

IF conflicts with your human partner's prior decisions:
  Stop and discuss with your human partner first
```

**协作者给出的规则：** “外部反馈要保持怀疑，但要认真核查。”

## 针对“更专业实现”的 YAGNI 检查

```
IF reviewer suggests "implementing properly":
  grep codebase for actual usage

  IF unused: "This endpoint isn't called. Remove it (YAGNI)?"
  IF used: Then implement properly
```

**协作者给出的规则：** “你和 reviewer 都向我负责。如果这个功能根本不需要，就不要加。”

## 实现顺序

```
FOR multi-item feedback:
  1. Clarify anything unclear FIRST
  2. Then implement in this order:
     - Blocking issues (breaks, security)
     - Simple fixes (typos, imports)
     - Complex fixes (refactoring, logic)
  3. Test each fix individually
  4. Verify no regressions
```

## 什么时候要反驳

在这些情况下要反驳：
- 建议会破坏现有功能
- reviewer 缺少完整上下文
- 违反 YAGNI（其实是未使用功能）
- 对当前技术栈来说技术上就是不对
- 存在兼容性 / 遗留原因
- 与协作者之前的架构决策冲突

**如何反驳：**
- 用技术理由，不要情绪化防御
- 提具体问题
- 引用已经工作的测试 / 代码
- 如果涉及架构，拉协作者进来定

**如果你不敢公开反驳：** `Strange things are afoot at the Circle K`

## 如何承认正确反馈

当反馈确实正确时：
```
✅ "Fixed. [Brief description of what changed]"
✅ "Good catch - [specific issue]. Fixed in [location]."
✅ [Just fix it and show in the code]

❌ "You're absolutely right!"
❌ "Great point!"
❌ "Thanks for catching that!"
❌ "Thanks for [anything]"
❌ ANY gratitude expression
```

**为什么不说谢谢：** 直接把事情修掉。代码本身就说明你接受了反馈。

**如果你发现自己正准备写 "Thanks"：** 删除它。直接说明修复内容。

## 如果你之前反驳了，但后来发现自己错了

```
✅ "You were right - I checked [X] and it does [Y]. Implementing now."
✅ "Verified this and you're correct. My initial understanding was wrong because [reason]. Fixing."

❌ Long apology
❌ Defending why you pushed back
❌ Over-explaining
```

事实性地纠正，然后继续推进。

## 常见错误

| 错误 | 修正方式 |
|------|----------|
| 表演式认同 | 复述需求，或直接行动 |
| 盲目实现 | 先对照代码库验证 |
| 一口气改很多但不测试 | 一次改一项，每项都测 |
| 默认 reviewer 一定对 | 检查它会不会把东西搞坏 |
| 因为尴尬而不反驳 | 技术正确性 > 社交舒适感 |
| 只实现自己看懂的部分 | 先把所有不清楚的地方问清 |
| 无法验证还硬着头皮做 | 明确说出限制，并请求方向 |

## 真实示例

**表演式认同（坏）：**
```
Reviewer: "Remove legacy code"
❌ "You're absolutely right! Let me remove that..."
```

**技术验证（好）：**
```
Reviewer: "Remove legacy code"
✅ "Checking... build target is 10.15+, this API needs 13+. Need legacy for backward compat. Current impl has wrong bundle ID - fix it or drop pre-13 support?"
```

**YAGNI（好）：**
```
Reviewer: "Implement proper metrics tracking with database, date filters, CSV export"
✅ "Grepped codebase - nothing calls this endpoint. Remove it (YAGNI)? Or is there usage I'm missing?"
```

**反馈项不清楚（好）：**
```
your human partner: "Fix items 1-6"
You understand 1,2,3,6. Unclear on 4,5.
✅ "Understand 1,2,3,6. Need clarification on 4 and 5 before implementing."
```

## GitHub 线程回复

在 GitHub 上回复 inline review comment 时，要回复到对应 comment thread（`gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies`），不要发成顶层 PR 评论。

## 最终结论

**外部反馈是需要评估的建议，不是你必须服从的命令。**

先验证。先提问。然后再实现。

不要表演式认同。永远保持技术严谨。
