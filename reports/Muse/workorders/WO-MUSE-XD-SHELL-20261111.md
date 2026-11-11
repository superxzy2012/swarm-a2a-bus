# WO-MUSE-XD-SHELL-20261111 — 给 Queen 在小豆 VPS 上开 shell 检查权限（BOSS 2026-11-11 12:45 亲令）

## 背景（Queen 实测）
- muse-relay@43.153.26.131 认证通过、退出码 0，但命令零输出——该账号只有端口转发、无 shell。Queen 查不了小豆的 Hermes 进程，每次都要绕人，不行。

## 要小卡办的（与小豆失联排查同一趟进箱，一并做）
1. 在小豆 VPS（43.153.26.131）上给 Queen 开一个**可 shell 登录**的账号（建议新账号 muse-xd；**不要放宽 muse-relay 转发账号的限制**，中继安全口径不变）。
2. 安装 Queen 公钥（指纹 SHA256:PQiUIKrIqEv30/rVG2bs0JnY/SABbwb1dbz3bX7Jrb8）：
   ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILdNdPJPu39IjtOZCFnquLAFUKqlBxJ/w2gzc47fpu4A root@htch-runtime
3. 权限范围：查看进程/服务状态与日志、查看内存、重启小豆的 Hermes 进程。不给改中继转发与 WireGuard 配置的权限。
4. 注意上次 authorized_keys 属主坑（BUILTIN\Administrators 属主会被 sshd strict mode 拒）：属主设为新账号本人，改完重启 sshd 前先自验一遍。
5. 回执：账号名 + 你侧自验结果；Queen 收到后亲自 SSH 实测（echo + 进程列表）验收。

时限：今日 14:00 前。卡住回原文。
