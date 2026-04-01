# Go Fractals CLI - 实现计划

请使用 `superpowers:subagent-driven-development` skill 执行这份计划。

## 上下文

要构建一个生成 ASCII 分形图的 CLI 工具。完整规格见 `design.md`。

## 任务

### Task 1：项目初始化

创建 Go module 和目录结构。

**执行：**
- 初始化 `go.mod`，module 名为 `github.com/superpowers-test/fractals`
- 创建目录结构：`cmd/fractals/`、`internal/sierpinski/`、`internal/mandelbrot/`、`internal/cli/`
- 创建最小可运行的 `cmd/fractals/main.go`，输出 `"fractals cli"`
- 添加 `github.com/spf13/cobra` 依赖

**验证：**
- `go build ./cmd/fractals` 成功
- `./fractals` 输出 `"fractals cli"`

---

### Task 2：CLI 框架与帮助输出

用 Cobra 建立 root command，并配置帮助信息。

**执行：**
- 创建 `internal/cli/root.go`
- 配置帮助文本，展示可用子命令
- 在 `main.go` 中接入 root command

**验证：**
- `./fractals --help` 能显示 usage，并列出 `"sierpinski"` 和 `"mandelbrot"`
- `./fractals`（无参数）会显示帮助

---

### Task 3：Sierpinski 算法

实现 Sierpinski 三角生成算法。

**执行：**
- 创建 `internal/sierpinski/sierpinski.go`
- 实现 `Generate(size, depth int, char rune) []string`，返回三角形各行内容
- 使用递归中点细分算法
- 创建 `internal/sierpinski/sierpinski_test.go`，测试包括：
  - 小三角（`size=4, depth=2`）与预期输出一致
  - `size=1` 返回单字符
  - `depth=0` 返回填满的三角形

**验证：**
- `go test ./internal/sierpinski/...` 通过

---

### Task 4：Sierpinski CLI 接入

把 Sierpinski 算法接入 CLI 子命令。

**执行：**
- 创建 `internal/cli/sierpinski.go`
- 添加参数：`--size`（默认 32）、`--depth`（默认 5）、`--char`（默认 `*`）
- 调用 `sierpinski.Generate()` 并把结果打印到 stdout

**验证：**
- `./fractals sierpinski` 能输出三角形
- `./fractals sierpinski --size 16 --depth 3` 能输出较小的三角形
- `./fractals sierpinski --help` 能显示参数说明

---

### Task 5：Mandelbrot 算法

实现 Mandelbrot 集 ASCII 渲染器。

**执行：**
- 创建 `internal/mandelbrot/mandelbrot.go`
- 实现 `Render(width, height, maxIter int, char string) []string`
- 把复平面区域（实轴 -2.5 到 1.0，虚轴 -1.0 到 1.0）映射到输出尺寸
- 把迭代次数映射到字符渐变 `" .:-=+*#%@"`（若提供 `char` 则使用单字符）
- 创建 `internal/mandelbrot/mandelbrot_test.go`，测试包括：
  - 输出尺寸与要求的 `width/height` 一致
  - 已知在集合内部的点（0,0）映射到最大迭代字符
  - 已知在集合外的点（2,0）映射到低迭代字符

**验证：**
- `go test ./internal/mandelbrot/...` 通过

---

### Task 6：Mandelbrot CLI 接入

把 Mandelbrot 算法接入 CLI 子命令。

**执行：**
- 创建 `internal/cli/mandelbrot.go`
- 添加参数：`--width`（默认 80）、`--height`（默认 24）、`--iterations`（默认 100）、`--char`（默认空字符串）
- 调用 `mandelbrot.Render()` 并打印结果

**验证：**
- `./fractals mandelbrot` 能输出可辨认的 Mandelbrot 图
- `./fractals mandelbrot --width 40 --height 12` 能输出更小版本
- `./fractals mandelbrot --help` 能显示参数说明

---

### Task 7：字符集配置

确保 `--char` 在两个命令里都工作一致。

**执行：**
- 验证 Sierpinski 的 `--char` 能正确传给算法
- 对 Mandelbrot 来说，`--char` 应让输出改用单字符，而不是渐变
- 为自定义字符输出补测试

**验证：**
- `./fractals sierpinski --char '#'` 使用 `#`
- `./fractals mandelbrot --char '.'` 对所有填充点使用 `.`
- 所有测试通过

---

### Task 8：输入校验与错误处理

为非法输入补充校验。

**执行：**
- Sierpinski：`size` 必须 > 0，`depth` 必须 >= 0
- Mandelbrot：`width/height` 必须 > 0，`iterations` 必须 > 0
- 对非法输入返回清晰错误信息
- 补错误场景测试

**验证：**
- `./fractals sierpinski --size 0` 会输出错误并以非 0 退出
- `./fractals mandelbrot --width -1` 会输出错误并以非 0 退出
- 错误信息清楚、有帮助

---

### Task 9：集成测试

补调用 CLI 的集成测试。

**执行：**
- 创建 `cmd/fractals/main_test.go` 或 `test/integration_test.go`
- 对两个命令都做完整 CLI 调用测试
- 验证输出格式和退出码
- 验证错误场景会返回非 0

**验证：**
- `go test ./...` 包括集成测试在内全部通过

---

### Task 10：README

补使用文档和示例。

**执行：**
- 创建 `README.md`，包含：
  - 项目描述
  - 安装方式：`go install ./cmd/fractals`
  - 两个命令的使用示例
  - 示例输出（小尺寸样例）

**验证：**
- README 准确描述工具行为
- README 里的示例都能实际运行
