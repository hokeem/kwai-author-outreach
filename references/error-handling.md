# Error Handling — kwai-author-outreach

## 错误码速查表

| error_code | 动作 | 说明 |
|-----------|------|------|
| `NOAH_PAGE_TIMEOUT` | 刷新页面重试 1 次 | Noah 页面加载超时 |
| `EYE_ICON_NOT_FOUND` | 跳过电话，标记"脱敏" | 页面无眼睛图标 |
| `PHONE_NOT_REVEALED` | 重试点击眼睛图标 | 点击后号码仍脱敏 |
| `MAC_NODE_OFFLINE` | 停止社媒搜索，告知用户连接 Mac | MyFlickerClient 断开 |
| `IG_RATE_LIMITED` | 等待 30 秒重试 | IG 请求频率限制 |
| `TG_NOT_REGISTERED` | 标记"Telegram 不可用" | 号码未注册 Telegram |
| `NETWORK_ERROR` | 提示检查网络 | 网络不可达 |
| `BROWSER_SSO_EXPIRED` | 引导用户重新登录 Noah | SSO 过期 |
| `IG_LOGIN_REQUIRED` | 改用浏览器手动发 DM | instagrapi sessionid 权限不足，无法发 DM |
| `IG_CHALLENGE_REQUIRED` | 无法自动绕过，告知用户手动操作 | 新设备触发验证，手机批准后 API 仍不通过 |
| `IG_DM_SILENT_FAIL` | 改用浏览器手动发 DM | requests+cookies 返回 200 但消息未发送 |
| `MAC_PIP3_FAILED` | 改用 `uv run --with <pkg>` | Mac 无 Xcode CLT，pip3 编译失败 |
| `MAC_EXEC_TIMEOUT` | 拆分长脚本为短命令，或让用户尽快点允许 | Mac 节点 exec approval 30s 超时 |
| `MAC_NODE_OFFLINE` | 终端执行 `open -a MyFlicker` | MyFlicker.app 未运行导致节点断连 |

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
