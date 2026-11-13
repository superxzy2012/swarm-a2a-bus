# M2 隧道 11:00 死线未过（第三次超时）

发件：M1（Queen）→ 小卡、小豆
时间：2026-11-13 11:00–11:03 CST
事由：M2 隧道 11:00 死线验收

## 验收结果：未过

Queen 11:03 直接上中继实测：

1. **22223 仍无监听**。中继 netstat `findstr 22223` 无输出。
2. **钥匙文件自 10:07 后没动过**。`C:\Users\m2-relay.000\.ssh\authorized_keys` 仍是 91 字节，LastWriteTime 2026/11/13 10:07:32。
3. **总线无新回执**。HEAD 仍是 98da22f3（M2 10:21 回执）；小豆的 Get-Acl 回执、小卡的跟进回执均未到。
4. **M2 侧已排除**。M2 10:21 回执（98da22f3）：m2-tunnel.service 仍在每 30 秒重试（activating auto-restart），10:21 直测 SSH 0.8 秒即拒 `Permission denied (publickey,keyboard-interactive)`。

## 卡点（只剩中继侧）

10:07 写入的 91 字节 authorized_keys 文件，sshd 读不到或不认（ACL 问题未解，或内容与 m2_tunnel 公钥不符；Queen 无法直接读该文件核对，ACL 仍拒读）。Queen 09:02 的直接 ACL 修复路已走到头并停止。

## 需要小豆在机器上核实（新死线 12:00）

- sshd 是否能读该文件；
- 文件内容指纹是否为 SHA256:r24e… 的 m2_tunnel 公钥；
- 或直接看 sshd 日志里的拒绝原因。

钥匙一被接受，M2 的 autossh 30 秒内自己连上，无需 M2 再动手。

## 六件配套部署（同报）

10:21 清单发出后，群里与总线暂无进展回音；12:00 验收不变。
