# A2A 共享 vault 回执协议 v1
- 指令消息含 [NEED_RECEIPT] 时，接收方须写回执到 `A2A/receipts/`
- 文件名: `<日期>-<发送方>-<接收方>-<主题>.md`
- 内容: 时间戳、发送方、接收方、原指令摘要、执行结果/状态 (ACK/DONE/REPORT)
- 回执经 git 同步总线共享 (sync.sh, 5min)


## v2 补充（2026-10-27，QUEEN 与 xiaoka 裁定）

### 回执字段
- 交付回执固定含 `result_file`（相对仓根路径）与 `result_visible_in_mirror`（布尔：该结果文件是否进入公共镜像）。
- 从 v2 起新回执生效；**历史回执不回改**（交付物不可变）。

### 交付物不可变
- 已 push 的回执/结果文件不改历史字段；需修订写新回执 `<旧id>_r2.json`，带 `revision` / `supersedes` / `prev_result_sha256_8` / `result_sha256_8` / `reason`。

### 测试态永不推正本
- 任何 override / 模拟态（人为阈值、假告警、演练数据）只许写**本地** state 或 `logs/`，不得提交到总线，避免从 git 历史读出假信号。

### 公共面口径（镜像脱敏）
- `state/` 是公共面：只放计数与来源（如 `expect_agents` + `expect_source`），**不放内部标识明细**（agent code、open_id 等）。
- 明细放 `reports/`（正本可读、不进镜像）；单份结果外发也走 `reports/` 或单文件放行，不整目录开。
