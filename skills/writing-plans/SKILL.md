---
name: writing-plans
description: "当你已经有 spec 或多步骤任务需求，但还没有开始改代码时使用。"
---

# 编写计划

## 概览

写实现计划时，要假设执行工程师对当前代码库几乎没有上下文，而且审美也不一定可靠。把他们真正需要知道的东西全部写下来：每个任务要改哪些文件、涉及哪些代码、测试要怎么看、文档要看哪里。给出完整计划，并拆成可执行的小任务。DRY。YAGNI。TDD。频繁提交。

要假设他们是熟练开发者，但对你的工具链和问题域几乎不了解。还要假设他们对测试设计也不够强。

**开始时要说明：** “我正在使用 writing-plans skill 来创建实现计划。”

**上下文：** 这一步应该在独立 worktree 中执行（通常由 brainstorming skill 创建）。

**计划保存位置：** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- 如果用户对计划存放位置有明确偏好，应以用户偏好为准

## 范围检查

如果 spec 同时覆盖多个彼此独立的子系统，那么它本该在 brainstorming 阶段就拆成多个子项目 spec。若没有拆，你应该建议把它分成多个独立 plan，每个子系统一个。每个 plan 都应能独立产出可运行、可测试的软件。

## 文件结构

在定义任务前，先把会创建或修改哪些文件，以及每个文件负责什么，梳理清楚。这个阶段会把拆分决策锁定下来。

- 把设计单元做出清晰边界和明确接口。每个文件只承担一个明确职责。
- 你在同时能放进上下文的代码规模内推理效果最好，而当文件保持聚焦时，你的编辑也更可靠。优先选择小而聚焦的文件，不要堆出大而全的文件。
- 会一起变化的文件应该放在一起。按职责拆，不要按技术层硬拆。
- 在已有代码库里，要遵循现有模式。如果代码库习惯大文件，不要擅自重构；但如果你要改的文件已经明显失控，把拆分写进计划是合理的。

这个结构会直接决定任务拆解。每个任务都应能独立成立，产出自洽改动。

## 细粒度任务拆分

**每一步只做一个动作（2 到 5 分钟）：**
- “写一个失败测试” 是一步
- “运行它并确认它确实失败” 是一步
- “实现使测试通过的最小代码” 是一步
- “运行测试并确认通过” 是一步
- “提交代码” 是一步

## 计划文档头部

**每个 plan 都必须从下面这个头部开始：**

```markdown
# [功能 Name] 实现 计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [一句话描述这项功能会构建什么]

**Architecture:** [用 2 到 3 句话说明整体方案]

**Tech Stack:** [关键技术 / 库]

---
```

## 任务结构

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## 不要留占位符

每一步都必须提供执行工程师真正需要的内容。下面这些都是 **计划失败**，绝不能写：
- “TBD”“TODO”“implement later”“fill in details”
- “Add appropriate error handling” / “add validation” / “handle edge cases”
- “Write tests for the above”（却不给具体测试代码）
- “Similar to Task N”（要把代码重复写出来，因为执行者可能是乱序阅读任务）
- 只描述做什么，却不展示怎么做的步骤（涉及代码的步骤必须给代码块）
- 引用某个类型、函数或方法，但它在任何任务里都没有定义

## 记住

- 永远给出精确文件路径
- 每个步骤都给出完整代码，如果要改代码，就把代码写出来
- 给出精确命令和期望输出
- DRY、YAGNI、TDD、频繁提交

## 自审

写完整份计划后，用新的眼光重新看 spec，并把 plan 对照一遍。这是你自己执行的检查清单，不是给 subagent 的。

**1. Spec 覆盖度：** 快速浏览 spec 的每一节 / 每个需求。你能指出哪个 task 实现了它吗？把缺口列出来。

**2. 占位符扫描：** 在 plan 里搜索红旗模式，也就是上面 “不要留占位符” 一节中的内容。找到就修。

**3. 类型一致性：** 你在后续任务里写到的类型、方法签名、属性名，是否和前面任务里定义的一致？如果 Task 3 写的是 `clearLayers()`，Task 7 却变成 `clearFullLayers()`，那就是计划里的 bug。

如果你发现问题，直接在文档里修掉，不需要重新发起 review。只要修好继续即可。如果 spec 里有需求没有对应 task，就补上。

## 执行交接

计划保存后，向用户提供执行方式选择：

**“计划已完成，已保存到 `docs/superpowers/plans/<filename>.md`。有两种执行方式：**

**1. Subagent-Driven（推荐）** - 我为每个 task 派发一个全新的 subagent，任务之间插入 review，迭代更快

**2. Inline Execution** - 在当前会话中通过 executing-plans 执行，按批次推进并设置检查点

**你想选哪一种？”**

**如果用户选择 Subagent-Driven：**
- **必需子 skill：** 使用 `superpowers:subagent-driven-development`
- 每个 task 一个全新 subagent，并配两阶段 review

**如果用户选择 Inline Execution：**
- **必需子 skill：** 使用 `superpowers:executing-plans`
- 按批次执行，并在关键点插入 review
