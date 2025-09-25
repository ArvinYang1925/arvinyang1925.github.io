---
layout: day
title: Day 11｜一鍵上線！完整部署到 Render 的實戰流程
date: 2025-09-25 13:42:25
tags:
  - TypeScript
  - Node.js
  - Render
---

昨天我們完成了部署的前置作業，今天當然要來實戰部署啦！這篇文章會帶你一步步把 Node.js + TypeScript 專案部署到 **Render**，並驗證 API 是否能正常運作。

<!-- more -->

---

## Render 部署設定步驟

### 1. 建立 Web Service

到 Render Dashboard 右上角點擊 **New → Web Service**，建立一個新的服務。

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/1-new-web-service.png?raw=true)

接著選擇 **連結 GitHub 專案**，點擊 **Connect**。

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/2-link-github-repo.png?raw=true)

---

### 2. 選擇專案與分支

- **Language** 選擇 `Node`
- **Branch** 選擇 GitHub 專案的 `develop` 分支

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/3-new-service-info.png?raw=true)

---

### 3. 設定建置與啟動指令

Render 需要知道如何建置與啟動專案。

**建置指令 (Build Command)**

```
npm install && npm run build
```

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/4-build-command.png?raw=true)

**啟動指令 (Start Command)**

```
npm run start
```

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/5-start-command.png?raw=true)

---

### 4. 選擇 Instance Type

如果是學習或 Side Project，可以先選擇 **Free 方案**。

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/6-instance-type.png?raw=true)

---

### 5. 配置環境變數

接著到 Environment Variables 區塊，新增專案需要的環境變數。

例如：

```
DB_HOST=<你的資料庫主機>
DB_PORT=5432
DB_USERNAME=<你的使用者名稱>
DB_PASSWORD=<你的密碼>
DB_NAME=<你的資料庫名稱>
```

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/7-env.png?raw=true)

點擊 **Add from .env**。

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/8-add-env.png?raw=true)

點擊 **Add variables**，就能看到變數成功新增。

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/9-env-result.png?raw=true)

---

### 6. 開始部署

設定完成後，點擊 **Deploy Web Service**，Render 就會開始自動建置與部署。

**部署中畫面**

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/10-deploying.png?raw=true)

**部署成功畫面**

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/11-deployed.png?raw=true)

---

## 驗證部署結果

部署完成後，Render 會提供一個公開網址。

透過該網址即可呼叫 API：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/12-req-link.png?raw=true)

使用 Postman 測試新增 Todo 資料，API 回應正常 ✅

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day11-Render/13-postman-test.png?raw=true)

---

🎉 恭喜你完成了一個完整的 Express + TypeScript + TypeORM 後端專案，並成功部署到 Render！

從這一步開始，你的專案已經能讓全世界存取了 。

⚠️ **小提醒：Render Free Plan 限制**

如果你使用的是 Render 免費方案，會有以下幾點需要注意：

- 服務若 **15 分鐘沒有流量**，會自動進入休眠狀態。
- 當再次有人存取時，服務會重新啟動，通常需要等待 **30 秒左右**。
- 免費方案的硬體資源有限，若專案流量大或需要長時間穩定服務，建議升級到付費方案。

---

## 結語：從準備到上線 🚀

在 **Day 10**，我們完成了部署的所有前置作業：

- 認識 Render 與雲端平台的定位
- 修改 `package.json`，指定 Node.js 版本
- 調整程式碼以支援環境變數的 `PORT`
- 學會透過 GitHub PR 把 `feature` 分支合併到 `develop`

接著在 **Day 11**，我們則把 `develop` 分支部署到 Render，並完成：

- 建立 Web Service
- 設定 Build 與 Start 指令
- 配置資料庫環境變數
- 驗證部署成功，API 可以正常使用

至此，我們已經把專案從「本地端」正式推上「雲端」，讓全世界都能透過網址存取 。

接下來的篇章，我們將來點輕鬆的主題，介紹 Prettier 、ESLint 小工具，並逐步探索更多進階主題（檔案上傳、驗證系統等），讓專案更接近真實商業應用。
