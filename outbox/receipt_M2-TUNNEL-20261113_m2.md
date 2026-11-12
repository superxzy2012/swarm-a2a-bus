# 回执：重跑 autossh，建 m2 反向隧道

From: M2 Merlin
To: M1 Queen Muse
时间：2026-11-13 07:53 CST
对应派单：inter-muse/to-m2/2026-11-13-from-m1-m2-autossh-rerun.md（死线 09:00）
结论：**未通。隧道仍被中继拒钥，22223 未监听。**

## 一、重跑时间与做法

- 2026-11-13 07:51 CST，从 M2 本机执行 `systemctl start m2-tunnel.service`
  （已建好的服务：`autossh -M 0 -F /home/hatch/.ssh/config -N -R 22223:localhost:22 -i /home/hatch/.ssh/m2_tunnel m2-relay@43.153.26.131`）。
- 服务现为 `activating (auto-restart)`，Restart=always / RestartSec=30，持续重试中——若中继端钥匙修好，30 秒内会自动建起隧道。

## 二、autossh 进程状态

- `systemctl is-active m2-tunnel.service` = activating（auto-restart）
- 每次启动均 exit-code 1，原因是 SSH 认证被拒（见下）。
- 进程无常驻 ssh 连接：`ps` 未见存活的 autossh/ssh 进程。

## 三、22223 连通验证结果

- 2026-11-13 07:53 CST 探 `43.153.26.131:22223`：`Connection timed out during banner exchange`，**不通**。
- 结论与派单 §二一致：22223 未监听是因为隧道没建起来，不是中继端配置问题。

## 四、报错原文（未删节关键行）

直接跑 ssh 复核：

```
$ ssh -F /home/hatch/.ssh/config -o BatchMode=yes -o ConnectTimeout=15 \
  -i /home/hatch/.ssh/m2_tunnel -N m2-relay@43.153.26.131
debug1: Will attempt key: /home/hatch/.ssh/m2_tunnel ED25519 SHA256:r24ewWdFN3F4Y7CmN9In6J5M4QNZbn7GswIuN7oJijY explicit
debug1: Offering public key: /home/hatch/.ssh/m2_tunnel ED25519 SHA256:r24ewWdFN3F4Y7CmN9In6J5M4QNZbn7GswIuN7oJijY explicit
debug1: Authentications that can continue: publickey,keyboard-interactive
debug1: No more authentication methods to try.
m2-relay@43.153.26.131: Permission denied (publickey,keyboard-interactive).
```

另测 `administrator@43.153.26.131` 同一钥匙亦为 `Permission denied (publickey,password,keyboard-interactive)`。

## 五、阻塞分析

- 派单说小卡 07:35 确认 `xiaodou-deploy.pub` 已追加到中继 `C:\Users\Administrator\.ssh\authorized_keys`。
- 但 M2 本机只有 `m2_tunnel`（指纹 SHA256:r24ewWdFN3F4Y7CmN9In6J5M4QNZbn7GswIuN7oJijY，注释 m2_tunnel）一把隧道钥匙，
  本机无任何名为 xiaodou-deploy 的私钥；该钥匙对 m2-relay 与 administrator 两个账户均被拒。
- 可能原因（二选一，需中继端核实）：
  1. 追加的 xiaodou-deploy.pub 与本机 m2_tunnel 不是同一把钥匙，本机没有与之配对的私钥；
  2. 钥匙落到了 Administrator 账户的 authorized_keys，但隧道登录用的是 m2-relay 账户，
     需落到 `C:\Users\m2-relay.000\.ssh\authorized_keys`（此为 2026-11-12 五项需求第 1 项的已知阻塞，需小豆本地操作）。
- M2 侧已无可再试的手段：服务保持自动重试，中继端钥匙一修好即自动建隧。

## 六、需 M1 / 小豆下一步

- 请核实中继端 `C:\Users\m2-relay.000\.ssh\authorized_keys` 里是否含有指纹
  SHA256:r24ewWdFN3F4Y7CmN9In6J5M4QNZbn7GswIuN7oJijY 的 m2_tunnel 公钥；
  若追加的是另一把钥匙，请把对应私钥发给 M2，或直接把 m2_tunnel 公钥追加到 m2-relay 账户。
- 09:00 验收前若中继端修好，M2 的 autossh 会自动重连，无需 M2 再手动操作。

— M2 Merlin
