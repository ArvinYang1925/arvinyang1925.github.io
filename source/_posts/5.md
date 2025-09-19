---
layout: day
title: Day 5 - TypeScript 核心語法 (3)：interface、type、generics
date: 2025-09-19 20:11:50
tags:
  - TypeScript
---

昨天我們介紹了 **Union** 和 **Enum**，今天要進一步學習如何用 **Interface、Type、Generics** 來讓程式更有結構、更可重用。

這次一樣用「搭火車」的例子 🚄，幫助你快速理解！

<!-- more -->

---

## 1. Interface（介面）

介面用來定義「物件的形狀」，就像規劃「火車乘客」需要有什麼資料。

```tsx
interface Passenger {
  id: number;
  name: string;
  seat?: string; // 可選屬性（可能沒劃位）
}

const passengerA: Passenger = {
  id: 1,
  name: "Arvin",
};

const passengerB: Passenger = {
  id: 2,
  name: "Alice",
  seat: "A12",
};
```

👉 特點：

- 適合用來描述「物件結構」。
- 支援 **extends**（繼承），方便擴充。

```tsx
interface VipPassenger extends Passenger {
  loungeAccess: boolean;
}

const vip: VipPassenger = {
  id: 3,
  name: "Bob",
  seat: "B10",
  loungeAccess: true,
};
```

---

## 2. Type（型別別名）

`type` 與 `interface` 很像，不過它更靈活，不只可以描述物件，還能用來定義 **聯合型別、基本型別別名**。

```tsx
// 車票編號可能是數字或字串
type TicketID = number | string;

// 乘客物件
type PassengerType = {
  id: TicketID;
  name: string;
};

const passengerC: PassengerType = {
  id: "T-123",
  name: "Charlie",
};
```

👉 差異比較：

- **interface**：專注描述物件結構，可繼承、擴充。
- **type**：更靈活，可以描述各種型別組合（物件、Union、Tuple 等）。

---

## 3. 泛型（Generics）

泛型就像「火車車廂」🚃：

同樣的車廂結構，可以載不同種類的乘客或貨物。

### 函式中的泛型

```tsx
function identity<T>(value: T): T {
  return value;
}

// 明確指定型別
const ticketNumber = identity<number>(12345);
const passengerName = identity<string>("David");

// 也可以自動推斷
const autoTicket = identity("E12");
```

---

### 介面中的泛型

```tsx
interface TrainResponse<T> {
  status: number;
  data: T;
}

// 回傳單一乘客
const passengerResponse: TrainResponse<Passenger> = {
  status: 200,
  data: { id: 1, name: "Arvin" },
};

// 回傳乘客陣列
const passengersResponse: TrainResponse<Passenger[]> = {
  status: 200,
  data: [
    { id: 2, name: "Alice" },
    { id: 3, name: "Bob", seat: "C3" },
  ],
};
```

---

### 類別中的泛型

```tsx
class TrainCar<T> {
  private items: T[] = [];

  add(item: T) {
    this.items.push(item);
  }

  getAll(): T[] {
    return this.items;
  }
}

// 建立「乘客車廂」
const passengerCar = new TrainCar<Passenger>();
passengerCar.add({ id: 1, name: "Arvin" });
passengerCar.add({ id: 2, name: "Alice" });

// 建立「行李車廂」
const luggageCar = new TrainCar<string>();
luggageCar.add("背包");
luggageCar.add("行李箱");

console.log(passengerCar.getAll());
console.log(luggageCar.getAll());
```

---

## 小結

- **Interface**：適合描述物件結構，例如「乘客」或「車票」。
- **Type**：更靈活，可以描述 **物件 + Union + Tuple**，常用來定義「票號、付款方式」等。
- **Generics**：像「火車車廂」，同一套結構可以裝不同型別，讓程式碼更可重用。

---

## Interface vs Type vs Generics 對照表

| 特點       | Interface                  | Type                                | Generics                   |
| ---------- | -------------------------- | ----------------------------------- | -------------------------- |
| **用途**   | 定義物件結構               | 定義各種型別（物件、Union、Tuple…） | 建立可重用、保留型別的模板 |
| **延伸性** | 可用 `extends` 繼承、擴充  | 不可繼承，但可用 `&` 做交集合併     | 可套用在函式、介面、類別   |
| **優點**   | 清楚描述物件結構、支援擴充 | 彈性大，可用於各種型別組合          | 可重用，提高型別安全性     |

---

👉 明天（Day 6）我們會開始進入實戰的一個全新篇章，開始初始化一個 **TypeScript + Node.js 專案 !**
