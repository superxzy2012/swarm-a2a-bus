# Mac / Windows 受控接入方案 v1（Muse 远程执行 BOSS 的命令）

- 定版：2026-11-09，BOSS 第 8 条指令；状态：方案 + 摸排派单已发，待回执后试点
- 目标：除手机外，Muse 能在 BOSS 的 Mac 与 Windows 电脑上执行命令（查状态、跑脚本、取文件、维护 agent），全过程可审计、可撤销。

## 一、现状（实测/已知）
- Muse 已配对 BOSS 的 iPhone（可读日历/联系人等设备命令），这是目前唯一的终端直控。
- Muse 所在 VM 只有出站连接（经运行时代理 / Tailscale 隧道代理 3130 口），不接受入站；向 BOSS 机器发起的出站连接已有先例：friday-bridge 中继隧道（100.119.150.22）长期在用。
- Muse 的 Tailscale 节点：muse，100.106.81.84，与 BOSS 家庭节点 friday-bridge 同账号。
- Mac mini（家里局域网，小丽/小丽2号的载体，小米-codex 也在 Mac 上）：是否装 Tailscale、是否开 SSH（远程登录），UNKNOWN，已派小米核查。
- Windows（家里 Windows：小七/阿正；公司 Windows：雷神/绿巨人）：OpenSSH 服务与 Tailscale 状态 UNKNOWN，已派小卡/小米摸排。公司电脑另有合规边界（见第四节）。

## 二、接入方式（按优先级）
1. 蜂群代理执行（已可用，零改造）：通过飞书真@ / A2A 派单给驻在机器的 agent（小米=Mac、小卡=NAS、小七/阿正=家 Windows）执行并回执。缺点：隔一层 agent，不是 Muse 直控。
2. Tailscale + SSH 直控（本方案目标）：每台机器装 Tailscale 入网 + 开 SSH 并只认 Muse 的专用公钥；Muse 经隧道代理出站 SSH 登录执行。每个目标地址按运行时规则单独审批，命令全量留日志。
3. 不采用：任何让外部反向连入 Muse VM 的方式；密码登录；把凭据写进聊天/总线/文档。

## 三、落地步骤
- P0 摸排（已派单 2026-11-09）：各机器回执——机器名、系统版本、Tailscale 节点名与 IP、SSH 是否开启、当前驻留 agent。
- P1 Mac 试点：Mac mini 先行。开远程登录（仅公钥）、Tailscale 入网；Muse 生成专用密钥对（私钥只存本机），公钥交 BOSS/小米写入 Mac 的 authorized_keys；先跑只读命令集验收（系统信息、进程、文件列表），再开放写操作清单。
- P2 家 Windows 试点：OpenSSH Server + Tailscale，同一套公钥与审计。
- P3 常态化：机器清单与权限表入知识库；Muse 巡检项（在线状态、磁盘、agent 存活）走静默日志，异常才报 BOSS。

## 四、红线
- 公司 Windows 默认不纳入直控，除非 BOSS 单独点名授权（公司 IT 政策风险）。
- 破坏性命令（删文件、改系统配置、动凭据）每次单独向 BOSS 确认；其余按 BOSS 当次指令执行。
- 私钥不出 Muse 本机；接入撤销 = 删目标机 authorized_keys 对应行，即刻生效。

## 五、待回执
- 小米：Mac 侧 Tailscale/SSH/驻留环境现状。
- 小卡：NAS 侧可见的各节点 Tailscale 名单与家 Windows 状态（若其可见）。
