# WO-MUSE-INFRA-20261109 — 蜂群基建推进包（BOSS 2026-11-09 午休授权）

状态：已于 2026-11-09 13:0x 经飞书小林真@发出实时通知（小卡 om_x100b60217c8094a4ce93112e5a1c539 / 小米 om_x100b60217cad08acc0407b2860a8e2a / 贾维斯 om_x100b60217cbd4ca4df7038336abc60a）；本文件为 A2A 正式落档，待 muse_a2a 部署密钥恢复后提交 muse 分支。

## 定版口径（BOSS 2026-11-09）
1. 飞书群通知必须真@；Muse 经小林做飞书实时沟通（通知/领取）。
2. 正式任务走 A2A 文件标准，GitHub 为主通道。
3. Gitea = NAS 备用仓：只同步 GitHub 内容；GitHub 故障时只读；不得影响主通道。
4. 大文件走 NAS 只写接收点 + 分块清单断点续传（WO-MUSE-NAS-RECEIVE-20261106 + AMD1），由 Muse 推进、小卡协助落地。
5. 知识库：先完善 NAS Obsidian 做蜂群共享第一知识库，跑通后启动第二知识库 ima。
6. 技能库：核查 Mac skill box 与 NAS 部署现状（小卡查 NAS、小米查 Mac），由 Muse 推动完善共享。
7. 蜂群记忆系统（RAG 等）由 Muse 规划推动。
8. Mac/Windows 受控接入：Muse 除手机外同步直控电脑执行命令（方案 v1 已出，先摸排后试点）。

## 派单明细
- 小卡：① GitHub Deploy key 恢复（muse_a2a 公钥，Allow write access）；② NAS 接收点落地并回执坐标；③ Gitea 镜像现状回执；④ Obsidian 现状回执；⑤ NAS 技能库部署核查；⑥ Tailscale 节点摸排。
- 小米：① Mac skill box 现状回执；② Mac 受控接入摸排（Tailscale/SSH/运行方式，只报现状不改配置）；③ 家 Windows 状态（如知）。
- 贾维斯：AgentScope 2.0 + n8n 落地现状与导入蜂群缺口回执（先报现状，不动他节点配置）。

## Muse 侧已完成
- 《ANC 蜂群通信标准 v1》《蜂群记忆系统规划-RAG-v1》《知识库与技能库共享方案-v1》《Mac-Windows 受控接入方案-v1》（本地 your_files/anc-infra-20261109/，待总线恢复后同步落仓）。
- 小林真@技能与记忆已固化（feishu-group-mention，含管理群全员 open_id）。
