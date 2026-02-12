---
title: "Next.js 部署避坑：腾讯云 EdgeOne Pages 的“实战手术刀”"
published: 2026-02-12
description: "记录一次使用 OpenClaw 自动化部署 Next.js 项目到腾讯云 EdgeOne Pages 的全过程，包含 404 错误解决、静态导出配置及 Node 版本切换问题的深度复盘。"
image: ""
tags: ["Next.js", "腾讯云", "EdgeOne", "部署指南", "小小芸实战"]
category: "技术分享"
draft: false
---

# 引言

在 2026 年的今天，部署一个 Next.js 项目理应是“一键式”的。然而，当你试图在腾讯云 **EdgeOne Pages** 这种兼具 CDN 加速与静态托管的平台上进行“深水区”操作时，往往会遇到一些意想不到的“惊喜”。

今天，我（和我的 AI 助手小小芸）经历了一场长达两小时的“代码手术”，终于打通了 Next.js 与 EdgeOne 的完美链路。

---

## 🛠 核心踩坑点与对策

### 1. 自动切换 Node 版本失败 (Switching Error)
**现象**：构建日志显示 `Switching node version` 后面跟着一串红色的 `Switching error`。
**对策**：
- 在根目录创建 `.nvmrc` 文件，明确指定 `20`。
- 在 `package.json` 的 `engines` 字段中锁定 Node 版本。

:::tip
EdgeOne 的构建环境有时对环境变量的感应比较迟钝，显式指定版本是最佳实践。
:::

### 2. 消失的适配器：@edgeone/next
**现象**：`npm install` 时报 404，找不到官方文档提到的适配器包。
**对策**：不要试图在本地安装它。腾讯云的 `eo-next` 指令实际上是内置在构建环境里的。在 `package.json` 的脚本中直接写 `next build && eo-next build` 即可。

### 3. 终极奥义：静态导出 (Static Export)
如果你追求极致的加载速度和稳定性（避免 `/posts?_rsc=...` 这种动态路由报 404），**静态导出**是唯一的正确答案。

```javascript
// next.config.mjs
const nextConfig = {
  output: 'export',
  images: {
    unoptimized: true, // 静态导出必须禁用图片优化
  },
};
```

---

## 🚀 总结

部署的本质是**环境的博弈**。通过这次实战，我们不仅拥有了一个清爽的个人主页，更摸清了 EdgeOne Pages 的底层脾气。

> [!IMPORTANT]
> 以后遇到构建失败，不要盲目重试，优先检查 `package-lock.json` 是否与构建环境兼容，并学会利用 GitHub Actions 进行“场外构建”。

---

**小小芸注**：本文由吴总指引，AI 助手小小芸整理完成。
