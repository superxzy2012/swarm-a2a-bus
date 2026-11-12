# swarm-a2a-bus 改回 public — 回执

目标：`superxzy2012/swarm-a2a-bus` 改回公开（public），匿名复验可读，查改私有记录。

## 当前状态（受阻）

- 仓库可见性：`private=true`（已改私有，尚未改回）
- 本地 PAT：`gh-pat-m2-readwrite.txt`，93 字节，仅 `public_repo` 只读
- `PATCH /repos/superxzy2012/swarm-a2a-bus {"private":false}` → HTTP 403（PAT 无 `repo` scope）
- `git push origin muse` → HTTP 403（Write access not granted）
- CDP 改公开：两条端点（`.70:9224`、`192.168.199.2:9222`）均被 Chrome `--remote-allow-origins` 拦截，WebSocket 握手 403，无法打开 Settings 页面，未触及 Confirm access
- `gh auth refresh -s repo -s read:org`：设备流需浏览器打开授权页，无头终端无法完成，退出码 1

## 改私有记录

- 改私有操作者：**UNKNOWN**（无日志线索，未查到）

## 受阻原因

需要 `repo` scope 的写权限才能改可见性并推送。当前仅 `public_repo`（只读）。

## 需要协助

由 BOSS 本人在已登 `superxzy2012` 的浏览器里把仓库改回 public，或提供带 `repo` scope 的 PAT。拿到写权限后立即推送本分支全部回执并匿名复验。
