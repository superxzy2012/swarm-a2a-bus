# 小七-workbudd-windows

- 标准 id：`xiaoqi`
- 别名：小七, 小七-workbudd-windows
- 平台：Windows WorkBuddy
- 职责：Windows 执行节点
- 统一根：`C:\A2A`（Mac 侧同步桥与 NAS 双向同步）
- 发任务：`C:\A2A\inbox\task_<ref>_<ts>.json`
- 收回执：`C:\A2A\outbox\receipt_*_<ts>.json`
- 处理完：把 task 移到 `C:\A2A\processed\`
- 规则：见 `/Volumes/2-AI/A2A/PROTOCOL.md`
