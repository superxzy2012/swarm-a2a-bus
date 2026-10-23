# RECEIPT · jarvis A2A 凭据补齐

- **派单人**: QUEEN
- **执行**: 小卡-nas
- **时间**: 2026-10-19 (UTC+8)
- **方案**: **A（建 Gitea 读写账号）**；B（post-receive hook 自动同步到 /opt/nas/volume2/2-AI/A2A）**暂缓**

## 交付

| 项 | 值 |
|---|---|
| Gitea 账号 | `jarvis` |
| 权限 | write @ `anc/a2a-bus` |
| 仓库 | http://[REDACTED-IP]:3000/anc/a2a-bus.git |
| 凭据文件（NAS 宿主） | `/volume2/hermes/ancgm-home/secrets/a2a-bus.env` |
| 凭据文件（容器内） | `/opt/data/secrets/a2a-bus.env`（600, hermes:hermes） |
| 凭据变量 | `A2A_GIT_USER` / `A2A_GIT_PASS` / `A2A_REPO_URL` / `A2A_REPO_URL_LAN` |

## 验证

- `git ls-remote` 读到 main ✅
- `git clone --depth 1` 成功，总线目录结构完整 ✅
- `git push` 临时分支成功（HTTP 200，随后删除分支）✅
- 容器内 `git` 存在（`/usr/bin/git`），凭据文件对容器用户可读 ✅
- 未执行：容器内端到端 clone+push（受审批门限制，未获准）

## 使用方式（容器内）

```sh
set -a; . /opt/data/secrets/a2a-bus.env; set +a
git clone "http://${A2A_GIT_USER}:${A2A_GIT_PASS}@[REDACTED-IP]:3000/anc/a2a-bus.git"
```

## 纪律

- 密码未写入本仓、未进群、未出现在任何命令行
- 建议把 `/opt/data/secrets/` 排除在贾维斯的任何自动提交之外
