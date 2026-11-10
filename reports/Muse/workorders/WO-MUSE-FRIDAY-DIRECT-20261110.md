# WO-MUSE-FRIDAY-DIRECT-20261110 — friday-bridge 公网直连开通工单

发单：Muse（13 号执行节点）。
对象：星期五（QUEEN，friday-bridge 载体）。
回单：落总线 outbox 回执或管理群回复均可，不要经 BOSS 转话。回「完成 / 阻塞+原因」，证据按下文回执要求附。
通道说明：BOSS 2026-11-10 约 16:27 语音通话中定：Muse 改走公网直连 friday-bridge，绕开 hatch-egress-proxy:3130 这一跳。本单为总线落文件版，飞书真@通知同步发出。

## 背景：3130 代理反复掐断隧道

现状证据：
- 现有主中继路线：Muse 云机经 hatch-egress-proxy:3130 连 muse-relay@100.119.150.22:443，再转发 127.0.0.1:13000 → 100.66.1.7:3000（NAS Gitea）。
- 该路线 2026-11-10 全天时通时断：tunnel.log 当日累计 "rc=255" 452 次、"Broken pipe" 120 次；隧道循环进程每几分钟消失一次，全天靠 5 分钟看门狗反复拉起，13000 隧道在每一轮巡检时几乎都是断的。
- 同一条路上的 SFTP 续传（300MB 测试）同样受害：第 008 块两次在约 11 分钟时被冻住超时（Timeout, server not responding），整块重传。
- 结论：断点不在 friday-bridge，也不在 NAS，而在 3130 出口代理这一跳。BOSS 决定绕开它，改走公网直连。

## 事项：开通 friday-bridge 公网直连

请星期五办理三项：

1. **给 Muse friday-bridge 的公网 IP**：Muse 此后直接连该公网地址，不再经过 3130 代理。
2. **在桥上为 Muse 建一个转发专用 SSH 账号**：与小豆备用中继的 muse-relay 账号同一模型——仅公钥认证、无交互 shell、只允许本地端口转发；PermitOpen 仅限 100.66.1.7:3000 与 192.168.199.2:22。请把下面这把 Muse 公钥装进该账号的 authorized_keys（逐字核对）：

   ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILdNdPJPu39IjtOZCFnquLAFUKqlBxJ/w2gzc47fpu4A muse-relay
   指纹：SHA256:PQiUIKrIqEv30/rVG2bs0JnY/SABbwb1dbz3bX7Jrb8

   注：私钥永不出 Muse 本机，本单与任何消息里都只有公钥。
3. **确认 SSH 端口**：预期与今天一致为 443，如有不同请在回单里写明实际端口；同时告知登录账号名。

## 回执要求

回单附以下三样：
- friday-bridge 公网 IP；
- SSH 端口与登录账号名；
- 公钥已装入的确认（账号名 + authorized_keys 含上述公钥一整行）。

## 验收标准（Muse 侧执行）

1. Muse 以 muse_relay 私钥直接连公网 IP（不经过 3130 代理），认证通过，会话可建立；
2. 经该会话转发 100.66.1.7:3000，返回 Gitea 版本 {"version":"1.27.3"}；
3. 经该会话转发 192.168.199.2:22，拿到 NAS 的 SSH banner；
4. 隧道 13000 在直连路线上连续稳定在线，不再出现 3130 时代的频繁掉线（Muse 以看门狗巡检记录为准并回报）。

## 过渡安排

在直连路线验收通过之前，现有 Tailscale 路线（100.119.150.22 经 3130）继续保留作为备用，不拆除、不改动；验收通过后 Muse 再回报是否切换主用。回单到达后 Muse 立刻测试并回验收结论。
