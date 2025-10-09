---
layout: day
title: Day25 - AWS RDS 入門：在雲端打造你的第一個資料庫服務
date: 2025-10-09 21:08:59
tags:
  - TypeScript
  - Node.js
  - AWS
---

> EC2 能架網站，那資料庫呢？
>
> 今天我們要一起開啟第一台 **RDS**！

---

## 為什麼要用 RDS？

理論上，我們也可以在 **EC2** 上自己安裝資料庫（例如 MySQL、PostgreSQL），

但這樣做會遇到幾個痛點 👇

<!-- more -->

| 項目     | 自架在 EC2                      | 使用 RDS                                        |
| -------- | ------------------------------- | ----------------------------------------------- |
| 安裝流程 | 需要手動安裝 MySQL / PostgreSQL | 一鍵建立，自動設定環境                          |
| 維護     | 要自己備份、更新、監控          | AWS 代管自動備份、更新、監控                    |
| 安全性   | 要自己配置防火牆、使用者        | 內建 VPC、Security Group、IAM 控制              |
| 擴展性   | 要手動調整硬體資源              | 可自動或手動擴充規模（vertical / read replica） |

換句話說，**RDS 就是幫你「代管資料庫」的 EC2**。

底層仍然是虛擬機（EC2），但 AWS 幫你把「系統管理」那層抽象掉了，

讓開發者只需要專注在「資料與應用程式」。

---

## RDS 的底層其實是 EC2 嗎？

是的，但差別在於：

- RDS 是 **受管服務（Managed Service）**
- AWS 會負責：
  - 建立與維護 EC2 instance
  - 安裝與配置資料庫引擎（MySQL / PostgreSQL / MariaDB / Oracle / SQL Server）
  - 定期 備份 / snapshot
  - 管理安全性群組與網路設定

簡單說：

> 你不再需要 sudo apt install postgresql，
>
> 而是直接在 AWS Console 上點幾下，資料庫就能用了 💥

---

## 跟著 AWS 一起開啟第一台 RDS

### Step 1：進入 AWS Console → 探索 AWS → 點擊「建立 RDS 資料庫」

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/1-start-rds.png?raw=true)

### Step 2：建立資料庫執行個體

- 選擇 輕鬆建立 (這是 AWS 提供的快速建立選項，適合新手體驗。)
- 選擇引擎 → **PostgreSQL**
- **資料庫執行個體大小 → 免費方案**
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/2-rds-info.png?raw=true)

### Step 3：設定帳號密碼

- **資料庫執行個體識別符**：database-1
- **主要使用者名稱**：postgres
- **憑證管理**：自我管理
- **使用者密碼**：勾選自動產生密碼
- 點擊 → 建立資料庫
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/3-rds-account.png?raw=true)
- 等待建立執行個體時 → 點擊檢視連線詳細資訊

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/4-rds-conn.png?raw=true)

- 保存資料庫使用者名稱與密碼 (等等連線時會用到)
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/5-rds-conn-info.png?raw=true)

### Step 4：設定 EC2 安全群組

回到 AWS Console → 點擊 EC2 → 側邊欄點擊安全群組

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/6-ec2-sg.png?raw=true)

點擊右上方建立安全群組 → 填寫名稱描述等資訊

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/7-create-sg.png?raw=true)

設定傳入規則 (填寫資訊如下圖) → 拉到最下方點擊建立安全群組

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/8-sg-in-rule.png?raw=true)

> ⚠️ 若開放 `0.0.0.0/0` 僅供測試使用，正式環境建議限制特定 IP 或使用 Private Subnet。

### Step 5：設定公開連線

- 進入 RDS 儀表板 → 點擊右上方修改
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/9-edit-rds.png?raw=true)
- 下拉到連線區塊 → 新增我們剛剛建立的安全群組 `my-pgsql-rule`
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/10-add-sg.png?raw=true)
- 連線區塊最下方有個其他組態的設定 → 勾選可公開存取
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/11-public-access.png?raw=true)
- 下拉到最下方按下繼續 → 檢視修改項目 → 點擊修改資料庫執行個體
- 下圖為修改中的資料庫執行個體
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/12-editing-rds.png?raw=true)

---

## 使用 DBeaver 連線測試

修改完 RDS 後，回到主控台的查看執行個體的「**端點**」與「**連接埠**」：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/13-rds-console.png?raw=true)

### 開啟 DBeaver → 新建連線：

- 選擇資料庫類型：PostgreSQL
- Host：輸入 RDS 端點
- Port：5432
- Database：postgres
- Username / Password：剛剛設定的帳密
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/14-db-conn.png?raw=true)

點擊 **Test Connection**

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day25-RDS-Intro/15-db-conn-test.png?raw=true)

如果顯示成功，就代表你成功連線了雲端資料庫！

---

## ⚠️ 開發測試的安全提醒

本篇教學屬於 **入門範例**，主要目的是讓大家體驗如何在雲端建立與連線資料庫。

實際正式專案部署時，需特別注意以下幾點：

1. **請勿長期開放 Public access**：

   若你的資料庫可被全網連線（`0.0.0.0/0`），將有較高的安全風險。

2. **測試完記得清理資源**：

   若僅為練習，建議在測試結束後 **終止或刪除 RDS 執行個體**，以避免產生額外費用。

3. **正式環境建議做法**：
   - 將 RDS 放在 **Private Subnet**，不開放外部連線。
   - 僅允許應用伺服器（EC2）內部網路存取。

---

## 結語

從 EC2 到 RDS，我們已經踏入雲端資料架構的世界！

明天我們將進一步探索：

- 如何讓 Node.js 專案連上 RDS
- 在 `.env` 中配置資料庫連線字串
- 並驗證資料是否能正常 CRUD

鐵人賽還剩下幾天呀～加油！加油！
