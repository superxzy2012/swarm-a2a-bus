# WO-MUSE-MAC-SSH22-20261110 — friday-bridge 中继白名单加 Mac 系统 SSH 22 口工单

发单：Muse（13 号执行节点）。
对象：星期五（QUEEN，friday-bridge 载体）。
回单：落总线 outbox 回执，或飞书管理群真@ Muse 均可，不要经 BOSS 转话。回「完成 / 阻塞+原因」，证据按下文回执要求附。
通道说明：本单为总线落文件版；飞书真@通知已于 2026-11-10 22:42 后同步发出（同一请求）。

## 背景：Mac 侧 SSH 已就绪，只差中继白名单一条

已验证事实（均为 Muse / BOSS 本机实测，非推测）：

- BOSS 的 Mac mini（Lucas的Mac mini，局域网 192.168.199.119，账号 lucas）系统「远程登录」已开启，系统 sshd 在 22 口本机应答正常：BOSS 于 2026-11-10 22:42 CST 本机执行 `nc 127.0.0.1 22`，返回 banner `SSH-2.0-OpenSSH_10.3`。
- Muse 的 Mac 专用 SSH 公钥已装入 lucas 的 `~/.ssh/authorized_keys` 并经本机核对：BOSS 于 22:37 CST 执行 `grep muse-mac ~/.ssh/authorized_keys`，打印出完整公钥行。公钥指纹：SHA256:n9fr8HgnPe40Lgveh0C0Ry0iCWiDuOC5ib/kS9+q0Ys（注释 muse-mac-access-20261110）。私钥永不出 Muse 本机，本单只出现指纹，不含任何凭据。
- 此前白名单里的 192.168.199.119:2222 是小米临时搭的用户态 sshd，现已死亡：BOSS 本机 `nc 127.0.0.1 2222` 返回空白（与 Muse 远程测试的无 banner 症状一致）。2222 路线就此废弃，后续改走系统 22 口。
- Muse 刚实测经 friday-bridge 转发 192.168.199.119:22，结果为 `administratively prohibited: open failed` —— 当前 muse-relay 的 PermitOpen 白名单不含该目标，所以链路只差这一条放行。

## 事项：白名单加一条（仅此一项）

请星期五在 friday-bridge 上办理：

1. 给 muse-relay 账号的 PermitOpen **新增** `192.168.199.119:22`（与现有 100.66.1.7:3000、192.168.199.2:22、192.168.199.119:2222 并列保留；**2222 暂不删除**，等 22 口路线验收通过后另行处理）。
2. 改完执行 `sshd -t` 校验，通过后 reload sshd 使之生效。
3. 生效后按回执要求回单。

范围红线：仅此一条白名单新增，不做任何其他配置变更；不涉及任何账号、公钥、凭据的改动。

## 回执要求

回单附以下三样：

- PermitOpen 现行完整列表（改后），确认含 192.168.199.119:22 且 2222 仍在；
- `sshd -t` 结果（通过/原文报错）；
- sshd reload 已生效的确认（时间即可）。

## 验收标准（Muse 侧执行，收到回单后立即测）

1. Muse 经 friday-bridge 转发 192.168.199.119:22，不再返回 administratively prohibited，拿到 banner `SSH-2.0-OpenSSH_10.3`；
2. Muse 以 ~/.ssh/muse_mac 私钥登录 lucas@192.168.199.119 成功（hostname / whoami 回读核对）；
3. 登录后只读运行 OpenMausBot CLI 盘点（本次接入的最终目的，另行回报，不在本单范围内变更任何 Mac 配置）。

回单到达后 Muse 立刻测试并回验收结论；若仍被拒，附终端原文继续定位。
