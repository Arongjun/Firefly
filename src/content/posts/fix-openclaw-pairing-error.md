---
title: "解决 OpenClaw WebUI 报错 disconnected (1008): pairing required"
description: "深度复盘并解决 OpenClaw 在 Nginx 转发环境下移动端无法连接 WebUI 的疑难杂症。"
pubDate: 2026-02-15 00:45:00
category: 技术方案
tags: [OpenClaw, Nginx, WebSocket, 安全配置]
---

在使用 OpenClaw 过程中，如果通过 Nginx 反向代理访问 WebUI，经常会遇到手机端或新设备报错 `disconnected (1008): pairing required`。即便输入正确的 Token 也无法登录。

本文将详细记录这一问题的病根及两种永久修复方法。

## 问题症状
- 电脑端访问正常，手机端通过域名（HTTPS）访问时报错。
- 界面提示 `1008` 错误代码。
- 无法弹出配对码（Pairing Code）输入框。

## 病根分析
OpenClaw Gateway 具有极高的安全性。当连接来自“非本地 IP”（如移动网络通过 Nginx 转发）时，Gateway 会认为这是一个潜在的不安全连接。
1. **信任缺失**：Gateway 默认不信任反向代理。
2. **安全防御**：对于未经过配对验证的设备，哪怕带了 Token，Gateway 也会出于自我保护强制要求一次配对握手。
3. **缓存死锁**：部分手机浏览器会记住连接失败的状态，导致不主动弹出配对输入框。

## 修复方法

### 方案 A：信任反向代理（推荐）
修改服务器上的 `~/.openclaw/openclaw.json`，在 `gateway` 配置段中加入 `trustedProxies`。这告诉 Gateway，Nginx 转发过来的流量是“自己人”。

```json
"gateway": {
  "trustedProxies": ["127.0.0.1"],
  ...
}
```

### 方案 B：允许非安全验证（最彻底）
如果通过上述方法手机依然无法连上，可以强制关闭配对校验，并允许非安全环境下的身份验证。在 `gateway` 下添加 `controlUi` 和 `pairing` 配置。

```json
"gateway": {
  "controlUi": {
    "allowInsecureAuth": true
  },
  "auth": {
    "mode": "token",
    "token": "你的TOKEN",
    "pairing": "off"
  }
}
```

## 操作贴士
- 修改配置文件后，务必执行 `openclaw gateway restart` 使其生效。
- 如果手机浏览器依然卡死，请尝试开启 **“隐私模式”** 或 **“清除缓存”**，强制浏览器发起全新的 WebSocket 握手。

:::note
**总结**：安全是一把双刃剑，OpenClaw 默认的高安全策略在复杂网络下需要通过 `trustedProxies` 手动进行调优。
:::
