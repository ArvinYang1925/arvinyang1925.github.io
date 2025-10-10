---
layout: day
title: Day 26 - 雲端串接實戰：Node.js 成功連上 AWS RDS！
date: 2025-10-10 23:10:26
tags:
  - TypeScript
  - Node.js
  - AWS
---

## 前情提要

Day 24，我們已經成功開啟 EC2，並在上面部署了第一版的 Node.js Express App。

Day 25，我們建立了 RDS 資料庫並成功測試連線。

今天，我們要來完成最關鍵的一步 ——

**讓 EC2 上的應用程式正式連上 RDS，打造完整的雲端架構！**

<!-- more -->

---

## 前置作業：關閉 RDS 公開存取權限，提高安全性

進入 **RDS 儀表板** → 點擊右上角「修改」 →

下滑到「連線」區塊 → 將「可公開存取」改為 **No** → 儲存修改。

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/1-rds-access.png?raw=true)

> 💡 昨天為了用 DBeaver 測試，我們暫時開啟了公開存取。
>
> 不過長期而言，這樣的設定會帶來極大的安全風險！

---

### 為什麼關閉「公開存取」能提升安全性？

### 1. 阻擋所有「外部網際網路」的流量

當你設定 `Public access = No` 時，RDS 僅會分配 **私有 IP**，

外部網路無法直接抵達它。

即使你的 Security Group 開得再寬（像 `0.0.0.0/0`），

外部世界也**完全 ping 不到**。

這能有效防止：

- 自動化掃描（masscan、nmap）
- 惡意登入爆破（如 PostgreSQL 密碼暴力破解）
- 被搜尋引擎（例如 Shodan）暴露

> 換句話說，它在「網路層」上就被隔離了！

---

### 2. 所有資料庫存取都需走「內部網路」

當 RDS 僅有私有 IP，能連線的主機僅限於：

- 同一個 **VPC** 內的 EC2
- 或透過 **VPN** 進來的內部節點

這樣，你能明確控制「誰」能夠存取資料庫，

將環境從「開放世界」轉變為「封閉社區」。

---

## 步驟 1：更新 EC2 上的專案版本

SSH 登入 EC2：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/2-ssh-conn.png?raw=true)

在 EC2 執行以下指令：

```bash
# 1. 進入專案資料夾
cd ~/iThome2025-node-ts

# 2. 停止目前的應用 (如果之前有開啟)
pm2 stop ithome-app

# 3. 切換並拉取最新分支
git checkout feature/init-ts-express
git pull origin feature/init-ts-express

# 4. 安裝套件
npm install

# 5. 重新編譯
npm run build
```

---

## 步驟 2：設定環境變數

建立 `.env`：

```bash
nano .env
```

輸入內容如下：

```bash
# RDS 資料庫設定
DB_HOST=node-ts-demo-db.xxxxxx.ap-northeast-1.rds.amazonaws.com
DB_NAME=postgres
DB_USERNAME=postgres
DB_PASSWORD=password
DB_PORT=5432

# JWT
JWT_SECRET=jwt_secret

# Firebase
FIREBASE_SERVICE_ACCOUNT={"type":"service_account",...,"client_x509_cert_url":"xxx>"}
FIREBASE_STORAGE_BUCKET=your-project.appspot.com

LOG_LEVEL=debug
NODE_ENV=production
```

儲存離開：`Ctrl + X → Y → Enter`

---

## 步驟 3：測試 RDS 連線

```bash
sudo apt update
sudo apt install postgresql-client -y

psql -h node-ts-demo-db.xxxxxx.ap-northeast-1.rds.amazonaws.com \
     -p 5432 \
     -U postgres \
     -d postgres
```

輸入密碼後若成功進入 `psql` 介面，代表連線順利 🎉

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/3-pgsql-conn.png?raw=true)

輸入 `\q` 可退出。

---

## 步驟 4：執行資料庫 Migration

```bash
# 查看 migration 檔案
ls -la src/migrations/

# 執行 migration 建立資料表
npm run migration:run

# 檢查 migration 狀態
npm run migration:show
```

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/4-migration.png?raw=true)

---

## ⚠️ 踩坑紀錄：Migration 成功，但 App 啟動失敗！

我原本滿心期待啟動應用程式，

結果一跑 `node dist/app.js`，瞬間炸裂：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/5-db-conn-failed.png?raw=true)

心想：「為何？Migration 都成功了，怎麼跑不動？」

---

### 問題分析

這是 TypeORM 常見陷阱之一：

- `migration:run` 是由 `typeorm-ts-node-commonjs` 執行，可直接讀 `.ts`
- 但 `node dist/app.js` 僅能執行 `.js`
  —— TypeORM 嘗試載入 `.ts` migration，自然報錯。

也就是說：

> 開發環境能吃 TS，但佈署環境只能吃 JS。

---

### ✅ 解決方案

在 `db.ts` 調整 migrations 路徑設定：

```tsx
migrations: process.env.NODE_ENV === "production"
  ? [__dirname + "/../migrations/*.js"]
  : ["src/migrations/**/*.ts"];
```

---

## 步驟 5：重新部署與驗證

```bash
# 拉取修改後程式碼
git pull origin feature/init-ts-express

# 查看版本
git log
```

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/6-git-pull.png?raw=true)

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/7-git-log.png?raw=true)

接著重新啟動：

```bash
node dist/app.js
```

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/8-db-conn-ok.png?raw=true)

成功連線 🎉

---

## 步驟 6：用 PM2 正式啟動

```bash
pm2 start ithome-app
pm2 logs ithome-app
```

應看到：

```
📦 DB Connected!
🚀 Server running on http://localhost:3000
```

---

## 步驟 7：全面驗證

### 7.1 從瀏覽器測試

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/9-browser-test.png?raw=true)

### 7.2 Postman 測試主要 API

註冊：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/10-postman-test-reg.png?raw=true)

登入：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/11-postman-test-login.png?raw=true)

新增 Todo：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/12-add-todo.png?raw=true)

### 7.3 檢查資料庫內容

```bash
# 在 EC2 上連接到 RDS
psql -h your-rds-endpoint -p 5432 -U postgres -d postgres

# 查看資料表
\dt

# 查看 todo 資料
SELECT * FROM "todo";

# 退出
\q
```

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day26-EC2-RDS/13-check-pgsql-todo.png?raw=true)

測試成功 🎉

---

## 結語

這一天，我們正式打通了 **Node.js 應用 + AWS RDS 資料庫** 的雲端實戰串接！

這次的流程看似繁瑣，

但我們一步步實作並理解了：

- 為何要關閉 Public Access
- 如何讓 EC2 連上 RDS
- TypeORM 在生產環境常見的 `.ts/.js` 落差問題

在雲端部署世界裡，**每一次錯誤都是最珍貴的學習歷程。**

下一篇，我們將進一步探索 AWS 的另一個明星服務 — **S3（Simple Storage Service）**。

它是許多應用的「檔案中樞」，可以安全地存放圖片、影片、PDF、甚至備份資料。

## 參考資源

> commit :
> fix: 🐛 update migrations config to support .js in production an
> 修正 TypeORM migration 設定，讓 production 使用 .js、開發使用 .ts

[Github 連結](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/659efb96fa58de0a07452d039a8f6997184842c4)
