---
layout: day
title: Day 10 - 部署啟程！從 Render 部署前置作業到 GitHub PR
date: 2025-09-24 12:38:13
tags:
  - TypeScript
  - Node.js
  - Render
---

## Render 服務簡介

![1-render.png](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day10-Render-deploy/1-render.png?raw=true)

在前幾天的系列文章中，我們已經完成了本地端的環境建置與功能實作。但真實專案裡，光是「我的電腦能跑」是不夠的，我們需要把服務放到雲端，讓全世界都能存取。

<!-- more -->

常見的雲端平台有 **AWS**、**GCP**、**Azure** 等，它們提供完整的基礎建設服務（IaaS）。但對於初學者或中小型專案來說，這些平台的學習曲線偏高、設定也比較繁瑣。

**Render** 屬於 **PaaS（平台即服務）**，提供比 AWS、GCP 更直覺的部署體驗，只需要專注於程式碼本身，Render 就能自動幫我們完成環境建置、佈署、監控。

👉 適合快速上手、部屬 Side Project 或個人練習專案。

---

## 用 Github 帳號第三方登入 Render 服務

![Screenshot 2025-09-23 at 7.56.36 PM.png](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day10-Render-deploy/2-signin-render.png?raw=true)

Render 提供 **Github 帳號登入**，這樣做的好處是可以直接與 GitHub 專案連動。

一旦我們在 GitHub 上有新的 commit（例如合併 PR），Render 就能自動觸發建置與部署。

**優點：**

- 自動同步 GitHub repo
- 省去手動上傳程式碼的麻煩
- CI/CD 整合更簡單

---

## Node.js 專案部署前置作業

### 修改 package.json

1. **新增啟動與建置指令**

   - 在 `package.json` 的 `scripts` 中加入：

     ```json
     "scripts": {
       "start": "node dist/app.js",
       "build": "tsc"
     }
     ```

   - `start`：執行編譯後的程式。
   - `build`：把 TypeScript 編譯為 JavaScript。

2. **指定 Node.js 版本**

   - 在 `package.json` 中加入：

     ```json
     "engines": {
       "node": "20.x"
     }
     ```

   - **為什麼要指定版本？**
     Render 預設可能使用不同的 Node.js 版本，如果與本地端差異過大，可能導致部署錯誤。
     指定 Node.js 版本能確保 Render 的執行環境與我們本地端一致，減少版本不相容的問題。

---

### 確認 app.ts

確保 `app.ts` 使用環境變數的 PORT：

```tsx
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

⚠️ **小提醒**

在 Render 上，服務的埠號（`PORT`）是 **自動分配** 的，並透過 `process.env.PORT` 傳入程式。

因此 **不需要在 Render 控制台手動新增 `PORT` 環境變數**。

只要程式碼有寫：

```tsx
const PORT = process.env.PORT || 3000;
```

就能正確接收 Render 給的 Port。

---

### 編譯與運行

在部署到 Render 前，先在本地端測試：

1. **編譯程式碼**

   - 執行：

     ```bash
     npm run build
     ```

   - 這會將 TypeScript 編譯為 JavaScript，輸出到 `dist` 資料夾。

2. **運行編譯後的程式**

   - 執行：

     ```bash
     npm run start
     ```

   - 檢查是否能正常啟動伺服器並連接到資料庫。

> [注意]：本地端測試成功後再進行部署，確保程式碼無誤。

> commit : Day 10 Render Deploy setting

[範例程式碼](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/dec62b5d6891897d92eb39d5367be0471c9c2f37)

---

## GitHub Merge PR 流程

在專案開發時，我們通常會用 **feature 分支** 來開發新功能，完成後再透過 **Pull Request (PR)** 合併到 `develop` 分支。以下是簡單的操作流程：

---

### Step 1. 建立 Pull Request

![Screenshot 2025-09-23 at 8.58.41 PM.png](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day10-Render-deploy/3-create-pr.png?raw=true)

點擊 **Compare & pull request** !

---

### Step 2. 撰寫 PR 描述

選擇要合併的分支：

- **base**：`develop`
- **compare**：`feature/init-ts-express`

![Screenshot 2025-09-23 at 9.01.06 PM.png](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day10-Render-deploy/4-pr-desc.png?raw=true)

在描述區塊中，列出這次 PR 的主要變更，例如：

- 初始化 Node.js + TypeScript 專案架構
- 建立基本 Express server
- 新增 TodoList API (CRUD)
- 整合 TypeORM 與資料庫連線

填寫 PR 標題與描述，準備送出 PR。按下 Create pull request !

---

### Step 3. 檢視 PR 狀態

送出 PR 後，可以看到 Commit 記錄與檔案變更。

此時 GitHub 會檢查與 `develop` 分支是否有衝突：

- ✅ **No conflicts** → 可以安全合併
- ⚠️ **有衝突** → 需要先解決衝突

![Screenshot 2025-09-23 at 9.01.35 PM.png](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day10-Render-deploy/5-check-pr.png?raw=true)

---

### Step 4. 合併到 develop

確認無誤後，點擊 **Merge pull request** → **Confirm merge**，就能把 feature 分支合併到 `develop`。

合併完成後，`develop` 分支就會更新，包含最新的功能。

![Screenshot 2025-09-23 at 9.02.07 PM.png](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day10-Render-deploy/6-merge-result.png?raw=true)

---

先完成這些步驟後，明天我們就能把 `develop` 分支部署到 Render 囉 🚀。
