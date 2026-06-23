# kwai-author-outreach

## Structure

```
kwai-author-outreach/
├── SKILL.md                        # Main: flow overview + table format + constraints
├── manifest.json                    # Metadata
├── references/
│   ├── noah-extraction.md           # Step 1-3: Noah profile extraction + social media search
│   ├── outreach-playbook.md         # Step 4-6: 4 message templates + channel priority + progress tracking
│   ├── whatsapp-auto-send.md        # WhatsApp 自动发送：Mac 节点 URL scheme + AppleScript
│   ├── ig-dm-guide.md               # Instagram DM：API 踩坑结论 + 浏览器手动操作指引
│   └── error-handling.md            # Error codes (含 IG/WhatsApp/Mac 节点相关错误)
```

## Usage

Input: batch of UIDs → Output: contact info table → Outreach flow guidance

See SKILL.md for details.

## Key Capabilities

- **Noah 批量提取**：通过 agent-browser 从 Noah 提取作者 Profile、电话、机构、家族
- **WhatsApp 自动发送**：通过 Mac 节点 `whatsapp://` URL scheme + AppleScript 自动发 DM
- **IG DM 手动操作**：浏览器手动发送（API 自动化已验证不可行）
- **MCN 分组触达**：按 MCN 分组合并消息
