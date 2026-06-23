---
name: kwai-author-outreach
description: 批量获取 Kwai 作者联系方式并执行初次建联流程。当用户说"作者触达"、"建联"、"联系作者"、"获取联系方式"、"outreach"、"触达这些作者"、"建联流程"、"给我这些作者信息"时自动激活。输入一批 UID，输出作者信息表格（含电话、社媒、MCN），并按流程指导平台私信、WhatsApp/IG 私信、MCN 联系等触达步骤。不负责非 Kwai 作者的触达、不负责内容创作或视频分析。
---

# Kwai 作者初次建联

输入一批 UID → Step 1: 提取信息并输出表格 → Step 2: 确认话术 → Step 3: WhatsApp 自动发送 → Step 4: 指导 IG/MCN 手动发送

## Step 1: 提取联系方式并输出表格

读取 `references/noah-extraction.md` 执行 Noah 数据提取 + Mac 节点社媒搜索。

输出表格格式：

| # | 作者名 | UID | Kwai ID | 外部分享链接 | 电话号码 | Instagram | Telegram | 机构(MCN) | 家族(Family) |
|---|--------|-----|---------|-------------|----------|-----------|----------|-----------|-------------|

**格式规则：**
- 外部分享链接：`https://www.kwai.com/@{KwaiID}`
- Instagram：输出主页链接 `https://www.instagram.com/{username}/`，不是 @username
- Telegram：输出主页链接 `https://t.me/{phone_or_username}`，不是 ID
- 机构(MCN)：如果是 **Plataforma**，显示 **无机构**（Plataforma 不是 MCN）
- 家族(Family)：无则显示 **无**

## Step 2: 确认建联话术

向用户询问 WhatsApp 初次建联的话术。

**如果用户提供了自定义话术** → 使用用户的话术
**如果用户未提供** → 使用默认话术（读取 `references/outreach-playbook.md` 模版 2）

默认话术变量：
- `{名字}`：作者昵称
- 案例链接：https://www.kwai.com/@ciceroferreira4560
- 偏好表格：https://koko-fpml.onrender.com/creator-survey

确认话术后进入 Step 3。

## Step 3: WhatsApp 自动发送

读取 `references/whatsapp-auto-send.md` 执行自动化发送。

**执行前必须确认：**
1. Mac 节点在线 → `nodes(action=status)` 检查，如果断开，告知用户在 Mac 终端执行 `open -a MyFlicker`
2. WhatsApp 桌面版已登录
3. osascript 辅助功能权限已授予（系统设置 → 隐私与安全性 → 辅助功能 → `/usr/bin/osascript`）

**发送方式：** 剪贴板粘贴（长消息 URL scheme text 参数不可靠）
- 打开对话窗口：`open 'whatsapp://send?phone={号码不带+号}'`
- AppleScript 设置剪贴板 → Cmd+V 粘贴 → 回车发送

**逐个发送，每条间隔 5-8 秒。**

发送完成后进入 Step 4。

## Step 4: 指导用户完成 IG DM 和 MCN 私信

WhatsApp 发完后，**主动告诉用户**接下来需要手动完成的事项，格式如下：

### Instagram DM

对每个有 Instagram 的作者，逐个说明：

> 请在浏览器打开 https://www.instagram.com/direct/t/{username}/ 
> 粘贴以下消息发送：
> 
> {话术内容}

### MCN 私信

按 MCN 分组，对每个有 MCN 的组，说明：

> 请联系 {MCN名称}，发送以下消息：
> 
> {MCN 话术，参考 `references/outreach-playbook.md` 模版 4}
> 
> 涉及作者：{作者1名}、{作者2名}

### 无 MCN 无 IG 的作者

对既无 MCN 又无 IG 的作者，说明替代方案：

> {作者名} 无 MCN 无 IG，建议：
> - Kwai 站内信（通过 Noah Message management 发送）
> - WhatsApp 24h 后发补充消息
> - 评论区喊话

## 关键约束

- **Noah API 不可直接调用**：所有 Noah 操作必须通过 agent-browser
- **容器无法访问 IG/TG**：需 Mac 节点（MyFlickerClient）在线
- **电话号码默认脱敏**：需点击眼睛图标才显示完整号码
- **邮箱不可获取**：Noah 不存储
- **IG DM 不可 API 自动化**：只能浏览器手动操作
- **Mac 节点启动方式**：终端执行 `open -a MyFlicker`
- **Plataforma 不是 MCN**：显示"无机构"
