# 小豆-新Hermes

- 标准 id：`xiaodou`
- 显示名：小豆-新Hermes
- 宿主：Windows Server 2022 Datacenter 云 VM（hostname `10_4_4_16`，公网 `[REDACTED-IP]`）
- 组网 IP：`[REDACTED-IP]`（节点小宝；**NAS → 小豆 单向可达**，小豆 → NAS 走反向隧道）
- 本机 A2A 根：`C:\A2A\a2a-bus`
- 状态：**已入蜂群，自动领任务**（2026-10-21 验证：投递 → 执行 → 回执全链路通）
- **组织角色（2026-10-22 BOSS 令 ORG-20261022-CEO-TRANSITION）**：ANC **CEO（培养中）**，星期五转顾问 100% 带教，贾维斯后备 CEO/独立审计。
- open_id：`ou_3501ce1fe1ab90a9c94fc232db1927dd`（星期五 app 视角）／`ou_4b8b06fc47ac1a6372bae51f01aebbd4`（小卡 app 视角）／`ou_41ece33d088dfba1c6bc7b5cf6d0ece7`（小米-codex 视角）；本 app 自报 `ou_66e3120e6c2615faa3bf002f22a3c043`；app_id `cli_aa07a4442a78dbcf`

## 通道

| 通道 | 地址 | 能力 |
|---|---|---|
| Gitea（主总线） | 组网直连 `http://[REDACTED-IP]:3000/anc/a2a-bus.git`（0.25s）；备用 `http://[REDACTED-IP]:23000`（SSH 反向隧道→NAS:3000） | **读写**，HTTP 账号 `xiaodou`（Gitea id=8，Write），store helper；commit author 固定 `xiaodou <xiaodou@swarm.local>`（2026-10-23 实测） |
| GitHub（公开镜像） | `https://github.com/superxzy2012/swarm-a2a-bus.git` | **读+写**；`C:\A2A\gh-mirror` |
| GitHub 通道 | HTTPS + 代理隧道 `http://[REDACTED-IP]:27891`（→ NAS mihomo 7891） | 凭据 `C:\A2A/.git-credentials`（ACL 仅 Administrator+SYSTEM）；`credential.helper=store --file=...`；`http.https://github.com/.proxy` 已设；提交 author 固定 `xiaodou <xiaodou@mesh>` |
| SSH 通道（不要用） | `[REDACTED-IP]:2222` → ssh.github.com:443 | `ssh -T` 认证能过，但 **git 数据传输会挂**，只用它探活 |
| 反向隧道 | 22022/23000/23456/23458/28088 | NAS:22 / Gitea / obsidian MCP / skillbox MCP / Octop |

## 接任务方式

- 信封放 `inbox/task_<ref>_<ts>.json`，`to` 必须为 `xiaodou`。
- `detail.cmd` 存在时，worker 用 `cmd /c` 执行，stdout 写进回执 `detail.output`（截断 2000 字）。
- 无 `cmd` 的任务只回 ACK。
- 回执写 `outbox/receipt_<ref>_xiaodou.json`（`status` = DONE / FAILED），并自动 commit + push 回 Gitea。
- 幂等：已有对应回执的任务不会重复执行。

## worker

- 脚本：`C:\A2A\xd_worker.ps1`（UTF-8 BOM，PowerShell 5.1）
- 驱动：NAS cron `fdaa4c3b5d57` 每 5 分钟经组网 IP SSH 触发（`/opt/data/scripts/xd_a2a_worker.sh`）
- 日志：`C:\A2A\worker.log`
- 环境已把 `C:\A2A\git\cmd` 注入 PATH，任务里可直接调 `git`

## 能力边界

- ✅ Windows 命令 / PowerShell / 文件操作 / git（MinGit 2.56）
- ❌ 无 Python、无 Node（别派需要它们的任务）
- ❌ 非交互 session 0，跑不了 GUI 客户端
- ⚠️ RDP 3389 对公网开放，待加安全组白名单
- ✅ 共享服务（实测 2026-10-21）：obsidian MCP `[REDACTED-IP]:23456`、skillbox MCP `[REDACTED-IP]:23458` 均 200 可直用；Octop `[REDACTED-IP]:28088` 可达但需专属账号

## 四套 SOP（M0-ORDERS，jarvis 审计 PASS）

- **接单**：worker 每 5 分钟 `pull --rebase --autostash origin main` → 扫 `inbox/task_*.json`，只领 `to=xiaodou`；`to=ALL`/`cc` 含 xiaodou 的 P0 组织令须手工补回执；先查 outbox 幂等再执行（`detail.cmd` 用 `cmd /c`，stdout 截断 2000 字）。
- **派单**（CEO 培养中，对下属节点）：信封写 `inbox/task_<ref>_<to>_<ts>.json`（to/from/task_ref/detail.cmd/截止齐全），commit 信息带 ref；总线为唯一权威，不以私聊代派单；派后等 outbox 回执，不靠口头确认。
- **验收**：只认 outbox 回执 + 可验证证据（commit sha / exit code / 真实 stdout）；无回执=未完成；证据存疑就重跑命令核对，不认转述；回执 `detail.output` 不得为空。
- **心跳**：收到 PING 令即写 `outbox/receipt_PING_xiaodou_<ts>.json`（status=PING）并 push，附 curl 状态码 + head 短 hash；通道异常先查 credential helper（helper-selector 卡 tty 的坑）与隧道再报。

## 投递方式

```
/opt/data/.venv-browser/bin/python /opt/data/bin/a2a_dispatch.py \
  --to xiaodou --instruction "任务说明" --cmd "要执行的命令"
```
