---
layout: day
title: Day 27 - AWS S3 實作檔案上傳：打造你的雲端檔案儲存中心
date: 2025-10-11 23:20:46
tags:
  - TypeScript
  - Node.js
  - AWS
---

## 前言

在前幾天，我們已經完成了 **EC2 主機部署** 以及 **RDS 資料庫串接**，

一個完整的後端雲端架構也漸漸成形。

但如果今天你的應用要讓使用者能上傳圖片、影片、報表、PDF…

這些「檔案」要放哪裡呢？

這時候，主角就登場了 —— **AWS S3 (Simple Storage Service)**。

> 💡 S3 是 AWS 三大基礎服務之一（EC2、RDS、S3），
>
> 它就像你在雲端的「硬碟」，負責安全地儲存、管理檔案。

今天我們就要實際操作：

從建立 S3 Bucket → 設定權限 → 實作 Node.js 上傳功能，

一步步完成「檔案雲端化」的實戰練習！

<!-- more -->

---

## 為什麼要使用 AWS S3？

### 1.超穩定的雲端儲存空間

S3 的資料儲存設計達到「**99.999999999%（11 個 9）**」的耐久性，

意思是：你的圖片幾乎不可能遺失。

### 2.價格便宜又按量計費

你只需為實際用到的空間與流量付費，

不用像傳統伺服器那樣一直維護硬碟或備份。

### 3.全球可存取

S3 的檔案可以透過 URL 在全世界直接被讀取，

搭配 CDN（如 CloudFront）後，速度更是飛快。

### 4.完美整合 AWS 生態系

與 EC2、Lambda、CloudFront、IAM、Route53 等無縫結合

不論是後端 API、影片平台、或電商後台都能輕鬆擴充。

---

## 第一部分：AWS S3 設定步驟

### 步驟 1：建立 S3 Bucket

1. **登入 AWS Console**
   - 前往 https://aws.amazon.com/console/
   - 使用您的 AWS 帳號登入
2. **進入 S3 服務**
   - 在上方搜尋欄輸入「S3」
   - 點選「S3」服務
3. **建立新的 Bucket**
   - 點擊「 **建立儲存貯體** 」按鈕
     ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/1-aws-s3.png?raw=true)
   - **AWS Region**: 選擇離您最近的區域（例如：`ap-northeast-1` 東京）
   - **Bucket name**: 輸入唯一名稱（例如：`my-ithome-2025-bucket`）
     - 注意：Bucket 名稱必須是全球唯一的
       ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/2-bucket-name.png?raw=true)
4. **設定 Object Ownership**
   - 選擇「ACLs disabled (recommended)」
     ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/3-ACL-dis.png?raw=true)
5. **Block Public Access settings**
   - **取消勾選**「Block all public access」
   - 勾選確認框（我了解這會讓檔案變成公開存取）
   - ⚠️ 注意：如果您想讓上傳的圖片可以公開存取，需要取消封鎖
     ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/4-public-access.png?raw=true)
6. **Bucket Versioning**
   - 選擇「Disable」
7. **其他設定保持預設**，點擊「 **建立儲存貯體** 」

### 步驟 2：設定 Bucket Policy（讓檔案可公開讀取）

1. 點進剛建立的 Bucket
2. 前往「Permissions」標籤
3. 找到「Bucket policy」區塊，點擊「Edit」

   ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/5-bucket-policy.png?raw=true)

4. 貼上以下 Policy（記得替換 `YOUR-BUCKET-NAME`）：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

5. 點擊「Save changes」

### 步驟 3：建立 IAM 使用者與取得 Access Key

1. **進入 IAM 服務**
   - 搜尋並點選「IAM」服務
2. **建立新使用者**
   - 左側選單點選「Users」
   - 點擊「Create user」
     ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/6-create-user.png?raw=true)
   - **User name**: 輸入 `s3-upload-user`
     ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/7-user-name.png?raw=true)
   - 點擊「Next」
3. **設定權限**
   - 選擇「Attach policies directly」
   - 搜尋並勾選「AmazonS3FullAccess」（簡單起見；生產環境建議使用更細緻的權限）
     ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/8-set-policy.png?raw=true)
   - 點擊「Next」，然後「Create user」
4. **建立 Access Key**
   - 點進剛建立的使用者
   - 選擇「Security credentials」標籤
   - 找到「Access keys」區塊，點擊「Create access key」
     ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/9-create-access-key.png?raw=true)
   - 選擇「Application running outside AWS」
     ![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/10-key-range.png?raw=true)
   - 點擊「Next」
   - 描述標籤：輸入「Node.js Upload App」
   - 點擊「Create access key」
   - **重要！** 複製並儲存：(等等需在 `.env` 設定)
     - **Access key ID** (類似：`AKIAIOSFODNN7EXAMPLE`)
     - **Secret access key** (類似：`wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`)
     - ⚠️ Secret key 只會顯示一次，請妥善保存！

---

## 第二部分：程式碼整合

### 1. 安裝 AWS SDK

```bash
npm install @aws-sdk/client-s3
npm install --save-dev @types/node
```

### 2. 設定環境變數

在您的 `.env` 檔案加入：

```
# AWS S3 設定
AWS_REGION=ap-northeast-1
AWS_ACCESS_KEY_ID=your-access-key-id
AWS_SECRET_ACCESS_KEY=your-secret-access-key
AWS_S3_BUCKET_NAME=my-ithome-2025-bucket
```

### 3. 建立 S3 工具檔案

建立新檔案：`src/utils/s3Utils.ts`

```tsx
import { S3Client } from "@aws-sdk/client-s3";

// 初始化 S3 Client
export const s3Client = new S3Client({
  region: process.env.AWS_REGION || "ap-northeast-1",
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID || "",
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY || "",
  },
});

export const bucketName = process.env.AWS_S3_BUCKET_NAME || "";
```

### 4. 修改上傳 Controller

修改 `src/controllers/uploadController.ts`，新增一個使用 S3 的函式：

```tsx
import { Response, NextFunction } from "express";
import path from "path";
import { PutObjectCommand } from "@aws-sdk/client-s3";
import { s3Client, bucketName } from "../utils/s3Utils";
import { AuthRequest } from "../middleware/isAuth";
import { AppDataSource } from "../config/db";
import { User } from "../entities/User";

/**
 * 上傳大頭照到 AWS S3
 */
export async function uploadAvatarToS3(
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

    // 3. 產生檔案路徑與名稱
    const timestamp = Date.now();
    const ext = path.extname(req.file.originalname).toLowerCase();
    const key = `images/avatars/user-${req.user.id}-${timestamp}${ext}`;

    // 4. 上傳到 S3
    const command = new PutObjectCommand({
      Bucket: bucketName,
      Key: key,
      Body: req.file.buffer,
      ContentType: req.file.mimetype,
    });

    await s3Client.send(command);

    // 5. 產生公開 URL
    const region = process.env.AWS_REGION || "ap-northeast-1";
    const publicUrl = `https://${bucketName}.s3.${region}.amazonaws.com/${key}`;

    // 6. 更新資料庫
    await AppDataSource.getRepository(User).update(
      { id: req.user.id },
      { profileUrl: publicUrl }
    );

    // 7. 回傳成功訊息
    res.status(200).json({
      status: "success",
      message: "大頭照上傳成功",
      data: { avatarUrl: publicUrl },
    });
  } catch (err) {
    next(err);
  }
}
```

### 5. 更新路由

修改 `src/routes/uploadRoutes.ts`：

```tsx
import { Router } from "express";
import {
  uploadAvatar,
  uploadAvatarToS3,
} from "../controllers/uploadController";
import { imageUpload } from "../middleware/imageUpload";
import { isAuth } from "../middleware/isAuth";

const router = Router();

// Firebase 版本（保留原有功能）
router.post("/avatar", isAuth, imageUpload.single("file"), uploadAvatar);

// AWS S3 版本（新增）
router.post("/avatar-s3", isAuth, imageUpload.single("file"), uploadAvatarToS3);

export default router;
```

---

## 測試上傳檔案

使用 Postman 測試：

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/11-postman-test.png?raw=true)

查看資料庫 `profile_url` 欄位 : (成功填入檔案位址)

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/12-check-db.png?raw=true)

到 S3 查看檔案是否已經上傳 :

![](https://github.com/ArvinYang1925/iThome-2025/blob/main/images/Day27-AWS-S3/13-check-s3.png?raw=true)

成功驗證檔案上傳！

---

## 重點提醒

1. **不要將 AWS credentials 提交到 Git**
   - 確保 `.env` 在 `.gitignore` 中
2. **成本考量**
   - S3 儲存與流量都會收費，但非常便宜
   - 可設定 Lifecycle policy 自動刪除舊檔案

---

## 結語

這次的實作，我們練習了：

- 建立並設定 S3 Bucket
- 讓檔案可公開讀取
- 透過 Node.js 上傳檔案到雲端
- 並將結果寫回資料庫

雖然看起來步驟很多，但這正是「真實世界後端」會遇到的流程。

加油！

---

## 參考資源

> commit :
>
> feat: 🎸 integrate AWS S3 upload feature
> 新增 AWS S3 檔案上傳功能（s3Utils + uploadController + route）

[Github 連結](https://github.com/ArvinYang1925/iThome2025-node-ts/commit/89e8d3de7355d24c174923dc0a4606874a4920fa)
