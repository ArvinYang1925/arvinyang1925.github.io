---
layout: day
title: Day 6 - 如何建立一個 TypeScript + Node.js 環境 (1)：初始化專案
date: 2025-09-20 20:26:24
tags:
  - TypeScript
  - Node.js
---

昨天我們聊了很多概念，今天開始要**動手實作**啦！💻

我們的目標很簡單：建立一個能跑起來的 **TypeScript + Node.js 專案環境**，並讓瀏覽器成功回應 **「Hello, iThome2025 !」** 🚀

準備好了嗎？Let’s go！

<!-- more -->

---

## 一、專案初始化

### 1. 開一個專案資料夾

你可以直接在電腦裡開一個新資料夾，或是從 GitHub Clone 專案下來。

這邊示範 Git 版本，順便練習一下分支管理：

```bash
git clone <你的專案網址>
cd <專案資料夾>

# 建立 develop 分支
git checkout -b develop
git push --set-upstream origin develop

# 建立功能分支
git checkout -b feature/init-ts-express
git push --set-upstream origin feature/init-ts-express

```

👉 這樣就能在一個乾淨的分支上開始開發，方便日後合併 PR。

---

### 2. 加上 `.gitignore`

為了避免「一些不必要的檔案」被推到 GitHub，我們要新增一個 `.gitignore`：

```
# Node modules
node_modules/

# Logs
logs/
*.log
npm-debug.log*

# Environment variables
.env

# Build output
dist/
build/

# TypeScript cache
*.tsbuildinfo

# OS / Editor 系統檔案
.DS_Store
Thumbs.db

# VS Code 設定
.vscode/

```

**小提醒**：這一步很容易被忽略，但很重要，因為上傳 `node_modules` 到 GitHub 真的有可能會被嫌棄 😅

---

### 3. 初始化 Node.js 專案

執行：

```bash
npm init -y

```

這會快速生成一個 `package.json`，裡面記錄專案資訊和套件清單。

按下 `-y` 代表接受所有預設值，超省時！👌

---

## 二、安裝必要套件

我們的工具包包含：

- **Express**：後端 API 框架
- **TypeScript**：型別安全
- **@types**：型別定義檔
- **ts-node-dev**：開發時自動編譯 + 重啟

一次裝好：

```bash
npm install express
npm install -D typescript @types/express @types/node ts-node-dev

```

裝完後，`package.json` 的 `dependencies` 和 `devDependencies` 就會多出這些小夥伴。

---

## 三、設定 TypeScript

### 1. 建立 `tsconfig.json`

輸入：

```bash
npx tsc --init

```

這會生成 `tsconfig.json`。接著編輯成以下版本（直接覆蓋即可）：

```json
{
  "compilerOptions": {
    "target": "ES2020", // 用 ES2020 的 JavaScript 版本
    "module": "commonjs", // 用 CommonJS 模組
    "outDir": "dist", // 編譯輸出目錄
    "rootDir": "src", // 程式碼根目錄
    "strict": true, // 啟用嚴格型別檢查
    "esModuleInterop": true, // 支援模組互操作
    "experimentalDecorators": true, // 啟用裝飾器語法
    "emitDecoratorMetadata": true // 為 TypeORM 生成元數據
  },
  "include": ["src/**/*"], // 編譯 src 資料夾內的檔案
  "exclude": ["node_modules"] // 排除 node_modules
}
```

這邊要記住幾個關鍵設定：

- `rootDir` → 原始碼放哪裡
- `outDir` → 編譯輸出到哪裡
- `strict` → 打開型別檢查，保護你的程式碼
- `experimentalDecorators` + `emitDecoratorMetadata` → 為之後用 TypeORM 預先鋪路

然後新增一個 `src` 資料夾：

```bash
mkdir src

```

---

## 四、寫第一支程式 🚀

在 `src` 裡新增 `app.ts`：

```bash
touch src/app.ts

```

寫入以下內容：

```tsx
import express from "express";

const app = express();

app.get("/", (req, res) => {
  res.send("Hello, iThome2025!");
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 Server running on http://localhost:${PORT}`);
});
```

到這裡，你已經完成一個最小可運行的 Express + TS 程式！🎉

---

## 五、啟動與測試

### 1. 修改 `package.json`

在 `scripts` 中加上：

```json
"scripts": {
  "dev": "ts-node-dev src/app.ts"
}

```

---

### 2. 啟動伺服器

執行：

```bash
npm run dev

```

如果一切順利，你會看到：

```
🚀 Server running on http://localhost:3000

```

---

### 3. 開瀏覽器測試

輸入：

```
http://localhost:3000

```

看到 **Hello, iThome2025 !**，就代表環境完成 ✅

---

## 小結

今天我們完成了專案初始化的整個流程：

- Git 分支管理
- 設定 `.gitignore`
- 初始化 `package.json`
- 安裝必要套件
- 設定 TypeScript
- 建立第一個 Express 程式

從現在開始，你已經有一個能跑起來的 **TypeScript + Node.js 開發環境** 🎯

明天，我們會進一步讓專案結構更有組織感，並準備好迎接更複雜的 API 開發。

---

## 補充資源

[Github 範例程式碼](https://github.com/ArvinYang1925/iThome2025-node-ts/tree/feature/init-ts-express)

> commit message: Day 6 - set up Express + TS environment
