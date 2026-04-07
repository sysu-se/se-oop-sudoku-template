# DESIGN

## 1. Sudoku / Game 的职责边界

- `Sudoku`
  - 负责保存和管理 9x9 grid
  - 提供 `guess(move)` 修改格子
  - 提供 `getGrid()` 读取局面
  - 提供 `clone()` 复制当前局面
  - 提供 `toJSON()` 和 `toString()` 作为外表化方式

- `Game`
  - 负责一局游戏流程
  - 持有当前 `Sudoku`
  - 管理 `undoStack` 和 `redoStack`
  - 提供 `guess/undo/redo/canUndo/canRedo`
  - 提供 `toJSON()` 供保存

---

## 2. Move 是值对象还是实体对象？为什么？

**值对象**。  
`Move` 只表示一次输入动作（row, col, value），不需要独立身份，也不需要生命周期管理,所以用普通对象传递更简单。

---

## 3. history 中存储的是什么？为什么？

在 history 中存储的是 `Sudoku` 的 JSON 快照（主要是 grid）。  
原因：
- Undo/Redo 时可以直接恢复到某一步局面
- 实现简单，逻辑直观

---

## 4. 复制策略是什么？哪些地方需要深拷贝？

复制策略：
- 使用 `map + slice` 对二维数组做拷贝（行级深拷贝）

需要拷贝的地方：
1. `createSudoku(input)` 时复制输入，避免外部修改影响内部状态
2. `getGrid()` 返回副本，避免调用方直接改内部 grid
3. `clone()` 生成独立对象

如果误用浅拷贝，多个对象会共享同一行数组，修改一个会污染另一个。

---

## 5. 序列化 / 反序列化设计

- `Sudoku.toJSON()` 导出 `{ grid }`
- `createSudokuFromJSON(json)` 从 `json.grid` 恢复 Sudoku
- `Game.toJSON()` 导出当前 `sudoku` 与历史栈
- `createGameFromJSON(json)` 从 `json.sudoku` 恢复当前局面（本作业先恢复核心状态）

---

## 6. 外表化接口是什么？为什么这样设计？

- `Sudoku.toString()`
  - 输出可读的 9x9 文本，方便调试查看局面
- `Sudoku.toJSON()`
  - 输出可序列化对象，便于保存和恢复
- `Game.toJSON()`
  - 输出游戏当前状态，便于持久化和测试

这样设计是为了同时满足“人可读调试”和“程序可序列化处理”。