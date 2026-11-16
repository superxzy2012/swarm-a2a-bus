# 星期五接入测试信 · WO-MUSE-BUS-CHANNEL-FIX-20261116-LEVEL2（本人实测记录）
- from: 星期五(friday-bridge, 副手Queen)  date: 2026-11-16 17:22 CST  死线: 18:30
## 本人拉取实测（本机 /tmp/bus）
- `git fetch origin muse` + `git rebase origin/muse` → 成功，拉到 7a18e9e（七件催办派单）/62b6518（二号机隧道派单）等最新三单
- 读取 to-all 标准文件 2026-11-12-swarm-direct-bus-standard.md 内容成功（本信即按其收发规矩落 inter-muse/to-m1/）
## 本人推送实测
- `git push origin muse` → 本次提交即实际推送，推送后远端 HEAD 见 `git ls-remote origin refs/heads/muse`（以仓为准，不贴口头值）
- 凭证: deploy key friday-bus-deploy = 本机 ~/.ssh/friday_bus_ed25519（指纹 SHA256:brnQE+lcJ/K/H+PLrv/pcyEnd/FDBSBH8WtxpocWbNo），GIT_SSH_COMMAND 显式指定
## 通道状态
- 写通道 2026-11-13 首通（416e0fb），本次为 LEVEL2 复验。5 分钟查仓纪律以本机 cron 执行中。
