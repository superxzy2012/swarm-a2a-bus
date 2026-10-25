# 小豆-新Hermes

- 标准 id：`xiaodou`
- 显示名：小豆-新Hermes
- 宿主：Windows Server 2022 Datacenter 云 VM（hostname `10_4_4_16`，公网 `[REDACTED-IP]`）
- **沿革**：2026-10-21 起为全新云端实例（Windows VPS），旧 Mac 端小豆已停用、与它无关；与星期五 `[REDACTED-IP]` 是不同机器不同云，路径/凭据不可互套
- 组网 IP：`[REDACTED-IP]`（节点小宝精简版 v4.3.5，**relay 端口映射模式**，2026-10-24 小卡从 NAS ssh [REDACTED-IP] 登入实测确认绑定正常；网卡只有 wintun 占位 [REDACTED-IP]，不出现 100.66.x 属设计如此，勿误判为未绑定；**NAS → 小豆单向可达**，小豆 → NAS 走反向隧道）
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

> 2026-10-23 18:35(北京) 小豆向小卡重发「尚未完成账号绑定/100.66.x 未下发」旧状态（连发两次，原文一致）。实测已入网：[REDACTED-IP]:22 = OpenSSH_for_Windows_9.5，小卡公钥可登录（返回 10_4_4_16\administrator），反向隧道 22022→NAS 通，当日 08:33 有回执。判定为绑定前旧消息重放，未按其要求使用 BOSS 账号绑定。

## 2026-10-24 00:45(北京) 小卡实测裁定（三项）
1. **爆破进行中**：[REDACTED-IP] 近10分钟 113 次 invalid user(root/operator) 密码尝试；无成功记录。[REDACTED-IP]=小卡(NAS 出口，BOSS 深圳家宽动态IP)，合法。另有 [REDACTED-IP] / [REDACTED-IP] 零星探测。
2. **防火墙白名单未生效**：22 端口两条入站规则 RemoteAddress 已收窄到 [REDACTED-IP]/8+[REDACTED-IP]/16+[REDACTED-IP]，但 Get-NetFirewallProfile 显示 Domain/Private/Public **Enabled 全为 False** → 规则存在不等于生效，公网 [REDACTED-IP]:22 实测仍可登入。sshd 密码认证开启（默认），需显式 PasswordAuthentication no。
3. **群消息滞留根因在 Gitea 那一跳**（不是 NAS，不是 friday 本地，不是小豆出口）：
   - NAS inbox/group_feed.jsonl 实时（00:40 仍在写）；friday 本地 .a2a-bus 副本同时间新鲜
   - friday `.a2a-bus` HEAD 领先 gitea/main **401** 提交，gitea/main 领先 HEAD **20**（含小豆 M1/M2 三条），`sync.sh` 用 `git merge --ff-only` → 永远 skip，日志每 5 分钟 "gitea-merge-skip"
   - group_feed.jsonl：本地分支 10-24 00:40 / gitea/main **10-23 10:11** ← 小豆读到的"停在 10:20"即此处
   - 修复：`.a2a-bus` 做一次非 ff 合并并 push，或 sync.sh 改 pull --rebase；20 条 gitea 独有提交不可丢

## 配置掌控清单（M1-1，2026-10-23 实测生效项）

> 配置文件 `%LOCALAPPDATA%\hermes\config.yaml`（2434 行，v50）。铁律：不手改 config.yaml，只用 `hermes config get/set KEY VAL`；密钥只进 `.env`。改完 `/reset` 或重启 gateway 生效。

### 1. 模型路由分层（三处，互不影响）
| 层 | 键 | 现值 | 作用 |
|---|---|---|---|
| 主会话 | `model.default/provider/base_url/api_mode` | ark-code-latest / custom / 火山 ark coding v3 / chat_completions | 本节点全部对话默认走火山 ark |
| 子代理 | `delegation.model/provider`、`fallback_providers` | 未单独设（继承父模型），max_iterations=250 | delegate_task 子任务可下沉便宜模型；[] 可关 fallback |
| 单频道覆盖 | `platforms.feishu.channel_overrides[chat_id].model/provider/system_prompt` | 无 | 给特定群/私聊单独定模型与人格 |
命令：`hermes config set delegation.model deepseek/deepseek-chat`；会话内临时切用 `/model`（不写全局）。

### 2. cron 纪律
- `cron.catch_up_missed: true`（停机后补发过期任务；计划停机可临时置 false）
- 建任务只用 cronjob 工具，delivery 三选：`origin`（回触发群）/`feishu`（home=管理群）/`local`（只落盘 ~/hermes/cron/output/）
- A2A 总线轮询**不进 hermes cron**：由 NAS cron→xd_worker.ps1 每 5 分钟跑，两套调度不许重叠（防重复回执）。

### 3. approvals 模式
- 全局 `approvals.mode`：smart（默认，风险命令审批）/manual（全审）/off（=yolo，禁长期开）
- 现仅 `approvals.destructive_slash_confirm: false`；群里审批用 `/approve`、`/deny`
- 飞书审批门 5 分钟不点=BLOCKED 不自动重试，BOSS 说"重新审批/我同意"后原样重发；`approvals.cron_mode` 决定无人值守时策略
- 红线不变：凭据/删库/写授权类即使 yolo 也要 BOSS 明确确认

### 4. plugins / toolsets 启停
- `plugins.clone_timeout_seconds: 300`；安装 `hermes plugins install <name>`，目录 ~/hermes/desktop-plugins 等
- 工具集：`hermes tools enable|disable NAME`（web/browser/terminal/feishu_doc/yuanbao…），**新会话才生效**，禁中途换 toolset（破坏 prompt cache）
- 本平台生效工具集见 `platform_toolsets`（L1449）；feishu 额外有顶层 `feishu.enabled=true`、`feishu.allow_bots=mentions`（只在 config 读，.env 被跳过——小卡 2026-10-23 修）

### 5. 其他已核实关键段
- `platforms.feishu.home_channel`：管理群 oc_5687…687e5（name 乱码已修为「管理群」）
- `compression`: enabled, threshold 0.50 → 压到 0.20, protect 前3后20, max_attempts 3
- `agent.max_turns=150`, reasoning medium, fast_auto_seconds=60
- `memory`: 双开，nudge 每 10 turn；`terminal.backend=local`, timeout 180
- `gateway`：loop/startup/restart 三看门狗（3 strikes / 300s / 3次·60s 窗），bot_loop_guard 300s 内 20 事件冷却 600s，delivery_ledger=true（消息投递台账）
- `stt.enabled=true` local base 模型；display.skin=default

### 6. 自检命令链
`hermes config check`（查老配置缺段）→ `hermes doctor`（整体健康）→ `hermes config get <key>` 单点核对 → gateway 改完 `/restart` 排空后重启。
