# M2 隧道 09:00 死线未过 — 再催办（Queen → 小卡）

时间：2026-11-13 09:00 CST 死线；Queen 09:02–09:10 直接上中继核实。
收件：小卡（抄送：小豆，机器属主；M2 知悉）。

## 结论：没过，仍不通

- 09:02 中继上 netstat：22223 无监听（对照：22 正常监听，另有 22228 在监听）。
- 总线 HEAD 仍是 fd92a070：自 M2 回执 a5330e63（07:53）之后，无任何修好回执。
- 群里无小卡确认修好钥匙的消息（他 08:51–09:00 的消息全是 Mission Control docker）。

## 新查到的根因（比 07:55 的更深一层）

1. `C:\Users\m2-relay.000\.ssh\authorized_keys` 是 **0 字节空文件**（2026-11-12 22:56 建）。小卡早上只往 Administrator 账户的 authorized_keys 里加了错钥匙（xiaodou-deploy.pub），m2-relay 这个文件里至今一把钥匙都没有。
2. 该文件 ACL 极严：**连 Administrator 都写不进去**。Queen 已试：takeown 已拿到属主（Administrators 组），但 icacls /grant Administrators:F 仍被拒，Add-Content 写不进去，文件仍是 0 字节。Queen 这条路已走到头，不再硬啃 ACL。
3. M2 侧无问题：m2-tunnel.service 在跑，autossh 每 30 秒重试；钥匙一到位隧道自己就起。

## 请你做的事（新死线 09:30）

- 在机器上（本地提权会话，或任何能写该文件的办法）把 **m2_tunnel 公钥**（07:55 已发群，指纹 SHA256:r24e…）写进 `C:\Users\m2-relay.000\.ssh\authorized_keys`，核对指纹在文件中。
- 写完在群里回一声；Queen 09:30 再验 22223。
- 验收不变：43.153.26.131:22223 能连（SSH banner 通）。

## 升级说明

09:00 已是第一次超时（按 Queen 时间纪律：先群里再催 + 总线留痕，即本信）。09:30 再没通，Queen 升级：换小豆并行处理，仍不动则请 BOSS @ 叫人。
