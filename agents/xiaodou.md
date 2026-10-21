# 小豆-新Hermes

- 标准 id：`xiaodou`
- 显示名：小豆-新Hermes
- 宿主：Windows Server 2022 Datacenter 云 VM（hostname `10_4_4_16`，公网 `[REDACTED-IP]`）
- 组网 IP：`[REDACTED-IP]`（节点小宝；**NAS → 小豆 单向可达**，小豆 → NAS 走反向隧道）
- 本机 A2A 根：`C:\A2A\a2a-bus`
- 状态：**已入蜂群，自动领任务**（2026-10-21 验证：投递 → 执行 → 回执全链路通）

## 通道

| 通道 | 地址 | 能力 |
|---|---|---|
| Gitea（主总线） | `http://[REDACTED-IP]:23000`（SSH 反向隧道 → NAS:3000） | **读写**，push 用 `arthur` |
| GitHub（公开镜像） | `https://github.com/superxzy2012/swarm-a2a-bus.git` | **只读**（SSH 到 GitHub 被墙，HTTPS clone 正常）；`C:\A2A\gh-mirror` |
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

## 投递方式

```
/opt/data/.venv-browser/bin/python /opt/data/bin/a2a_dispatch.py \
  --to xiaodou --instruction "任务说明" --cmd "要执行的命令"
```
