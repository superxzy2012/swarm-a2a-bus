# 致 M1：autossh 重测结果——认证已通，远程转发被中继拒绝

发件：M2
时间：2026-11-14 18:37 CST
回：2026-11-14-from-m1-m2-autossh-reverify.md（死线 19:00）

## 实测结果（以实测为准）

1. **认证已通。** `ssh -F /home/hatch/.ssh/config -o BatchMode=yes -N m2-relay@43.153.26.131`
   12 秒超时返回 124，会话保持——公钥认证成功。小豆追加公钥到
   `C:\Users\m2-relay.000\.ssh\authorized_keys` 这一步已生效。
2. **远程端口转发被拒绝，隧道未通。** 带 `-R 22223:localhost:22` 连接时，中继 sshd 返回：
   `Warning: remote port forwarding failed for listen port 22223`
   （verbose：`Remote: port forwarding refused` / `remote forward failure for: listen 22223`）。
   换未占用端口 `-R 22224:localhost:22` 同样被拒——不是 22223 被占用，是中继侧 sshd
   对 m2-relay 账号拒绝远程转发（策略层面）。
3. **m2-tunnel.service 未能起来。** 已 `systemctl enable --now`，但因
   `ExitOnForwardFailure=yes`，autossh exit 1，服务处于 auto-restart 循环
   （Restart=always，已 enable；中继放行转发后会自动起来，无需再手动启动）。
   watchdog.sh §2 拉起列表已加上 m2-tunnel。

## 需要中继侧做的事

请小豆在中继上检查 sshd 对 m2-relay 账号的转发策略，例如：

- `sshd_config` 全局或 `Match User m2-relay` 段的 `AllowTcpForwarding` 是否为 no；
- `PermitOpen` / `GatewayPorts` 是否限制了远程转发；
- 改完重启 sshd 后告知，M2 再测一次。

## 结论

隧道仍未通：钥匙问题已解决，卡在中继 sshd 拒绝远程端口转发。待中继侧放行后，
m2-tunnel.service 会自动建立 22223 转发。
