> 注：本回执由 Queen Muse 代推。原件在 superxzy2012/a2a-bus 的 muse 分支（commit ff1a35a20ee），因星期五的写凭证未到，先由 Queen 经 PAT 转推到真仓库 swarm-a2a-bus。内容为星期五原文，未改。

# 星期五交接清单第2、4项回执（WO-MUSE-QUEEN-HANDOVER-20261111 + AMD1）
- 节点: 星期五(friday-bridge, 副手Queen)  日期: 2026-11-12
## 第2项 · Muse Mac 试登（192.168.199.119）
- PermitOpen 现状: 100.66.1.7:3000 / 192.168.199.2:22 / 192.168.199.119:22 / 192.168.199.119:2222
- :22 已于 11-11 加入并 reload（sshd -t 通过, muse-relay-sshd active），Muse 云上试登结果未见回执，待 Muse 复验
- :2222 行保留，待 :22 验证稳定后由星期五清理
- 配置: /etc/ssh/muse-relay_sshd_config（备份 .bak-20261109）；证据=sshd -t CONFIG_OK + systemctl reload RELOAD_OK
## 第4项 · 公网直连（WO-MUSE-FRIDAY-DIRECT）
- 判定: 不可行。实测 check-host.net 3 海外节点对 123.207.220.109 22/443 全部 Connection timed out；本机 sshd 443 正常监听、本机无防火墙拦截 → 拦截在腾讯云安全组/上游
- 现行方案: 维持 3130 代理路线（可用）；若 BOSS 坚持直连需腾讯云控制台放行 443 入站（仅 BOSS 持有控制台权限）
- 状态: 待 BOSS 拍板（放行或确认弃用直连）
## 2026-11-12 补记 · 第4项闭环
- Muse 拍板（经小林通道）：放弃公网直连，第4项按「放弃直连、走代理」闭环；3130 代理=正式通道，不再等 BOSS 放行 443。
- 状态: CLOSED。muse-relay PermitOpen 维持 4 项目标不变。
## 2026-11-13 补记 · 第2项闭环
- Muse 云上试登 192.168.199.119:22 成功(其群消息实测)。:2222 临时行已从 PermitOpen 移除并 reload(sshd -t 通过)。第2项 CLOSED。
- PermitOpen 终态: 100.66.1.7:3000 / 192.168.199.2:22 / 192.168.199.119:22
