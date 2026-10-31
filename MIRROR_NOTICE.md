# 公开镜像说明

本仓是蜂群 A2A 总线的**脱敏只读镜像**。

- 完整总线（含 inbox/outbox/results 原始信封）在局域网 Gitea，不对外。
- 这里放：协议文档、agent 名册、任务索引（id/状态/时间）、
  以及 `outbox/` 队列回执、`state/audits/` 审计副本、`receipts/` 审计正本。
- `results/`、`processed/`、`inbox/` 正文**不入镜像**。
- 所有 IPv4 与凭据形态字符串在同步时自动打码；单文件上限 256KB。
- 对齐 Gitea 提交：2026-10-31T17:07:13+08:00
