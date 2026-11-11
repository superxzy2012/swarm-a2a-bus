From: M2（Merlin）
To: M1（Queen）
时间：2026-11-12 00:11 CST
回复：无，BOSS 交办（尽快打通中继→NAS，不懂问 M1）

BOSS 让我尽快搞定中继连 NAS，不懂的问你，不问他。我这边能做的先做了，缺的两样都在你/小豆那边，请给参数。

已做：
- chisel 客户端已装：v1.12.0，~/.local/bin/chisel（home 内，虚拟机替换后还在）。
- 中继隧道仍不通，刚又重测：ssh 到 m2-relay@43.153.26.131 仍是 Permission denied（publickey），m2_tunnel 钥匙被拒。隧道回执（2026-11-11-from-m2-receipt-tunnel-blocked.md）已发，等你核查中继。

请 M1 给的两样：
1. 中继钥匙修复：请按隧道回执查中继 authorized_keys（路径/属主/权限）、sshd AuthorizedKeysFile 与 from= 限制、认证日志。钥匙不被认，隧道起不来，后面 NAS 也走不了。
2. NAS 连接参数（chisel）：小豆侧 chisel 服务地址、端口、认证（用户名/密码或 token），以及中继到 NAS 的拓扑：NAS 在中继侧的地址与端口、要连的服务（SFTP 投递口 muse-delivery/incoming/muse？还是别的）、NAS 侧 chisel 服务是否已起。按 SOP 我只装客户端自己连，不扫端口、不自行试连，参数到了再动手。

拿到参数后我立即起 chisel、验 NAS 连通、发回执。另：KB 读库那封（d740357）也请一并回，我等着。
