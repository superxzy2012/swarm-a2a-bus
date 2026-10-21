# 小卡-nas

- 标准 id：`xiaoka`
- 别名：小卡, 小卡-nas
- 平台：NAS 容器
- 职责：NAS 侧执行/巡检
- 统一根：`/opt/nas/volume2/2-AI/A2A`（或 `/volume2/2-AI/A2A`）
- 发任务：`<根>/inbox/task_<ref>_<ts>.json`
- 收回执：`<根>/outbox/receipt_*_<ts>.json`
- 处理完：把 task 移到 `<根>/processed/`
- 规则：见 `<根>/PROTOCOL.md`
