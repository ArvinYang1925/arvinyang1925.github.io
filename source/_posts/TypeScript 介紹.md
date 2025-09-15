---
title: TypeScript 完整介紹：從入門到實戰
date: 2025-01-27 10:00:00
updated: 2025-01-27 10:00:00
tags:
  - TypeScript
  - JavaScript
  - 前端開發
  - 程式語言
categories:
  - 技術教學
description: 深入了解 TypeScript 的核心概念、語法特性，以及如何在實際專案中應用這個強大的 JavaScript 超集語言。
keywords: TypeScript, JavaScript, 型別系統, 前端開發, 編程語言
---

## 什麼是 TypeScript？

TypeScript 是由 Microsoft 開發的一種開源程式語言，它是 JavaScript 的一個**型別化超集**（typed superset），最終會編譯成純 JavaScript 代碼。簡單來說，任何有效的 JavaScript 代碼都是有效的 TypeScript 代碼。

<!-- more -->

### 🎯 為什麼選擇 TypeScript？

1. **靜態型別檢查** - 在編譯時期就能發現錯誤
2. **更好的 IDE 支援** - 智能提示、自動完成、重構工具
3. **增強的可讀性** - 代碼更容易理解和維護
4. **漸進式採用** - 可以逐步將現有 JavaScript 專案遷移到 TypeScript

## 🚀 快速開始

### 安裝 TypeScript

```bash
# 全域安裝
npm install -g typescript

# 專案內安裝
npm install --save-dev typescript

# 驗證安裝
tsc --version
```

### 第一個 TypeScript 檔案

建立 `hello.ts` 檔案：

```typescript
function greet(name: string): string {
  return `Hello, ${name}!`;
}

const userName: string = "Arvin";
console.log(greet(userName));
```

編譯並執行：

```bash
# 編譯
tsc hello.ts

# 執行生成的 JavaScript
node hello.js
```

## 📚 核心概念

### 1. 基本型別

```typescript
// 基本型別
let isDone: boolean = false;
let decimal: number = 6;
let color: string = "blue";
let list: number[] = [1, 2, 3];
let tuple: [string, number] = ["hello", 10];

// Any 型別（盡量避免使用）
let notSure: any = 4;

// Void 型別
function warnUser(): void {
  console.log("This is a warning message");
}

// Null 和 Undefined
let u: undefined = undefined;
let n: null = null;
```

### 2. 介面 (Interface)

```typescript
interface Person {
  firstName: string;
  lastName: string;
  age?: number; // 可選屬性
  readonly id: number; // 唯讀屬性
}

function createPerson(person: Person): string {
  return `${person.firstName} ${person.lastName}`;
}

const user: Person = {
  id: 1,
  firstName: "John",
  lastName: "Doe",
  age: 30,
};
```

### 3. 類別 (Classes)

```typescript
class Animal {
  private name: string;
  protected species: string;
  public age: number;

  constructor(name: string, species: string, age: number) {
    this.name = name;
    this.species = species;
    this.age = age;
  }

  public makeSound(): void {
    console.log(`${this.name} makes a sound`);
  }

  protected getInfo(): string {
    return `${this.name} is a ${this.species}`;
  }
}

class Dog extends Animal {
  private breed: string;

  constructor(name: string, breed: string, age: number) {
    super(name, "Canine", age);
    this.breed = breed;
  }

  public bark(): void {
    console.log("Woof! Woof!");
    this.makeSound();
  }

  public getBreedInfo(): string {
    return `${this.getInfo()} of breed ${this.breed}`;
  }
}
```

### 4. 泛型 (Generics)

```typescript
// 泛型函數
function identity<T>(arg: T): T {
  return arg;
}

let output1 = identity<string>("myString");
let output2 = identity<number>(100);

// 泛型介面
interface GenericIdentityFn<T> {
  (arg: T): T;
}

// 泛型類別
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;

  constructor(zero: T, addFn: (x: T, y: T) => T) {
    this.zeroValue = zero;
    this.add = addFn;
  }
}
```

### 5. 聯合型別與交集型別

```typescript
// 聯合型別
type StringOrNumber = string | number;

function padLeft(value: string, padding: StringOrNumber): string {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}

// 交集型別
interface ErrorHandling {
  success: boolean;
  error?: { message: string };
}

interface ArtworksData {
  artworks: { title: string }[];
}

type ArtworksResponse = ArtworksData & ErrorHandling;
```

## ⚙️ 進階特性

### 1. 型別別名 (Type Aliases)

```typescript
type Point = {
  x: number;
  y: number;
};

type EventHandler = (event: Event) => void;

type Status = "loading" | "success" | "error";
```

### 2. 條件型別

```typescript
type MessageOf<T> = T extends { message: infer U } ? U : never;

interface Email {
  message: string;
}

interface Dog {
  bark(): void;
}

type EmailMessageContents = MessageOf<Email>; // string
type DogMessageContents = MessageOf<Dog>; // never
```

### 3. 映射型別

```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type ReadonlyTodo = Readonly<Todo>;
type PartialTodo = Partial<Todo>;
```

## 🛠️ 實戰應用

### TypeScript 設定檔 (tsconfig.json)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020", "DOM"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### React + TypeScript 範例

```typescript
import React, { useState, useEffect } from "react";

interface User {
  id: number;
  name: string;
  email: string;
}

interface UserListProps {
  initialUsers?: User[];
}

const UserList: React.FC<UserListProps> = ({ initialUsers = [] }) => {
  const [users, setUsers] = useState<User[]>(initialUsers);
  const [loading, setLoading] = useState<boolean>(false);

  useEffect(() => {
    const fetchUsers = async (): Promise<void> => {
      setLoading(true);
      try {
        const response = await fetch("/api/users");
        const userData: User[] = await response.json();
        setUsers(userData);
      } catch (error) {
        console.error("Failed to fetch users:", error);
      } finally {
        setLoading(false);
      }
    };

    if (initialUsers.length === 0) {
      fetchUsers();
    }
  }, [initialUsers.length]);

  if (loading) {
    return <div>Loading...</div>;
  }

  return (
    <ul>
      {users.map((user: User) => (
        <li key={user.id}>
          {user.name} - {user.email}
        </li>
      ))}
    </ul>
  );
};

export default UserList;
```

## 📝 最佳實踐

### 1. 型別定義

```typescript
// ✅ 好的做法
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// ❌ 避免使用 any
function processData(data: any): any {
  return data;
}

// ✅ 使用具體型別
function processUserData(data: User[]): ProcessedUser[] {
  return data.map((user) => ({
    id: user.id,
    displayName: `${user.firstName} ${user.lastName}`,
    isActive: user.status === "active",
  }));
}
```

### 2. 錯誤處理

```typescript
type Result<T, E = Error> =
  | {
      success: true;
      data: T;
    }
  | {
      success: false;
      error: E;
    };

async function fetchUserData(id: string): Promise<Result<User>> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      return {
        success: false,
        error: new Error(`HTTP ${response.status}: ${response.statusText}`),
      };
    }
    const user = await response.json();
    return { success: true, data: user };
  } catch (error) {
    return {
      success: false,
      error: error instanceof Error ? error : new Error("Unknown error"),
    };
  }
}
```

## 🔍 常見問題與解決方案

### 1. 型別斷言

```typescript
// 型別斷言（謹慎使用）
const someValue: unknown = "this is a string";
const strLength: number = (someValue as string).length;

// 更安全的做法
function isString(value: unknown): value is string {
  return typeof value === "string";
}

if (isString(someValue)) {
  const strLength: number = someValue.length; // TypeScript 知道這裡是 string
}
```

### 2. 可選鏈與空值合併

```typescript
interface NestedObject {
  level1?: {
    level2?: {
      value?: string;
    };
  };
}

const obj: NestedObject = {};

// 可選鏈
const value = obj.level1?.level2?.value;

// 空值合併
const displayValue = value ?? "Default Value";
```

## 🎓 學習資源與工具

### 推薦工具

- **VS Code** - 最佳的 TypeScript 開發體驗
- **TypeScript Playground** - 在線練習和測試
- **TSLint/ESLint** - 代碼檢查工具
- **Prettier** - 代碼格式化工具

### 學習資源

- [TypeScript 官方文檔](https://www.typescriptlang.org/docs/)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [React TypeScript 速查表](https://github.com/typescript-cheatsheets/react)

## 📈 總結

TypeScript 為 JavaScript 開發帶來了強大的型別系統，幫助開發者：

1. **提前發現錯誤** - 編譯時期的型別檢查
2. **提升開發效率** - 更好的 IDE 支援和智能提示
3. **改善代碼品質** - 更清晰的 API 設計和文檔
4. **降低維護成本** - 重構更安全，代碼更容易理解

對於現代 JavaScript 開發，特別是大型專案，TypeScript 已經成為不可或缺的工具。它不僅提升了開發體驗，也讓代碼更加健壯和可維護。

## 🚀 下一步

如果您剛開始學習 TypeScript，建議：

1. 從簡單的型別標註開始
2. 逐步學習介面和類別
3. 在小專案中實踐
4. 探索進階特性如泛型和條件型別
5. 整合到您喜愛的框架中（React, Vue, Angular）

開始您的 TypeScript 之旅吧！這將是一個值得投資的技術選擇。

---

> 💡 **小提示**: 記住，TypeScript 的目標不是替代 JavaScript，而是讓 JavaScript 開發更加安全和高效。漸進式採用是最佳策略！
