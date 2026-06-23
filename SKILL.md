---
name: kwai-author-outreach
description: 批量获取 Kwai 作者联系方式并执行初次建联流程。当用户说"作者触达"、"建联"、"联系作者"、"获取联系方式"、"outreach"、"触达这些作者"、"建联流程"、"给我这些作者信息"时自动激活。输入一批 UID，输出作者信息表格（含电话、社媒、MCN），并按流程指导平台私信、WhatsApp/IG 私信、MCN 联系等触达步骤。不负责非 Kwai 作者的触达、不负责内容创作或视频分析。
---

# Kwai 作者初次建联

输入一批 UID → 输出联系信息表格 → 指导触达流程。

## 流程总览

```
输入 UID 列表
  → Step 1: Noah 提取 Profile（姓名/Kwai ID/机构/家族/Bio/外部链接）
  → Step 2: Noah 点击眼睛图标获取完整电话
  → Step 3: Mac 节点搜索 Instagram / Telegram
  → Step 4: 输出联系信息表格
  → Step 5: 按流程发送触达消息（平台私信 + WhatsApp/IG + MCN）
  → Step 6: 24h 跟进
```

## Step 1-3: 获取联系方式

读取 `references/noah-extraction.md` 执行 Noah 数据提取 + Mac 节点社媒搜索。

提取完成后输出如下表格：

| # | 作者名 | UID | Kwai ID | 外部分享链接 | 电话号码 | Instagram/其他社媒 | 机构(MCN) | 家族(Family) |
|---|--------|-----|---------|-------------|----------|-------------------|-----------|-------------|

外部分享链接格式：`https://www.kwai.com/@{KwaiID}`

## Step 4-5: 触达流程

读取 `references/outreach-playbook.md` 获取完整话术模版和发送顺序。

**触达顺序（同步进行）：**
1. Kwai 站内私信 — 验证身份 + 引导到 WhatsApp
2. WhatsApp 自动私信 — 通过 Mac 节点 URL scheme 自动发送
3. Instagram DM — 通过 Mac 节点浏览器手动发送（IG API 不可用）
4. MCN 私信 — 通过机构引荐未回复的作者

**24h 跟进：**
对未回复作者发补充消息，参考 `references/outreach-playbook.md` 模版 3。

**WhatsApp 自动发送：**
读取 `references/whatsapp-auto-send.md` 执行自动化发送。核心原理：Mac 节点 + `whatsapp://` URL scheme + AppleScript 模拟回车。

**Instagram DM：IG API 自动化不可行，只能通过 Mac 浏览器手动发送。**

## 关键约束

- **Noah API 不可直接调用**：所有 Noah 操作必须通过 agent-browser
- **容器无法访问 IG/TG**：需 Mac 节点（MyFlickerClient）在线
- **电话号码默认脱敏**：需点击 `.noah-icon-eye-close` 眼睛图标才显示完整号码
- **邮箱不可获取**：Noah 不存储，巴西 Comedy 作者普遍不留邮箱
- **IG DM 不可 API 自动化**：只能浏览器手动操作
- **Mac 节点启动方式**：终端执行 `open -a MyFlicker` 启动 MyFlicker.app 即可连接
