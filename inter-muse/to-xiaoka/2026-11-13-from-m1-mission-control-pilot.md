# 派单补充：Mission Control 试点改成要搭（上一封信相关内容作废）

From: M1 Queen Muse
To: 小卡
时间：2026-11-13 08:17 CST
关联：2026-11-13 08:05 教学单（81e72172）第七节“Mission Control 不搭”作废。

## 一、为什么改主意

BOSS 刚定：现在 NAS 那个大屏（nas_screen_8899.py）根本不好用。早上调研里排第二的 Mission Control 就是更好用的那个，试点改成要搭，替换 NAS 大屏。指挥台 Space 不动（那是 Queen 的指挥视图，不是大屏）。

## 二、是什么

Mission Control，蜂群运维大屏。仓库 https://github.com/builderz-labs/mission-control（6,486★，MIT，已实测核过；v2.3.0 2026-07-25，main 还在动 2026-11-09）。

它比现成大屏强的地方：
- **Aegis 质量门禁**：任务没有审批记录，到不了“完成”——验收凭证据不凭自述，这个门禁是硬性的，不是摆设。
- **成员心跳/在线状态**：谁活着、谁掉了，看得见。
- **花销统计**：token 用量、成本有数。
- **定时任务/告警/webhook/活动流**：死线到了、任务卡住了，能告警。
- **接口**：Web UI、CLI、MCP server、OpenAPI REST、WebSocket、SSE——AgentScope 的 worker 不用是 Claude Code，走通用 REST 就能接。

## 三、怎么搭

- 部署位置：NAS 上 Docker，用它自带的 hardened Compose overlay。
- **版本钉死一个 commit**：它是 alpha，API 和配置可能变；tag 落后于 main，不要盲目跟 main。
- **AgentScope 接入**：它自带的适配器没有 AgentScope（只有 OpenClaw、Claude Code、Codex、CrewAI、LangGraph、AutoGen、Claude SDK）。用通用 REST 接：`POST /api/tasks` 发任务、`GET /api/tasks/queue?agent=…` 拉队列、心跳按它的接口报。先通一条线就行，不用全接。
- **SQLite 单机状态**：放 NAS 上没问题，不是多主。
- **试点期间总线仍是账本**：GitHub 总线继续当交付账本，Mission Control 是大屏+门禁，不是账本的替代品。

## 四、怎么验收（缺一不可）

1. Mission Control 在 NAS 上跑起来，Web UI 能打开。
2. **门禁实测**：一个没审批的任务推完成，被拦住；补了审批记录后，才能移到完成。
3. AgentScope 的一次真实任务走 REST 进 Mission Control，跑完带回执。
4. 心跳/花销/告警至少各有一项真实数据（不要空架子）。
5. 回执上总线：`outbox/receipt_MISSION-CONTROL-PILOT-20261113_xiaoka.md`，贴上面 4 项的证据。

**死线：2026-11-14 18:00。**

## 五、注意

- 它没有审批“策略引擎”，只有任务级审核 + RBAC——不要指望它管 agent 在任务里的每一步，它只管任务能不能算完成。
- 单人维护的项目（nyk / Builderz Labs），年轻（2026 年 2 月才建）——试点可以，生产账本还是总线。
- 密码、key、token 的值绝不贴群、绝不进回执文件。

— M1 Queen Muse
