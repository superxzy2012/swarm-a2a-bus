# 星期五（QUEEN）云端迁移手册

> 状态：**准备中**（待 BOSS 提供新云凭据后执行）
> 生成时间：2026-10-20
> 生成者：friday（本机，迁移执行前自证）

## 0. 迁移目标

把 QUEEN（Hermes 云端实例）从当前腾讯云迁到 BOSS 自己名下的新云。
原则：**不丢记忆、不丢总线、不断飞书链路、模型路由无缝、可回滚。**

---

## 1. 现状盘点（源机，2026-10-20 实测）

### 机器
- 云厂商：腾讯云 CVM，实例 `ins-ijz9hynb`，公网 IP `[REDACTED-IP]`，内网 `[REDACTED-IP]`
- hostname：`VM-43-229-ubuntu`，OS：Ubuntu 24.04.4 LTS，内核 6.8.0-101-generic
- ⚠️ 该云在**非 BOSS 名下**（历史结论：微信扫码看不到这台机）

### Hermes
- 版本：Hermes Agent v0.21.5 (2026.9.24)
- 可执行：`~/.local/bin/hermes`，安装目录 `~/.hermes/hermes-agent`，venv 同目录
- Python 3.11.16

### 数据体量（必搬）
| 路径 | 体量 | 说明 |
|---|---|---|
| `~/.hermes` | ~1.4G | config.yaml / .env / memories / skills / plugins / cron / state.db / kanban.db / feishu_seen_message_ids.json |
| `~/sync` | 2.1G | 三方 Obsidian 同步库 + `github_deploy_key` |
| `~/A2A` | 1.2M | A2A 总线本地 + 台账 + 脚本 |
| `~/.a2a-bus` | 8.1M | git 中继仓 |
| `~/frp-monitor` | 2.9M | sync_obsidian.sh 等 |
| `~/.ssh` | 28K | obs_ro deploy key / config / authorized_keys / known_hosts |
| `~/.gnupg` | 8K | GPG |
| home 散落脚本 | ~50K | swarm-watch-v2.py / model-watch.py / gw-restart-once.sh 等 |

### 模型路由
- `model.default = deepseek-v4-pro-202606`，provider=custom
- base_url = `https://token.lightvela.com/v1`，api_mode=chat_completions
- 兜底：`model-watch.py` cron */10，连续3败回退 auto，日志 `~/A2A/model_watch.log`
- ⚠️ 本机跑着 `/opt/lightvela/dsh`（@deepseek-ai/dsh node 进程）—— lightvela 本地组件

### 连接器 / 平台（.env 凭据，key 名）
- FEISHU_APP_ID / FEISHU_APP_SECRET（应用维度，不绑 IP）
- LIGHTCLAW_API_KEY_usr_0a04b6b1b5ee70714444f172 + LIGHTCLAW_SITE
- WEIXIN_ACCOUNT_ID / WEIXIN_TOKEN / WEIXIN_BASE_URL / WEIXIN_CDN_BASE_URL
- CUSTOM_API_KEY / CUSTOM_BASE_URL（模型网关）
- BROWSERBASE_*（浏览器服务）
- 插件：lightclawbot、weixinpay

### A2A 总线 git
- origin = `github-a2a:superxzy2012/a2a-bus.git`（ssh 走 443，IdentityFile `~/.ssh/obs_ro`）
- gitea = `http://<queen>:<密>@[REDACTED-IP]:3000/anc/a2a-bus.git`（走组网隧道，账密见 .git/config，勿外泄）

### 组网（访问 NAS 的关键）
- 进程：`ownbsv/NodeBabyLinkService -peerid A6EACE9FF0406464OW`（贝锐/小宝组网）
- 提供 `[REDACTED-IP]` 隧道，承载：Gitea :3000、MCP obsidian :3456、skillbox :3458
- ⚠️ peerid 可能绑设备指纹，跨机复用待验证

### cron（系统 crontab）
```
*/15 * * * * ~/frp-monitor/sync_obsidian.sh
*/5  * * * * ~/A2A/sync.sh >> ~/A2A/sync.log
*/2  * * * * python3 ~/swarm-watch-v2.py
*/15 * * * * python3 ~/A2A/mcp_probe.py
*    * * * * [ -f ~/.gw_restart_once ] && ~/gw-restart-once.sh
*/10 * * * * python3 ~/model-watch.py
```
### hermes cron（jobs.json）
- 启用：AI资讯(0 7 * * *)、Obsidian同步晨检(0 7 * * *)、群消息监听-15min(*/15)
- 禁用：蜂群群消息巡检/任务板巡查/每日健康看板/T-013盯梢

---

## 2. 迁移分类

### A. 搬数据（rsync/tar 全量）
`~/.hermes`、`~/sync`、`~/A2A`、`~/.a2a-bus`、`~/frp-monitor`、`~/.ssh`、`~/.gnupg`、home 散落脚本 + 用户文件（PDF/PPT/PNG）。

### B. 重建环境
1. 新机装 Hermes（同版本 v0.21.5）→ 覆盖数据目录（或装好后 rsync 回灌）
2. Python 3.11 + 依赖
3. 组网客户端 NodeBabyLinkService 重装 + 入网（验证 [REDACTED-IP] 可达）
4. lightvela dsh 组件（若网关需要本地组件）

### C. 重接外部依赖
1. 模型网关 lightvela：验证 CUSTOM_API_KEY 是否绑 IP，绑则新 IP 加白
2. 飞书：同 app_id/secret 即可（需确认是否允许新 IP）
3. lightclawbot / weixin：验证是否绑设备/IP，绑则重新授权
4. Gitea + MCP：走组网隧道，依赖 B3 组网成功

---

## 3. 迁移步骤（草案）

1. **快照备份**：源机 tar 全量核心数据 → 存一份到 GitHub/Gitea（或直传新机）
2. **新机就绪**：装 Hermes v0.21.5 + Python3.11 + 组网客户端
3. **倒数据**：rsync 核心目录，校验 .env/config/memories/skills/plugins/cron
4. **修路径**：venv 绝对路径、脚本里 /home/agentuser 路径（若用户名不同）
5. **重接依赖**：验证 lightvela / 飞书 / lightclawbot / weixin / 组网
6. **恢复 cron**：系统 crontab 6 条 + hermes jobs.json
7. **验证**：见第 5 节验收清单
8. **切换**：新机接管飞书 app（老机停 gateway，避免双实例抢消息）
9. **观察 24h**：模型路由、心跳、总线、群监听全绿后，老机封存/释放

---

## 4. 风险与待验证（执行前逐项确认）

| # | 风险 | 影响 | 验证方式 |
|---|---|---|---|
| 1 | lightvela 网关绑 IP | 迁后模型哑火 | 新机 curl 测 token.lightvela.com |
| 2 | 组网 peerid 绑设备 | NAS 连不上 | 新机装组网后 ping [REDACTED-IP] |
| 3 | 微信/lightclawbot 绑设备 | 连接器掉线 | 新机登录后看连接器状态 |
| 4 | 飞书 app 绑 IP/回调 | 收不到消息 | 新机发测试消息看回执 |
| 5 | 双实例抢消息 | 老机新机同时收飞书 | 切换期只留一个 gateway |

---

## 5. 验收清单（迁移成功判据）

- [ ] `hermes --version` 正常，记忆/skills/plugins 完整加载
- [ ] 模型压测：deepseek-v4-pro 流式 + tool_calls 通过
- [ ] 飞书 DM 收发正常（BOSS 发消息，QUEEN 回复）
- [ ] A2A 总线：`sync.sh` 推 GitHub + Gitea 双成功，ls-remote 一致
- [ ] MCP 探针：[REDACTED-IP]:3456/3458 initialize OK
- [ ] 组网：[REDACTED-IP] 可达
- [ ] cron：6 条系统 cron + 3 条 hermes cron 全部跑通无报错
- [ ] 群监听 swarm-watch-v2 正常落 feed
- [ ] model-watch.py 兜底生效

---

## 6. 回滚方案

老机在切换后**保留 ≥7 天不释放**（封存状态：停 gateway、停 cron，数据不动）。
新机任何一项验收不过 → 老机重启 gateway 即刻接管，飞书链路 5 分钟内回切。

---

## 7. 需要 BOSS 提供（一次问清）

1. 新云 IP + SSH 登录方式（root 密码 / 私钥 / 账号密码）
2. 新云云厂商 + 账号是否 BOSS 自己名下（决定能否自助加安全组白名单）
3. 新云规格是否够（建议 ≥2C4G，磁盘 ≥20G，因 sync 2.1G + hermes 1.4G + 缓存）
