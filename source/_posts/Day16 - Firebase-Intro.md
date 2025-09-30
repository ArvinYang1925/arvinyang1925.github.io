---
layout: day
title: Day 16 - Firebase Storage 初探：輕鬆搞定專案檔案上傳前置作業
date: 2025-09-30 22:48:18
tags:
  - TypeScript
  - Node.js
  - Tools
---

在前幾天，我們的 API 主要處理「資料」的 CRUD。

但在真實專案裡，除了文字資料，**圖片與檔案上傳** 也幾乎是必備功能（例如：會員大頭貼、商品圖片、文章配圖）。

這時候，**Firebase Storage** 就能派上用場啦！

它是 Google 提供的雲端檔案儲存服務，適合開發者快速上手，特別適合 side project 或 prototype。

<!-- more -->

---

## 1. Firebase Storage 是什麼？

- 由 Google Firebase 提供的 **雲端物件儲存** 服務
- 主要拿來存放 **圖片、影片、音訊、PDF…** 各種檔案
- 優點：
  - 直接綁 Firebase 專案（不用額外維護伺服器）
  - 提供安全規則（Security Rules）控制誰能讀寫
  - 有免費額度（適合 side project）

---

## 2. 建立 Firebase 專案 & 開啟 Storage

### 步驟一：建立 Firebase 專案

1. 前往 [Firebase Console](https://console.firebase.google.com/) (點擊右上角 Go to console)

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/1-friebase.png?raw=true)

2. 點擊**建立新的 Firebase 專案**

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/2-new-project.png?raw=true)

3. 輸入專案名稱（例如：`iThome2025-storage-demo`）

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/3-project-name.png?raw=true)

4. 完成建立後，就能進到專案儀表板

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/4-proj-dashboard.png?raw=true)

### 步驟二：啟用 Firebase Storage

1. 在左側選單選 **Storage**

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/5-menu-storage.png?raw=true)

2. 點擊 **升級專案 (這邊會需要連接信用卡帳戶)**

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/6-upgrade-proj.png?raw=true)

3. 點擊連結帳戶 (免費額度：**1 GB 儲存空間)**

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/7-link-account.png?raw=true)

4. 選擇地區（免付費位置）

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/8-choose-area.png?raw=true)

5. 勾選以正式模式啟動

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/9-stander-start.png?raw=true)

6. 完成後會自動建立一個 **Bucket**（其實就是一個儲存檔案的空間）

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/10-finish-bucket.png?raw=true)

📌 之後我們的專案就會透過這個 bucket 來存取檔案。

---

## 3. 專案設定與金鑰下載

1. 點擊小齒輪選專案設定

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/11-proj-setting.png?raw=true)

2. 選擇服務帳戶 → Firebase Admin SDK → 選 Node.js → 產生新的私密金鑰

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/12-firebase-sdk.png?raw=true)

3. 點擊產生金鑰後把這份 JSON 檔案保存好，後面會使用到 (**金鑰不要存在公開的地方**)

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/13-gen-key.png?raw=true)

4. JSON 檔案大致如下

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/14-json-file.png?raw=true)

---

## 4. 專案設定：.env 檔案

在 `.env` 檔案中設定環境變數：

```
FIREBASE_SERVICE_ACCOUNT={放入將 JSON 檔案壓縮成一行的資訊}
FIREBASE_STORAGE_BUCKET=ithome2025-storage-demo.firebasestorage.app
```

1.設定 `FIREBASE_SERVICE_ACCOUNT` → 小技巧 (將 JSON 檔案壓縮成一行)

VSCode → cmd+shift+P → Join Lines

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/15-join-lines.png?raw=true)

2.設定 `FIREBASE_STORAGE_BUCKET` → 點擊複製資料夾路徑 → 去除 (gs://)

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day16-Firebase/16-bucket-name.png?raw=true)

```json
ithome2025-storage-demo.firebasestorage.app
```

---

## 5.結語

到這裡，我們已經把 Firebase Storage 的基礎環境準備好：

- 建立 Firebase 專案
- 啟用 Storage 並建立 Bucket
- 下載服務金鑰 & 設定 `.env`

換句話說，**我們已經準備好讓專案具備「檔案上傳」的能力**啦！ 🚀

不過今天還沒寫程式碼，先到這裡收尾。

明天 Day 17 ✨，我們會進入實戰，使用 **Node.js + multer** 串接 Firebase Storage，實際測試檔案上傳，並取得公開 URL。
