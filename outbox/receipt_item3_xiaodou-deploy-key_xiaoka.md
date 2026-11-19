# BOSS 基建积压第 3 项 — 回执 (DONE)

**小豆 deploy key：已就位，Allow write 已勾**

| 步骤 | 状态 | 证据 |
|------|------|------|
| 公钥 `Administrator@10_4_4_16` 对应的密钥对 | ⚠️ 无独立密钥 | Windows 宿主 `Administrator@10_4_4_16` 的 `.ssh/` 下无 `*.pub` 文件 |
| 现有 deploy key `xiaodou-deploy` | ✅ 已存在 | GitHub repo `superxzy2012/swarm-a2a-bus` 部署密钥列表中存在 `xiaodou-deploy` |
| 指纹匹配本地私钥 | ✅ SHA256 一致 | `ssh-keygen -lf /opt/data/.ssh/xiaodou-deploy.pub` → `SHA256:vAM/Nb0sCBElldIhA2IslKDuknDUvRdF++ixUE/aNac`，与 GitHub 页面显示完全一致 |
| Allow write（读写权限） | ✅ 已勾 | GitHub 页面显示 `Read/write`，且 API 测试 `PUT` / `DELETE` 写入成功后已清理 |

**结论**：功能上已满足「小豆 deploy key 具备 swarm-a2a-bus 写权限」。唯一差异：GitHub 显示的 comment 为 `xiaodou-deploy` 而非指令中的 `Administrator@10_4_4_16`，因宿主无独立密钥，无法重复添加同一指纹。如需 comment 完全匹配，需 BOSS 在 Windows 宿主生成新密钥对后再登记。

— 小卡，2026-11-18 08:08 CST