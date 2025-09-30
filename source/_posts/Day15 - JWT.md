---
layout: day
title: Day 15 - API 安全升級：JWT 登入驗證全流程實作
date: 2025-09-29 14:43:17
tags:
  - TypeScript
  - Node.js
  - Tools
---

到目前為止，我們的 TodoList API 已經能跑起來，還能把資料存進資料庫。

但是，有沒有發現一個大漏洞？

👉 **任何人都可以操作 todos，不需要登入！**

但現在我們希望做到「一人一帳號，一人一份 TodoList」。

今天我們就來幫 API 加上 **JWT 登入驗證**，讓系統更有安全感 💪。

<!-- more -->

P.S 本次專案程式碼修改幅度較大，下面列出重點步驟，詳細可參考最下方**補充資源**的內容。

---

## 1. 先裝上兩大神器

要實作登入驗證，通常會用到這兩個套件：

- **jsonwebtoken**：產生與驗證 JWT（JSON Web Token），用來確認使用者身份。
- **bcryptjs**：幫密碼做雜湊（Hash），確保不會把明碼存進資料庫。

```bash
npm install jsonwebtoken bcryptjs
npm install -D @types/jsonwebtoken @types/bcryptjs
```

裝好之後，我們就能開始搞定 **註冊 / 登入 / 驗證 middleware** 了 🚀。

---

## 2. 新增 User 實體

我們需要一個使用者資料表，才能綁定 Todo。

```tsx
// src/entities/User.ts
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  OneToMany,
} from "typeorm";
import { Todo } from "./Todo";

@Entity()
export class User {
  @PrimaryGeneratedColumn("uuid")
  id!: string;

  @Column({ type: "varchar", length: 50, nullable: false })
  name!: string;

  @Column({ type: "varchar", length: 320, unique: true, nullable: false })
  email!: string;

  @Column({ type: "varchar", length: 72, nullable: false })
  password!: string;

  @CreateDateColumn({ name: "created_at" })
  createdAt!: Date;

  @UpdateDateColumn({ name: "updated_at" })
  updatedAt!: Date;

  // 與 Todo 的一對多關係
  @OneToMany(() => Todo, (todo) => todo.user)
  todos?: Todo[];
}
```

再來，把 Todo 也連上使用者：(可以覆蓋本來的 Todo.ts 程式碼)

```tsx
// src/entities/Todo.ts
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  ManyToOne,
  JoinColumn,
} from "typeorm";
import { User } from "./User";

@Entity()
export class Todo {
  @PrimaryGeneratedColumn("uuid")
  id!: string;

  @Column({ type: "varchar", length: 255, nullable: false })
  title!: string;

  @Column({ default: false })
  completed!: boolean;

  @CreateDateColumn({ name: "created_at" })
  createdAt!: Date;

  @UpdateDateColumn({ name: "updated_at" })
  updatedAt!: Date;

  // 與 User 的多對一關係
  @Column({ name: "user_id" })
  userId!: string;

  @ManyToOne(() => User, (user) => user.todos, { onDelete: "CASCADE" })
  @JoinColumn({ name: "user_id" })
  user!: User;
}
```

這樣一來，Todo 就會跟 User 綁定囉 。

---

## 3. 密碼加密工具

不建議把使用者輸入的密碼直接存進 DB。

所以我們先寫一個小工具來處理加密與比對：

```tsx
// src/utils/passwordUtils.ts
import bcrypt from "bcryptjs";

export async function hashPassword(plain: string): Promise<string> {
  const salt = await bcrypt.genSalt(10);
  return bcrypt.hash(plain, salt);
}

export function comparePassword(
  plain: string,
  hashed: string
): Promise<boolean> {
  return bcrypt.compare(plain, hashed);
}
```

---

## 4. JWT 工具

JWT 就像是使用者的「通行證」。

我們需要能夠 **簽發 token** 以及 **驗證 token**。

```tsx
// src/utils/jwtUtils.ts
import * as jwt from "jsonwebtoken";

export interface JWTPayload {
  id: string;
  email: string;
}

if (!process.env.JWT_SECRET) {
  throw new Error("Missing JWT_SECRET in .env");
}
const SECRET = process.env.JWT_SECRET;
const EXPIRES_IN = process.env.JWT_EXPIRES_IN || "24h";

export function generateToken(payload: JWTPayload): string {
  return jwt.sign(payload, SECRET, {
    expiresIn: EXPIRES_IN as jwt.SignOptions["expiresIn"],
  });
}

export function verifyToken(token: string): JWTPayload {
  return jwt.verify(token, SECRET) as JWTPayload;
}
```

---

## 5. 註冊與登入流程

現在來寫 API：

```tsx
// src/controllers/authController.ts
import { Request, Response, NextFunction } from "express";
import {
  registerSchema,
  loginSchema,
} from "../validator/authValidationSchemas";
import { AppDataSource } from "../config/db";
import { User } from "../entities/User";
import { hashPassword, comparePassword } from "../utils/passwordUtils";
import { generateToken } from "../utils/jwtUtils";
import jwt from "jsonwebtoken";

const userRepo = AppDataSource.getRepository(User);

export async function register(
  req: Request,
  res: Response,
  next: NextFunction
) {
  try {
    const parsed = registerSchema.safeParse(req.body);
    if (!parsed.success) {
      const issue = parsed.error.issues[0];
      res.status(400).json({ status: "failed", message: issue.message });
      return;
    }

    const { name, email, password } = parsed.data;

    // 檢查是否已註冊
    const exists = await userRepo.findOneBy({ email });
    if (exists) {
      res.status(409).json({ status: "failed", message: "Email 已被使用" });
      return;
    }

    // 建立 User（移除角色相關邏輯）
    const hashed = await hashPassword(password);
    const user = userRepo.create({
      name,
      email,
      password: hashed,
    });
    const saved = await userRepo.save(user);

    // 生成 token
    const token = generateToken({ id: saved.id, email: saved.email });
    const { exp, iat } = jwt.decode(token) as { exp: number; iat: number };
    const expiresIn = exp - iat;

    res.status(201).json({
      status: "success",
      message: "註冊成功",
      data: {
        token,
        expiresIn,
        userInfo: {
          id: saved.id,
          name: saved.name,
          email: saved.email,
        },
      },
    });
  } catch (err) {
    next(err);
  }
}

export async function login(req: Request, res: Response, next: NextFunction) {
  try {
    const parsed = loginSchema.safeParse(req.body);
    if (!parsed.success) {
      res.status(401).json({ status: "failed", message: "帳號或密碼錯誤" });
      return;
    }

    const { email, password } = parsed.data;
    const user = await userRepo.findOneBy({ email });

    if (!user || !(await comparePassword(password, user.password))) {
      res.status(401).json({ status: "failed", message: "帳號或密碼錯誤" });
      return;
    }

    const token = generateToken({ id: user.id, email: user.email });
    const { exp, iat } = jwt.decode(token) as { exp: number; iat: number };
    const expiresIn = exp - iat;

    res.status(200).json({
      status: "success",
      message: "登入成功",
      data: {
        token,
        expiresIn,
        userInfo: {
          id: user.id,
          name: user.name,
          email: user.email,
        },
      },
    });
  } catch (err) {
    next(err);
  }
}

export async function logout(req: Request, res: Response, next: NextFunction) {
  try {
    res.status(200).json({
      status: "success",
      message: "登出成功",
    });
  } catch (err) {
    next(err);
  }
}
```

登入的流程則是比對密碼 → 發 token。

然後 routes 新增 authRoutes.ts

```json
// src/routes/authRoutes.ts
import { Router } from "express";
import { register, login, logout } from "../controllers/authController";
import { isAuth } from "../middleware/isAuth";

const router = Router();

router.post("/register", register);
router.post("/login", login);
router.post("/logout", isAuth, logout);

export default router;
```

.env 記得也要加上 `JWT_SECRET`

```json
JWT_SECRET=jwt_secret
```

---

## 6. 認證中間件

有了 token 之後，還要能檢查它是否合法。

這時候就需要 middleware 出場啦：

```tsx
// src/middleware/isAuth.ts
import { Request, Response, NextFunction } from "express";
import { verifyToken, JWTPayload } from "../utils/jwtUtils";

export interface AuthRequest extends Request {
  user?: JWTPayload;
}

export function isAuth(req: AuthRequest, res: Response, next: NextFunction) {
  const auth = req.headers.authorization;
  if (!auth?.startsWith("Bearer ")) {
    res.status(401).json({ status: "failed", message: "請先登入" });
    return;
  }

  const token = auth.split(" ")[1];
  try {
    const payload = verifyToken(token);
    req.user = payload;
    next();
  } catch {
    res.status(401).json({ status: "failed", message: "Token 無效或已過期" });
    return;
  }
}
```

---

## 7. 保護 Todo API

最後，讓所有 Todo 操作都必須帶上 token 才能執行：

```tsx
// src/routes/todoRoutes.ts
import { Router } from "express";
import {
  getTodos,
  createTodo,
  updateTodo,
  deleteTodo,
} from "../controllers/todoController";
import { isAuth } from "../middleware/isAuth";

const router = Router();

router.get("/", isAuth, getTodos);
router.post("/", isAuth, createTodo);
router.put("/:id", isAuth, updateTodo);
router.delete("/:id", isAuth, deleteTodo);

export default router;
```

---

## 8.實作成果

用 postman 測試註冊 API :

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day15-JWT/1-register.png?raw=true)

測試登入 API :

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day15-JWT/2-login.png?raw=true)

在 postman 傳送請求時帶上 token

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day15-JWT/3-token.png?raw=true)

嘗試新增待辦事項

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day15-JWT/4-add-todo.png?raw=true)

可發現待辦事項綁定了 userId

現在，每個人都只能操作自己的 Todo！🎉

---

## 🔑 本日重點複習

1. **bcryptjs**：加密密碼，保護使用者資料。
2. **jsonwebtoken**：簽發 / 驗證 Token，實現登入驗證。
3. **middleware**：保護 API，只允許合法使用者操作。
4. **User 與 Todo 關聯**：實現一人一份 TodoList。

---

## 寫在最後

做到這裡，我們的 TodoList API 已經升級到「多人系統」。

每個人都必須註冊 / 登入，才能建立屬於自己的 Todo。

---

## 補充資源

> commit : install jwt plugin and implement auth feature

[Github 連結](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/cbaf0b87023e4232ee25ca63cfe873dde4613b4a)
