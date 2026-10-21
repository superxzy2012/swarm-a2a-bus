# winpc — 远端公司 Windows 工作站

| 项 | 值 |
|---|---|
| 主机名 | `DESKTOP-4PV5HTQ` |
| 系统 | Windows 11 专业版（PS 5.1） |
| 登录用户 | `Lucas`（管理员） |
| 组网地址 | `[REDACTED-IP]`（节点小宝 mesh） |
| 局域网 | `[REDACTED-IP]` |
| NAS 侧入口 | `ssh company-win`（密钥 `/opt/data/.ssh/company_lucas`） |
| Gitea 账号 | `winpc`（Write on `anc/a2a-bus`） |
| 工具 | MinGit 2.56.0（`C:\A2A\git`）、curl、PowerShell；**无 Python、无 Node** |
| 工作目录 | `C:\A2A\a2a-bus`（clone 自 `http://[REDACTED-IP]:3000/anc/a2a-bus.git`） |

## 通道

| 方向 | 路径 |
|---|---|
| NAS → winpc | 直连 mesh `[REDACTED-IP]:22`（无需隧道） |
| winpc → NAS | 直连 `[REDACTED-IP]:3000`（Gitea）✅ |
| winpc → GitHub | ❌ 未配（未装代理通道，暂不开放） |

## 任务收发

- worker：`C:\A2A\win_worker.ps1`，计划任务 `A2A-Worker-winpc`（SYSTEM，每 5 分钟）
- 收件：`inbox/task_*.json` 里 `to == "winpc"`
- 回执：`outbox/receipt_<task_ref>_winpc.json`
- 日志：`C:\A2A\worker.log`

## 边界

- **只推 `main`（回执），不建分支**；push 失败自动 rebase 重试 3 次
- 执行 `detail.cmd` 走 `cmd.exe /c`，**多命令用 `&` 分隔**（不是 PowerShell 的 `;`）
- 这是 BOSS 公司的办公电脑：**不装重量级环境、不常驻服务、不做长期计算**，只当短任务执行节点
- 凭据 `C:\A2A\.git-credentials`，ACL 仅 Lucas/Administrators/SYSTEM
