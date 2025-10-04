---
layout: day
title: Day 20 - 從 0 到自動化：開啟你的第一個 GitHub Actions 旅程
date: 2025-10-04 21:26:55
tags:
  - TypeScript
  - Node.js
  - Tools
---

## 前言

在專案開發的過程中，常常會遇到這種情況：

- 有時候是忘了跑 ESLint，專案裡到處都是紅線。
- 有時候是格式亂掉，Prettier 沒跑，code review 變成「排版大戰」。
- 更慘的是，大家都說「我電腦可以跑」，結果一併到 main 就全壞。

這些其實都不是誰的錯，而是我們缺少一個「自動幫我們檢查」的機制。

👉 這就是 **CI/CD** 登場的時候：

它就像一個不會喊累的機器人，每次你 push 或發 PR，就自動幫你檢查、測試、甚至部署。

今天，我們就來用 **GitHub Actions** 簡單幫專案加上這個「自動守護者」，開啟 CI/CD 的第一步！

<!-- more -->

---

## CI / CD 是什麼？有啥好處

- **CI（Continuous Integration，持續整合）**
  每次 push 或 PR 時，自動跑測試、檢查程式碼，確保專案隨時健康。
  **好處**：快速發現 bug、避免合併衝突、提升專案穩定度。
- **CD（Continuous Delivery / Deployment，持續交付/部署）**
  - Delivery：確保程式隨時可以部署（但最後一步通常人工按下去）。
  - Deployment：自動一路推到正式環境。
    **好處**：讓產品更新流程更快、更穩、更少人為失誤。

---

## 什麼是 GitHub Actions？

**GitHub Actions** 是 GitHub 提供的 **自動化工作流程服務**，可以在 push、PR、發版 tag 等事件觸發時，執行你定義的工作，例如：

- 跑 **Lint / Test / Build**
- 自動部署到 Render、Vercel、AWS
- 發送通知到 Slack / Discord

---

## 同類型工具有哪些？

除了 GitHub Actions，常見的 CI/CD 工具有：

- **GitLab CI/CD** – 和 GitHub Actions 很像，但內建在 GitLab。
- **CircleCI / Travis CI** – 早期很流行的雲端服務。
- **Jenkins** – 傳統自架 CI/CD 工具，彈性高但維護成本大。

---

## ✨ 為何選擇 GitHub Actions 入門？

- **零門檻整合**：Repo 在 GitHub，就能直接用。
- **社群豐富**：Marketplace 上有各種現成 actions。
- **免費額度夠用**：對個人專案和開源很友善。
- **YAML 配置直覺**：學一次就能套用到其他專案。

---

## GitHub Actions 只負責 CI 嗎？

不是！

- 它可以跑 **CI（測試 / 檢查 / 編譯）**。
- 也能做 **CD（自動部署）**。

今天我們先聚焦在 **CI 自動化檢查**，讓專案更安全。

---

## 🛠️ 實戰步驟

### **步驟一：創建 GitHub Actions 工作流程目錄**

首先，在專案根目錄創建 `.github/workflows` 資料夾。

---

### **步驟二：創建 CI Workflow 文件**

我們建立一個最簡單的 GitHub Actions workflow，包含以下檢查：

- ✅ 代碼風格檢查 (ESLint)
- ✅ 格式檢查 (Prettier)
- ✅ TypeScript 編譯檢查

在剛剛建立的資料夾底下建立 `ci.yml`檔案內容如下：

```yaml
name: CI

# 觸發條件：當有程式碼推送到任何分支，或有 Pull Request 時
on:
  push:
    branches: ["**"]
  pull_request:
    branches: ["**"]

jobs:
  # 代碼品質檢查工作
  code-quality:
    runs-on: ubuntu-latest

    steps:
      # 步驟 1: 檢出程式碼
      - name: 📥 檢出程式碼
        uses: actions/checkout@v4

      # 步驟 2: 設置 Node.js 環境
      - name: 📦 設置 Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20.17.0"
          cache: "npm"

      # 步驟 3: 安裝依賴
      - name: 📚 安裝依賴套件
        run: npm ci

      # 步驟 4: 執行 Prettier 格式檢查
      - name: 🎨 檢查程式碼格式
        run: npm run format:check

      # 步驟 5: 執行 ESLint 檢查
      - name: 🔍 執行 ESLint 檢查
        run: npm run lint

      # 步驟 6: TypeScript 編譯檢查
      - name: 🔨 TypeScript 編譯檢查
        run: npx tsc --noEmit
```

---

## 詳細說明

### **1. Workflow 觸發時機**

```yaml
on:
  push:
    branches: ["**"] # 任何分支有推送時觸發
  pull_request:
    branches: ["**"] # 任何 Pull Request 時觸發
```

### **2. 執行環境**

- `runs-on: ubuntu-latest` → 使用最新的 Ubuntu 環境
- `node-version: '20.17.0'` → 使用專案指定的 Node.js 版本

### **3. 檢查步驟**

1. **格式檢查** → 確保程式碼符合 Prettier 規範
2. **Lint 檢查** → 確保程式碼符合 ESLint 規則
3. **編譯檢查** → 確保 TypeScript 沒有型別錯誤

---

## 如何使用

### **步驟三：提交並推送到 GitHub**

```bash
# 1. 將 workflow 文件加入版本控制
git add .github/workflows/ci.yml

# 2. 提交變更
git commit -m "chore: add GitHub Actions CI workflow"

# 3. 推送到 GitHub
git push
```

---

### **步驟四：查看 Actions 執行結果**

1. 前往你的 GitHub repository
2. 點擊上方的 **"Actions"** 標籤
3. 你會看到 workflow 正在執行或已完成

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day20-Github-Action/1-CI-InProgress.png?raw=true)

4. 點擊任何一次執行可以查看詳細的執行記錄 (這邊可以發現程式碼格式上有些問題) → CI 檢查失敗

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day20-Github-Action/2-migration-prob.png?raw=true)

5. 修復關於 migrations 資料夾底下程式碼格式問題 → 目標是讓 CI 能夠正確完整執行

   **推薦方法：讓 Prettier 和 ESLint 忽略 migrations 資料夾的檢查**

   這是最常見的做法，因為：

   - ✅ migration 是 TypeORM 自動生成的，不應該手動或格式化修改
   - ✅  每次生成新的 migration 都會有相同的問題

   在 `.prettierignore` 加上

   ```bash
   src/migrations/**/*.ts
   ```

   修改 `eslint.config.js` 加上 ( `ignores: ["src/migrations/**/*.ts"]` )

   ```bash
   const js = require("@eslint/js");
   const globals = require("globals");
   const prettier = require("eslint-config-prettier");
   const tsParser = require("@typescript-eslint/parser");
   const tsPlugin = require("@typescript-eslint/eslint-plugin");

   module.exports = [
     {
       // 新增：忽略 migrations 資料夾
       ignores: ["src/migrations/**/*.ts"],
     },
     {
       files: ["**/*.ts"], // 檢查 .ts 檔案
       languageOptions: {
         parser: tsParser, // 用 TypeScript 的解析器
         sourceType: "module", // 支援 ES 模組
         globals: {
           ...globals.node, // Node.js 的全域變數（像 process）
         },
       },
       // ... existing code ...
     },
   ];
   ```

6. 進行 commit 與推送程式碼

   commit : update ESLint/Prettier config to ignore migrations folder

7. 一樣的方式查看 Github Action
8. 結果 : 偵測出我們之前測試 ESLint 套件所建立的一個小檔案有問題 `testLint.ts`

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day20-Github-Action/3-testLint-prob.png?raw=true)

9. 移除 `testLint.ts` 後再 push 一次
10. 結果 : 成功通過 CI 測試啦！

    ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day20-Github-Action/4-CI-success.png?raw=true)

---

## ✅ 完成！

到這裡，你已經成功在專案中導入 **第一個 GitHub Actions CI Workflow**。

從現在開始，每次 push 或發 PR，GitHub 會幫你自動檢查程式碼，確保品質無虞。

---

## 補充

如果只想在 `main` 和 `develop` 分支執行，可以改成：

```yaml
on:
  push:
    branches: ["main", "develop"]
  pull_request:
    branches: ["main", "develop"]
```

---

## 結語

到今天為止，我們已經幫專案加上了第一層保護網：

- push / PR → 自動跑 **ESLint / Prettier / TypeScript 檢查**
- 任何錯誤 → CI 立刻偵測，保持程式碼品質

CI 就像一個安靜但可靠的守護者，幫助團隊維持專案品質。

---

## 參考資料

[chore: 🤖 add GitHub Actions CI workflow](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/eb0291ea9426aef7bf640a69d98fd03e8b0930b1)

[chore: 🤖 update ESLint/Prettier config to ignore migrations](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/13d901b086c26472e51a6f1fd34603dd8393bfbb)

[fix: 🐛 reomve testLint.ts](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/7dd74bae3d93d95e3397f6fc3feed9c85d1048e3)
