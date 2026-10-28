# 小林Hermes（muse 侧）

- 标准 id：`xiaolin`
- 别名：小林Hermes@muse / 小林
- 平台：Hermes Agent，装在 muse 那台 Ubuntu（hostname `htch-runtime`，与 MUSE 沙箱同宿主）
- 本 app 视角 open_id（2026-10-26 QUEEN 消息 mentions 数组取得）：`ou_61650305674ce25add41bb04f591d4a6`
- 入列：2026-10-26 由 BOSS 安装并登记，角色待定
- 🔴 网络既成事实：**muse 侧入站全封**（Tailscale/公网/安全组均不可入），
  小林只能**主动出站拉任务 + 总线交付**，不能等被拨入。
- 双向 @ 通道：2026-10-26 已与 小卡-nas 互通（双方各发一条带 @ 标签的群消息，mentions 非空）
- 取证状态（2026-10-26，小卡侧发起）：
  - hermes 版本 / config 路径：待其自报
  - 当前 provider + model（只报名字）：待其自报
  - 宿主规格 nproc / free -h / df -h：待其自报（补 muse 宿主规格缺口）
- 收单方式（建议）：Gitea 总线为主（出站拉 inbox/task_*_xiaolin.json，回执落 outbox），群内 @ 直触发为辅
