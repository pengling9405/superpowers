# Go Fractals CLI - 设计

## 概览

一个用来生成 ASCII 艺术分形图的命令行工具。支持两种分形类型，并允许配置输出参数。

## 使用方式

```bash
# 谢尔宾斯基三角
fractals sierpinski --size 32 --depth 5

# 曼德博集合
fractals mandelbrot --width 80 --height 24 --iterations 100

# 自定义字符
fractals sierpinski --size 16 --char '#'

# 帮助
fractals --help
fractals sierpinski --help
```

## 命令

### `sierpinski`

通过递归细分生成谢尔宾斯基三角。

参数：
- `--size`（默认：32）- 三角形底边宽度（字符数）
- `--depth`（默认：5）- 递归深度
- `--char`（默认：`*`）- 用于填充点的字符

输出：把三角形逐行打印到 stdout。

### `mandelbrot`

把曼德博集合渲染成 ASCII 图。通过迭代次数映射到不同字符。

参数：
- `--width`（默认：80）- 输出宽度（字符数）
- `--height`（默认：24）- 输出高度（字符数）
- `--iterations`（默认：100）- 逃逸计算的最大迭代次数
- `--char`（默认：gradient）- 单个字符；如果省略，则使用渐变 `" .:-=+*#%@"`

输出：把矩形图像打印到 stdout。

## 架构

```
cmd/
  fractals/
    main.go           # 入口，CLI 配置
internal/
  sierpinski/
    sierpinski.go     # 算法实现
    sierpinski_test.go
  mandelbrot/
    mandelbrot.go     # 算法实现
    mandelbrot_test.go
  cli/
    root.go           # 根命令、帮助信息
    sierpinski.go     # Sierpinski 子命令
    mandelbrot.go     # Mandelbrot 子命令
```

## 依赖

- Go 1.21+
- `github.com/spf13/cobra` 用于构建 CLI

## 验收标准

1. `fractals --help` 能正常显示用法
2. `fractals sierpinski` 能输出可辨认的三角形
3. `fractals mandelbrot` 能输出可辨认的曼德博集合
4. `--size`、`--width`、`--height`、`--depth`、`--iterations` 参数都能生效
5. `--char` 可以自定义输出字符
6. 非法输入会给出清晰错误信息
7. 所有测试通过
