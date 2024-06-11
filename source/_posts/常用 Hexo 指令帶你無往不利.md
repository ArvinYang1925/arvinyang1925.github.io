---
title: 常用 Hexo 指令帶你無往不利
date: 2024-06-07 16:51:13
tags:
  - Hexo 指令
---

Hexo 是一个快速、简单且功能强大的静态博客框架。以下是一些常用的 Hexo 指令：

```sh
   hexo new 文章名稱
   hexo generate 產生 public 資料夾內容
   hexo server
   hexo clean 清理緩存
   hexo deploy
```

<!--more-->

1. **安装 Hexo**

   ```sh
   npm install -g hexo-cli
   ```

2. **初始化 Hexo 项目**

   ```sh
   hexo init <folder>
   cd <folder>
   npm install
   ```

3. **生成静态文件**

   ```sh
   hexo generate
   ```

4. **启动本地服务器**

   ```sh
   hexo server
   ```

5. **新建文章**

   ```sh
   hexo new <title>
   ```

6. **发布文章**

   ```sh
   hexo deploy
   ```

7. **清理缓存和生成的静态文件**

   ```sh
   hexo clean
   ```

8. **列出所有 Hexo 命令**

   ```sh
   hexo list
   ```

9. **查看 Hexo 帮助信息**

   ```sh
   hexo help
   ```

10. **安装插件**
    ```sh
    npm install <plugin-name> --save
    ```

以下是一些额外的有用命令和操作：

- **新建页面**

  ```sh
  hexo new page <page-name>
  ```

- **草稿转发布**
  如果你使用 `hexo new draft <title>` 创建了草稿，可以通过以下命令将草稿发布为正式文章：

  ```sh
  hexo publish <title>
  ```

- **生成并部署**
  这会在生成静态文件后立即部署它们：

  ```sh
  hexo generate --deploy
  ```

- **查看当前配置**
  你可以直接编辑 `_config.yml` 文件来配置 Hexo，但也可以用以下命令查看当前配置：
  ```sh
  hexo config
  ```

这些命令涵盖了 Hexo 的大部分基本功能，可以帮助你快速上手并管理你的博客。
