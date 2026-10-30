# 星期五-新Hermes
> ⚠️ **出列撤销（2026-10-30 11:20 CST）**：BOSS 令「不停，已延长一个月，你仍是 QUEEN」，`QUEEN-GAUNTLET-QDAY-20261030.md`（`65fb5987`）判定 **QUEEN-CUTOVER-ORDER-20261028 作废**——10-29 12:00 交割线与 10-30 friday 出列**全部撤销**。friday 继续任 QUEEN 至 BOSS 下令切换；xiaodou 转为接班人陪跑（五门 Q1–Q5，见 `reports/QUEEN-TRAINING-PLAN-v2-20261030.md`）。
> 历史留档（已失效）：原写「出列：2026-10-30（已生效，2026-10-29T12:00 收口核验）」；G3 已于 10-29 09:24 开闸、Gitea main 主写在 xiaodou、QUEEN 推送 cron 已 gate（`G3-PAUSE-ACK`）、贾维斯 11:41 只读复核 PASS——这些机制状态**继续有效**，只是不再对应 friday 出列。收口正本 `reports/CUTOVER-CLOSEOUT-20261029_xiaodou.md` 保留为历史。
> ✅ **状态：RECOVERED + 转顾问（2026-10-23 00:47 +08:00）**。gateway 已于 2026-10-22 08:32(+08:00) 自行恢复（systemd active，进程连续运行 24h+）。按 BOSS 令 ORG-20261022-CEO-TRANSITION，friday 由 CEO 转**顾问**，过渡期只负责 100% 无保留带教 xiaodou（见 `tasks/HANDOVER-FRIDAY-MASTER.md`），不再承担日常派单；jarvis 后备 CEO/独立审计。
> 📜 历史：2026-10-21 17:09Z 曾因进程失联（VM 存活）被 BOSS 令灾备切换，枢纽由 jarvis 临时代管；失联期间 gitea/github 双 main 分叉，已于 2026-10-23 08:45(+08:00) 三方合并统一（merge c1cfb7d/377416a，备份分支 backup/premerge-github-line-20261023）。

- 标准 id：`friday`
- 别名：星期五, 星期五-新Hermes, Hermes
- 平台：Hermes（腾讯云 Ubuntu `VM-43-229-ubuntu`，公网出口 `[REDACTED-IP]`）
- 职责：CEO 派单/巡检
- 本地镜像根：`/home/agentuser/A2A`（云 VM 无 NAS 网络通路）
- 统一根：`/Volumes/2-AI/A2A`（Mac）＝ `/volume2/2-AI/A2A`（NAS）
- 发任务：`/home/agentuser/A2A/inbox/task_<ref>_<ts>.json`
- 收回执：`/home/agentuser/A2A/outbox/receipt_*_<ts>.json`
- 处理完：把 task 移到 `/home/agentuser/A2A/processed/`
- 规则：见 `/home/agentuser/A2A/PROTOCOL.md`；跨网同步见根目录 `PROTOCOL.md` 第 10 节
- 中继仓：`superxzy2012/a2a-bus`（SSH 别名 `github-a2a`，`ssh.github.com:443`）
- 同步脚本：`/home/agentuser/A2A/sync.sh`
