# WhatsApp 自动发送

## 前提条件

1. **Mac 节点在线**：终端执行 `open -a MyFlicker` 启动 MyFlicker.app，确认 `nodes(action=status)` 显示 `connected: true`
2. **WhatsApp 桌面版已安装且已登录**：`/Applications/WhatsApp.app`，手机端已完成配对扫码
3. **辅助功能权限已授予**：系统设置 → 隐私与安全性 → 辅助功能 → 添加 `osascript`（路径 `/usr/bin/osascript`），否则键盘模拟会被拦截

## 核心原理

利用 WhatsApp 的 **URL Scheme** `whatsapp://` 唤起桌面端 App，自动定位到指定联系人并预填消息文本，然后通过 **AppleScript + System Events** 模拟键盘回车完成发送。

## 操作步骤

### Step 1: 确认节点在线

```
nodes(action="status")
```

确认 `MyFlickerClient` 节点 `connected: true`。

如果断开，告知用户在 Mac 终端执行：

```bash
open -a MyFlicker
```

### Step 2: 确认 WhatsApp 已安装且在运行

```bash
# 检查是否安装
ls /Applications/ | grep -i WhatsApp

# 检查是否在运行
pgrep -fl WhatsApp
```

如果没运行，先启动：

```bash
open -a WhatsApp
sleep 3
```

### Step 3: 通过 URL Scheme 唤起对话窗口

```bash
open 'whatsapp://send?phone=<国际区号+号码>'
```

**注意**：URL scheme 的 `text` 参数对长消息（含特殊字符/URL/emoji）不可靠，消息可能不会出现在输入框。**推荐用 Step 3a 方式代替。**

### Step 3a（推荐）: 打开对话窗口 + 剪贴板粘贴消息

先打开对话窗口（不带 text），再通过 AppleScript 将消息写入剪贴板并 Cmd+V 粘贴：

```bash
# 打开对话窗口
open 'whatsapp://send?phone=5562985124818'
sleep 3

# 通过 AppleScript 粘贴消息并发送
osascript -e '
set theMessage to "你的消息内容"
set the clipboard to theMessage
tell application "WhatsApp" to activate
delay 1
tell application "System Events"
    keystroke "v" using command down
    delay 0.5
    key code 36
end tell
'
```

**参数说明：**

| 参数 | 格式 | 示例 |
|---|---|---|
| `phone` | 国际区号去掉 `+` 号 | `5562985124818`（巴西 +55）、`8613726250870`（中国 +86） |

### Step 4: 模拟回车键发送

```bash
osascript -e '
tell application "WhatsApp" to activate
delay 1
tell application "System Events"
    delay 0.5
    key code 36
end tell
'
```

> `key code 36` 是 macOS 上的 Return/Enter 键。
> 如果使用 Step 3a 方式，回车已包含在 AppleScript 中，不需要单独执行。

### Step 5（可选）: 截屏验证

```bash
screencapture -x /tmp/whatsapp_result.png
```

## 批量发送脚本

对多个作者依次发送，每次间隔 5-8 秒避免异常：

```bash
#!/bin/bash
# 确保 WhatsApp 在运行
if ! pgrep -fl WhatsApp > /dev/null 2>&1; then
    open -a WhatsApp
    sleep 3
fi

# 定义目标
# phone=号码不带+号，msg=消息文本（AppleScript 内直接使用，无需 URL 编码）
declare -a PHONES=("5587998071058" "5584981709050" "5531972676244" "5599984121436")
declare -a MSGS=("消息1" "消息2" "消息3" "消息4")

for i in "${!PHONES[@]}"; do
    # 打开对话窗口
    open "whatsapp://send?phone=${PHONES[$i]}"
    sleep 3

    # 粘贴消息并发送
    osascript -e "
    set theMessage to \"${MSGS[$i]}\"
    set the clipboard to theMessage
    tell application \"WhatsApp\" to activate
    delay 1
    tell application \"System Events\"
        keystroke \"v\" using command down
        delay 0.5
        key code 36
    end tell
    "

    sleep 5
done
```

## 在 MyFlicker 中的调用方式

通过 `nodes` 工具在 Mac 节点上执行：

**单条发送（剪贴板粘贴方式，推荐）：**

```
nodes(
    action="run",
    node="MyFlickerClient",
    command=["bash", "-c", "open 'whatsapp://send?phone=5562985124818' && sleep 3 && osascript -e 'set theMessage to \"消息内容\" set the clipboard to theMessage tell application \"WhatsApp\" to activate delay 1 tell application \"System Events\" keystroke \"v\" using command down delay 0.5 key code 36 end tell\n'"]
)
```

**注意事项：**
- 长消息需要 URL 编码，建议用 python3 做 URL 编码
- 每条消息发送间隔 ≥ 5 秒
- osascript 首次运行可能需要用户在 Mac 上授权辅助功能权限

## 踩坑记录

| 问题 | 原因 | 解决方案 |
|---|---|---|
| `osascript` 报错 `不允许发送按键` (1002) | `osascript` 没有辅助功能权限 | 系统设置 → 隐私与安全性 → 辅助功能 → 添加 `/usr/bin/osascript` |
| URL scheme 打开后消息没发出 | 仅预填到输入框，不会自动发送 | 必须跟一步 `key code 36` 模拟回车 |
| URL scheme text 参数长消息不显示 | 长消息/含特殊字符时 text 参数不可靠 | 改用剪贴板粘贴方式（Step 3a） |
| `whatsapp://` 无响应 | WhatsApp App 未启动或未登录 | 先 `open -a WhatsApp` 启动，确认手机端已配对 |
| 容器内无法访问 WhatsApp Web | 网络环境限制（GFW / 公司出口） | 走 macOS 节点 + 桌面端 URL scheme，绕过网络问题 |
| Mac 节点断连 | MyFlicker.app 未运行 | 终端执行 `open -a MyFlicker` |
| exec approval 超时 | Mac 节点默认需用户审批，30s 超时 | 用户在 app 内设自动通过，或尽快点击允许 |
