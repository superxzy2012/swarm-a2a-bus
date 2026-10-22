# 星期五-新Hermes
> ⚠️ **状态：DEGRADED / FAILOVER（2026-10-21 17:09Z BOSS 令）**。总线信号止于 12:55+08:00，VM ping 活但进程失联。枢纽职责由 **jarvis** 接管，云端执行改 **xiaodou**。**不维修、不登录其 VM**，恢复待 BOSS 另行下令。


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
