---
layout: day
title: Day 7 - 如何建立一個 TypeScript + Node.js 環境 (2)：專案架構與資料庫設定
date: 2025-09-21 12:25:24
tags:
  - TypeScript
  - Node.js
---

嗨～昨天我們已經有一個能跑起來的 **TypeScript + Node.js 開發環境**。接著，在正式開發 API 之前，先來點暖身 —— 建立一個清晰、可維護的專案架構，並完成環境變數與資料庫連線設定。這些基礎工具就是未來專案能否穩定推進的關鍵。

<!-- more -->

---

## 專案架構整理

### 建立資料夾結構

為了讓專案更有條理、方便後續維護，我們在 `src` 底下建立以下資料夾：

- `config`：存放設定檔（例如資料庫連線設定）。
- `controllers`：處理 API 業務邏輯。
- `entities`：定義資料庫模型 (Entity)。
- `middleware`：存放中間件。
- `routes`：定義 API 路由。
- `utils`：常用工具函數。

建立指令：

```bash
cd src
mkdir config controllers entities middleware routes utils
```

最終結構如下：

```
src/
  ├── config/
  ├── controllers/
  ├── entities/
  ├── middleware/
  ├── routes/
  ├── utils/
  └── app.ts
```

---

## 環境變數設定

為什麼要用環境變數？

因為專案常常需要存放敏感資訊（像是資料庫密碼、API 金鑰），這些東西絕不能直接寫死在程式碼裡。

### 安裝與設定 dotenv

1. 安裝套件與型別：

   ```bash
   npm install dotenv
   npm install @types/dotenv --save-dev
   ```

2. 在專案根目錄建立 `.env` 檔案：

   ```bash
   touch .env
   ```

   在檔案內輸入：

   ```
   DB_HOST=localhost
   DB_NAME=db_name
   DB_USERNAME=你的資料庫使用者
   DB_PASSWORD=你的密碼
   DB_PORT=5432
   ```

3. 確認 `.gitignore`有下方配置：

   ```
   .env
   ```

> ⚠️ .env 內含敏感資訊，務必避免上傳到 GitHub！

---

## 資料庫連線設定

這裡我們會使用 **PostgreSQL** 搭配 **TypeORM** 作為 ORM 工具。

### 安裝套件

```bash
npm install typeorm pg
```

### 設定 TypeORM 連線

在 `src/config` 建立 `db.ts`：

```tsx
import "reflect-metadata";
import { DataSource } from "typeorm";
import { Todo } from "../entities/Todo";
import dotenv from "dotenv";

dotenv.config();

export const AppDataSource = new DataSource({
  type: "postgres",
  host: process.env.DB_HOST,
  port: Number(process.env.DB_PORT),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  entities: [Todo],
  synchronize: true, // 開發階段建議 true，正式環境請改成 false
  logging: true,
});
```

> ⚠️ synchronize: true 會自動產生 ORM 所對應的資料表，僅限開發階段使用；正式環境建議使用 migration！

---

## 定義資料模型

在 TypeORM 裡，**Entity 就是資料庫模型**，負責定義表格結構。

### 建立 Todo Entity

1. 新增檔案：

   ```bash
   touch src/entities/Todo.ts

   ```

2. 編輯內容：

   ```tsx
   import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

   @Entity()
   export class Todo {
     @PrimaryGeneratedColumn("uuid")
     id!: string;

     @Column()
     title!: string;

     @Column({ default: false })
     completed!: boolean;
   }
   ```

3. 確認 `tsconfig.json`，確保支援裝飾器：

   ```json
   "experimentalDecorators": true,
   "emitDecoratorMetadata": true
   ```

> 💡 小技巧：
>
> - `!` 表示該屬性必填。

---

👉 明天（Day 8）我們即將開始開發 API ，讓 **TypeScript + Node.js 專案更豐富 🚀 !**

## 補充資源

[Github 範例程式碼](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/ea9757e05ffe4d0a7dd48d65c1e4cc37f25361d8)

> git commit : setup backend structure and typeORM setting
