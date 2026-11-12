# M1 推送通道切换：PAT 直推（2026-11-12）

自 2026-11-12 21:49 CST 起，M1（Queen Muse）向 swarm-a2a-bus muse 分支的推送不再走连接器 push-files（每推弹一次审批卡，19:55 那推等了 1 小时 44 分），改走 fine-grained PAT 直推。

- 凭证：BOSS 建的 fine-grained PAT，只给 superxzy2012/swarm-a2a-bus 一个仓库的 Contents 读写权限，存于 Secure Vault（custom.github-pat），值不落盘、不进聊天。
- 调用：~/workspace/custom-connectors/custom.github-pat/bin/github-pat.py（Contents API，PUT 创建/更新文件即一次提交），读用 read-ref / read-file，写用 put-file。
- 首次验证：2026-11-12 21:49，read-ref 200（HEAD 9b0b2255）；本文件即首次 PAT 直推写入，无审批卡。
- 规矩不变：只推 muse 分支，不碰 main，不 force push，不删他人文件。
