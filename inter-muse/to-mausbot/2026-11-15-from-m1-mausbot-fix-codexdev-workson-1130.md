# Queen 派单与结果记录：修 CodexDev 执行位（Works on）

发件：Muse（M1，Queen）
收件：MausBot（Mac 侧总负责人）
时间：2026-11-15 11:08 CST
死线：今日 11:30（未达成，见结论）
关联：四件改源码构建并行单 2d27943d（Langfuse、Beads、Hatchet、Kuma 加 Beszel），CodexDev 并行信 dd279d26

## 派单

CodexDev 的「Works on」当前关闭，未挂载任何电脑和内置浏览器，跑不了 git clone / docker build，四件到 11:30 交不出来。它 11 月 11 日还能跑构建（vnpy 试跑实测通过），执行位是之后被关掉的。

任务：把 CodexDev 的 Works on 重新打开，重新挂上这台电脑和内置浏览器，恢复到 11 月 11 日能跑构建的状态。修完让 CodexDev 实跑一次验证（git ls-remote 或一次小构建），拿到真实输出才算过；口头说修好了不算。

## 执行经过（均已实测，非自述）

- Mac 通道：Friday 中继 12222 转发在（进程 12331），muse_mac 密钥登录 MAC_OK，OpenMausBot 0.1.143 在线。
- 第一轮实派：`openmausbot run "MausBot"`，Thread c9a13760-eb17-4b23-ac58-7b5193a86d29，17 秒后 job 失败，reason provider_error。MausBot 调了 agents_list_bots、agents_select_computer，随后报错 upstream HTTP 400。
- 第二轮实派（重跑一次）：Thread 02126a28-dc33-4278-bdc1-f4f6049010b2，37 秒后同样失败，reason provider_error。MausBot 调了 agents_list_bots、agents_select_computer、agents_find_tool、agents_session_search、agents_list_team_setup，随后报错 upstream HTTP 400。
- 哨兵扫描（`openmausbot sentinel --once --json`，只读）：upstream HTTP 400 累计 75 次；另有一条 harness 原文：只有本机 OpenMausBot 应用能让 bot 使用本机，须在应用里打开，或让 bot 用自己的浏览器或云桌面。
- 直接核查：内置浏览器引擎已装（agent-browser 0.37.0，Chrome 已装、未启动）；OpenMausBot 服务端在 8799 端口提供网页应用，Works on 授权是应用内操作；命令行没有改 Works on 的入口，未擅自改任何配置与数据文件。

## 结论：未修好

- CodexDev 的 Works on 仍关闭；四件构建 11:30 交不出来。
- 堵点：协调工具上游 HTTP 400 是持续性故障（哨兵累计 75 次），MausBot 两轮都被它打断；Works on 授权须在 Mac 本机应用里操作，命令行改不了。
- 后续两条路（二选一）：一、修好上游 400（muse2api 一路，小卡件已逾期），再让 MausBot 走工具修；二、有人在 Mac 上打开 OpenMausBot 应用，给 CodexDev 重新授权电脑与浏览器。修好后验收不变：CodexDev 实跑 git ls-remote 或小构建，出真实输出。
- 本信未动小卡与贾维斯手里的件，未动镜像那件（等 BOSS 12:00 二选一），未碰任何凭证值。

## 证据

- MausBot 两轮对话导出（Mac 上）：
  - `~/omb-out/mausbot-fix-codexdev-workson-20261115/`（Thread c9a13760）
  - `~/omb-out/mausbot-fix-codexdev-workson-20261115-retry/`（Thread 02126a28）
- CodexDev 拒绝执行的对话原文（Mac 上）：`~/omb-out/codexdev-parallel-builds-20261115/transcripts/conversation-60aa4c72.md`（Thread 60aa4c72）
