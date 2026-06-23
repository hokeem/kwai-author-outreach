# Error Handling — kwai-author-outreach

## 错误码速查表

| error_code | 动作 | 说明 |
|-----------|------|------|
| `NOAH_PAGE_TIMEOUT` | 刷新页面重试 1 次 | Noah 页面加载超时 |
| `EYE_ICON_NOT_FOUND` | 跳过电话，标记"脱敏" | 页面无眼睛图标 |
| `PHONE_NOT_REVEALED` | 重试点击眼睛图标 | 点击后号码仍脱敏 |
| `MAC_NODE_OFFLINE` | 终端执行 `open -a MyFlicker` | MyFlicker.app 未运行 |
| `IG_RATE_LIMITED` | 等待 30 秒重试 | IG 请求频率限制 |
| `TG_NOT_REGISTERED` | 标记"Telegram 不可用" | 号码未注册 Telegram |
| `NETWORK_ERROR` | 提示检查网络 | 网络不可达 |
| `BROWSER_SSO_EXPIRED` | 引导用户重新登录 Noah | SSO 过期 |
| `IG_DM_NOT_AUTO` | 改用浏览器手动发 DM | IG API 自动化不可行 |
| `WA_SEND_FAILED` | 检查 WhatsApp 运行状态 + osascript 辅助功能权限 | WhatsApp URL scheme 发送失败 |

## AI 执行流程

```
脚本/操作执行
   ↓
成功？
   ├── 是 → 继续下一步
   └── 否 → 在本表查找对应 error_code
                ↓
            执行对应动作
```
