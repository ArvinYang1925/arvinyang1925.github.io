---
layout: day
title: Day 14 - API 驗證救星：用 Zod 驗證來檢查
date: 2025-09-28 19:36:53
tags:
  - TypeScript
  - Node.js
  - Tools
---

在開發後端 API 的時候，你一定遇過這些狀況：

- 前端傳來的資料少了一個欄位。
- 輸入的字串太長，直接讓資料庫報錯。
- 原本應該是 `boolean`，結果卻收到 `"true"` 或 `1`。

如果每次都要手動檢查 `req.body`，程式碼會變得又長又難維護。

👉 這時候，**Zod 就是我們的防呆神器**。

<!-- more -->

---

## 1. 為什麼要用 Zod？

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day14-Zod/1-zod.png?raw=true)

Zod 是一個 **TypeScript-first 的資料驗證工具**，它的特點包括：

- **語法直覺**：`z.string().min(1)` 就能定義規則。
- **與 TS 整合**：Schema 本身就是型別，可以直接推導。
- **後端防呆**：避免錯誤資料寫進資料庫。

---

## 2. Zod 基礎知識

先來看一個小範例：

```tsx
import { z } from "zod";

// 定義 schema
const userSchema = z.object({
  name: z.string().min(1, "Name is required"),
  age: z.number().int().min(0),
});

// 驗證資料
const result = userSchema.safeParse({ name: "", age: -1 });

if (!result.success) {
  console.log(result.error.errors);
  // [
  //   { message: "Name is required", path: ["name"] },
  //   { message: "Number must be greater than or equal to 0", path: ["age"] }
  // ]
}
```

常見型別：

- `z.string()`：字串
- `z.number()`：數字
- `z.boolean()`：布林
- `z.object({...})`：物件

常用驗證方法：

- `.min(n)` / `.max(n)`：限制範圍
- `.optional()`：欄位可選
- `.email()`：檢查 Email 格式

也可用線上 Zod Playground 快速測試一下:

https://zod-playground.vercel.app/?appdata=N4IgzgxgFgpgtgQxALhALwHQHsBGArGCAFwApgAdAOwAJrKE4ZlrMwiAnAS0oHMSBKDHG4kAjABpq5EADkGMapzDV2MAI4BXTqoAm0-uKq0EPJiwyUNcHDHYCM3UoOGUSABgNUAvvwDcIcRAANwQAGw0YMBQAbRBgOnlmaQApLChKaUkTMwAmN2ovEABdQKDbME4sShQQABYMUQbRAJAlAC0sHQBZbk4UADMwsBgvIA

---

## 3. 安裝 Zod

在專案根目錄執行：

```bash
npm install zod
```

---

## 4. 建立驗證 Schema

建立 `src/validator/todoValidation.ts`：

```tsx
import { z } from "zod";

export const createTodoSchema = z.object({
  title: z
    .string()
    .min(1, { message: "Title is required" })
    .max(100, { message: "Title must be at most 100 characters" }),
});

export const updateTodoSchema = z.object({
  title: z.string().optional(),
  completed: z.boolean().optional(),
});
```

---

## 5. 在 Controller 使用驗證

在 `todoController.ts` 中對 `createTodo`、`updateTodo` 套用 `.safeParse()`：

```tsx
import { Request, Response, NextFunction } from "express";
import { AppDataSource } from "../config/db";
import { Todo } from "../entities/Todo";
import {
  createTodoSchema,
  updateTodoSchema,
} from "../validator/todoValidation";

const todoRepository = AppDataSource.getRepository(Todo);

export async function createTodo(
  req: Request,
  res: Response,
  next: NextFunction
) {
  try {
    const parsed = createTodoSchema.safeParse(req.body);
    if (!parsed.success) {
      res.status(400).json({ status: "failed", message: "標題過長或過短" });
      return;
    }

    const newTodo = todoRepository.create({ title: parsed.data.title });
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
) {
  try {
    const { id } = req.params;
    const parsed = updateTodoSchema.safeParse(req.body);

    if (!parsed.success) {
      res.status(400).json({ status: "failed", message: "更新資料格式錯誤" });
      return;
    }

    const todo = await todoRepository.findOneBy({ id });
    if (!todo) {
      res.status(404).json({ status: "error", message: "Todo not found" });
      return;
    }

    todo.title = parsed.data.title ?? todo.title;
    todo.completed = parsed.data.completed ?? todo.completed;

    const updatedTodo = await todoRepository.save(todo);
    res.json({ status: "success", data: updatedTodo });
  } catch (error) {
    next(error);
  }
}
```

---

## 6. 成果展示

- ✅ 正確資料 → 正常新增 / 更新
- ❌ 錯誤資料 → 直接被擋下並回傳錯誤訊息
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day14-Zod/2-postman-test.png?raw=true)

---

## 7. 總結

今天我們學到：

- Zod 是一個簡單直覺的 **驗證工具**。
- 透過 `safeParse()` 可以有效攔截不合法資料。
- 在 API Controller 直接使用 Schema，就能大幅提升專案的安全性與可維護性。

下次寫 API 時，別忘了這個小幫手！

---

## 補充資源

> commit : add zod validation

[Github 連結](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/2bf8d89d9096edd2960abf384ceb98528f43fc99)

- [Zod 官方文件](https://zod.dev/)
- [Zod YouTube 教學](https://www.youtube.com/results?search_query=zod+validation)
