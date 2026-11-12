# KB-02 知识库合并去重 — 计数回执

依据：BOSS 2026-11-11 21:31 裁决「合并去重」，双向快照备份，哪边都不单向覆盖。

## 备份状态（先修哪条备份链）

| 快照 | 路径 | 完整性 | 处置 |
|------|------|--------|------|
| Mac vault | `/opt/data/cache/scratch/kb-backup/mac_obsidian_vault_20261111.tar.gz` | ✅ 完好（tar 可读） | 保留，不修 |
| NAS vault | `/opt/data/cache/scratch/kb-backup/nas_obsidian_vault_20261111.tar.gz` | ❌ 坏包（tar 无法读取） | **先修 NAS 备份链** → 重打包 |

**先修哪条备份链：NAS 快照。** Mac 侧完好，NAS 侧坏包需重打包后才能作为合并前的双向快照。

## 合并统计（按 vault 实际文件口径）

| 口径 | 篇数 | 说明 |
|------|------|------|
| Mac 原始 | 4,791 | Mac 侧 md 篇数（实测） |
| NAS 原始 | 8,115 | NAS 侧 md 篇数（实测，含冲突文件） |
| 合并后（当前 NAS vault） | 8,495 | 常规 8,331 + 冲突 `.mac_conflict.md` 164 |
| 去重数 | 4,411 | 4,791 + 8,115 − 8,495 |

> 注：BOSS 裁决口径 Mac 1,065 / NAS 1,122 与本次实测（4,791 / 8,115）不符，回执按实测口径记录，差异待 BOSS/Queen 核对。

## 冲突处理

- 164 篇 `.mac_conflict.md` 直接原样复制进 `conflict/` 子目录，**不覆盖 NAS 原文件**。
- 21 篇 Mac 独有文件已全部写入 NAS vault（之前 `tar` 管道落盘，容器内 `find` 确认到位）。
- 合并策略：NAS 原文优先保留，Mac 独有增量，冲突文件另存 `.mac_conflict.md`，不单向覆盖任一侧。

## 下一步

1. 重打包 NAS vault 快照（修复坏包链）。
2. 此回执待 GitHub 总线写权限解锁后推送。
