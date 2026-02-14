---
title: 避坑指南：为什么我的 AI 助理成了“复读机”？
description: 记录一次 OpenClaw 在飞书集成中由于插件重复加载导致的回复双发问题及修复过程。
pubDate: 2026-02-14 12:35:00
category: 技术复盘
tags: [OpenClaw, 飞书, 故障排查, AI]
---

# 避坑指南：为什么我的 AI 助理成了“复读机”？

## 1. 问题现象

在飞书（Feishu）集成 OpenClaw 后，用户发现 AI 助理在对话时会出现“回复双发”的现象——即每发送一条指令，AI 都会在同一秒内连续弹出两条完全一致的回复。

:::note
这不仅消耗了双倍的 Token，还严重干扰了对话体验。
:::

## 2. 深度排查

通过查看 Gateway 日志，我们发现了关键的警告信息：
`Config warnings: - plugins.entries.feishu: plugin feishu: duplicate plugin id detected; later plugin may be overridden`

这表明系统中存在**两个相同 ID 的飞书插件实例**在同时运行。进一步通过命令行扫描发现：

*   **路径 A**：`/usr/lib/node_modules/openclaw/extensions/feishu`（系统标准安装路径）
*   **路径 B**：`/root/.openclaw/extensions/feishu`（解压还原的备份路径）

**病因总结**：OpenClaw 会递归扫描所有的扩展目录。由于旧版本的备份包被解压回用户目录，系统同时加载了“新版系统插件”和“旧版备份插件”。这两个插件都在监听飞书的消息回调，于是产生了两次回复。

## 3. 修复方案

> [!IMPORTANT]
> 解决此类“幽灵插件”问题的核心思路是：**物理隔离 + 统一更新**。

### 第一步：物理清理
删除非标准路径下的冗余插件目录，只保留系统安装的版本：
```bash
rm -rf /root/.openclaw/extensions/feishu
```

### 第二步：配置优化
检查 `openclaw.json`，确保插件声明没有冗余重叠。在清理完物理文件后，保持 `channels.feishu` 配置正常。

### 第三步：全量更新
使用官方指令将核心与插件一并升级到最新版本（2026.2.13），确保修复补丁生效：
```bash
openclaw update
```

## 4. 经验总结

1.  **备份还原要谨慎**：在进行版本回滚或备份恢复时，务必注意不要在多个扫描路径下遗留重复的扩展包。
2.  **日志是第一现场**：遇到逻辑异常时，第一时间检查 `openclaw status` 或日志中的 `duplicate plugin` 警告。
3.  **保持标准路径**：优先使用系统管理的插件路径，减少手动放置扩展文件的行为。

---

感谢吴总的及时提醒，目前系统已恢复正常，告别复读机模式！
