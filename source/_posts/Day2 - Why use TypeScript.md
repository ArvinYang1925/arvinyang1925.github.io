---
layout: day
title: Day 2 - 為什麼要用 TypeScript 開發 Node.js？（價值、趨勢）
date: 2025-09-16 10:36:28
tags:
  - TypeScript
---

在開始之前，先讓我們來談談今天的主角 —— **TypeScript**。

許多人在接觸 TypeScript 的第一反應是：「這不就是 JavaScript 加上型別嗎？」沒錯，這句話雖然簡化了很多細節，但卻說中了核心。TypeScript 本質上就是 **JavaScript 的超集（superset）**，它在 JavaScript 的基礎上加上了型別系統與一些進階語法糖，讓開發者能寫出更可靠、更可維護的程式。

<!-- more -->

---

## 一、TypeScript 簡介與發展歷史

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day2/typescript-officle.png?raw=true)

TypeScript 由 **微軟（Microsoft）** 團隊在 **2012 年**發表，設計之父是著名的程式語言專家 **Anders Hejlsberg**，他同時也是 C# 的主要設計者。當時 JavaScript 在大型專案中經常遇到可維護性與錯誤檢查的瓶頸，因此 TypeScript 誕生的目標就是：

👉 **讓 JavaScript 更適合大型專案的開發**。

隨著前端框架（Angular、React、Vue）逐漸壯大，以及 Node.js 進軍後端，TypeScript 的生態圈也迅速擴展。

---

## 二、TypeScript 與 JavaScript 的關係

可以把 TypeScript 想成是 **加強版的 JavaScript**。

- **JavaScript**：瀏覽器和 Node.js 都能直接執行的語言。
- **TypeScript**：在 JavaScript 的基礎上，增加了 **型別系統**、**介面（interface）**、**泛型（generics）** 等功能。

所有 TypeScript 程式碼最終都會被「編譯（**compile**）」成純 JavaScript，才能在 Node.js 或瀏覽器中執行。換句話說：

💡 **任何合法的 JavaScript 程式碼，本身就是合法的 TypeScript 程式碼。**

---

## 三、TypeScript 的優點

那麼 TypeScript 到底解決了什麼痛點呢？以下列幾個主要優點：

1. **靜態型別檢查**
   - 在編譯階段就能發現型別錯誤，減少 runtime bug。
   - 例如：
     ```tsx
     let age: number = 25;
     age = "twenty-five"; // 編譯錯誤！
     ```
2. **更好的 IDE 支援**
   - 自動補全、即時型別提示與錯誤檢查，都比純 JavaScript 更精準。
   - 這能大幅提升開發體驗，尤其在大型專案中。
3. **程式結構更清晰**
   - 使用 interface、type、enum 可以清楚描述資料結構，團隊成員更容易理解。
4. **降低維護成本**
   - 型別提示與嚴格檢查，讓新成員接手專案更容易上手，也避免「看名字猜型別」的情況。

---

## 四、為什麼要用 TypeScript 開發 Node.js？（價值與趨勢）

很多人會問：

> Node.js 已經用 JavaScript 就能開發了，為什麼還要多學一個 TypeScript？

理由其實很簡單，歸納成兩個關鍵字：**價值**與**趨勢**。

### 1. 價值

- **提升專案可靠度**：伺服器端通常牽涉到資料庫、API、使用者資料，任何錯誤都可能導致重大問題。TypeScript 的型別檢查能大幅降低這種風險。
- **更好的團隊協作**：Node.js 專案通常不是一人獨自開發，而是多人協作。TypeScript 的型別定義就是團隊之間的「契約（contract）」，避免誤解。
- **長期維護更輕鬆**：後端系統常常需要跑好幾年，TypeScript 能讓專案在 1 年、3 年後依然可讀、可維護。

### 2. 趨勢

- **主流框架全面支援**：像是 NestJS、Next.js 都是以 TypeScript 為核心打造。
- **開發者市場需求**：求職網站上，TypeScript 幾乎已經是 Node.js 工程師的必備技能。
- **生態系成熟**：越來越多 npm 套件都提供官方的型別定義，不再需要自己額外補齊。

換句話說，現在的 TypeScript 已經不是「可有可無的加分題」，而是 Node.js 開發中逐漸成為「基本配備」的技能。

---

## 結語

總結來說，TypeScript 不是要取代 JavaScript，而是要 **讓 JavaScript 更適合在大型、長期、多人協作的專案中發光發熱**。

在 Node.js 的後端開發世界，可靠性、維護性與可擴展性是第一優先，而這正是 TypeScript 最擅長的領域。

所以，與其問「為什麼要用 TypeScript？」更應該問的是：

👉 **如果不用 TypeScript，我會損失什麼？**

在未來的系列文章裡，我們會先介紹一些 TypeScript 基礎，然後一步一步示範，如何把 TypeScript 和 Node.js 結合起來，打造一個專案級的開發環境 🚀。
