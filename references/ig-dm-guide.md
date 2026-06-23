# Instagram DM 发送指南

## 结论：IG DM 无法通过 API 自动化，只能浏览器手动操作

经过完整测试，以下方案均失败：

| 方案 | 结果 | 原因 |
|------|------|------|
| instagrapi `login_by_sessionid` | ✅ 可以查用户信息 | sessionid 有只读权限 |
| instagrapi `login_by_sessionid` + `direct_send` | ❌ `login_required` | sessionid 权限不足以发 DM |
| instagrapi `login()` + 密码 | ❌ `ChallengeRequired` | 新设备触发验证，手机批准后仍不生效 |
| instagrapi `direct_message(msg, [uid])` | ❌ 参数传反，消息文本被当 thread ID | instagrapi API 签名在不同版本有变化 |
| requests + Chrome cookies → `/api/v1/direct_v2/threads/broadcast/text/` | ❌ 返回 200 但实际未发送 | IG 检测到非浏览器请求，静默丢弃 |
| instagrapi `login_by_sessionid` + 密码 login（同设备） | ❌ 仍然 `login_required` | 设备配置复用无法绕过权限检查 |

**根因**：Instagram 对第三方客户端发 DM 限制严格。sessionid 仅授权只读操作，写入操作（发消息）需要完整的浏览器 session，而 API 调用会被 IG 检测并拦截。

## 可行方案：Mac 浏览器手动操作

### 操作步骤

1. 在 Mac Chrome 中打开 Instagram（确保已登录）
2. 逐个打开 DM 页面：`https://www.instagram.com/direct/t/{username}/`
3. 在消息输入框粘贴话术内容
4. 点击发送

### 话术内容

参考 `references/outreach-playbook.md` 模版 2，所有 IG DM 末尾统一加：

> Me adiciona no WhatsApp: +86 13726250870 — posso te explicar melhor por lá! 👈

### 已确认的 IG 用户名

| 作者 | IG 用户名 | IG User ID |
|------|-----------|------------|
| jeomix | @jeomixoficial | 77479397349 |
| MyrellaeVictor | @myrellaevictor01 | 59244884915 |
| Goió | @goio_oficial | 71828927944 |
| Mairton | @mairton_baturite | 29052257751 |

## instagrapi 可用的操作

虽然发 DM 不可行，instagrapi + Chrome sessionid 可用于：

- 查询用户信息：`cl.user_info_by_username(username)` → 获取 user_id、粉丝数等
- 查询用户 ID：用于构造浏览器 DM URL

```python
import browser_cookie3
from instagrapi import Client

cj = browser_cookie3.chrome(domain_name='.instagram.com')
cookies = {c.name: c.value for c in cj}
cl = Client()
cl.login_by_sessionid(cookies['sessionid'])

# 查询用户信息（可用）
user = cl.user_info_by_username('jeomixoficial')
print(user.pk, user.follower_count)
```

## 运行方式（Mac 节点）

```bash
source ~/.local/bin/env && uv run --with instagrapi --with browser-cookie3 python3 <script.py>
```

**注意**：Mac 没有 Xcode CLT，`pip3 install` 会失败，必须用 `uv run`。
