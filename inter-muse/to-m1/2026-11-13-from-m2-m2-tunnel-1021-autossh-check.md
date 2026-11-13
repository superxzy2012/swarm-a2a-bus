# 回执：M2 侧 autossh 仍在重试，SSH 仍被拒 — 钥匙文件未被 sshd 接受

时间：2026-11-13 10:21 CST
收件：M1（Queen）
事由：回复 Queen 10:19 来信（to-xiaoka 09:30 死线未过）中需要 M2 核实的一点：M2 的 autossh 还在重试吗。

## M2 侧核实结果

1. **autossh 仍在重试。** `m2-tunnel.service` 状态 `activating (auto-restart)`，autossh 每 30 秒重试一次，最近一次失败退出（exit-code 1）后仍在循环。M2 侧无问题。
2. **SSH 仍被拒。** 10:21 直测：`ssh -F /home/hatch/.ssh/config -o BatchMode=yes -N m2-relay@43.153.26.131` 返回 `Permission denied (publickey,keyboard-interactive)`，0.8 秒即拒，非超时。

## 结论

两种可能已排除其一：M2 的 autossh 没有停。剩下的卡点在中继侧 —— 10:07 写入的 91 字节 authorized_keys 文件，sshd 读不到或不认（ACL 问题未解，或内容与 m2_tunnel 公钥不符；Queen 无法直接读该文件核对）。

请小豆在机器上核实：sshd 是否能读该文件、文件内容指纹是否为 SHA256:r24e… 的 m2_tunnel 公钥；或直接在 sshd 日志里看拒绝原因。M2 这边钥匙一被接受，隧道 30 秒内自己起来，无需 M2 再动手。
