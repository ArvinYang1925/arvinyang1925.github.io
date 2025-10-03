---
layout: day
title: Day 19 - 專案升級必備：資料庫 Migration 實戰
date: 2025-10-03 21:31:48
tags:
  - TypeScript
  - Node.js
  - Tools
---

## 前言

在專案開發時，我們常常會遇到「資料庫 Schema 需要修改」的情況。

例如新增欄位、刪除欄位、或是調整欄位型別。

最直覺的方式是 —— 直接改 Entity，然後靠 TypeORM 的 `synchronize: true` 來自動同步。

但是，這樣做真的安全嗎？ 🤔

<!-- more -->

---

## ⚠️ `synchronize: true` 的隱藏風險

在 `DataSource` 設定中，大家應該都看過這段：

```tsx
synchronize: true, // 開發階段方便，正式環境建議 false
```

這個參數會讓 TypeORM 在每次啟動專案時，自動比對 Entity 與資料庫結構，並「直接修改資料庫」。

**好處：**

- 開發初期很快：改 `User.ts` 或 `Todo.ts`，資料庫自動更新。
- 不需要寫 SQL，完全自動化。

**壞處：**

1. **資料可能直接被刪掉**：欄位名稱改了，TypeORM 可能直接 drop + recreate table。
2. **多人協作很危險**：大家本地 Entity 不一致 → DB schema 會亂掉。
3. **Production 大忌**：上線環境如果還開 `synchronize: true`，一個小改動就可能讓線上資料消失。

👉 這就是我們需要 Migration 的理由。

---

## 為什麼要用 Migration？

Migration 就像「資料庫的 Git」，能幫助我們做到：

- **版本控制**：每次改動都被記錄下來。
- **可回滾**：跑錯了可以 revert。
- **多人協作**：大家跑同一份 Migration，確保一致性。
- **安全上線**：Production 可以放心執行，避免不可預期的自動修改。

---

## Migration 實戰：導入 Migration 的步驟

### 第 1 步：在 `package.json` 添加 Migration 指令

```json
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "dev": "ts-node-dev src/app.ts",
  "format": "prettier --write \"src/**/*.{js,ts}\"",
  "format:check": "prettier --check \"src/**/*.{js,ts}\"",
  "lint": "eslint \"src/**/*.ts\" --format=stylish",
  "lint:fix": "eslint \"src/**/*.ts\" --fix",
  "migration:generate": "typeorm-ts-node-commonjs migration:generate -d src/config/db.ts",
  "migration:run": "typeorm-ts-node-commonjs migration:run -d src/config/db.ts",
  "migration:revert": "typeorm-ts-node-commonjs migration:revert -d src/config/db.ts",
  "migration:show": "typeorm-ts-node-commonjs migration:show -d src/config/db.ts"
}
```

---

### 第 2 步：建立 `migrations` 資料夾

```bash
mkdir -p src/migrations
```

---

### 第 3 步：修改 `src/config/db.ts`

將 `synchronize` 改為 `false`，並加入 migration 設定：

```tsx
import "reflect-metadata";
import { DataSource } from "typeorm";
import { Todo } from "../entities/Todo";
import { User } from "../entities/User";
import dotenv from "dotenv";

dotenv.config();

export const AppDataSource = new DataSource({
  type: "postgres",
  host: process.env.DB_HOST,
  port: Number(process.env.DB_PORT),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  ssl: { rejectUnauthorized: false },
  entities: [Todo, User],
  synchronize: false, // ⚠️ 改為 false，改用 migration 管理
  logging: true,
  migrations: ["src/migrations/**/*.ts"], // 📁 migration 檔案路徑
  migrationsTableName: "migrations_history", // 📊 migration 歷史記錄表名稱
});
```

---

### 第 4 步：生成初始 Migration

如果之前用 `synchronize: true` 已經有資料表，建議先清空，再重新用 migration 建立。

清空方式如下: (Dbeaver 選取 Table 右鍵點擊 Delete)

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day19-Migration/1-db-delete-table.png?raw=true)

```bash
npm run migration:generate src/migrations/InitialMigration
```

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day19-Migration/2-init-migration.png?raw=true)

這會生成一個 migration 檔案，內容類似這樣：

```tsx
import { MigrationInterface, QueryRunner } from "typeorm";

export class InitialMigration1759480233749 implements MigrationInterface {
  name = "InitialMigration1759480233749";

  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`CREATE TABLE "user" (...)`);
    await queryRunner.query(`CREATE TABLE "todo" (...)`);
    await queryRunner.query(`ALTER TABLE "todo" ADD CONSTRAINT ...`);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`ALTER TABLE "todo" DROP CONSTRAINT ...`);
    await queryRunner.query(`DROP TABLE "todo"`);
    await queryRunner.query(`DROP TABLE "user"`);
  }
}
```

---

### 第 5 步：執行 Migration

執行前可以先查看一下狀態

```bash
npm run migration:show   # 查看狀態
```

可以發現有偵測到一個 migration 檔案尚未執行

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day19-Migration/3-migration-show.png?raw=true)

```bash
npm run migration:run    # 執行 migration
```

下圖為 migration 檔案執行結果 :

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day19-Migration/4-migration-run.png?raw=true)

然後可以到資料庫查看目前狀況 :

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day19-Migration/5-db-after-migration.png?raw=true)

可發現多了一張 migrations_history 表，這是用來記錄 migration 的資訊

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day19-Migration/6-migration-table.png?raw=true)

---

### 第 6 步：其他常用指令

```bash
# 回退最後一次 migration
npm run migration:revert

# 修改 Entity 後，重新生成 migration
npm run migration:generate src/migrations/你的Migration名稱
```

---

## 完整工作流程範例

1. **首次設定**

   ```bash
   mkdir -p src/migrations
   npm run migration:generate src/migrations/InitialMigration
   npm run migration:run
   ```

2. **未來修改 Entity 後 (這邊舉例在 User.ts Entity 加上一個 phone 欄位)**

   ```bash
   @Entity()
   export class User {
     @PrimaryGeneratedColumn("uuid")
     id!: string;

   	...

     @Column({ type: "varchar", length: 10, nullable: true })
     phone?: string;

   	...
   }
   ```

   執行下方指令生成 migration 檔案

   ```bash
   npm run migration:generate src/migrations/AddPhoneToUser
   ```

   生成結果如下

   ```bash
   import { MigrationInterface, QueryRunner } from "typeorm";

   export class AddPhoneToUser1759482580830 implements MigrationInterface {
       name = 'AddPhoneToUser1759482580830'

       public async up(queryRunner: QueryRunner): Promise<void> {
           await queryRunner.query(`ALTER TABLE "user" ADD "phone" character varying(10)`);
       }

       public async down(queryRunner: QueryRunner): Promise<void> {
           await queryRunner.query(`ALTER TABLE "user" DROP COLUMN "phone"`);
       }

   }
   ```

   執行下方指令執行 migration 檔案

   ```bash
   npm run migration:run
   ```

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day19-Migration/7-migration-add-phone-col.png?raw=true)

   可查看資料庫新增了 phone 欄位在最尾端

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day19-Migration/8-db-result.png?raw=true)

   成功執行 migration 啦 🎉

3. **如果需要回退**

   ```bash
   npm run migration:revert
   ```

---

## Migration 開發流程

1. 修改/新增 Entity
2. 建立對應的 Migration
3. 在本地測試 `migration:run`
4. Commit Migration 檔案 → 其他人同步
5. 部署時，CI/CD 自動跑 Migration

---

## 🔑 小提醒 & Best Practice

- **開發可以先用 `synchronize: true`，但正式專案要關掉。**
- 每個 Migration 檔案盡量只做「一件事」（新增欄位、刪表、調型別）。
- Migration 檔案要 **一起 Commit**，不然別人會少版本。
- 別忘了 `migration:revert` 可以救命 🙌。

---

## 結語

`synchronize: true` 很方便，但就像「測試環境的捷徑」，不能拿到 Production。

真正穩定的專案，必須靠 **Migration** 來管理資料庫版本。 🚀

---

## 參考資料

> commit : disable synchronize and setup migration scripts

[Github 連結](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/d0f8a0c22be61f2b2487a0db5dbaa23df09c1d94)
