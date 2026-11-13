# 派单：搭建 ANC 配套部署（教学单）

From: M1 Queen Muse
To: 小卡
时间：2026-11-13 08:05 CST
前置：m2 隧道修复优先（死线 09:00），修完再开这一单。

## 一、总述

BOSS 已定：把 GitHub 成熟项目调研里对 ANC 有用的配套部署搭起来。调研报告（`~/workspace/research_notes/anc-swarm-github-projects-20261112-2318/report.md`，总线另有汇总页）给的 Top 5 是 Beads、Mission Control、LiteLLM、Langfuse、Hatchet。分工如下：

- **你搭 5 项**（按顺序，先 1 后 2）：LiteLLM → Langfuse → Beads 试点 → Hatchet 试点 → Uptime Kuma + Beszel。
- **Mission Control 不搭**：它的活（大屏 + Aegis 质量门禁 + 心跳 + 花销）指挥台 Space 已经做了，再搭一个是重复。先记为 REFERENCE，以后指挥台不够用再议。

部署位置：NAS 上 Docker（Uptime Kuma 例外，见第五节）。每项验收都一样：**部署好 + 真实调用跑通 + 回执上总线**，只说“装好了”不算。

## 二、第一项：LiteLLM（最急，今天 18:00）

**是什么**：模型网关，一个 OpenAI 兼容端点，后面挂多个免费模型上游，自动 fallback。仓库 https://github.com/BerriAI/litellm（63,190★，v1.105.1 / v1.106.2，2026 年 10 月）。

**为什么先搭它**：今天早上 AgentScope 就是被 kilo 550B 空回复拖死的——单个免费模型一死，整个 worker 就卡住。有了 fallback 链，一个上游死就自动切下一个，不再卡住。这是当前最痛的点。

**怎么搭**：
- Docker 部署 LiteLLM Proxy，配 `config.yaml`：`model_list` 列上游（OpenRouter `:free` → Kilo → Groq/Gemini 免费档），`router_settings` 写 fallback 链；开虚拟 key（每 agent 一把）+ 每 key 预算 + 花销统计。
- **版本钉死**：不要用 v1.82.7 / v1.82.8（2026-03-24 出过供应链投毒事件，PyPI 上被篡改约 40 分钟）。用 v1.105.1、v1.106.2 或更新的已核实版本，核对签名。
- **安全**：只绑 127.0.0.1 + master key 鉴权；**不要碰 `enterprise/` 目录**（商业授权，生产用要订阅）。
- 2026 年有 27 条安全公告（含 4 条严重），搭完顺手看一遍公告，该升的升。

**怎么验收（缺一不可）**：
1. Proxy 起起来，健康检查过。
2. 一次真实 chat completion 走 Proxy 返回 200。
3. **Failover 实测**：把链首上游指到一个死的模型（或临时停掉一个上游），再调一次，调用仍成功——证明真的切过去了，不是只配了个样子。
4. AgentScope 的一次真实任务走 LiteLLM 跑到 DONE（切生产流量前必须先过这一项）。
5. 回执上总线：`outbox/receipt_LITELLM-DEPLOY-20261113_xiaoka.md`，贴上面 4 项的证据（不要只写结论）。

**死线：今天 18:00。**

**注意**：切生产流量前不要动 AgentScope 现有的模型配置；LiteLLM 先并行跑通，第 4 项过了再切。

## 三、第二项：Langfuse（11-14 12:00）

**是什么**：自托管 LLM 追踪 + 成本 + 评估平台。仓库 https://github.com/langfuse/langfuse（v4.72.0，2026 年 10 月）。MIT 核心。

**为什么**：验收要凭证据不凭自述——每个 worker 跑了什么工具、花了多少 token、多少钱、延迟多少，都要有 trace 可查。以后完成回执要链 trace + 产物，不再只凭一句话。

**怎么搭**：Docker/Helm 自托管。只用 MIT 核心，**不要开 `ee/` 里的企业功能**（RBAC、审计日志、保留期、掩码、SCIM 等 9 项是授权收费的）。

**怎么验收**：一次真实 worker 运行的 trace 能在 Langfuse 里查到（工具、延迟、token、成本齐全）；成本看板有数。回执上总线。

**死线：2026-11-14 12:00。**

## 四、第三项：Beads 试点（11-14 18:00）

**是什么**：给 agent 用的分布式任务账本（Dolt 数据库后端）。仓库 https://github.com/gastownhall/beads（28,694★，MIT，112 个 release）。命令：`bd create / ready / claim / close`，任务带依赖图，认领是原子的，关闭要附证据。

**为什么**：现在的总线是“文件放分支上、轮询靠运气”，工人慢、不回、死线过了没人知道，都看不见。账本让任务状态可见：没人认领、认领了不干，都看得见。

**怎么搭**：
- 在 NAS 上装 `bd`，建账本，走现有 git 远端同步（Beads 本来就是走 git remote 同步的，不用新通道）。
- **先只试点一条线**（Ozon 线）：Queen 在账本里建任务，工人 `bd ready` 看、认领、关闭附证据。
- **大坑**：真相来源是 Dolt 数据库（gitignored），JSONL 只是导出。数据库必须备份/推送，**不要只留 JSONL**，丢了库就丢了账本。

**怎么验收**：`bd create → ready → claim → close` 全程跑通一次（含附证据关闭）；Dolt 库备份就位。回执上总线。

**死线：2026-11-14 18:00。**

## 五、第四项：Hatchet 试点（11-15 12:00）

**是什么**：Postgres 打底的可靠任务队列 + 编排。仓库 https://github.com/hatchet-dev/hatchet（~8,300★，MIT）。DAG 工作流、超时、带退避的重试、限速、并发控制、自带 Web UI。

**为什么**：蜂群缺的规矩——任务超时没人认领就重派、认领了半天没动静就升级给 Queen、失败 N 次就挂起并通知人——要由运行时自动执行，不靠人盯着。

**怎么搭**：Postgres + Hatchet 服务（比 Temporal 轻得多，NAS 跑得动；Temporal 是以后长大了再换的选项，现在不要上）。

**怎么验收**：一条带超时/重试/升级规则的测试工作流跑通，UI 能看到。回执上总线。

**死线：2026-11-15 12:00。**

## 六、第五项：Uptime Kuma + Beszel（11-15 18:00）

**是什么**：两个轻量监控。Uptime Kuma（https://github.com/louislam/uptime-kuma，v2.5.6，MIT）：服务存活监控，HTTP/TCP/心跳推送，死了就告警。Beszel（https://github.com/henrygd/beszel，v0.18.8，MIT）：主机监控，CPU/内存/磁盘/Docker/S.M.A.R.T./GPU。

**为什么**：agent 死了、网关掉了、NAS 磁盘满了，都要第一时间知道，而不是等 BOSS 来问。

**怎么搭**：
- **Uptime Kuma 必须装在 NAS 之外**（比如一台云 VM），不然 NAS 挂了、监控跟着挂，等于没装。主机你定，报给我。
- Beszel：hub 放 NAS，各主机装 agent。

**怎么验收**：Kuma 上挂好所有关键服务（网关、AgentScope、n8n、隧道），手动停一个服务能收到告警；Beszel 能看到各主机指标。回执上总线。

**死线：2026-11-15 18:00。**

## 七、不要碰的（调研结论，直接执行）

- AutoGen（已归档）、OpenAI Swarm（已归档）、Gas Town（2026 年 9 月退役）、Backtrader（2024 年 8 月后没动静）、OpenAI Evals（2026-11-30 下线）——不要装。
- 社区共享 key 的免费网关（FreeRideV3 之类）——不要接， key 不可信、无审计。
- Medusa/Saleor/Vendure（自建商城平台）——我们是在平台上卖货，不是要建商城，形状不对。
- 许可证陷阱：backtesting.py 是 AGPL-3.0、Freqtrade/Backtrader 是 GPL、vectorbt 带 Commons Clause——这些是量化线的，以后要用先读许可证。

## 八、规矩

- 每项死线前交回执上总线；到点交不出，死线前在群里说卡在哪、证据是什么，不要到点才说“没弄完”。
- 部署不要动现有生产配置（AgentScope、OmniRoute、muse2api）——新东西先并行验证，验收过了再切。
- 密码、key、token 的值绝不贴群、绝不进回执文件。

— M1 Queen Muse
