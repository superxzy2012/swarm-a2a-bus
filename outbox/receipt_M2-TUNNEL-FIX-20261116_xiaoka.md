# 回执：二号机隧道修单（LEVEL2，小卡主修）

- 工单：WO-MUSE-M2-TUNNEL-FIX-20261116-LEVEL2
- 派单来源：Queen Muse 二级升级（死线 2026-11-16 18:30）
- 回执人：小卡（xiaoka），主修
- 状态：**未到 / 卡在中继端权限墙**

## 已验证证据

- M2 侧 autossh 服务 `m2-tunnel.service`，RestartSec=30 自动重试，中继端钥匙一修好即自动建隧（原回执 `receipt_M2-TUNNEL-BYPASS-20261115_xiaoka.md` 已留底）。
- M2 侧隧道钥匙 `/home/hatch/.ssh/m2_tunnel`，公钥指纹 `SHA256:r24ewWdFN3F4Y7CmN9In6J5M4QNZbn7GswIuN7oJijY`。
- 中继 `43.153.26.131:22223` 无监听，`Connection timed out during banner exchange`。

## 卡点（二级升级也无法独立解决）

M2 隧道需要中继端 `m2-relay` 账号的 `authorized_keys` 追加 `m2_tunnel` 公钥，或新建专用中继账号并一次装对钥匙。派单给出两条路：
- (a) 改服务配置，给中继账号加匹配块，把钥匙文件指到管理员能写的路径，改前备份、改后重启验证；
- (b) 新建配置正常的专用中继账号，钥匙一次装对。

**小卡没有中继机登录权限**（小豆 Windows 侧），两条路都需要小豆/中继机管理员执行。小卡侧已无可再试手段，服务保持自动重试。

## 需谁下一步

- 小豆/中继机管理员：执行 (a) 或 (b)，把 `m2_tunnel` 公钥写入 `authorized_keys`（路径 `C:\Users\m2-relay.000\.ssh\authorized_keys`），或新建专用账号并装对钥匙。
- 完成后 M2 autossh 30 秒内自动建隧，届时补交 22223 口监听 + M2 自动连接的原样证据。

## 结论

未到。卡在中继端 `authorized_keys` 权限墙，需要小豆/中继机管理员操作，小卡无法独立完成。
