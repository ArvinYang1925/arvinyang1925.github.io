---
layout: day
title: Day 8 - 打造你的第一個 TodoList API：一步步實現 CRUD 功能
date: 2025-09-22 15:14:15
tags:
  - TypeScript
  - Node.js
---

在學習 **Express + TypeScript + TypeORM** 的過程中，`TodoList API` 是非常適合新手上手的練習案例。

因為它的邏輯簡單（新增、讀取、更新、刪除），卻又涵蓋了 RESTful API 的核心概念：

- **CRUD**（Create / Read / Update / Delete）
- **Controller / Route 分層**
- **與資料庫的互動（Repository）**

這樣的練習不僅可以打好基礎，還能快速理解 **後端架構設計** 的常見模式。

<!-- more -->

---

## 建立 Controller

Controller（控制器）是 **專門負責處理業務邏輯** 的地方：

- 收到請求 (`req`) → 驗證/處理 → 回傳回應 (`res`)
- 不直接決定路由，而是提供「功能」讓 Route 使用
- 好處是：**程式碼結構清楚、可讀性佳、方便測試與維護**

建立 `src/controllers/todoController.ts`：

```tsx
import { Request, Response, NextFunction } from "express";
import { AppDataSource } from "../config/db";
import { Todo } from "../entities/Todo";

const todoRepository = AppDataSource.getRepository(Todo);

export async function getTodos(
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> {
  try {
    const todos = await todoRepository.find();
    res.json({ status: "success", data: todos });
  } catch (error) {
    next(error);
  }
}

export async function createTodo(
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> {
  try {
    const { title } = req.body;
    if (!title) {
      res.status(400).json({ status: "error", message: "Title is required" });
      return;
    }
    const newTodo = todoRepository.create({ title });
    const savedTodo = await todoRepository.save(newTodo);
    res.status(201).json({ status: "success", data: savedTodo });
  } catch (error) {
    next(error);
  }
}

export async function updateTodo(
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> {
  try {
    const { id } = req.params;
    const { title, completed } = req.body;
    const todo = await todoRepository.findOneBy({ id });

    if (!todo) {
      res.status(404).json({ status: "error", message: "Todo not found" });
      return;
    }

    todo.title = title !== undefined ? title : todo.title;
    todo.completed = completed !== undefined ? completed : todo.completed;
    const updatedTodo = await todoRepository.save(todo);
    res.json({ status: "success", data: updatedTodo });
  } catch (error) {
    next(error);
  }
}

export async function deleteTodo(
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> {
  try {
    const { id } = req.params;
    const result = await todoRepository.delete({ id });
    if (result.affected === 0) {
      res.status(404).json({ status: "error", message: "Todo not found" });
      return;
    }
    res.json({ status: "success", data: result });
  } catch (error) {
    next(error);
  }
}
```

---

## 設定路由

Route（路由）的工作就是 **決定請求要交給哪個 Controller 處理**。

它就像是 **導航地圖**：

- `GET /todos` → 查詢所有代辦事項 → `getTodos`
- `POST /todos` → 建立新代辦 → `createTodo`
- `PUT /todos/:id` → 更新某筆代辦 → `updateTodo`
- `DELETE /todos/:id` → 刪除某筆代辦 → `deleteTodo`

這樣一來，Controller 和 Route 的責任就分得很清楚：

- **Route = 請求分配器**
- **Controller = 處理邏輯**

建立 `src/routes/todoRoutes.ts`：

```tsx
import { Router } from "express";
import {
  getTodos,
  createTodo,
  updateTodo,
  deleteTodo,
} from "../controllers/todoController";

const router = Router();

router.get("/", getTodos);
router.post("/", createTodo);
router.put("/:id", updateTodo);
router.delete("/:id", deleteTodo);

export default router;
```

---

## 整合到主程式

最後一步就是在 `app.ts` 裡，把 `todoRoutes` 整合進主程式。

這樣當使用者發送請求到 `/api/todos` 時，Express 就會把它交給我們剛剛寫好的 `todoRoutes`，再由對應的 Controller 去處理。

修改 `src/app.ts`：

```tsx
import "reflect-metadata";
import express from "express";
import { AppDataSource } from "./config/db";
import todoRoutes from "./routes/todoRoutes";
import dotenv from "dotenv";

dotenv.config();

const app = express();
app.use(express.json());

app.use("/api/todos", todoRoutes); // 加上 Todo 路由

app.get("/", (req, res) => {
  res.send("Hello, iThome 2025!");
});

const PORT = process.env.PORT || 3000;

AppDataSource.initialize()
  .then(() => {
    console.log("📦 DB Connected!");
    app.listen(PORT, () => {
      console.log(`🚀 Server running on http://localhost:${PORT}`);
    });
  })
  .catch((err) => {
    console.error("❌ DB connection failed:", err);
  });
```

---

👉 到這裡，我們就完成了一個最基礎的 TodoList API。

但是目前還沒有資料庫可以測試 (還不能用 Postman 打 API 😂)，下一篇我們就來介紹 Render 服務上的資料庫應用，把整段串接起來。

---

## 補充資料

[Github 範例程式碼](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/d766050e40229dd97cd90a5c94c667d14cc90c2f)

> commit : Day 8 initialize todo‑list API

---
