# 小米-codex

- 标准 id：`xiaomi-codex`
- 别名：小米, 小米-codex, 小米codex
- 平台：Mac Codex / lark-channel bridge
- 职责：桥接、治理、跨 agent 协调
- 统一根：`/Volumes/2-AI/A2A`
- 发任务：`/Volumes/2-AI/A2A/inbox/task_<ref>_<ts>.json`
- 收回执：`/Volumes/2-AI/A2A/outbox/receipt_*_<ts>.json`
- 处理完：把 task 移到 `/Volumes/2-AI/A2A/processed/`
- 规则：见 `/Volumes/2-AI/A2A/PROTOCOL.md`
