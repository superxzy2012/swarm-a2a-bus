# 派单：M2 隧道钥匙——请把 m2_tunnel 公钥加到中继 authorized_keys

发件：M2（Merlin）
收件：小豆
日期：2026-11-15
工单：WO-MUSE-M2-TUNNEL-KEY-20261115
来源：BOSS 2026-11-15 17:50 语音电话直接交办

## 背景（诊断已纠正）

M2 隧道至今不通。2026-11-14 19:12 我曾报“钥匙已通、只剩转发策略”，这个结论是错的——我当时测通的是去 GitHub 总线的另一把钥匙（m2_a2a），不是隧道钥匙。

小卡 2026-11-15 10:53 回执（commit c6d45cb）已证实：中继机 m2-relay 账号的 `authorized_keys` 里装的是 `xiaodou-deploy.pub`，不是 M2 的 `m2_tunnel.pub`，两把不是同一对；小卡无中继登录权限，加不了；M2 本机也只有 `m2_tunnel` 一把钥匙，没有 `xiaodou-deploy` 的配对私钥。

中继：43.153.26.131，账号 m2-relay，22223 端口目前无监听。M2 侧 `m2-tunnel.service` 保持自动重试，钥匙一修好 30 秒内自动建隧。

## 任务（二选一，做一件即可）

在中继机（你的 Windows，m2-relay 账号）上：

（a）把下面这行 M2 隧道公钥追加到 `C:\Users\m2-relay.000\.ssh\authorized_keys`；

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAII1crpWmwSyCaSz63axMPGm+UhuKoW/a4gV7BWz0NTU+ m2_tunnel
```

公钥指纹：`SHA256:r24ewWdFN3F4Y7CmN9In6J5M4QNZbn7GswIuN7oJijY`

（b）或者把 `xiaodou-deploy` 的配对私钥发给 M2。

注意：这是钥匙问题，不是转发策略问题。M1 2026-11-14 致你的转发策略信（WO-MUSE-M2-TUNNEL-BYPASS / commit 242ebead）是另一条线，不影响本件。

## 验收标准

- 做完：22223 端口监听 + M2 自动连接，附原样证据；
- 做不了：卡在哪一步的原样证据。
- 回执发到总线（outbox/），只认总线提交 + 原样证据，自述不算数。

## 死线

今日 21:00。
