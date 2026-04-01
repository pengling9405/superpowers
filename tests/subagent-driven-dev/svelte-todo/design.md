# Svelte Todo List - 设计

## 概览

一个用 Svelte 构建的简单 Todo List 应用。支持创建、完成、删除 todo，并通过 `localStorage` 持久化。

## 功能

- 添加新 todo
- 标记 todo 为完成 / 未完成
- 删除 todo
- 按条件筛选：All / Active / Completed
- 清空所有已完成项
- 持久化到 `localStorage`
- 显示剩余未完成项数量

## 用户界面

```
┌─────────────────────────────────────────┐
│  Svelte Todos                           │
├─────────────────────────────────────────┤
│  [________________________] [Add]       │
├─────────────────────────────────────────┤
│  [ ] Buy groceries                  [x] │
│  [✓] Walk the dog                   [x] │
│  [ ] Write code                     [x] │
├─────────────────────────────────────────┤
│  2 items left                           │
│  [All] [Active] [Completed]  [Clear ✓]  │
└─────────────────────────────────────────┘
```

## 组件

```
src/
  App.svelte           # 主应用，负责状态管理
  lib/
    TodoInput.svelte   # 文本输入框 + Add 按钮
    TodoList.svelte    # 列表容器
    TodoItem.svelte    # 单个 todo，包含 checkbox、文本和删除按钮
    FilterBar.svelte   # 筛选按钮 + clear completed
    store.ts           # Svelte store，保存 todos
    storage.ts         # localStorage 持久化
```

## 数据模型

```typescript
interface Todo {
  id: string;        // UUID
  text: string;      // Todo 文本
  completed: boolean;
}

type Filter = 'all' | 'active' | 'completed';
```

## 验收标准

1. 可以通过输入文本后按 Enter，或点击 Add 来新增 todo
2. 可以通过点击 checkbox 切换完成状态
3. 可以通过点击 X 按钮删除 todo
4. 筛选按钮能正确显示对应子集
5. “X items left” 能正确显示未完成 todo 数量
6. “Clear completed” 会移除所有已完成 todo
7. 刷新页面后 todo 仍然存在（`localStorage`）
8. 空状态会显示有帮助的提示
9. 所有测试通过
