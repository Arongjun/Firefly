---
title: "解决 OpenClaw WebUI 报错 disconnected (1008): pairing required"
description: "深度复盘并解决 OpenClaw 在 Nginx 转发环境下移动端无法连接 WebUI 的疑难杂症。"
pubDate: 2026-02-15 00:45:00
category: 技术方案
tags: [OpenClaw, Nginx, WebSocket, 安全配置]
---

在使用 OpenClaw 过程中，如果通过 Nginx 反向代理访问 WebUI，经常会遇到手机端或新设备报错 `disconnected (1008): pairing required`。即便输入正确的 Token 也无法登录。

本文将详细记录这一问题的病根及三种行之有效的修复方法。

## 问题症状
- 电脑端访问正常，手机端通过域名（HTTPS）访问时报错。
- 界面提示 `1008` 错误代码。
- 无法弹出配对码（Pairing Code）输入框。

## 病根分析
OpenClaw Gateway 具有极高的安全性。当连接来自“非本地 IP”（如移动网络通过 Nginx 转发）时，Gateway 会认为这是一个潜在的不安全连接。
1. **信任缺失**：Gateway 默认不信任反向代理。
2. **安全防御**：对于未经过配对验证的设备，哪怕带了 Token，Gateway 也会出于自我保护强制要求一次配对握手。

## 修复方案

### 方法一：后台手动批准（命令行模式）
如果您能访问服务器终端，这是最直接的方法。通过手动批准待处理的配对请求来建立信任。

1. **查看待处理请求**：
   ```bash
   openclaw devices list
   ```
2. **批准请求**：
   找到对应的 `request id` 后运行：
   ```bash
   openclaw devices approve <request id>
   ```

### 方法二：信任反向代理（推荐配置）
修改服务器上的 `~/.openclaw/openclaw.json`，在 `gateway` 配置段中加入 `trustedProxies`。这告诉 Gateway，Nginx 转发过来的流量是“自己人”。

```json
"gateway": {
  "trustedProxies": ["127.0.0.1"],
  ...
}
```

### 方法三：修改配置文件强制放行（最彻底/暴力）
如果上述方法依然无效（通常是由于手机浏览器缓存或 Nginx 头部透传问题），可以强制允许非安全环境下的身份验证。在 `gateway` 配置段中添加 `controlUi` 项。

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
- **重启生效**：修改配置文件后，务必执行 `openclaw gateway restart`。
- **强制刷新**：如果手机浏览器依然卡在旧报错界面，请尝试开启 **“隐私模式”** 或 **“清除缓存”**，强制发起全新的 WebSocket 握手。

:::note
**总结**：安全是一把双刃剑，OpenClaw 默认的高安全策略在复杂网络下需要通过手动授权或 `trustedProxies` 进行调优。
:::
