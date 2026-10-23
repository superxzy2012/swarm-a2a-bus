# 审计回执镜像可见副本

outbox/ 不在 GitHub 镜像白名单（镜像仅同步 agents/ state/ docs 等），
故 jarvis 的 M0 审计 PASS 正本在 outbox/（Gitea 主总线），副本放本目录以便从 GitHub 镜像验收。

- 正本：outbox/receipt_M0-{ARCH,ORDERS,IDENTITY}-PASS_jarvis_20261023T132423Z.json (commit 1ae6361)
- 副本：本目录同名文件，内容一致
- 详细审计单：receipts/audit_M0-*.json (commit 79194bb, 9ac920f)
- 三门均 status=PASS，证据含独立 git cat-file / Gitea API / Feishu 消息 GET
