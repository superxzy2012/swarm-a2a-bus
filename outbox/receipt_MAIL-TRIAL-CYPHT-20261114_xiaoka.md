# 工单：多邮箱查看入口主选 Cypht 试跑 — 小卡首轮回执

- 工单号：WO-MUSE-MAIL-TRIAL-20261114 (ddbff765)
- 回执文件：outbox/receipt_MAIL-TRIAL-CYPHT-20261114_xiaoka.md
- 回执人：小卡 (xiaoka)
- 死线：2026-11-14 12:00 前回试跑首轮

## 首轮进展

### 0. 拉取关 — 阻塞
`docker pull cypht/cypht:latest` 在 NAS 上执行：**超时**，
`Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)`

本机 `git clone https://github.com/cypht-org/cypht.git` 及 `codeload` tarball 均受阻（404/极慢）。Mac (192.168.199.119) SSH 转发通道也建立不上（端口 12022 监听失败）。出网受限：GitHub 全线不可达，Docker Hub 超时。

**后续**：若 12:00 前仍无法获取源码，按派单指示报 Queen 转 Mac，或由你另行提供源码包。

### 1. 钉最新版 — 未开始
待源码到位后，钉定版本 ≥2.12.2，核对 2026-09 安全问题修复，生成全新站点密钥（只放 NAS 本地，不进总线、不进群）。

### 2. JMAP 第一关 — 准备中
Atomic Mail JMAP 服务端：https://api.atomicmail.ai，用户名 `muse-anc-1`，密钥已从投递点文件 `atomicmail-apikey-20261114.txt` 读取（已按要求不在群里贴明文，不发任何密钥原文；轮换由你统一处理，旧密钥已作废）。

待源码到位后，给 Cypht 加 Atomic Mail JMAP 账户，先试密钥作密码的 Basic 认证；不通则记录原文并转备选评估，不硬啃。

### 3~6. QQ / Gmail / Outlook / 验收 — 未开始
等 JMAP 第一关见分晓后再推进。QQ 邮箱授权码、Gmail OAuth2 客户端等均需 BOSS/Queen 在试跑到对应步骤时提供。

## 结论
首轮回单已交：拉取关阻塞（Docker Hub 超时 + GitHub 不可达），源码未到位，后续步骤待解。不干等，若 12:00 前仍未通，直接升级。
