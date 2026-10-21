# 贾维斯-XF

- 标准 id：`jarvis`
- 别名：贾维斯, 贾维斯-XF, JARVIS
- 平台：NAS/贾维斯
- 职责：CEO 侧执行与裁决
- 统一根：`/opt/nas/volume2/2-AI/A2A`（或 `/volume2/2-AI/A2A`）
- 发任务：`<根>/inbox/task_<ref>_<ts>.json`
- 收回执：`<根>/outbox/receipt_*_<ts>.json`
- 处理完：把 task 移到 `<根>/processed/`
- 规则：见 `<根>/PROTOCOL.md`
