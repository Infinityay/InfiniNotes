---
title: "Hugo 部署到 EdgeOne Pages 完整指南"
date: 2025-12-05
draft: false
tags: ["Hugo", "EdgeOne", "部署", "博客"]
categories: ["技术"]
summary: "详细介绍如何将 Hugo 静态博客部署到腾讯 EdgeOne Pages，解决常见报错问题。"
---

## 前言

今天心血来潮想试试腾讯的 EdgeOne Pages，结果折腾了一下午，部署过程各种报错令人头疼，最后终于折腾出能顺利部署的方法了！

本文整理了完整的部署流程，希望能帮助遇到同样问题的朋友。

---

## 准备工作

### 1. 确保你的 Hugo 项目结构正确

```
your-blog/
├── archetypes/
├── content/
│   └── posts/
├── layouts/
├── static/
├── themes/
│   └── PaperMod/  (或其他主题)
├── hugo.toml      (配置文件)
└── package.json   (关键！下面会创建)
```

### 2. 创建 package.json 文件

在项目根目录创建 `package.json` 文件，这是 **关键步骤**：

```json
{
  "name": "your-blog-name",
  "version": "1.0.0",
  "scripts": {
    "build": "./node_modules/.bin/hugo --minify",
    "postinstall": "git submodule update --init",
    "dev": "hugo server -D",
    "clean": "rm -rf public",
    "preview": "hugo server"
  },
  "dependencies": {
    "hugo-extended": "^0.152.2"
  },
  "devDependencies": {
    "cross-env": "^7.0.3"
  }
}
```

> **注意**：`hugo-extended` 的版本号请根据你本地使用的 Hugo 版本填写。可以通过 `hugo version` 命令查看。

### 3. 配置 .gitignore

确保 `.gitignore` 排除构建产物：

```gitignore
# Hugo 生成的文件
/public/
/resources/_gen/
/.hugo_build.lock

# Node
node_modules/
package-lock.json
```

### 4. 推送到 GitHub

```bash
git add .
git commit -m "准备 EdgeOne 部署"
git push
```

---

## EdgeOne Pages 部署配置

在 EdgeOne Pages 控制台导入 Git 项目后，**敲黑板！这里要注意配置**：

| 配置项 | 值 |
|--------|-----|
| **输出目录** | `public` |
| **构建命令** | `npm run build` |
| **安装命令** | `HUGO_BIN_EXTENDED=true npm install --save-dev cross-env` |

填好这三项后，点击 **"开始部署"**，等待一小会就完成啦！

---

## 常见问题

### Q1: 构建报错找不到 hugo 命令

**原因**：EdgeOne 环境没有预装 Hugo。

**解决**：使用 `package.json` 中的 `hugo-extended` 依赖，通过 npm 安装 Hugo。

### Q2: 主题没有加载

**原因**：主题是 git submodule，没有正确初始化。

**解决**：`package.json` 中的 `postinstall` 脚本会自动执行 `git submodule update --init`。

### Q3: GitHub Actions workflow 报错

**原因**：如果你之前配置过 GitHub Pages 的 workflow，但没有开启 GitHub Pages。

**解决**：删除 `.github/workflows/` 目录，因为用 EdgeOne 就不需要 GitHub Actions 了。

```bash
rm -rf .github
git add -A
git commit -m "删除 GitHub Actions，改用 EdgeOne"
git push
```

---

## 后续更新文章

部署成功后，每次写新文章只需：

```bash
# 创建新文章
hugo new posts/新文章.md

# 编辑文章内容...

# 推送到 GitHub
git add .
git commit -m "新增文章"
git push
```

EdgeOne 会自动检测到仓库更新并重新部署。

---

## 参考

- [EdgeOne Pages 官方文档](https://cloud.tencent.com/document/product/1552)
- [Hugo 官方文档](https://gohugo.io/documentation/)
- [原文参考](https://www.bilibili.com/opus/1090179935173083161)
