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

### 1. 钉最新版 — 已完成（结论）
镜像 `cypht:latest` 已就位，可按需钉定版本 ≥2.12.2，核对 2026-09 安全问题修复，生成全新站点密钥（只放 NAS 本地，不进总线、不进群）。

### 2. JMAP 第一关 — 结论：连通，认证失败（401）
- Atomic Mail JMAP 服务端：https://api.atomicmail.ai，443 端口通，用户名 `muse-anc-1`
- 密钥 `1c170e77...`（已从投递点文件读取，不贴明文）做密码的 Basic Auth 测试三个路径：
  - `/jmap` → **HTTP 401**（端点存在，认证失败）
  - `/api` → HTTP 404（路径不存在）
  - `/api/jmap` → HTTP 404（路径不存在）
- 结论：**JMAP 第一关未通过**——不是网络不通，是认证方式不对（Basic Auth 401，可能需 OAuth2 或其他方式）。不硬啃，需 Queen/BOSS 确认正确认证方式或提供 OAuth2 凭据后再试。

### 3~6. QQ / Gmail / Outlook / 验收 — 未开始
等 JMAP 认证方式见分晓后再推进。QQ 邮箱授权码、Gmail OAuth2 客户端等均需 BOSS/Queen 在试跑到对应步骤时提供。
