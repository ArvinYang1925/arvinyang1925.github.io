---
layout: day
title: Day 13 - 一致的程式碼：ESLint 導入
date: 2025-09-27 00:13:49
tags:
  - TypeScript
  - Node.js
  - Tools
---

## ESLint 的歷史與簡介

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day13-ESLint/1-eslint.png?raw=true)

在 JavaScript 的早期，開發者缺乏統一的規範，程式碼容易因個人習慣而變得混亂。

- 2000 年代，出現了 **JSLint**（由 Douglas Crockford 開發，JavaScript 語言守護者之一），用來檢查常見錯誤。
- 之後社群又發展出 **JSHint**，提供更彈性的規則與設定。
- 到了 **2013 年**，Nicholas C. Zakas（前 Yahoo! JS 團隊成員）推出 **ESLint**，透過「規則驅動 + 插件機制」，讓開發者可以自由擴展並制定專案專屬規範。

今天，ESLint 已經成為 JavaScript/TypeScript 專案裡最常見的程式碼檢查工具。

<!-- more -->

---

## 為什麼需要 ESLint？

和昨天的 **Prettier** 不同，ESLint 專注於 **程式邏輯與最佳實踐**。

### 常見檢查範例

- **程式錯誤**：未定義變數、未使用的變數、錯誤的語法。
- **最佳實踐**：禁止使用 `var`、避免 `any`、規定函式宣告風格。

### 為什麼重要？

1. **降低 Bug 風險**：提早發現潛在錯誤，避免程式在 runtime 才爆炸。
2. **提升可讀性**：團隊都遵守相同規範，程式碼更一致。
3. **改善協作體驗**：Code Review 時，機器會先抓語法與規範問題，讓人類專注於商業邏輯。

👉 簡單來說：

- **Prettier**：讓程式碼「好看」
- **ESLint**：讓程式碼「正確」

---

## VSCode 安裝 ESLint

這邊也建議在 VSCode 裡安裝 ESLint 套件：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day13-ESLint/2-vscode-eslint.png?raw=true)

---

## 專案安裝 ESLint

在專案裡安裝 ESLint 以及相關套件：

```bash
npm install eslint @eslint/js eslint-config-prettier eslint-plugin-prettier @typescript-eslint/parser @typescript-eslint/eslint-plugin --save-dev
```

---

## 設定 ESLint

在專案根目錄新增 `eslint.config.js`：

```jsx
const js = require("@eslint/js");
const globals = require("globals");
const prettier = require("eslint-config-prettier");
const tsParser = require("@typescript-eslint/parser");
const tsPlugin = require("@typescript-eslint/eslint-plugin");

module.exports = [
  {
    files: ["**/*.ts"], // 檢查 .ts 檔案
    languageOptions: {
      parser: tsParser, // 用 TypeScript 的解析器
      sourceType: "module", // 支援 ES 模組
      globals: {
        ...globals.node, // Node.js 的全域變數（像 process）
      },
    },
    plugins: {
      "@typescript-eslint": tsPlugin, // TypeScript 專屬插件
      prettier: require("eslint-plugin-prettier"), // Prettier 插件
    },
    rules: {
      ...js.configs.recommended.rules, // ESLint 基本規則
      ...tsPlugin.configs.recommended.rules, // TypeScript 推薦規則
      ...prettier.rules, // 正確使用 prettier 配置
      "prettier/prettier": "error", // Prettier 格式錯誤會顯示
      semi: ["error", "always"], // 行尾一定要有分號
      quotes: ["error", "double"], // 用雙引號
      "no-var": "error", // 不准用 var
      "@typescript-eslint/no-unused-vars": ["error"], // 不准有沒用到的變數
      "@typescript-eslint/no-explicit-any": "error", // 禁止使用 any 類型
      // 改用 TypeScript 版本的 func-style
      "func-style": ["error", "declaration", { allowArrowFunctions: true }],
      // 禁止一般函式表達式
      "no-restricted-syntax": [
        "error",
        {
          selector: "FunctionExpression",
          message: "請使用函式宣告式或箭頭函式，避免使用一般函式表達式 😎",
        },
      ],
    },
  },
];
```

在 `package.json` 裡加上 script：

```json
"scripts": {
    "lint": "eslint \"src/**/*.ts\"",
    "lint:fix": "eslint \"src/**/*.ts\" --fix"
}
```

- `npm run lint`：檢查程式碼規則
- `npm run lint:fix`：自動修復能修的錯誤

---

## 測試 ESLint

在 `/src` 下新增 `testLint.ts`：

```tsx
import express from "express";

const app = express();

// 違反 1: sayHello 宣告但未使用
function sayHello(name: any) {
  // 違反 2: 用 any 型別（TypeScript 規則）
  var greeting = "Hello " + name; // 違反 3: 用 var 宣告

  let unusedVar = 42; // 違反 4: 未使用的變數
  console.log(greeting);
}

// 違反 5:請使用函式宣告式或箭頭函式，避免使用一般函式表達式
const a = function (name: string) {
  console.log(name);
};
a("Peter");

app.get("/test", (req, res) => {
  res.send("Test route");
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

執行：

```bash
npm run lint
```

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day13-ESLint/3-npm-run-lint.png?raw=true)

你會看到 ESLint 把所有違規都列出來，幫助你更快找到問題。

---

## 小結

- ESLint 誕生於 2013 年，是目前最常見的 JavaScript/TypeScript 程式碼檢查工具。
- 與 Prettier 搭配：
  - **Prettier** → 排版風格一致
  - **ESLint** → 邏輯與規範正確
- 在專案中安裝、設定後，可以自動檢查、修復程式，避免錯誤。

---

## 補充資料

> commit : setup ESLint

[Github 連結](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/2786c17e588e8598f42e8349a901f3d6717afe46)

👉 明天 (Day14) 我們則會來介紹 Zod 驗證 (API 驗證守護神)，期待期待 💪
