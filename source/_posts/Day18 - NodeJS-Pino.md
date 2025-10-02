---
layout: day
title: Day 18 - console.log 退役啦！Node.js Pino 帶你升級專案 Log
date: 2025-10-02 20:25:01
tags:
  - TypeScript
  - Node.js
  - Tools
---

## 前言

在寫程式的過程中，你是不是也常常這樣做？

```tsx
console.log("資料有進來嗎？", data);
console.log("API 回傳結果：", response);
console.log("進到這裡了嗎？");
```

👆 這些 `console.log` 幾乎是每個工程師的「人生必經之路」。它快速、直覺，甚至帶點安全感，因為我們可以立即知道程式有沒有跑到某一段。

不過，當專案越來越大、功能越來越複雜，`console.log` 的缺點就會暴露出來。今天，我們來聊聊什麼是「日誌 (log)」，以及它在專案中的價值，並實戰介紹一個超好用的 Node.js 日誌套件 **Pino** 。

<!-- more -->

---

## 為什麼需要日誌 (Log)？

在系統開發中，日誌就是程式的「黑盒子紀錄器」。它能幫助我們：

- **Debug 除錯**：快速找到程式碼的執行狀況與錯誤來源。
- **監控系統**：觀察 API 請求、伺服器效能與錯誤比例。
- **追蹤使用者行為**：紀錄誰登入了系統、誰刪除了哪筆資料。
- **預警通知**：當錯誤異常增加時，可以觸發告警，馬上通知工程師。

簡單來說，log 是專案穩定性和維運效率的「關鍵後盾」。

---

## `console.log` 的問題

雖然 `console.log` 很方便，但它在大型專案中有許多限制：

1. **沒有層級**：錯誤與提示長得一樣，難以過濾。
2. **缺乏結構**：只會印文字，不利於後續分析。
3. **難以集中管理**：如有多台伺服器各印 log 的情況，較難彙整。
4. **效能問題**：高併發環境下，`console.log` 會拖慢程式速度。

所以，我們需要一個更專業的解決方案：**Pino**。

---

## Pino 是什麼？

`Pino` 是目前 Node.js 生態中最熱門的日誌套件之一，特色包括：

- **效能超快**（比 winston 快 5–10 倍）。
- **預設 JSON 格式**，方便集中管理（如 Grafana Loki）。
- **開發模式友善**，可搭配 `pino-pretty` 輸出彩色、人類可讀的 log。
- **支援多層級**（info、warn、error、debug…），避免訊息混雜。

接下來就實戰，帶你把專案的 `console.log` 全部升級成 Pino 吧！

---

## 步驟 1：安裝套件 📦

```bash
npm install pino
npm install -D pino-pretty @types/pino
```

**套件說明：**

- `pino`：主要的日誌記錄套件。
- `pino-pretty`：開發環境用的美化輸出工具（devDependencies）。
- `@types/pino`：TypeScript 型別定義。

---

## 步驟 2：建立 Logger 配置 📝

在 `src/utils/logger.ts` 中建立一個共用的 logger：

```tsx
import pino from "pino";

// 判斷是否為開發環境
const isDevelopment = process.env.NODE_ENV !== "production";

// 建立 logger 實例
const logger = pino({
  level: process.env.LOG_LEVEL || "info",
  transport: isDevelopment
    ? {
        target: "pino-pretty",
        options: {
          colorize: true,
          translateTime: "SYS:yyyy-mm-dd HH:MM:ss",
          ignore: "pid,hostname",
        },
      }
    : undefined, // 生產環境直接輸出 JSON
});

export default logger;
```

這樣，我們就有一個可以隨時呼叫的 `logger`，而且會根據環境自動調整輸出格式。

---

## 步驟 3：在 app.ts 使用 Logger 🚀

修改 `app.ts`，測試不同級別的日誌：

```tsx
import "reflect-metadata";
import express from "express";
import { AppDataSource } from "./config/db";
import todoRoutes from "./routes/todoRoutes";
import authRoutes from "./routes/authRoutes";
import uploadRoutes from "./routes/uploadRoutes";
import dotenv from "dotenv";
import logger from "./utils/logger"; // 導入 logger

dotenv.config();

const app = express();
app.use(express.json());

// 測試不同級別的日誌
logger.info("這是一般資訊");
logger.debug("這是除錯訊息");
logger.warn("這是警告");
logger.error("這是錯誤");

app.use("/api/todos", todoRoutes);
app.use("/api/auth", authRoutes);
app.use("/api/upload", uploadRoutes);

app.get("/", (req, res) => {
  res.send("Hello, iThome 2025!");
});

const PORT = process.env.PORT || 3000;

AppDataSource.initialize()
  .then(() => {
    logger.info("📦 DB Connected!");
    app.listen(PORT, () => {
      logger.info(`🚀 Server running on http://localhost:${PORT}`);
    });
  })
  .catch((err) => {
    logger.error({ err }, "❌ DB connection failed:");
  });
```

不同級別的日誌測試結果 :

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day18-Pino/1-log-test.png?raw=true)

---

## 步驟 4：HTTP Request Logger 中間件 🔍

想要紀錄 API 請求？加上 `pino-http` 就搞定：

```bash
npm install pino-http
```

在 `app.ts` 加入：

```tsx
import pinoHttp from "pino-http";

app.use(pinoHttp({ logger }));
```

這樣，每一次 API 請求和回應都會自動被記錄。

---

## 步驟 5：在 Controller 中使用 Logger 💡

在任何檔案都能這樣呼叫：

```tsx
import logger from "../utils/logger";

logger.info("一般資訊");
logger.debug("除錯訊息");
logger.warn("警告訊息");
logger.error("錯誤訊息");
```

---

## 實際測試

當我們新增一個 Todo 時，log 會長這樣：(紀錄 API 請求和回應)

```json
[2025-10-02 15:34:09] INFO: request completed
    req: {
      "id": 1,
      "method": "POST",
      "url": "/api/todos",
      "query": {},
      "params": {},
      "headers": {
        "content-type": "application/json",
        "authorization": "Bearer <token>",
        "user-agent": "PostmanRuntime/7.48.0",
        "accept": "*/*",
        "postman-token": "77cca9bc-0c90-4539-9df1-923e409eddc7",
        "host": "localhost:3000",
        "accept-encoding": "gzip, deflate, br",
        "connection": "keep-alive",
        "content-length": "38"
      },
      "remoteAddress": "::1",
      "remotePort": 63412
    }
    res: {
      "statusCode": 201,
      "headers": {
        "x-powered-by": "Express",
        "content-type": "application/json; charset=utf-8",
        "content-length": "245",
        "etag": "W/\"f5-HbNtbBTEZGIV/ZC0+J4ggDvV1Qo\""
      }
    }
    responseTime: 1033
```

---

## 額外提示 💡

1. **環境變數**

   在 `.env` 檔設定 log 層級：

   ```
   LOG_LEVEL=debug
   NODE_ENV=development
   ```

2. **日誌層級順序（低 → 高）**

   `trace < debug < info < warn < error < fatal`

3. **生產環境建議**
   - 輸出 JSON 格式（方便集中收集）。
   - 搭配 log 管理工具（如 Grafana Loki）。

---

## 總結

- `console.log` 缺乏層級、結構化，中大型系統維護上較難。
- **Pino** 提供高效能、結構化、可擴展的 log 解決方案。
- 透過層級、格式化、HTTP 中間件，我們能輕鬆掌握系統健康狀況。

👉 從今天開始，讓我們跟 `console.log` 說掰掰，迎接更專業的日誌管理吧！

---

## 參考資源

> commit : add pino plugin

[Github 連結](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/1fd0dd5fc31366fb503f98116fd2ab2166647263)
