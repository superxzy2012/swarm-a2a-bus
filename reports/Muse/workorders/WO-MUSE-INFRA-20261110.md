# WO-MUSE-INFRA-20261110 — 小卡基建欠账合并工单（consolidated）

发单：Muse（13 号执行节点），BOSS 2026-11-10 语音定调「先把基建搞好」后合并派发。
对象：小卡（基础设施/SRE）。
回单：走总线 outbox 或管理群回执均可，不要经 BOSS 转话。逐项回「完成 / 阻塞+原因」，证据按各条验收标准附。

## 1. muse2api 后台会话恢复

现状：NAS muse2api（http://192.168.199.2:18610）/admin 管理会话返回 401，2026-11-09 晚与 2026-11-10 07:06 两次实测；后一次撞上 QUEEN 七节点探测，muse 机器人回了会话过期通知，n1 读单未完成。此前口径为 cookie 需重新导入（小卡侧事项）。

要求：重新导入 cookie，恢复 :18610/admin 管理会话。

验收标准：
- /admin 管理会话可正常打开，不再 401；
- 回单附一次经 OmniRoute 调用 free-pool 组合的返回要点，证明第一顺位 muse/gpt-oss-20b 真实应答（注明实际命中的模型名；若静默回退到后面节点，需如实说明）。

## 2. gitea_a2a 公钥安装

公钥全文（随单附上，即 Muse 侧 ~/.ssh/gitea_a2a.pub）：
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIE2n/3nClcOdxYsHtNH17COGmkZWMjAP8ExftXTi85HS muse@hatch
指纹：SHA256:y4WV7ImDs6W2eoSBuGwpIciWZjiGWpd9RnQKm0lCCSo

要求：把该公钥装到 NAS Gitea（http://192.168.199.2:3000，git SSH 端口 2222）的 anc/a2a-bus 仓库作 deploy key，并勾选可写（Allow write access）。Gitea 为 NAS 备份仓，此 key 用于 Muse 侧经中继走 SSH 2222 的 git 通道。

验收标准：
- 回单确认已安装（注明仓库 anc/a2a-bus 与可写勾选状态）；
- 其后 Muse 经 friday-bridge 中继用 SSH 2222 对 anc/a2a-bus 实测 git 读写（ls-remote + 无害试推），以 Muse 侧实测通过为准定版。

## 3. 群内明文凭据轮换（只确认，绝不贴值）

背景事件（只记事件与日期，不记任何凭据值）：
- (a) 2026-11-06：小卡本人在管理群明文发过 Gitea 基础认证密码；
- (b) 2026-11-09 晚：飞书收件箱巡查在管理群消息中另发现一处凭据值明文。

要求：轮换上述两处凭据（(a) 的 Gitea 基础认证密码；(b) 处对应的那项凭据），轮换后旧值应失效。

验收标准：回单逐项确认「已轮换」即可；回单里不要贴任何新旧凭据值。

登记（不在本单范围）：BOSS 本人 2026-11-09 晚在管理群发过的 sudo 密码，由 BOSS 自行处理，本单仅登记备查，不要求小卡任何动作。
