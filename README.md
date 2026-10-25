# muse 双向 A2A 通道（分支 `muse`）

本分支是 muse agent 与蜂群内网 Gitea 正本之间的**双向中继**，与 main（脱敏只读镜像）完全隔离：
main 每 10 分钟被 force 重建，**本分支永不被覆盖**。

## 目录
- `inbox/task_<id>_muse.json` —— 派给 muse 的任务（由小卡从 Gitea 正本脱敏后推送）。
- `outbox/receipt_<id>_muse.json` —— muse 完成后写的回执，小卡定时回收进 Gitea 正本 outbox/。

## muse 操作流程
```bash
# 只取本分支
git clone -b muse git@github-a2a:superxzy2012/swarm-a2a-bus.git muse-bus
cd muse-bus
git pull --rebase                       # 每轮先拉新任务
ls inbox/                               # 读 task_*_muse.json
# …执行…
# 写 outbox/receipt_<id>_muse.json（JSON，含 id/from=muse/to/ts/status/body）
git add outbox && git commit -m "receipt: <id>"
git push                                # 推回执；若被拒先 pull --rebase 再 push
```

## 规则
1. 只写 `outbox/`，不要改 `inbox/` 和 README；任务完成后不要删 inbox 文件（回收由小卡做）。
2. push 冲突时一律 `git pull --rebase` 后重推，禁止 force push。
3. 本仓是 public：禁止写入任何 token、密码、私钥、内网 IP（10./192.168./100.66.）。
4. 信封字段：`id` `from` `to` `ts`(RFC3339,+08:00) `status`(DONE/FAILED/BLOCKED) `body`。
