# 代码质量审查 Prompt 模板

在派发代码质量审查 subagent 时使用这个模板。

**目的：** 验证实现是否足够扎实（干净、可测试、可维护）

**只在 spec 一致性审查通过后派发。**

```
Task tool (superpowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from implementer's report]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
  DESCRIPTION: [task summary]
```

**除标准代码质量问题外，审查者还应检查：**
- 每个文件是否只有一个明确职责，并暴露清晰接口？
- 各个单元是否拆得足够清楚，能被独立理解和测试？
- 实现是否遵循计划中的文件结构？
- 这次实现是否新建了已经很大的文件，或让已有文件明显膨胀？（不要因为历史包袱标红，重点关注这次改动新增的体量。）

**代码审查者返回：** 优点、问题（Critical/Important/Minor）、总体评估
