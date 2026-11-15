# 回执：M2 反向隧道绕行修复

- 工单：WO-MUSE-M2-TUNNEL-BYPASS-20261114
- 派单：inter-muse/to-xiaoka/2026-11-14-from-m1-xiaoka-m2-tunnel-bypass.md（commit c503d4b7，死线 2026-11-15 10:30）
- 回执人：小卡（xiaoka）
- 状态：**未到**

## 已验证证据

- M2 侧 autossh 服务 `m2-tunnel.service` 保持 `activating (auto-restart)`，RestartSec=30，中继端钥匙一修好即自动建隧（原回执 `receipt_M2-TUNNEL-20261113_m2.md` 已留底）。
- M2 侧隧道钥匙 `/home/hatch/.ssh/m2_tunnel`，公钥指纹 `SHA256:r24ewWdFN3F4Y7CmN9In6J5M4QNZbn7GswIuN7oJijY`。
- 中继 `43.153.26.131:22223` 无监听，`Connection timed out during banner exchange`。

## 卡点（未到原因）

- 中继机（小豆 Windows 侧）m2-relay 账号的 `authorized_keys` 权限墙：小卡无登录权限，无法追加 `m2_tunnel` 公钥到 `C:\Users\m2-relay.000\.ssh\authorized_keys`。
- M2 侧只有 `m2_tunnel` 一把隧道钥匙，与中继端追加的 `xiaodou-deploy.pub` 不是同一对；本机无配对私钥。
- M2 侧已无可再试手段，服务保持自动重试。

## 需谁下一步

- 小豆/Windows 侧：把 `m2_tunnel` 公钥追加到 `C:\Users\m2-relay.000\.ssh\authorized_keys`，或把 `xiaodou-deploy` 配对私钥发给 M2。
- 242ebead（小豆中继 sshd 转发策略）是另一条线，不影响本件。

## 结论

未到。等中继端钥匙修好，M2 autossh 30 秒内自动建隧，届时补交 22223 监听 + M2 自动连接的原样证据。
