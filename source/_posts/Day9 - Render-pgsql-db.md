---
layout: day
title: Day 9｜Render 雲端啟動：資料庫連線全攻略
date: 2025-09-23 20:20:50
tags:
  - TypeScript
  - Node.js
  - Render
  - pgSQL
---

## 前言 - 什麼是資料庫？

簡單來說，**資料庫（Database）** 是一種用來 **儲存、管理與存取資料** 的系統。

它能幫助我們把資料有條理地組織起來，並透過查詢語言（例如 SQL）快速檢索與更新。

常見的資料庫類型有：

- **關聯式資料庫（RDBMS）**：以表格（Tables）方式存放資料，支援 SQL，例如 MySQL、PostgreSQL、SQLite。
- **非關聯式資料庫（NoSQL）**：不一定使用表格，常見於 JSON 文件、Key-Value、Graph 等結構，例如 MongoDB、Redis。

<!-- more -->

### 常見資料庫比較

| 特點         | MySQL                           | PostgreSQL                             |
| ------------ | ------------------------------- | -------------------------------------- |
| 歷史         | 1995 年誕生，社群廣泛           | 1996 年誕生，功能強大                  |
| 使用場景     | 網站後端（WordPress、電商系統） | 複雜商業系統、數據分析平台             |
| SQL 標準相容 | 普及但部分不相容                | 高度相容 ANSI-SQL                      |
| 擴充能力     | 功能完整，但擴展性略少          | 支援更多型別（JSON、陣列）、自定義函數 |
| 性能         | 讀取速度快，適合高頻查詢        | 交易一致性（ACID）更嚴謹               |
| 儲存 JSON    | 有支援，但功能較基本            | 支援 JSONB，操作更強大                 |
| 授權         | 開源 GPL                        | 開源 PostgreSQL License                |

👉 **簡單來說**：

- **MySQL**：適合中小型專案或網站開發。
- **PostgreSQL**：功能更全面，適合大型專案、金融系統、數據分析。

以上為兩種資料庫的主要定位，但實際選擇時還是要根據具體團隊需求和情境來決定。

---

## Render 資料庫服務簡介

Render 除了能部署前後端應用程式，也提供 **PostgreSQL 資料庫服務**，其優點包括：

- 免費方案可用（適合開發 / 測試）。
- 內建 **自動備份** 功能。
- 提供 **Connection Info**，可直接複製連線字串使用。
- 支援外部連線（可用 DBeaver、pgAdmin、psql 連線）。
- 內建 SSL 安全機制，保障連線安全。

---

## ✅ 建立 PostgreSQL 資料庫

1. 點擊右上角的 **「New」** 按鈕。
2. 選擇「**Postgres**」。

![Screenshot 2025-04-24 at 9.21.05 PM](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day9-Render-DB/1-create-pg-database.png?raw=true)

1. 設定資料庫細節（如下）：
   - **Name**：資料庫 Instance 名稱。
   - **Region**：選擇靠近你的地區（例如 Singapore 新加坡）。
   - **Database Name / User / Password**：Render 會自動產生，也可以自訂。
   - **Plan Option**：先選擇 Free 方案即可，方便測試使用。
     ![dashboard.render.com_new_database](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day9-Render-DB/2-new-pg-db-info.png?raw=true)
2. 點擊「**Create Database**」。
3. 等待 DB Instance 生成後，即可在 Dashboard 中看到。

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day9-Render-DB/3-db-instance.png?raw=true)

4. 點擊對應的 Service Name 以查看資料庫詳細資訊。

---

## ✅ 資料庫存取方式

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day9-Render-DB/4-external-url.png?raw=true)

Render 會自動產生一組 **外部連線網址（External Connection URL）**，可供應用程式或團隊使用：

- 進入 Postgres 資料庫頁面。
- 在「**Connection Info**」找到 `External Database URL`。
- 複製這組 URL，並用於 `psql`、DBeaver、應用程式或 ORM。

ℹ️ 注意：這組 URL 含有帳號密碼，**務必保護好，不要放在 GitHub 等公開地方**。

---

### External Database URL

```
postgresql://node_ts_demo_db_user:<password>@dpg-d33uq4q4d50c73ejo3rg-a.singapore-postgres.render.com/node_ts_demo_db
```

格式通常為：

```
postgres://<user>:<password>@<host>/<dbname>
```

拆解（設定 DBeaver 可用）：

- **使用者**：node_ts_demo_db_user
- **密碼**：password
- **主機**：[dpg-d33uq4q4d50c73ejo3rg-a.singapore-postgres.render.com](http://dpg-d33uq4q4d50c73ejo3rg-a.singapore-postgres.render.com/)
- **資料庫名稱**：node_ts_demo_db

---

## 用 DBeaver 測試連線

[DBeaver](https://dbeaver.io/) 是一款跨平台的開源資料庫管理工具，支援 MySQL、PostgreSQL、SQLite、MongoDB 等多種資料庫。

### 安裝與連線步驟

1. 下載並安裝 DBeaver。
2. 點選「**New Database Connection**」。
3. 選擇「**PostgreSQL**」。
4. 輸入 Render 提供的資訊：
   - **Host**：`dpg-xxxxx.singapore-postgres.render.com`
   - **Database**：`ts_template_db`
   - **Username**：`ts_template_db_user`
   - **Password**：`password`
   - **Port**：`5432`
     ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day9-Render-DB/5-dbeaver-test-conn.png?raw=true)
5. 點擊 **Test Connection**，成功會顯示「Connected」。

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day9-Render-DB/6-db-connected.png?raw=true)

---

## Node.js 專案連線 Render 資料庫與 API 測試

### 修改 `.env` 檔案，加入資料庫連線資訊

```bash
DB_HOST=dpg-d33uq4q4d50c73ejo3rg-a.singapore-postgres.render.com
DB_NAME=node_ts_demo_db
DB_USERNAME=node_ts_demo_db_user
DB_PASSWORD=password
DB_PORT=5432
```

### 啟動伺服器並測試連線

使用先前 Day 6 設定好的 script：

```bash
npm run dev
```

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day9-Render-DB/7-ssl-problem.png?raw=true)

會發現連線失敗，因為 SSL 相關問題。

---

### 解決 SSL 問題

- **開發環境**：可暫時關閉驗證以便快速測試。
- **正式環境（Production）**：建議保持預設驗證（`true`），以確保安全性。

若遇到 SSL 連線錯誤，在 `db.ts` 的 `AppDataSource` 配置中加入：

```tsx
ssl: { rejectUnauthorized: false },
```

更新後的 `src/config/db.ts`：

```tsx
export const AppDataSource = new DataSource({
  type: "postgres",
  host: process.env.DB_HOST,
  port: Number(process.env.DB_PORT),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  entities: [Todo],
  synchronize: true,
  logging: true,
  ssl: { rejectUnauthorized: false },
});
```

✅ 成功連線的終端機畫面：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day9-Render-DB/8-nodejs-conn-success.png?raw=true)

(可觀察到 typeORM 已經跟資料庫互動並藉由 Todo Entity 創立 Todo 表格)

DBeaver 畫面：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day9-Render-DB/9-todo-table.png?raw=true)

---

## API 測試範例

使用 Postman 測試幾個簡單的 API：

| 方法   | 路徑                | Body 範例                      | 回應範例                                                                                         |
| ------ | ------------------- | ------------------------------ | ------------------------------------------------------------------------------------------------ |
| GET    | `/api/todos`        | -                              | `{ "status": "success", "data": [ ... ] }`                                                       |
| POST   | `/api/todos`        | `{ "title": "學 TypeScript" }` | `{ "status": "success", "data": { "id": "xxx", "title": "學 TypeScript", "completed": false } }` |
| PUT    | `/api/todos/<uuid>` | `{ "completed": true }`        | `{ "status": "success", "data": { "id": "1", "title": "學 TypeScript", "completed": true } }`    |
| DELETE | `/api/todos/<uuid>` | -                              | `{ "status": "success", "data": { "affected": 1 } }`                                             |

---

## Postman 測試結果

成功新增 API 資料：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day9-Render-DB/10-postman-create-todo.png?raw=true)

---

## 🎯 今日收穫

今天我們不只學會了如何在 Render 上建立 PostgreSQL 資料庫，還實際完成了幾件關鍵任務：

- 了解了 **資料庫的基本概念**，並比較了 MySQL 與 PostgreSQL 的差異。
- 在 Render 建立了 **Postgres 資料庫服務**，並熟悉了連線方式。
- 使用 **DBeaver** 成功測試連線，確認資料庫可以正常使用。
- 在 **Node.js + TypeORM 專案**中完成連線，並處理 SSL 問題。
- 用 **Postman** 驗證了 API 與資料庫的互動，確認 CRUD 功能正常。

👉 到這一步，我們的專案已經有了「**完整的後端 + 真實的雲端資料庫**」，下一步就能進一步嘗試部署專案了！ 🚀

## 補充資料

[Github 範例程式碼](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/280a18f58c08410d2aebf95f6781766a45b18946)

> git commit : setup render database connection info
