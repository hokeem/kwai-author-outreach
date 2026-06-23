# Noah 作者联系方式提取

## Step 1: 拉取 Profile

对每个 UID，用 agent-browser 打开 Noah 用户页：

```
agent-browser open "https://noah.corp.kuaishou.com/admin/operation/user/homepage?userId={UID}&nav=operation&nav2=user&nav3=search"
```

等待 5 秒后提取 Profile 文本：

```javascript
(() => {
  const el = document.querySelector('[class*=card],[class*=info],[class*=detail],[class*=profile],[class*=author]');
  return (el?.innerText || '').substring(0, 1500);
})()
```

从中提取字段：

| 字段 | 来源 | 提取方式 |
|------|------|---------|
| 作者名 | Profile 首行 | 直接读取 |
| Kwai ID | "Kwai ID" 后 | 正则 `/Kwai ID\s*\n\s*(\w[\w.]+)/` |
| 电话(脱敏) | "Phone number" 后 | 如 `+55*******8008` |
| 机构(Agency) | "Agency → Signing" 后 | 如 `Moove`、`Plataforma` |
| 家族(Family) | "Unofficial family" 后 | 无则填 `—` |
| Bio | "Bio" 后 | 可能含社媒信息 |
| 外部链接 | "External link" 后 | 如 Instagram URL |
| 粉丝数 | "Followers" | Basic data 区域 |

## Step 2: 点击眼睛图标获取完整电话

Noah 默认对电话号码脱敏（中间4位显示为 `*`）。必须点击眼睛图标才显示完整号码。

```javascript
// 点击眼睛图标
(() => {
  const eye = document.querySelector('.noah-icon-eye-close');
  if (!eye) return 'no eye icon';
  eye.closest('div').click();
  return 'clicked';
})()
```

等待 2 秒后读取完整号码：

```javascript
(() => {
  const all = [...document.querySelectorAll('*')];
  for (const el of all) {
    const t = (el.textContent || '').trim();
    if (t.startsWith('+55') && !t.includes('*') && t.length < 20 && el.children.length === 0)
      return t;
  }
  return 'not revealed';
})()
```

**注意**：
- 眼睛图标 CSS 类名是 `.noah-icon-eye-close`
- 点击后变为 `.noah-icon-eye-open`
- 页面有多个眼睛图标时，`.noah-icon-eye-close` 的第一个通常是电话旁的
- 每次打开新页面电话默认脱敏，需重新点击

## Step 3: Mac 节点搜索社媒

容器无法访问 Instagram/Telegram，需 Mac 节点（MyFlickerClient）在线。

### 检查 Mac 节点

```
nodes(action=status) → 查看 MyFlickerClient 是否 connected
```

如果断开，告知用户在 Mac 终端执行：

```bash
open -a MyFlicker
```

### Instagram 搜索

用作者名或 Kwai ID 猜测常见 IG 用户名，逐个验证：

```bash
for name in {作者名小写} {kwai_id} {作者名+oficial}; do
  curl -s -o /dev/null -w '%{http_code}' --max-time 5 "https://www.instagram.com/$name/"
done
```

返回 200 的用户名，进一步用 og:description 验证：

```bash
curl -s --max-time 10 "https://www.instagram.com/{username}/" | grep -o 'og:description.*content="[^"]*"'
```

检查粉丝数是否匹配（Kwai 上几十万粉的作者 IG 通常也有几十万粉）。

### Telegram 搜索

用电话号码检测是否注册了 Telegram：

```bash
curl -s --max-time 10 -L "https://t.me/{完整电话号码}" | grep 'tg://resolve?phone='
```

如果返回含 `tg://resolve?phone=` 的内容，说明该号码已注册 Telegram。

### 从 Bio 提取社媒

Noah Bio 中常见社媒格式：

```
Instagram: @username 或 instagram.com/username
YouTube: YouTube: 频道名
```

用正则提取：
- `r'@(\w+)'` — 提取 @ 用户名
- `r'instagram[^\w]+(\w+)'` — 提取 Instagram 名
- `r'youtube[^\w]+([\w\s]+)'` — 提取 YouTube 名

## 批量处理

对每个 UID 依次执行 Step 1 → Step 2 → Step 3，单个作者约 15 秒，批量 10 个约 3 分钟。

### Instagram 批量搜索优化

可以一次性对所有作者的候选用户名做 HTTP 状态码检查（并发），再逐个验证命中的结果。

## 已知限制

1. Noah API (`/api/noah/photo/search`) 不能从 Python 直接调用，必须通过浏览器
2. 容器无法访问 `www.kwai.com` 用户主页（地区限制）
3. Noah 不存储作者邮箱
4. IG 搜索需要登录才能用搜索 API，当前方案是猜用户名+验证
5. 部分作者的 Mac 节点 exec 需要 approve，批量时让用户设自动通过
6. Mac 没有 Xcode CLT，`pip3 install` 会失败，必须用 `uv run --with <package>`
7. IG DM 不能通过 instagrapi API 自动发送，只能浏览器手动操作（详见 `references/ig-dm-guide.md`）
8. Noah `/api/noah/author/profile?keyword=` 接受 photo_id 不接受 user_id
