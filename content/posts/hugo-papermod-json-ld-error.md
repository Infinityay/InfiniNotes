---
title: "解决 Hugo PaperMod 主题构建时 JSON-LD 解析错误"
date: 2025-12-05
draft: false
tags: ["Hugo", "PaperMod", "JSON-LD", "踩坑"]
categories: ["技术"]
summary: "记录在使用 Hugo PaperMod 主题时，因文章中包含 JSON 代码块导致构建失败的问题及解决方案。"
---

## 问题描述

在使用 Hugo + PaperMod 主题构建博客时，遇到如下报错：

```
ERROR failed to process "/posts/xxx/index.html": 
"/tmp/hugo-transform-error:116:40": expected comma character 
or an array or object ending on line 116 and column 40
   12:     {
           ^
```

这个错误非常迷惑，看起来像是 JSON 解析错误，但明明只是在写 Markdown 文章。

## 问题原因

经过排查，发现问题出在 **PaperMod 主题的 JSON-LD 结构化数据生成**。

PaperMod 主题会在每个页面生成 JSON-LD（结构化数据），用于 SEO 优化。其中有一段模板代码：

```html
"articleBody": {{ .Content | safeJS | htmlUnescape | plainify }}
```

这行代码会把**整篇文章的内容**转换成 JSON 字符串嵌入到 `<script type="application/ld+json">` 中。

问题来了：如果你的文章里有 JSON 代码块，比如：

```json
{
  "name": "example",
  "version": "1.0.0"
}
```

这段代码会被直接嵌入 JSON-LD，导致 JSON 格式错误，Hugo 在处理时就会报错。

## 解决方案

### 方案一：禁用 JSON-LD

在项目的 `layouts/partials/templates/` 目录下创建一个空的 `schema_json.html` 文件：

```bash
mkdir -p layouts/partials/templates
echo '{{/* 禁用 JSON-LD schema 以避免代码块中的 JSON 导致解析错误 */}}' > layouts/partials/templates/schema_json.html
```

这个空模板会覆盖主题的 JSON-LD 模板，从而禁用 JSON-LD 输出。



## 禁用 JSON-LD 的影响

| 方面 | 影响 |
|------|------|
| 网站功能 | ✅ 无影响 |
| 页面显示 | ✅ 无影响 |
| SEO 富文本摘要 | ⚠️ 搜索结果不会显示增强信息（作者、日期等） |
| 页面体积 | ✅ 略有减小 |

对于个人博客来说，这个影响可以忽略不计。

## 总结

这是 Hugo PaperMod 主题的一个已知问题。当文章内容包含 JSON 代码块时，JSON-LD 生成会失败。

最简单的解决方案就是禁用 JSON-LD，用一个空模板覆盖主题的 `schema_json.html`。

希望这篇文章能帮助遇到同样问题的朋友！

## 参考

- [PaperMod 主题](https://github.com/adityatelange/hugo-PaperMod)
- [Hugo 官方文档](https://gohugo.io/)
- [JSON-LD 介绍](https://json-ld.org/)
