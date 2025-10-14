---
layout: day
title: Day 30 - 讓 TypeScript 把你的 Node.js 開發再升級 — 心得與完整目錄
date: 2025-10-14 21:32:13
tags:
  - TypeScript
  - Node.js
  - AWS
---

## 前言 & 心得

終於完賽了！🎉

每天數著日子發文，終於順利完成了第十七屆鐵人賽——真的超怕中途斷賽 😭

其實從以前開始開發時，就常常透過搜尋看到許多前輩在鐵人賽上的文章，

那時候覺得這是一個很酷的挑戰，也從中學到了不少東西。

所以今年就想給自己一個機會，從讀者變成參賽者，親自體驗這 30 天的歷程。

<!-- more -->

由於這系列文章有不少實作與程式碼範例，

中間花了很多時間在測試、除錯、重跑流程，

一開始還有囤一點備稿，但進入中後期幾乎是「邊學邊寫、日更實戰」。

回頭看，真的蠻酷的。

每天一小步、一點一滴的累積，最後竟然能完成一個完整的教學系列。

這份成就感比想像中還要強烈。

---

## 系列目錄

為了讓讀者能更方便回顧整個系列，以下我將內容分為五大階段 👇

---

### **1. TypeScript 核心語法與基礎複習**

進入實戰前，先複習並掌握 TypeScript 的核心觀念，建立「型別思維」。

- [Day 3 ｜ TypeScript 核心語法 (1)：型別系統、基本型別](https://ithelp.ithome.com.tw/articles/10381757)
- [Day 4 ｜ TypeScript 核心語法 (2)：Union、Enum](https://ithelp.ithome.com.tw/articles/10382560)
- [Day 5 ｜ TypeScript 核心語法 (3)：interface、type、泛型](https://ithelp.ithome.com.tw/articles/10382689)

---

### **2. 基礎建置與環境設定**

從零開始建立一個完善的 TypeScript + Node.js 開發環境，

完成簡單的 API 實作，最後部署到 Render 雲端服務。

- [Day 6 ｜建立 TypeScript + Node.js 環境 (1)：初始化專案](https://ithelp.ithome.com.tw/articles/10384075)
- [Day 7 ｜建立 TypeScript + Node.js 環境 (2)：專案架構與資料庫設定](https://ithelp.ithome.com.tw/articles/10384080)
- [Day 8 ｜打造你的第一個 TodoList API：一步步實現 CRUD 功能](https://ithelp.ithome.com.tw/articles/10385326)
- [Day 9 ｜ Render 雲端啟動：資料庫連線全攻略](https://ithelp.ithome.com.tw/articles/10386281)
- [Day 10 ｜部署啟程！從 Render 部署前置作業到 GitHub PR](https://ithelp.ithome.com.tw/articles/10386893)
- [Day 11 ｜一鍵上線！完整部署到 Render 的實戰流程](https://ithelp.ithome.com.tw/articles/10387580)

---

### **3. 專案工具與進階功能**

讓專案更專業！導入程式碼格式化工具、API 驗證、使用者登入驗證，

並探索如何串接雲端檔案上傳。

- [Day 12 ｜程式碼自動排版神器：Prettier 實戰導入](https://ithelp.ithome.com.tw/articles/10388279)
- [Day 13 ｜一致的程式碼：ESLint 導入](https://ithelp.ithome.com.tw/articles/10388768)
- [Day 14 ｜ API 驗證救星：用 Zod 驗證來檢查](https://ithelp.ithome.com.tw/articles/10388774)
- [Day 15 ｜ API 安全升級：JWT 登入驗證全流程實作](https://ithelp.ithome.com.tw/articles/10390867)
- [Day 16 ｜ Firebase Storage 初探：輕鬆搞定專案檔案上傳前置作業](https://ithelp.ithome.com.tw/articles/10391259)
- [Day 17 ｜ Firebase Storage 實戰：用 Node.js + multer 上傳圖片到雲端](https://ithelp.ithome.com.tw/articles/10391803)

---

### **4. 後端常見進階應用**

這部分涵蓋專案升級必備的實務主題：日誌系統、Migration 管理、CI/CD 自動化流程。

- [Day 18 ｜ console.log 退役啦！Node.js Pino 帶你升級專案 Log](https://ithelp.ithome.com.tw/articles/10392321)
- [Day 19 ｜專案升級必備：資料庫 Migration 實戰](https://ithelp.ithome.com.tw/articles/10392467)
- [Day 20 ｜從 0 到自動化：開啟你的第一個 GitHub Actions 旅程](https://ithelp.ithome.com.tw/articles/10393411)

---

### **5. 雲端部署與維護**

最後，我們將專案部署到 AWS 雲端，探索雲端的核心服務與概念。

- [Day 21 ｜ AWS 初探 (1)：什麼是雲端服務？](https://ithelp.ithome.com.tw/articles/10393772)
- [Day 22 ｜ AWS 初探 (2)：破關拿獎金・預算防護](https://ithelp.ithome.com.tw/articles/10394112)
- [Day 23 ｜從零啟動雲端主機：帶你開出第一台 AWS EC2！](https://ithelp.ithome.com.tw/articles/10394375)
- [Day 24 ｜從本地到雲端：把你的 Node.js 專案部署上 AWS EC2！](https://ithelp.ithome.com.tw/articles/10394986)
- [Day 25 ｜ AWS RDS 入門：在雲端打造你的第一個資料庫服務](https://ithelp.ithome.com.tw/articles/10395279)
- [Day 26 ｜雲端串接實戰：Node.js 成功連上 AWS RDS！](https://ithelp.ithome.com.tw/articles/10395419)
- [Day 27 ｜ AWS S3 實作檔案上傳：打造你的雲端檔案儲存中心](https://ithelp.ithome.com.tw/articles/10395913)
- [Day 28 ｜ AWS VPC 入門：初探雲端世界的隱形網路](https://ithelp.ithome.com.tw/articles/10396471)
- [Day 29 ｜ AWS IAM 入門：讓雲端安全運作的身分與權限管理](https://ithelp.ithome.com.tw/articles/10396573)

---

## 結語

挑戰鐵人賽真的不容易。

每天想主題、寫內容、驗證程式、排版、上圖，還要兼顧可讀性與教學節奏。

有時候邊寫邊懷疑：「我真的寫得完嗎？」

但一步步走到這裡，才發現**堅持本身就是一種力量。**

感謝這三十天以來所有閱讀、按讚、留言、給建議的朋友們，

你們的互動都是我撐下去的重要動力 ❤️

如果這個系列能讓你對 TypeScript + Node.js

或是部署到雲端的流程有更清晰的理解，那就太值得了。

未來我也會持續更新相關技術主題，

讓我們在開發這條路上，一起學習、一起成長。

## 參考資源

- [GitHub 專案原始碼](https://github.com/ArvinYang1925/iThome2025-node-ts/tree/feature/init-ts-express)
- [個人部落格](https://arvinyang1925.github.io/)
