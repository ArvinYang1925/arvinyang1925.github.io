---
layout: day
title: Day 12 - 程式碼自動排版神器：Prettier 實戰導入
date: 2025-09-26 12:24:13
tags:
  - TypeScript
  - Node.js
  - Tools
---

## 前言

在團隊開發時，大家常常會因為一些小細節爭得不可開交：

有人愛用單引號 `'`，有人卻堅持雙引號 `"`；有人在每行結尾必加分號，有人則覺得省略更簡潔；甚至連一行程式碼能寫多長，都可能成為爭論焦點。

這些差異雖然不影響程式能不能跑，但會導致專案越來越凌亂，Code Review 也常常淪為「引號大戰」、「縮排大戰」。

這些關於 Coding Style 的問題，當然要來利用一些好用的工具來解決。

<!-- more -->

---

## 為什麼需要 Coding Style？

在開發專案時，每個人都有自己的習慣：

- 有人喜歡 `單引號`，有人堅持 `雙引號`。
- 有人會在每行 **80 字元**就換行，有人覺得 **120 字元**也沒問題。
- 有人堅持在每行結尾加上 `;`，有人認為可以省略。

這些差異雖然不會影響程式能不能執行，但會造成：

1. **可讀性降低**：程式碼風格不一致，團隊閱讀時需要額外的腦力轉換。
2. **協作成本上升**：Code Review 時會浪費時間在「引號」、「縮排」這些小事，而不是程式邏輯。
3. **維護困難**：專案越大，風格越不一致，日後接手的人會更痛苦。

👉 為了避免這種情況，我們需要一個「統一的 Coding Style 工具」來幫忙。這就是 **Prettier** 出現的理由。

---

## Prettier 是什麼？

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day12-Prettier/1-Prettier.png?raw=true)

[Prettier](https://prettier.io/) 是一個「有主見的程式碼格式化工具」。

它會自動幫你排版程式碼，確保整個專案的風格一致。

### 特點

- **專注在排版**（縮排、引號、換行、逗號…）。
- **避免爭論**：大家都用相同規則，開發者不需要再為風格吵架。
- **支援多種語言**：JavaScript、TypeScript、JSON、Markdown、HTML、CSS…

📌 簡單比喻：

- **Prettier** = 幫你「整理房間」，讓程式碼整齊乾淨。
- **ESLint**（明天會介紹）= 規範你的「生活習慣」，檢查程式寫法是否符合最佳實踐。

---

## VSCode 安裝 Prettier 套件

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day12-Prettier/2-vscode-prettier.png?raw=true)

## 專案加入 Prettier

安裝 Prettier：

```bash
npm install --save-dev prettier
```

---

## 設定 Prettier

在專案根目錄新增 `.prettierrc`，統一規則：

```json
{
  "semi": true,
  "singleQuote": false,
  "trailingComma": "all",
  "tabWidth": 2,
  "printWidth": 140
}
```

說明：

- `semi: true` → 每行結尾加上分號
- `singleQuote: false` → 使用雙引號
- `trailingComma: "all"` → 物件最後一項也加逗號（方便日後新增項目）
- `tabWidth: 2` → 縮排 2 格
- `printWidth: 140` → 每行最長 140 字元

---

## 忽略檔案：`.prettierignore`

和 `.gitignore` 類似，這裡設定 Prettier 不要處理的檔案：

```
# 忽略依賴套件
node_modules

# 忽略編譯後的輸出
dist
build

# 忽略環境變數設定檔
.env
.env.*

# 忽略版本控制與鎖檔
.git
.gitignore
package-lock.json
yarn.lock
pnpm-lock.yaml

# 測試覆蓋率報告
coverage

# Log 檔案與紀錄
logs
*.log

# 靜態資源或上傳資料
public/
static/
uploads/

# 忽略 Prettier 自己的 ignore 設定
.prettierignore

# 圖片與媒體檔案
*.png
*.jpg
*.jpeg
*.svg
*.gif
*.mp4
*.webm

# 可選：忽略資料庫檔案或 dump 檔
*.sqlite
*.sql
*.dump
```

### 為什麼要新增 `.prettierignore`？

1. **不必要格式化**：像 `node_modules` 或 `dist`，不是我們自己寫的程式碼。
2. **效能考量**：避免 Prettier 處理大量無意義的檔案。
3. **避免衝突**：像 `.env` ，格式被改可能會出錯。

---

## 專案格式化範例

在 `package.json` 加入 scripts：

```json
"scripts": {
  "format": "prettier --write \"src/**/*.{js,ts}\"",
  "format:check": "prettier --check \"src/**/*.{js,ts}\""
}
```

- `npm run format:check`：只檢查，不會修改程式碼 (可發現 `todoController.ts` 和 `todoRoutes.ts` 有格式上的問題)
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day12-Prettier/3-format-check.png?raw=true)
- `npm run format`：自動格式化 `src` 底下的程式碼
  ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day12-Prettier/4-format-code.png?raw=true)
- 附上專案程式碼經過 prettier 格式化後的結果 (https://github.com/ArvinYang1925/iThome2025-node-ts/commits/feature/init-ts-express/)

`todoController.ts`

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day12-Prettier/5-format-controller.png?raw=true)

`todoRoutes.ts`

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day12-Prettier/6-format-route.png?raw=true)

成功透過 Prettier 格式化程式碼 💪

---

## 小結

- **Coding Style**：避免專案變得凌亂，提高團隊協作效率。
- **Prettier**：專注在「排版」，讓程式碼看起來統一且乾淨。
- **搭配 ESLint**：Prettier 負責「長相」，ESLint 負責「習慣」，兩者搭配才完整。

👉 明天 (Day12) 我們會介紹 **ESLint**，看看它如何幫助我們寫出更正確、更有品質的程式碼。

---

## 參考資料

[Github 連結](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/ce334a8f7f3366eb00ba95c1977326a03e3e72e2)

> commit : setup prettier

[隨時隨地格式化 - Prettier](https://ithelp.ithome.com.tw/m/articles/10294321)

[你終究要用 Prettier，為什麼不一開始就用「Prettier」呢？](https://israynotarray.com/javascript/20231031/1586150719/)
