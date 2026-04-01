# Persuasion Principles for 技能 设计

## 概览

LLMs respond to the same persuasion principles as humans. Understanding this psychology helps you design more effective skills - not to manipulate, but to ensure critical practices are followed even under pressure.

**Research foundation:** Meincke et al. (2025) tested 7 persuasion principles with N=28,000 AI conversations. Persuasion techniques more than doubled 遵循率 rates (33% → 72%, p < .001).

## 七个原则

### 1. 权威
**含义：** Deference to expertise, credentials, or official sources.

**在技能文档中的用法：**
- Imperative language: "YOU MUST", "Never", "Always"
- Non-negotiable framing: "没有例外"
- Eliminates decision fatigue and rationalization

**适用场景:**
- 纪律-enforcing skills (TDD, 验证 requirements)
- Safety-critical practices
- Established best practices

**示例：**
```markdown
✅ Write code before test? Delete it. Start over. No exceptions.
❌ Consider writing tests first when feasible.
```

### 2. 承诺
**含义：** Consistency with prior actions, statements, or public declarations.

**在技能文档中的用法：**
- Require announcements: "Announce skill usage"
- Force explicit choices: "Choose A, B, or C"
- Use 跟踪: TodoWrite for checklists

**适用场景:**
- Ensuring skills are actually followed
- Multi-step processes
- Accountability mechanisms

**示例：**
```markdown
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
❌ Consider letting your partner know which skill you're using.
```

### 3. 稀缺性
**含义：** Urgency from time limits or limited availability.

**在技能文档中的用法：**
- Time-bound requirements: "Before proceeding"
- Sequential dependencies: "Immediately after X"
- Prevents procrastination

**适用场景:**
- Immediate 验证 requirements
- Time-sensitive 工作流
- Preventing "I'll do it later"

**示例：**
```markdown
✅ After completing a task, IMMEDIATELY request code review before proceeding.
❌ You can review code when convenient.
```

### 4. 社会认同
**含义：** Conformity to what others do or what's considered normal.

**在技能文档中的用法：**
- Universal patterns: "每次都要", "Always"
- Failure modes: "X without Y = failure"
- Establishes norms

**适用场景:**
- Documenting universal practices
- Warning about 常见 failures
- Reinforcing standards

**示例：**
```markdown
✅ Checklists without TodoWrite tracking = steps get skipped. Every time.
❌ Some people find TodoWrite helpful for checklists.
```

### 5. 团结感
**含义：** Shared identity, "we-ness", in-group belonging.

**在技能文档中的用法：**
- Collaborative language: "our codebase", "we're colleagues"
- Shared goals: "we both want 质量"

**适用场景:**
- Collaborative 工作流
- Establishing team culture
- Non-hierarchical practices

**示例：**
```markdown
✅ We're colleagues working together. I need your honest technical judgment.
❌ You should probably tell me if I'm wrong.
```

### 6. 互惠
**含义：** Obligation to return 收益 received.

**运作方式：**
- 谨慎使用，过度使用会显得在操控对方
- 在技能文档中通常很少需要

**何时避免：**
- Almost always (other principles more effective)

### 7. 喜好
**含义：** Preference for cooperating with those we like.

**运作方式：**
- **DON'T USE for 遵循率**
- Conflicts with honest feedback culture
- Creates sycophancy

**何时避免：**
- 在纪律约束场景下一律不要使用

## Principle Combinations by 技能类型

| 技能类型 | Use | Avoid |
|------------|-----|-------|
| 纪律-enforcing | 权威 + 承诺 + 社会认同 | 喜好, 互惠 |
| 指导/technique | Moderate 权威 + 团结感 | Heavy 权威 |
| Collaborative | 团结感 + 承诺 | 权威, 喜好 |
| 参考 | 清晰度 only | All persuasion |

## 为什么有效：背后的心理机制

**Bright-line rules reduce rationalization:**
- "YOU MUST" removes decision fatigue
- Absolute language eliminates "is this an exception?" questions
- Explicit anti-rationalization counters close specific loopholes

**Implementation intentions create automatic behavior:**
- Clear triggers + required actions = automatic execution
- "When X, do Y" more effective than "generally do Y"
- Reduces cognitive load on 遵循率

**LLMs are parahuman:**
- Trained on human text containing these patterns
- 权威 language precedes 遵循率 in training data
- 承诺 sequences (statement → action) frequently modeled
- 社会认同 patterns (everyone does X) establish norms

## 合乎伦理的使用方式

**合理用途：**
- Ensuring critical practices are followed
- Creating effective documentation
- Preventing predictable failures

**不合理用途：**
- Manipulating for personal gain
- Creating false urgency
- Guilt-based 遵循率

**判断标准：** Would this technique serve the user's genuine interests if they fully understood it?

## 研究引用

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.
- Seven principles of persuasion
- Empirical foundation for influence 调研

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania.
- Tested 7 principles with N=28,000 LLM conversations
- 遵循率 increased 33% → 72% with persuasion techniques
- 权威, 承诺, 稀缺性 most effective
- Validates parahuman model of LLM behavior

## 快速参考

When designing a skill, ask:

1. **What 类型 is it?** (纪律 vs. 指导 vs. 参考)
2. **What behavior am I trying to change?**
3. **Which principle(s) apply?** (Usually 权威 + 承诺 for 纪律)
4. **Am I combining too many?** (Don't use all seven)
5. **Is this ethical?** (Serves user's genuine interests?)
