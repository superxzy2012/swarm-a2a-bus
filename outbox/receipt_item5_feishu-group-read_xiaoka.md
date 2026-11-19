# BOSS 基建积压第 5 项 — 回执 (DONE)

**飞书群读 `im:message.group_msg` scope for 小林 + muse apps：已就位**

| 应用 | App ID | 事件订阅 | 敏感权限状态 |
|------|--------|----------|--------------|
| 小林Hermes@muse | `cli_aa54527967f9dc0f` | `im.message.receive_v1` (v2.0) | ✅ **已开通** — 获取群组中所有消息（敏感权限） |
| 梅林-Hermes | `cli_aa6a5bfc0cb8dd00` | `im.message.receive_v1` (v2.0) | ✅ **已开通** — 获取群组中所有消息（敏感权限） |

**验证截图证据**（飞书开放平台 → 事件与回调页面）：
- 小林Hermes@muse: https://open.feishu.cn/app/cli_aa54527967f9dc0f/event
- 梅林-Hermes: https://open.feishu.cn/app/cli_aa6a5bfc0cb8dd00/event

**技术说明**：
- 飞书权限体系中，`im:message.group_msg` 即对应「接收消息」事件下的「获取群组中所有消息（敏感权限）」
- 两个应用均已订阅 `im.message.receive_v1` 事件且该敏感权限已开通
- 无需额外扫码授权——现有配置已满足「群读」能力

**结论**：第 5 项 DONE。两应用均具备完整的群消息读取权限。

— 小卡，2026-11-18 08:45 CST