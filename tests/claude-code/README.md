# Claude Code 技能 测试

使用 Claude Code CLI 对 superpowers skills 进行自动化测试。

## 概览

这套测试用于验证 skills 是否被正确加载，以及 Claude 是否按预期遵循这些 skills。测试通过无头模式调用 Claude Code（`claude -p`），然后验证行为。

## 要求

- 已安装 Claude Code CLI，并且在 PATH 中（`claude --version` 应该能运行）
- 已安装本地 superpowers 插件（安装方式见主 README）

## 运行测试

### 运行全部快速测试（推荐）：
```bash
./run-skill-tests.sh
```

### 运行集成测试（较慢，10 到 30 分钟）：
```bash
./run-skill-tests.sh --integration
```

### 运行单个测试：
```bash
./run-skill-tests.sh --test test-subagent-driven-development.sh
```

### 以详细输出模式运行：
```bash
./run-skill-tests.sh --verbose
```

### 设置自定义超时：
```bash
./run-skill-tests.sh --timeout 1800  # 给集成测试 30 分钟
```

## 测试结构

### 测试-helpers.sh

skills 测试通用函数：
- `run_claude "prompt" [timeout]` - 用 prompt 运行 Claude
- `assert_contains output pattern name` - 验证输出中包含某个模式
- `assert_not_contains output pattern name` - 验证输出中不包含某个模式
- `assert_count output pattern count name` - 验证某模式出现的精确次数
- `assert_order output pattern_a pattern_b name` - 验证先后顺序
- `create_test_project` - 创建临时测试目录
- `create_test_plan project_dir` - 创建示例计划文件

### 测试文件

每个测试文件都应该：
1. 引入 `test-helpers.sh`
2. 使用特定 prompt 运行 Claude Code
3. 用断言验证预期行为
4. 成功时返回 0，失败时返回非 0

## 示例测试

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
source "$SCRIPT_DIR/test-helpers.sh"

echo "=== Test: My Skill ==="

# 向 Claude 询问这个 技能
output=$(run_claude "What does the my-skill skill do?" 30)

# 验证响应
assert_contains "$output" "expected behavior" "Skill describes behavior"

echo "=== All tests passed ==="
```

## 当前测试

### 快速测试（默认运行）

#### 测试-subagent-driven-development.sh

测试 skill 内容和要求（约 2 分钟）：
- skill 是否能被加载与访问
- 工作流顺序是否正确（spec compliance 在 code quality 之前）
- 是否记录了 self-review 要求
- 是否记录了 plan 读取效率要求
- 是否记录了 spec compliance reviewer 的怀疑态度
- 是否记录了 review loop
- 是否记录了 task 上下文提供方式

### 集成测试（使用 `--integration`）

#### 测试-subagent-driven-development-integration.sh

完整工作流执行测试（约 10 到 30 分钟）：
- 创建真实测试项目和 Node.js 配置
- 创建包含 2 个任务的实现计划
- 使用 `subagent-driven-development` 执行计划
- 验证真实行为：
  - plan 只在开始时读取一次（而不是每个 task 读一次）
  - subagent prompt 中提供了完整 task 文本
  - subagent 在汇报前会先自审
  - spec compliance review 发生在 code quality 之前
  - spec reviewer 会独立阅读代码
  - 最终能产出可工作的实现
  - 测试通过
  - 正确创建 git commit

**它验证什么：**
- 这套工作流是否真的能端到端工作
- 我们的改进是否真的被应用
- subagent 是否正确遵循该 skill
- 最终代码是否可用且经过测试

## 新增测试的方法

1. 新建测试文件：`test-<skill-name>.sh`
2. 引入 `test-helpers.sh`
3. 使用 `run_claude` 和断言编写测试
4. 把它加入 `run-skill-tests.sh` 的测试列表
5. 赋可执行权限：`chmod +x test-<skill-name>.sh`

## Timeout 说明

- 默认超时：每个测试 5 分钟
- Claude Code 可能需要一定时间响应
- 如有需要，可以通过 `--timeout` 调整
- 测试本身应尽量聚焦，避免跑得过久

## 调试失败测试

使用 `--verbose` 可以看到完整 Claude 输出：

```bash
./run-skill-tests.sh --verbose --test test-subagent-driven-development.sh
```

如果不加 `--verbose`，只有失败时才会显示输出。

## CI/CD 集成

在 CI 中运行：

```bash
# 为 CI 环境设置明确超时
./run-skill-tests.sh --timeout 900

# 退出码 0 = 成功，非 0 = 失败
```

## 备注

- 这些测试验证的是 skill **指令**，不是完整执行过程
- 完整工作流测试会非常慢
- 重点是验证关键 skill 要求
- 测试应尽量保持确定性
- 避免测试具体实现细节
