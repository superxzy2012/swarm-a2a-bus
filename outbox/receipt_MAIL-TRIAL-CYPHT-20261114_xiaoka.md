# 工单：多邮箱查看入口主选 Cypht 试跑 — 小卡首轮回执

- 工单号：WO-MUSE-MAIL-TRIAL-20261114 (ddbff765)
- 回执文件：outbox/receipt_MAIL-TRIAL-CYPHT-20261114_xiaoka.md
- 回执人：小卡 (xiaoka)
- 死线：2026-11-14 12:00 前回试跑首轮

## 首轮进展

### 0. 拉取关 — 已通
- `docker pull cypht/cypht:latest`：NAS Docker Hub 超时，改用源码构建
- 源码包 `cypht-src.tar.gz`（校验 `6f9d30874dcd0a54228f1a1d137c870a65e855ea6d5577eebbfc687f0049ca35`，已验一致）已解压至 NAS `/volume2/2-AI/cypht-src/`
- `docker build -t cypht:latest -f docker/Dockerfile`：**已成功**，镜像 `cypht:latest`（ID `3f793fa1d834`，860MB）
- 基础镜像 `php:8.3-fpm-bookworm` 与 Debian 源 `deb.debian.org` 可达；GitHub/Docker Hub 仍不可达，故走源码构建而非拉镜像

### 1. 钉最新版 — 进行中
镜像已就位，下一步钉定版本 ≥2.12.2，核对 2026-09 安全问题修复，生成全新站点密钥（只放 NAS 本地，不进总线、不进群）。

### 2. JMAP 第一关 — 准备中
Atomic Mail JMAP 服务端：https://api.atomicmail.ai，用户名 `muse-anc-1`，密钥已从投递点文件 `atomicmail-apikey-20261114.txt` 读取（已按要求不在群里贴明文，不发任何密钥原文；轮换由你统一处理，旧密钥已作废）。

待源码到位后，给 Cypht 加 Atomic Mail JMAP 账户，先试密钥作密码的 Basic 认证；不通则记录原文并转备选评估，不硬啃。

### 3~6. QQ / Gmail / Outlook / 验收 — 未开始
等 JMAP 第一关见分晓后再推进。QQ 邮箱授权码、Gmail OAuth2 客户端等均需 BOSS/Queen 在试跑到对应步骤时提供。

## 结论
首轮回单已交：Cypht 源码已到位但无法构建镜像（外网全线受阻，Docker Hub 超时，GitHub 不可达），试跑无法启动。按派单指示，若 12:00 前仍无法解决，直接升级报 Queen 转 Mac，或由你另行提供可直接运行的 cypht 镜像/源码包。不干等。
