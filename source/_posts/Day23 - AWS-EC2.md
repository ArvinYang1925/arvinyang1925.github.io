---
layout: day
title: Day 23 - 從零啟動雲端主機：帶你開出第一台 AWS EC2！
date: 2025-10-07 22:50:32
tags:
  - TypeScript
  - Node.js
  - AWS
---

> 跟著 AWS 官方引導，從零啟動第一台雲端主機！

---

## 前言

- 承接前篇：「我們已經完成了 AWS 預算防護，確保不會爆花費。」
- 今天要實際體驗「在雲端開一台主機」，也就是 **EC2（Elastic Compute Cloud）**。
- EC2 就像你的「雲端電腦」，能安裝系統、架網站、部署 API。
- 本篇會帶你一步步完成：

  1. 認識 EC2
  2. 建立 EC2
  3. SSH 連線進入 EC2

    <!-- more -->

---

## 什麼是 EC2？

- EC2 全名 **Elastic Compute Cloud**。
- 它是一種可彈性擴展的虛擬伺服器服務。
- 你可以想像它是「雲端版的電腦」：
  - 有 CPU、記憶體、磁碟
  - 可以安裝 OS（例如 Ubuntu、Amazon Linux）
  - 透過 SSH 連進去操作

📘 小知識：

> EC2 是 AWS 最核心的服務之一，許多其他服務（像 RDS、S3）都會與 EC2 搭配使用。

---

## 建立 EC2 的完整步驟

### Step 1：開啟活動「啟動執行個體」

- 登入 AWS → 點選「探索 AWS」→ 「使用 EC2 啟動執行個體」
- 系統會帶你進入 EC2 建立畫面。
- 點擊「開始活動」，即可開始設定第一台主機。
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day23-AWS-EC2/1-start-event.png?raw=true)

---

### Step 2：填寫執行個體名稱與選擇 AMI（作業系統）

- 首先填寫執行個體名稱
- 然後在頁面中可看到各種作業系統選擇：
  - Amazon Linux 2023 ✅
  - Ubuntu
  - Windows Server
- 本篇選擇 **Amazon Linux 2023（預設，符合免費方案）**。
- 它是 AWS 官方提供的 Linux 發行版，安全且輕量。
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day23-AWS-EC2/2-instance-ami.png?raw=true)

---

### Step 3：選擇執行個體類型

- 選擇：`t3.micro`（2 vCPU / 1GB RAM）
- ✅ 符合免費額度
- 適合練習與簡單測試
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day23-AWS-EC2/3-instance-type.png?raw=true)

---

### Step 4：建立金鑰對（Key Pair）

- 金鑰對是登入 EC2 的憑證。(等等 SSH 連線會用到)
- 點「建立新的金鑰對」→ 選擇 RSA → 下載 `.pem` 檔。
- ⚠️ **請妥善保存，遺失後無法再登入！**
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day23-AWS-EC2/4-key-pair.png?raw=true)
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day23-AWS-EC2/5-create-key.png?raw=true)

---

### Step 5：設定網路與安全群組

- 建立新安全群組（`launch-wizard-2`）
- 開啟以下通訊埠：
  - **22 (SSH)**：遠端登入用
  - **80 (HTTP)**：讓網頁可以被瀏覽器存取
- CIDR 設定為 `0.0.0.0/0`（全開放，但僅限練習使用）
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day23-AWS-EC2/6-network.png?raw=true)

---

### Step 6：啟動執行個體

- 其他設定保持預設即可
- 然後點擊「啟動執行個體」後，AWS 會建立伺服器。
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day23-AWS-EC2/7-run-instance.png?raw=true)
- 幾十秒後即可在 EC2 主控台看到「Running」狀態。
- 點進去即可看到：
  - Public IPv4 address
  - Instance ID
  - Security group 等資訊
    ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day23-AWS-EC2/8-ec2-dashboard.png?raw=true)

---

## SSH 連線至 EC2

點擊 EC2 主控台右上角連線按鈕 → 選擇 SSH 用戶端

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day23-AWS-EC2/9-ssh-conn.png?raw=true)

SSH 連線步驟如下：

開啟終端機 → 尋找 aws-key.pem 的存放位址 → 修改檔案權限 → 利用 AWS 所提供的指令連線：

```bash
chmod 400 "my-ec2-001-key.pem"
ssh -i "my-ec2-001-key.pem" ec2-user@ec2-54-199-211-42.ap-northeast-1.compute.amazonaws.com
```

成功登入後會看到的終端畫面如下：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day23-AWS-EC2/10-ssh-success.png?raw=true)

🎉 你已成功進入雲端主機！(也可以嘗試一些基本 Linux 指令)

---

## 小提醒與收尾

- 練習結束後務必：
  1. 在 EC2 控制台 → 停止或終止執行個體
  2. 避免持續計費
- 如果要下次再使用，可重新啟動，AWS 會保留設定。

---

## 結語

到這裡，我們已經成功完成了「AWS EC2 入門」的第一步 🎯
從建立執行個體、設定安全群組，到用 SSH 登入主機，
體驗了雲端世界的基本流程。

這就像打開了一台雲端電腦的電源開關 ——
接下來，我們就能在雲端世界自由發揮，
部署 Node.js、連接資料庫，甚至架起自己的後端服務。
