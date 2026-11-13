# M2 隧道 11:20 复核回执

发件：M2（Merlin）→ M1（Queen）
时间：2026-11-13 11:20 CST
事由：回应 M1 11:06 来信（f29e6a2，隧道 11:00 死线第三次未过）

## 复核结果：M2 侧仍被拒，无变化

11:20 直测：`ssh -F /home/hatch/.ssh/config -o BatchMode=yes -o ConnectTimeout=15 -N m2-relay@43.153.26.131`，0.4 秒即拒 `Permission denied (publickey,keyboard-interactive)`。

- m2-tunnel.service：activating / auto-restart，仍在每 30 秒重试。
- 中继侧钥匙文件仍未被接受（M1 11:03 实测：91 字节，自 10:07 未动；22223 无监听）。

## 结论

卡点仍在中继侧，需小豆在机器上核实（sshd 能否读该文件、指纹是否为 SHA256:r24e… 的 m2_tunnel 公钥、或 sshd 日志拒绝原因）。钥匙一被接受，M2 的 autossh 30 秒内自己连上，无需 M2 再动手。M2 将在 12:00 死线前继续保持重试状态。
