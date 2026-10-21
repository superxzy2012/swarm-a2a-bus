# octop — Octop 专家平台（NAS 8088）

| 项 | 值 |
|---|---|
| 标准 id | `octop` |
| 显示名 | Octop 专家平台 / 章鱼 |
| 平台 | NAS `[REDACTED-IP]:8088`，容器 `octop` v1.0.2 |
| 宿主 | Hermes 容器（小卡侧）代跑 worker：`/opt/data/bin/octop_a2a_worker.py` |
| 轮询 | cron 每 3 分钟扫 `inbox/`，只处理 `to` 以 `octop` 开头的任务 |

## 子路由（写在 to 字段里）

| to 写法 | 派给谁 |
|---|---|
| `octop` | 默认 → `main`（小通·通用助手） |
| `octop:小安` / `octop:CMW4WR` | 小安·合规哨兵（Ozon 合规、禁售、认证） |
| `octop:小热` / `octop:JF6ADQ` | 小热·热点雷达（科技/AI 热点、外网抓数） |
| `octop:小办` / `octop:Q8NY3B` | 小办·办公自动化（表格、文档、流程） |
| `octop:老钱` / `octop:TAQG6V` | 老钱·证券观察员（A 股行情） |
| `octop:小维` / `octop:X1N8RQ` | 小维·运维工程师 |
| `octop:阿文` / `octop:9VQHTY` | 阿文·一文多发 |
| `octop:小调` / `octop:4GK9GF` | 小调·任务编排 |
| `octop:team:XJWVMB` | 跨境选品小队（小热+小安+小办 协同） |

## 能力边界

- ✅ 知识库语义检索（697 篇 vault 笔记，本地 ONNX 嵌入）
- ✅ 外网抓数（容器走 7891 代理，插件 4 个）
- ✅ 多专家协同（团队异步拆活）
- ✅ 每专家独立长期记忆（自动提炼 atoms）
- ❌ 不能发飞书消息（无渠道，需小卡转述）
- ❌ 不能跑代码/改文件（ACP 无 node）
- ⚠️ 数字类答案可能串（实测 14/15 天串题），引用前必须回原文核

## 任务信封示例

```json
{
  "task_ref": "T-20261021-001",
  "from": "xiaoka",
  "to": "octop:小安",
  "ts": "2026-10-21T11:30:00+08:00",
  "type": "task",
  "priority": "P1",
  "instruction": "便携榨汁杯上 Ozon 俄罗斯站需要哪些认证？给结论，不要科普。",
  "deadline_hours": 2
}
```

回执写在 `outbox/receipt_<task_ref>_octop_<ts>.json`，正文落 `results/result_<task_ref>_octop_<ts>.md`。
