---
layout: day
title: Day 3 - TypeScript 核心語法 (1)：型別系統與核心
date: 2025-09-17 20:48:03
tags:
  - TypeScript
---

前兩天我們介紹了 TypeScript 的背景與價值，今天要開始動手寫程式碼，從最常見的 **基本型別** 與 **型別推斷 / 型別註記** 講起。

---

## 1. 基本常用型別

TypeScript 在 JavaScript 基礎上，提供了更嚴謹的型別檢查。以下是常見的基本型別：

- **string**：字串
- **number**：數字（整數、浮點數都屬於 number）
- **boolean**：布林值（true/false）
- **array**：陣列
- **object**：物件，可指定 key 與 value 的型別
- **null** / **undefined**：空值與未定義
<!-- more -->
- **any**：跳過型別檢查，不建議常用
- **unknown**：安全版的 any，需要檢查後才能使用
- **void**：通常用在函式沒有回傳值時

我們用「搭火車」這個情境，來把 **基本型別** 串在一起，這樣學起來更有畫面。以下是範例：

```tsx
// 搭火車的情境

// 乘客名字（string）
let passengerName: string = "Arvin";

// 年齡（number）
let age: number = 25;

// 是否已經買票（boolean）
let isBuyTicket: boolean = true;

// 行李（array）
let packages: string[] = ["背包", "行李箱", "Mac"];

// 車票（object）
let ticket: { start: string; destination: string; price: number } = {
  start: "台南",
  destination: "台北",
  price: 750,
};

// 不確定的外部輸入（unknown）
let inputSeat: unknown = "A12";
if (typeof inputSeat === "string") {
  console.log(`座位號碼是 ${inputSeat}`);
}

// 不小心遺失票券（null & undefined）
let lostTicket: null = null;
let notAssignedSeat: undefined = undefined;

// any（不建議，但有時候需要先用）
let randomInfo: any = "隨便的資訊";
randomInfo = 123;

// void：檢查是否有買票（沒有回傳值）
function checkTicket(name: string, hasTicket: boolean): void {
  if (hasTicket) {
    console.log(`${name} 已經買票，可以進站！`);
  } else {
    console.log(`${name} 尚未購票！`);
  }
}

checkTicket(passengerName, isBuyTicket);
```

---

## 2. 型別推斷 vs 型別註記

TypeScript 在很多情況下會自動「推斷型別」：

```tsx
let trainStation = "台北"; // 推斷為 string
let ticketPrice = 500; // 推斷為 number
```

但有時候我們需要 **手動註記型別**，讓程式碼更清楚：

```tsx
let trainStation: string = "台北";
let ticketPrice: number = 500;
```

### 差異比較

- **型別推斷**：程式更簡潔，讓 TS 自動判斷。
- **型別註記**：在需要 **可讀性**、**強制規範**、或 **複雜物件** 時更適合。

---

## 3. 簡單使用場景

### ✅ 適合用型別推斷

當值一看就知道型別，TS 自動判斷即可。

```tsx
let trainName = "自強號"; // string
let travelTime = 120; // number
```

### ✅ 適合用型別註記

1. **函式參數與回傳值**（TS 無法從空參數自動推斷，必須標註）

```tsx
function buyTicket(start: string, destination: string, price: number): string {
  return `已購買從 ${start} 到 ${destination} 的車票，票價 ${price} 元`;
}
```

1. **物件結構清楚表達**

```tsx
let passenger: { id: number; name: string; hasTicket: boolean } = {
  id: 1,
  name: "Arvin",
  hasTicket: true,
};
```

1. **與 API 或外部資料互動**

```tsx
type TrainApiResponse = {
  status: number;
  stations: string[];
};

const res: TrainApiResponse = {
  status: 200,
  stations: ["台北", "新竹", "台中"],
};
```

---

## 小結

- **基本型別**是 TypeScript 的基礎：`string`、`number`、`boolean`、`array`、`object`、`null`、`undefined`…
- **型別推斷**：讓程式更簡潔。
- **型別註記**：在函式、物件、API 回傳值等情境中特別重要。

👉 明天（Day 4）我們會進一步介紹 union、enum，讓型別系統更靈活！

---
