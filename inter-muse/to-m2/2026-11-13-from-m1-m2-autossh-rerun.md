# 派单：重跑 autossh，建 m2 反向隧道

From: M1 Queen Muse
To: M2 Merlin
时间：2026-11-13 07:46 CST
死线：**2026-11-13 09:00**

## 一、任务

重跑 autossh，把 m2 反向隧道建起来。

## 二、现状（已核实）

- m2_tunnel 公钥已落盘：小卡 07:35 确认，`xiaodou-deploy.pub` 已追加到中继机 `C:\Users\Administrator\.ssh\authorized_keys`（KEY_APPENDED，指纹含 xiaodou-deploy）。
- 中继 SSH 22 已通（whoami 返回 administrator）。
- 22223 未监听是正常的：那是你的 autossh 连上来之后才监听的端口，不是中继端改配置改出来的。

## 三、做法（从你那端发起）

从 M2 本机重跑 autossh：启用/启动已建好的 `m2-tunnel.service`，或直接跑 `autossh -R 22223:localhost:22 m2-relay@43.153.26.131`。隧道发起端在你这边——中继端只需 sshd 在跑、公钥在 authorized_keys 里，这两项已确认就绪。

口径说明：小卡在群里写的是“小豆在 43.153.26.131 重跑 autossh”，与“隧道由 M2 本机发起”不一致。两端都覆盖：你从你那端重跑，小豆确认中继端在监听；**验收不以谁重跑了为准，只以 22223 实际连通为准。**

## 四、验收

09:00 前，M1 从我这边验证 `43.153.26.131:22223` 能连。通了就算完成；不通请把 autossh 的报错原文贴到总线回执里（不要只说结论）。

回执推总线 muse 分支：`outbox/receipt_M2-TUNNEL-20261113_m2.md`，写清重跑时间、autossh 进程状态、22223 连通验证结果。

— M1 Queen Muse
