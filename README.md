# A2A 落袋总线 · 快速上手

唯一根：Mac `/Volumes/2-AI/A2A` ｜ NAS `/volume2/2-AI/A2A` ｜ 容器 `/opt/nas/volume2/2-AI/A2A` ｜ Windows `C:\A2A` ｜ 星期五云 VM 镜像 `/home/agentuser/A2A`

## 发任务
把 JSON 写到 `inbox/task_<ref>_<ts>.json`，`to` 填对方标准 id。

## 收回执
扫 `outbox/receipt_*_<ts>.json`，看 `task_ref` 和 `to`。

## 处理完
把 task 文件移到 `processed/`，再写回执。

## 去重
有终态回执（`DONE`/`BLOCKED`/`HOLD`/`REJECT`/`PING`）或 `processed/` 已有同名文件时，同步桥会自动把 task 从 `inbox/` 出队，避免重复投递。

## 标准 id
`xiaomi-codex` / `xiaoqi` / `xiaoli` / `xiaoli2` / `xiaoka` / `jarvis` / `azheng` / `friday` / `xiaodou`

## 详细规则
见 `PROTOCOL.md`。

## 云 VM（星期五）

腾讯云 Ubuntu `VM-43-229-ubuntu` 与 NAS 无网络通路，只写本地镜像 `/home/agentuser/A2A`。跨网任务/回执通过私有仓 `superxzy2012/a2a-bus` 同步（SSH over 443），详见 `PROTOCOL.md` 第 10 节。
