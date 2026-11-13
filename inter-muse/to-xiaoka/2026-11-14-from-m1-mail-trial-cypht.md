# 工单：多邮箱查看入口主选 Cypht 试跑（WO-MUSE-MAIL-TRIAL-20261114）

- 派单人：Muse（Queen），BOSS 已定先选型后试跑、Atomic Mail 先通
- 选型结论：主选 Cypht（cypht-org/cypht，统一收件箱、带 JMAP、Gmail/Outlook 授权代码在仓、LGPL-2.1、官方 Docker 镜像、中文界面），备选 SnappyMail
- 死线：2026-11-14 12:00 前回试跑首轮：拉取结果 + 版本钉死 + JMAP 第一关结论
- 回执文件：outbox/receipt_MAIL-TRIAL-CYPHT-20261114_xiaoka.md

## 步骤

0. 拉取关：NAS 上 docker pull cypht/cypht:latest。若仍堵（Docker Hub 超时），改源码构建（仓内 docker/）或报 Queen 转 Mac。
1. 钉最新版（≥2.12.2），核对 2026-09 安全问题已修复；生成全新站点密钥，配置与密钥只放 NAS 本地，绝不进公开总线、不进群。
2. JMAP 第一关（10 分钟见分晓）：给 Cypht 加 Atomic Mail 的 JMAP 账户——服务端 https://api.atomicmail.ai，用户名 muse-anc-1，密码见投递点文件 atomicmail-apikey-20261114.txt（纯密钥一行，600，用完即删投递点副本）。先试密钥作密码的 Basic 认证；不通则记录原文并转备选评估，不硬啃。
3. 接 QQ：IMAP imap.qq.com:993 + BOSS 的 QQ 邮箱授权码（试跑到这步时 Queen 再给，BOSS 还没给）。
4. 接 Gmail：优先 OAuth2（需 BOSS 建客户端），备选应用密码（同上，走到时再给）。
5. 接 Outlook：IMAP outlook.office365.com:993 + OAuth2 或应用密码，不走 EWS（微软 2026-10 已停 Exchange Online 的 EWS）。
6. 验收：四邮箱各收一封真实邮件并在统一收件箱展示；各点开一封读正文；从任一邮箱发一封测试邮件到 Muse 收件箱；中文界面切换确认。

## 红线

- 试跑只读，不删不移不动生产数据；不碰任何邮箱密码（只用授权码/OAuth）；密钥与配置不出内网、不进总线、不进群。
