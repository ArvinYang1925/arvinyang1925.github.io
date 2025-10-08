---
layout: day
title: Day 24 - 從本地到雲端：把你的 Node.js 專案部署上 AWS EC2！
date: 2025-10-08 19:54:23
tags:
  - TypeScript
  - Node.js
  - AWS
---

## 前言

昨天我們已經成功開啟第一台 AWS EC2 主機，體驗了雲端主機。

今天，我們要更進一步 —— **把 Node.js 專案實際部署到雲端主機上！**

這篇文章將帶你完成從 0 到上線的所有流程：

1. 建立 Ubuntu EC2 實例
2. SSH 連線
3. 安裝 Node.js / npm / Git
4. 部署並執行專案
5. 使用 PM2 常駐應用程式

<!-- more -->

---

## AWS EC2 部署完整步驟

### **步驟 1：在 AWS EC2 啟動 Ubuntu 實例**

1. **名稱** : `ithome2025-ec2`
2. **選擇 AMI**：選擇 Ubuntu（這邊使用筆者較熟悉的 Ubuntu 22.04 LTS）

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day24-EC2-NodeJS/1-name-os.png?raw=true)

3. **選擇實例類型**：`t3.micro`
4. **金鑰對（重要）**：
   - 建立新的金鑰對或選擇現有的
   - 下載 `.pem` 檔案並妥善保存（例如：`my-key.pem`）
5. **網路設定**：
   - ✅ 允許 SSH 流量（來自 0.0.0.0/0 或您的 IP）
   - ✅ 允許 HTTP 流量（port 80）
   - ✅ 允許來自網際網路的 HTTPS 流量（可選）
   - 🔧 **額外新增自訂 TCP 規則**：port 3000（Node.js 應用預設埠）
     ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day24-EC2-NodeJS/2-tcp-3000.png?raw=true)
6. **儲存空間**：預設 8GB 足夠

點擊「啟動實例」

---

### **步驟 2：SSH 連接到您的 EC2 實例 (詳細可參考上一篇文章)**

等待執行個體狀態變為 "執行中" 後：

1. 點擊 EC2 主控台右上角連線按鈕 → 選擇 SSH 用戶端
2. 開啟終端機 → 尋找 aws-key.pem 的存放位址 → 修改檔案權限 → 利用 AWS 所提供的指令連線

成功連接後會看到 Ubuntu 的歡迎訊息。

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day24-EC2-NodeJS/3-welcome-ubuntu.png?raw=true)

---

### **步驟 3：在 EC2 上安裝 Node.js 和 npm**

在 EC2 實例中執行：

```bash
# 更新系統套件
sudo apt update
sudo apt upgrade -y

# 安裝 Node.js 20.x（LTS 版本）
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# 驗證安裝
node --version  # 應顯示 v20.x.x
npm --version   # 應顯示 10.x.x

# 安裝 Git（用於複製專案）
sudo apt install -y git
```

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day24-EC2-NodeJS/4-node-ver.png?raw=true)

---

### **步驟 4：將專案部署到 EC2**

接下來，我們要把專案實際部署到 **AWS EC2** 上。

不過目前我們還沒有啟用 RDS 或其他雲端服務，因此這次的部署目標很單純：

> 先讓 Node.js + Express 伺服器能在雲端成功啟動運作，
>
> 暫時不涉及資料庫連線設定。

為了確保環境一致，我們會回到專案中尚未整合資料庫的版本，也就是以下這個 commit：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day24-EC2-NodeJS/5-git-commit.png?raw=true)

```bash
# 複製專案
git clone https://github.com/ArvinYang1925/iThome2025-node-ts.git
cd iThome2025-node-ts

# 切換到特定 commit
git checkout 342fd7f5ee9ece202ef2ec2a00c10b05d1c76a4a
```

---

### **步驟 5：安裝依賴並編譯專案**

在 EC2 實例中：

```bash
# 進入專案目錄
cd ~/iThome2025-node-ts

# 安裝依賴
npm install

# 編譯 TypeScript 到 JavaScript
npx tsc
```

編譯後會在 `dist/` 目錄產生 JavaScript 檔案。

---

### **步驟 6：測試運行應用**

```bash
# 直接運行編譯後的 JS
node dist/app.js
```

您應該會看到：

```
🚀 Server running on http://localhost:3000
```

測試方式：

- 或在瀏覽器訪問 `http://您的EC2公有IP:3000`
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day24-EC2-NodeJS/6-hello-tihome.png?raw=true)

如果看到 "Hello, iThome2025 !"，恭喜成功！🎉

---

### **步驟 7：使用 PM2 讓應用程式持續運行**

PM2 是 Node.js 的程序管理工具，可以讓應用程式在背景持續運行：

```bash
# 安裝 PM2
sudo npm install -g pm2

# 使用 PM2 啟動應用
pm2 start dist/app.js --name "ithome-app"

# 查看應用狀態
pm2 status

# 查看日誌
pm2 logs

# 設定開機自動啟動
pm2 startup
```

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day24-EC2-NodeJS/7-pm2.png?raw=true)

PM2 常用指令：

```bash
pm2 restart ithome-app  # 重啟
pm2 stop ithome-app     # 停止
pm2 delete ithome-app   # 刪除
pm2 logs ithome-app     # 查看日誌
```

---

## 結語

至此，我們已經完成了從本地開發環境到 **AWS** **雲端部署的流程** 🎉

這一步的意義不只是讓網站能被全世界訪問，更代表著你掌握了開發者的三大核心能力：

- 開發（Node.js + TypeScript）
- 部署（AWS EC2）
- 維運（PM2）

接下來，我們會逐步探索 AWS RDS 等雲端服務

慢慢讓這個 Node.js 專案成為一個完整的雲端應用 🚀

---

## 補充資源

[Github 範例程式碼](https://github.com/ArvinYang1925/iThome2025-node-ts/tree/feature/init-ts-express)

> commit message: Day 6 - set up Express + TS environment
