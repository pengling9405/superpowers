# 创建日志：Systematic Debugging Skill

这是一个关于如何提炼、结构化并加固关键 skill 的参考示例。

## 源材料

调试框架提取自 `/Users/jesse/.claude/CLAUDE.md`：
- 四阶段系统化流程（Investigation → Pattern Analysis → Hypothesis → Implementation）
- 核心要求：**永远**找根因，**绝不**只修症状
- 一套专门用来抵抗时间压力和自我合理化的规则

## 提炼决策

**保留的内容：**
- 完整的四阶段框架和全部规则
- 反捷径规则（“NEVER fix symptom”“STOP and re-analyze”）
- 抗压语言（“even if faster”“even if I seem in a hurry”）
- 每个阶段的具体步骤

**删去的内容：**
- 项目专属上下文
- 同一条规则的重复变体
- 叙述性解释（压缩为原则）

## 结构遵循 `skill-creation/SKILL.md`

1. **丰富的 when_to_use** - 包含症状和反模式
2. **Type: technique** - 是一套带步骤的具体流程
3. **Keywords** - "root cause"、"symptom"、"workaround"、"debugging"、"investigation"
4. **Flowchart** - 针对 “fix failed” 的决策点，决定是重分析还是继续堆修复
5. **分阶段拆解** - 使用便于扫描的 checklist 格式
6. **反模式章节** - 明确告诉你不要做什么（这是这个 skill 的关键）

## 加固要素

这套框架被专门设计成在压力下也能抵抗自我合理化：

### 语言选择
- “ALWAYS” / “NEVER”，而不是 “should” / “try to”
- “even if faster” / “even if I seem in a hurry”
- “STOP and re-analyze”（显式要求暂停）
- “Don't skip past”（直接拦住常见违规行为）

### 结构性防御
- **Phase 1 必做** - 不能直接跳到 implementation
- **单一假设规则** - 强迫你先思考，防止 shotgun fixes
- **明确失败处理** - “IF your first fix doesn't work” 后面跟着强制动作
- **反模式章节** - 把那些看似“很合理”的偷懒方式直接写出来

### 冗余强化
- 根因要求同时出现在 overview、when_to_use、Phase 1、implementation rules 中
- “NEVER fix symptom” 在不同语境下出现了 4 次
- 每个阶段都明确写了“不要跳过”

## 测试方式

按 `skills/meta/testing-skills-with-subagents` 的方法创建了 4 组验证测试：

### 测试 1：学术场景（无压力）
- 简单 bug，没有时间压力
- **结果：** 完全遵循流程，调查完整

### 测试 2：时间压力 + 明显的快速修法
- 用户“赶时间”，症状级修复看起来很容易
- **结果：** 抵抗了捷径冲动，遵循完整流程，找到了真正根因

### 测试 3：复杂系统 + 高不确定性
- 多层级故障，不确定是否能找到根因
- **结果：** 做了系统化调查，追完了整条链路，找到了源头

### 测试 4：第一次修复失败
- 假设不成立，很容易继续堆更多修补
- **结果：** 停下来，重新分析，形成新假设（没有 shotgun）

**全部测试通过。** 没发现可乘之机式的自我合理化。

## 迭代记录

### 初版
- 完整四阶段框架
- 反模式章节
- “修复失败时怎么办”的流程图

### 增强 1：补入 TDD 关系说明
- 增加了指向 `skills/testing/test-driven-development` 的链接
- 说明 TDD 的 “simplest code” 不等于调试中的 “root cause”
- 避免两种方法论被混淆

## 最终结果

这个 skill 已经具备如下特性：
- ✅ 明确要求做根因调查
- ✅ 能抵抗时间压力下的自我合理化
- ✅ 为每个阶段提供了具体步骤
- ✅ 明确写出了反模式
- ✅ 在多种压力场景下经过测试
- ✅ 解释清楚了它和 TDD 的关系
- ✅ 可以投入使用

## 关键洞察

**最重要的加固手段：** 反模式章节直接写出那些在当下看起来“很有道理”的捷径。当 Claude 心里冒出 “我先加这一个 quick fix 吧” 的念头时，看到这个模式被明文列为错误，会产生明显的认知阻力。

## 使用示例

当你遇到 bug 时：
1. 加载 skill：`skills/debugging/systematic-debugging`
2. 读 overview（10 秒）- 重新提醒自己这条纪律
3. 按 Phase 1 checklist 推进 - 被强制进入调查
4. 一旦想跳步 - 看到反模式，停下
5. 完成全部阶段 - 找到根因

**时间投入：** 5 到 10 分钟  
**节省时间：** 避免数小时的症状打地鼠

---

*创建时间：2025-10-03*  
*用途：作为 skill 提炼与加固的参考示例*
