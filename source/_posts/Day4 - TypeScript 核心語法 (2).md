---
layout: day
title: Day 4 - TypeScript 核心語法 (2)：union、enum
date: 2025-09-18 13:34:26
tags:
  - TypeScript
---

昨天我們介紹了基本型別，今天要進一步學會 **Union Type（聯合型別）** 和 **Enum（列舉）**。

這次一樣用「搭火車」的例子 🚄，讓程式碼更貼近生活！

<!-- more -->

---

## 1. Union Type（聯合型別）

Union 的特色是用 **`|`** 將多種可能型別區分開來。

當一個值可能有「多種型別」時，Union 非常好用。

### 搭火車範例：乘客的票號

```tsx
let ticketId: number | string;

ticketId = 12345; // 數字型票號
ticketId = "A12"; // 字串型票號
// ticketId = true; // ❌ 編譯錯誤
```

---

### 常見搭配：型別守衛 (Type Guards)

在使用 Union 時，我們通常需要用 **`typeof`** 來判斷型別後再操作，避免出錯。

```tsx
function printTicketId(id: number | string) {
  if (typeof id === "string") {
    console.log("票號（字串）：" + id.toUpperCase());
  } else {
    console.log("票號（數字）：" + id);
  }
}

printTicketId(12345);
printTicketId("b23");
```

---

### Union Type 補充說明 🚉

- **優點**：
  1. 適合處理「多種可能型別」的情境，例如 **API 回傳值** 或 **使用者輸入**。
  2. 可以搭配 **Type Guards** 提高安全性。
- **常見使用場景**：
  - 車票 ID（可能是數字 / 字串）
  - 車廂座位（可能是 "A12" 也可能是 12）
  - 付款方式（可能是 "現金" | "信用卡" | "行動支付"）

```tsx
type PaymentMethod = "Cash" | "CreditCard" | "MobilePay";

let payment: PaymentMethod;

payment = "Cash"; // ✅
payment = "MobilePay"; // ✅
// payment = "Coupon";  // ❌ 不在定義範圍
```

---

## 2. Enum（列舉）

Enum 適合定義「固定範圍內的值」，比 Union 更有「名稱 → 值」的對應關係。

### 搭火車範例：車票狀態

```tsx
enum TicketStatus {
  NotBought, // 尚未購買
  Bought, // 已購票
  Used, // 已使用
  Expired, // 已過期
}

let myTicket: { passenger: string; status: TicketStatus } = {
  passenger: "Arvin",
  status: TicketStatus.Bought,
};

console.log(myTicket);
```

---

### Enum + 陣列應用：乘客清單

```tsx
let passengers: { name: string; status: TicketStatus }[] = [];

function addPassenger(name: string) {
  passengers.push({
    name: name,
    status: TicketStatus.NotBought,
  });
}

addPassenger("Alice");
addPassenger("Bob");

console.log(passengers);
```

---

### Enum 設定字串值（更直觀）

```tsx
enum TrainClass {
  Express = "自強號",
  Local = "區間車",
  HighSpeed = "高鐵",
}

function printTrainClass(c: TrainClass) {
  console.log("列車種類：" + c);
}

printTrainClass(TrainClass.Express);
```

---

### Enum 補充說明 🚆

- **優點**：
  1. 讓「固定選項」更清楚（比字串常數更有語意）。
  2. 避免拼錯字串（TS 編譯時就會檢查錯誤）。
  3. 可讀性高，適合大型專案。
- **常見使用場景**：
  - 票券狀態（已購買 / 未購買 / 已使用 / 過期）
  - 列車種類（自強號 / 區間車 / 高鐵）
  - 座位等級（普通 / 商務 / 頭等）

---

## 小結

- **Union Type**：讓一個變數可以有多種型別，常用於「票號、付款方式」這類不確定型別的場景。
- **Enum**：讓「固定範圍的值」更有語意，常用於「票券狀態、列車種類、座位等級」。
- Union 靈活，Enum 嚴謹 —— 搭配使用可以讓程式碼更乾淨又安全。

👉 明天（Day 5）我們會介紹 **interface** 和 **type**，讓我們的火車旅程更結構化！ 🚉
