---
layout: day
title: Day 22 - AWS 初探 (2) - 破關拿獎金・預算防護
date: 2025-10-06 23:33:24
tags:
  - TypeScript
  - Node.js
  - AWS
---

## 前情提要

目前申辦 AWS 帳號會提供一組「探索 AWS」活動，只要完成幾個指定任務，就能獲得最多 **US$100 的抵用金**。

這些任務包含：

- 使用 Amazon Bedrock
- 在 Budgets 中設定成本預算
- 建立一個 Lambda 應用程式
- 建立一個 RDS 資料庫
- 啟動一台 EC2 虛擬主機

<!-- more -->

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day22-AWS-budgets/1-aws-tasks.png?raw=true)

筆者目前已完成兩項任務，共獲得 **US$40** 的抵用金。

接下來要挑戰的就是——**建立 AWS Budgets 預算防護**，避免帳單炸裂 。

---

## 🧾 為什麼要設定預算防護？

筆者去年使用 GCP Cloud SQL 時，

因為忘記關閉服務，結果帳單爆出 **七千多元台幣 QQ**。

這次吸取教訓，覺得在 AWS 設定「預算警報」非常重要。

這樣當使用量或金額超過設定值時，系統會自動寄信提醒，避免再度出事。

---

## 實作：設定 AWS Budgets 預算防護

以下步驟超簡單，大約 5 分鐘即可完成！

---

### Step 1. 參與 AWS 探索活動

1. 註冊新帳號 → 登入 AWS Console
2. 尋找探索 AWS 區塊
3. 點擊**使用 AWS Budgets 設定成本預算**
4. 按下開始活動

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day22-AWS-budgets/2-start-event.png?raw=true)

---

### Step 2. 選擇預算類型

這裡我們選擇使用教學範本 → 每月成本預算

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day22-AWS-budgets/3-budget-type.png?raw=true)

---

### Step 3. 設定預算名稱與金額

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day22-AWS-budgets/4-set-budget.png?raw=true)

在「**每月成本預算 – 範本**」區域：

- **預算名稱**：輸入 `MyBudget`
- **輸入預算金額 (US$)**：設定每月預算，例如 `5 USD`
- **電子郵件收件人**：填入要收到通知的電子郵件

> 小技巧：
>
> 如果你只是測試用途，設定 5 或 10 美元即可。
>
> 一旦超過這個金額，AWS 就會發信通知。

---

### Step 4. 確認與建立

檢查設定沒問題後，按下 建立預算。

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day22-AWS-budgets/5-create-budget.png?raw=true)

接著回到 Budgets 列表，你就能看到剛建立的預算監控項目。

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day22-AWS-budgets/6-budget-dashboard.png?raw=true)

---

## ✅ 領取獎金

上述步驟都完成後就可以回到主控台 → 探索 AWS 區塊查看是否有成功領取抵用金額啦！

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day22-AWS-budgets/7-get-aws-money.png?raw=true)

---

## 小結：用預算防護安心玩雲端

這一步看似簡單，卻非常關鍵。

它就像你在玩雲端服務時荷包的「安全保障」，

讓你可以放心探索，而不用擔心月底帳單爆炸。

今天解鎖了「探索 AWS」任務、多拿到 20 美元抵用金 💰  太棒啦！
