---
layout: day
title: Day 17 - Firebase Storage 實戰：用 Node.js + multer 上傳圖片到雲端
date: 2025-10-01 18:09:24
tags:
  - TypeScript
  - Node.js
  - Tools
---

昨天我們完成了 **Firebase Storage 的環境設定**：

- 建立專案 & Bucket
- 下載服務金鑰
- 設定 `.env`

今天就要正式進入**實戰篇**！

想像一下：現在我們的服務需要讓使用者可以上傳**大頭貼**。

那我們該怎麼做？

👉 就是用 **Node.js + multer** 串接 Firebase Storage，把檔案安全地存到雲端，最後產生一個公開可存取的 URL。

<!-- more -->

### **步驟 1：安裝必要套件**

```bash
# 核心套件
npm install multer firebase-admin

# TypeScript 開發依賴（如果使用 TypeScript）
npm install -D @types/multer
```

**套件說明：**

- `multer`: 處理 `multipart/form-data` 的文件上傳中間件
- `firebase-admin`: Firebase Admin SDK，用於操作 Firebase Storage

---

### **步驟 2：設定環境變數 (.env)**

在 `.env` 檔案中加入 Firebase 設定（詳細可參考 Day16 文章)

```bash
# Firebase 設定
FIREBASE_SERVICE_ACCOUNT={"type":"service_account",...,"client_x509_cert_url":"xxx>"}
FIREBASE_STORAGE_BUCKET=your-project.appspot.com
```

**重要提醒：**

- `FIREBASE_SERVICE_ACCOUNT` 必須是**完整的 JSON 字串**（單行）
- 記得把 `.env` 加入 `.gitignore`，避免洩漏金鑰

---

### **步驟 3：新增 User Entity 欄位**

在 `User.ts` 增加 `profileUrl`，用來存放大頭貼位址：

```json
@Entity()
export class User {
  @PrimaryGeneratedColumn("uuid")
  id!: string;

  @Column({ type: "varchar", length: 50, nullable: false })
  name!: string;

  @Column({ type: "varchar", length: 320, unique: true, nullable: false })
  email!: string;

  ...

  @Column({ name: "profile_url", length: 2048, nullable: true })
  profileUrl?: string;

	...
}
```

---

### **步驟 4：建立 Firebase 工具檔 (utils/firebaseUtils.ts)**

```tsx
import admin from "firebase-admin";
import dotenv from "dotenv";

dotenv.config();

// 解析環境變數中的 JSON 字串
const serviceAccount = JSON.parse(process.env.FIREBASE_SERVICE_ACCOUNT!);

// 初始化 Firebase Admin
admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
});

// 取得 Storage Bucket
const bucket = admin.storage().bucket();

export { admin, bucket };
```

---

### **步驟 5：建立圖片上傳中間件 (middleware/imageUpload.ts)**

```tsx
import multer from "multer";
import path from "path";
import { Request } from "express";
import { Express } from "express";

// 使用記憶體儲存（不寫入硬碟）
const storage = multer.memoryStorage();

// 檔案過濾器：只接受 JPG 和 PNG
const imageFileFilter = (
  req: Request,
  file: Express.Multer.File,
  cb: multer.FileFilterCallback
) => {
  const ext = path.extname(file.originalname).toLowerCase();
  if (![".jpg", ".png", ".jpeg"].includes(ext)) {
    return cb(new Error("只接受 JPG/PNG 格式的圖片檔案"));
  }
  cb(null, true);
};

// 限制：2 MB、JPG/PNG 格式
export const imageUpload = multer({
  storage,
  limits: { fileSize: 2 * 1024 * 1024 }, // 2 MB
  fileFilter: imageFileFilter,
});
```

**設計重點：**

- `memoryStorage()`: 檔案存在記憶體中（`req.file.buffer`），適合直接上傳到雲端
- `fileFilter`: 限制只接受圖片格式
- `limits`: 限制檔案大小為 2 MB

---

### **步驟 6：建立上傳 Controller (controllers/uploadController.ts)**

```tsx
import { Response, NextFunction } from "express";
import path from "path";
import { bucket } from "../utils/firebaseUtils";
import { AuthRequest } from "../middleware/isAuth";
import { AppDataSource } from "../config/db";
import { User } from "../entities/User";

/**
 * 上傳大頭照到 Firebase Storage
 */
export async function uploadAvatar(
  req: AuthRequest,
  res: Response,
  next: NextFunction
) {
  try {
    // 1. 檢查是否有上傳檔案
    if (!req.file) {
      res.status(400).json({
        status: "failed",
        message: "請選擇要上傳的圖片檔案",
      });
      return;
    }

    // 2. 檢查使用者是否已登入
    if (!req.user) {
      res.status(401).json({
        status: "failed",
        message: "請先登入",
      });
      return;
    }

    // 3. 產生遠端檔案路徑
    const timestamp = Date.now();
    const ext = path.extname(req.file.originalname).toLowerCase();
    const remotePath = `images/avatars/user-${req.user.id}-${timestamp}${ext}`;

    // 4. 取得 Firebase Storage 檔案參考
    const file = bucket.file(remotePath);

    // 5. 建立寫入串流
    const stream = file.createWriteStream({
      metadata: {
        contentType: req.file.mimetype,
      },
    });

    // 6. 錯誤處理
    stream.on("error", (err) => next(err));

    // 7. 上傳完成後的處理
    stream.on("finish", async () => {
      try {
        // 設定檔案為公開存取
        await file.makePublic();

        // 產生公開 URL
        const publicUrl = `https://storage.googleapis.com/${bucket.name}/${remotePath}`;

        // 8. 更新資料庫（根據你的 ORM/資料庫架構調整）
        await AppDataSource.getRepository(User).update(
          { id: req.user?.id },
          { profileUrl: publicUrl }
        );

        // 9. 回傳成功訊息
        res.status(200).json({
          status: "success",
          message: "大頭照上傳成功",
          data: { avatarUrl: publicUrl },
        });
      } catch (err) {
        next(err);
      }
    });

    // 10. 將檔案緩衝區寫入串流
    stream.end(req.file.buffer);
  } catch (err) {
    next(err);
  }
}
```

---

### **步驟 7：設定路由 (routes/uploadRoutes.ts)**

```tsx
import { Router } from "express";
import { uploadAvatar } from "../controllers/uploadController";
import { imageUpload } from "../middleware/imageUpload";
import { isAuth } from "../middleware/isAuth";

const router = Router();

router.post(
  "/avatar",
  isAuth, // 1. 驗證 JWT token
  imageUpload.single("file"), // 2. 處理單一檔案上傳，欄位名稱為 "file"
  uploadAvatar // 3. 執行上傳邏輯
);

export default router;
```

**中間件順序很重要：**

1. 先驗證使用者身份 (`isAuth`)
2. 再處理檔案上傳 (`imageUpload.single("file")`)
3. 最後執行業務邏輯 (`uploadAvatar`)

---

### **步驟 8：註冊路由到主應用程式 (app.ts)**

```tsx
import express from "express";
import uploadRoutes from "./routes/uploadRoutes";

const app = express();

// ... 其他中間件設定

// 註冊上傳路由
app.use("/api/upload", uploadRoutes); // 上傳路由

export default app;
```

---

### **步驟 9：上傳範例**

使用 **Postman** 測試：

1. 選擇 `POST` 方法
2. URL: `http://localhost:3000/api/upload/avatar`
3. Headers: `Authorization: Bearer YOUR_JWT_TOKEN`
4. Body → form-data → Key: `file` (選擇 File 類型) → 選擇圖片

經由 Postman 回傳結果可得知成功上傳！成功後就能拿到一個 Firebase Storage 的公開 URL 🎉

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day17-Firebase-Multer/1-postman.png?raw=true)

接著，查看一下資料庫 `profileUrl` 欄位 → 發現已經有正確的 URL 存入

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day17-Firebase-Multer/2-database.png?raw=true)

最後，再到我們的 Firebase Storage 查看檔案 :

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day17-Firebase-Multer/3-firebase.png?raw=true)

太棒了！我們用 Node.js + multer 成功串接了 Firebase Storage 服務 🍻

---

## 小結

今天我們完成了：

- 使用 `multer` 處理圖片上傳
- 串接 Firebase Storage
- 把檔案存雲端並取得公開連結
- 更新資料庫，讓使用者能擁有自己的大頭貼

到這裡，我們的服務具備了「圖片上傳」的能力！ 🚀

## 補充資源

> commit : use multer and firebase storage to upload file

[Github 連結](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/e46e7f49aa707b49da1f733534fd2747898974d1)
