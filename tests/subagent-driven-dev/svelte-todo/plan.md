# Svelte Todo List - 实现计划

请使用 `superpowers:subagent-driven-development` skill 执行这份计划。

## 上下文

要用 Svelte 构建一个 todo list 应用。完整规格见 `design.md`。

## 任务

### Task 1：项目初始化

使用 Vite 创建 Svelte 项目。

**执行：**
- 运行 `npm create vite@latest . -- --template svelte-ts`
- 运行 `npm install` 安装依赖
- 验证 dev server 可正常启动
- 清理 `App.svelte` 中默认的 Vite 模板内容

**验证：**
- `npm run dev` 能启动服务
- 页面显示最小化的 `"Svelte Todos"` 标题
- `npm run build` 成功

---

### Task 2：Todo Store

创建用于管理 todo 状态的 Svelte store。

**执行：**
- 创建 `src/lib/store.ts`
- 定义带 `id`、`text`、`completed` 的 `Todo` 接口
- 创建初始为空数组的 writable store
- 导出函数：`addTodo(text)`、`toggleTodo(id)`、`deleteTodo(id)`、`clearCompleted()`
- 创建 `src/lib/store.test.ts`，为每个函数写测试

**验证：**
- 测试通过：`npm run test`（如果需要就先安装 vitest）

---

### Task 3：localStorage 持久化

给 todos 增加持久化层。

**执行：**
- 创建 `src/lib/storage.ts`
- 实现 `loadTodos(): Todo[]` 和 `saveTodos(todos: Todo[])`
- 优雅处理 JSON 解析错误（返回空数组）
- 与 store 集成：初始化时加载、变更时保存
- 为读取 / 保存 / 错误处理编写测试

**验证：**
- 测试通过
- 手工验证：新增 todo，刷新页面后仍然存在

---

### Task 4：TodoInput 组件

创建添加 todo 的输入组件。

**执行：**
- 创建 `src/lib/TodoInput.svelte`
- 文本输入框绑定本地状态
- Add 按钮调用 `addTodo()`，并清空输入框
- Enter 键也能提交
- 当输入为空时禁用 Add 按钮
- 补组件测试

**验证：**
- 测试通过
- 组件能正常渲染输入框和按钮

---

### Task 5：TodoItem 组件

创建单个 todo 项组件。

**执行：**
- 创建 `src/lib/TodoItem.svelte`
- Props：`todo: Todo`
- Checkbox 切换完成状态（调用 `toggleTodo`）
- 完成时文本显示删除线
- Delete 按钮（X）调用 `deleteTodo`
- 补组件测试

**验证：**
- 测试通过
- 组件能正确渲染 checkbox、文本和删除按钮

---

### Task 6：TodoList 组件

创建列表容器组件。

**执行：**
- 创建 `src/lib/TodoList.svelte`
- Props：`todos: Todo[]`
- 为每个 todo 渲染一个 TodoItem
- 为空时显示 `"No todos yet"`
- 补组件测试

**验证：**
- 测试通过
- 组件能正确渲染 TodoItem 列表

---

### Task 7：FilterBar 组件

创建筛选与状态栏组件。

**执行：**
- 创建 `src/lib/FilterBar.svelte`
- Props：`todos: Todo[]`、`filter: Filter`、`onFilterChange: (f: Filter) => void`
- 显示计数：`"X items left"`（未完成项数量）
- 三个筛选按钮：All、Active、Completed
- 当前激活筛选器需有高亮
- `"Clear completed"` 按钮（当没有完成项时隐藏）
- 补组件测试

**验证：**
- 测试通过
- 组件能正确渲染计数、筛选按钮和 clear 按钮

---

### Task 8：App 集成

把所有组件接入 `App.svelte`。

**执行：**
- 引入所有组件和 store
- 添加筛选状态（默认 `'all'`）
- 根据当前筛选状态计算 `filtered todos`
- 按顺序渲染：标题、TodoInput、TodoList、FilterBar
- 为各组件传入正确 props

**验证：**
- App 能渲染全部组件
- 添加 todo 正常
- 切换完成状态正常
- 删除正常

---

### Task 9：筛选功能

确保筛选逻辑端到端可用。

**执行：**
- 验证筛选按钮会改变展示内容
- `all` 显示全部 todos
- `active` 只显示未完成项
- `completed` 只显示已完成项
- `Clear completed` 会移除已完成项，并在需要时重置筛选器
- 添加集成测试

**验证：**
- 筛选测试通过
- 手工验证所有筛选状态

---

### Task 10：样式与打磨

补充 CSS 样式，保证可用性。

**执行：**
- 让整体样式与设计稿一致
- 完成项显示删除线和弱化颜色
- 当前激活筛选按钮高亮
- 输入框有 focus 样式
- 删除按钮在 hover 时出现（或在移动端始终显示）
- 适配响应式布局

**验证：**
- 应用视觉上可用
- 样式不会破坏功能

---

### Task 11：端到端测试

为完整用户流程补 Playwright 测试。

**执行：**
- 安装 Playwright：`npm init playwright@latest`
- 创建 `tests/todo.spec.ts`
- 覆盖以下流程：
  - 添加 todo
  - 完成 todo
  - 删除 todo
  - 筛选 todos
  - 清空已完成
  - 持久化（添加 → 刷新 → 验证）

**验证：**
- `npx playwright test` 通过

---

### Task 12：README

补项目文档。

**执行：**
- 创建 `README.md`，包含：
  - 项目描述
  - 安装：`npm install`
  - 开发：`npm run dev`
  - 测试：`npm test` 与 `npx playwright test`
  - 构建：`npm run build`

**验证：**
- README 准确描述项目
- README 中的指令都能执行
