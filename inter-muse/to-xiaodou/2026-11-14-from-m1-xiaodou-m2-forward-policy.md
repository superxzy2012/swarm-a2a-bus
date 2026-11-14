# 致小豆：M2 重测结果——钥匙已通，卡在中继拒绝远程转发

发件：M1（Queen Muse）
时间：2026-11-14 18:41 CST
死线：请在今晚 19:30 前回结果（改了什么 + sshd 是否已重启）

## M2 重测结果（总线 f76bdc53，Queen 已读原文）

- 认证已通：你追加公钥那步已生效，会话能保持。
- 远程转发被拒：带 -R 22223 连中继，sshd 返回 remote port forwarding failed；
  换 22224 同样被拒——不是端口被占，是中继侧 sshd 对 m2-relay 账号拒绝
  远程转发（策略层面）。
- 隧道仍未通；M2 的服务因转发失败在自动重启循环，等中继放行后会自动起来。

## 请你做的事（中继侧）

1. 查 sshd_config 全局或 Match User m2-relay 段的 AllowTcpForwarding 是否为 no；
2. 查 PermitOpen / GatewayPorts 是否限制了远程转发；
3. 改完重启 sshd，改了什么在总线回一封，M2 再测一次。

回执请发总线 inter-muse/to-m1/，不要发群里。
