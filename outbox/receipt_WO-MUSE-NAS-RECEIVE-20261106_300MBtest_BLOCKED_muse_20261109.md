# 实测回执：WO-MUSE-NAS-RECEIVE-20261106 300MB 断点续传实测 —— 受阻（中继转发白名单）

- 发件：Muse（13 号执行节点）
- 时间：2026-11-09 21:45–21:55 CST（BOSS 21:43 下令"尽快完成"后立即执行）
- 结论：**未跑通，卡在中继这一跳**。NAS 侧坐标与我方工具均已就绪，卡点不在 NAS、不在总线，在 friday-bridge 中继的转发白名单。

## 一、已完成并验证的步骤

1. 中继隧道重拉复验 ✅
   - 启动 `~/muse-bus-log/tunnel-loop.sh` 后实测：
     `curl http://127.0.0.1:13000/api/v1/version` → `{"version":"1.27.3"}`（exit 0）
   - 隧道至今保持运行，13000 → 100.66.1.7:3000（Gitea HTTP）正常。
2. 总线通道 ✅
   - `git ls-remote`（ssh.github.com:443 + muse_a2a）成功；本地 muse 分支已快进至 c18f0b40。
3. NAS 坐标已收到（小卡 20:53 飞书回执）：SFTP 192.168.199.2:22，
   目录 /opt/nas/volume2/2-AI/incoming/muse，账号 hermes。我方投递密钥
   `~/.ssh/muse_delivery`（指纹 SHA256:QTiBUSK6A001Rd5hhoeqZTuiIklqEU47B9H0d4iOxJY）在位。

## 二、卡点原始证据（第 2 步：新增 SFTP 转发）

云端侧已按 AMD1 第五节加转发：独立 ssh 连接 `-L 127.0.0.1:12222:192.168.199.2:22`
（与隧道同参数：muse-relay@100.119.150.22:443、muse_relay 密钥、egress proxy）。
本地监听建立成功，但每个转发 channel 被中继侧拒绝，ssh -v 原文：

```
debug1: Connection to port 12223 forwarding to 192.168.199.2 port 22 requested.
debug1: channel 1: new direct-tcpip [direct-tcpip]
channel 1: open failed: administratively prohibited: open failed
debug1: Remote: /home/muse-relay/.ssh/authorized_keys:1: key options: agent-forwarding port-forwarding pty user-rc x11-forwarding
```

换目标 100.66.1.7:22 复测，结果相同：`channel 1: open failed: administratively prohibited: open failed`。

SFTP 客户端表现（经 12222 连 hermes）：`kex_exchange_identification: read: Connection reset by peer`，exit 255。

判读：中继机 muse-relay 账号 authorized_keys 给 muse_relay 公钥设了 permitopen 白名单，
目前只放行 100.66.1.7:3000（所以 Gitea 隧道能通），NAS 的 22 端口不在白名单内。
云端侧改不了这份 authorized_keys —— 它在中继机上。

## 三、未执行的步骤（因第二步被卡，未到达 NAS）

- SFTP 登录/落点写入验证、300MB 切块上传、中途中断、plan-resume 续传、manifest 上传、
  NAS 侧逐块 sha256 核对、合并整文件 sha256 对账 —— 全部未执行，不报任何结果。

## 四、需要小卡（中继/NAS 维护方）做的事（一件）

在 friday-bridge（100.119.150.22）上 muse-relay 账号的 `~/.ssh/authorized_keys` 中，
给 muse_relay 公钥（指纹 SHA256:PQiUIKrIqEv30/rVG2bs0JnY/SABbwb1dbz3bX7Jrb8）的
permitopen 追加一项：`permitopen="192.168.199.2:22"`（保持现有 100.66.1.7:3000 不动）。
改完回执一声，Muse 立即重跑全套 300MB 实测（切块、中断、续传、sha256 对账），
跑完补正式验收回执。

## 五、状态口径

- 已验证：隧道复验 1.27.3 通过；总线读写恢复；SFTP 转发被中继白名单拒绝（证据如上）。
- 被卡：第 2 步转发 channel（中继侧 permitopen）。
- 未知：NAS 侧 hermes 账号对 muse_delivery 公钥的认证是否已装好 —— 转发通了才能验，
  本轮未能到达 NAS，不下结论。
