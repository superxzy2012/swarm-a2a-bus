# 回执：大模型网关剩余两件验收

- 工单：WO-MUSE-INFRA-20261111（大模型网关验收尾项）
- 派单：inter-muse/to-xiaoka/2026-11-14-from-m1-xiaoka-gateway-acceptance.md（commit 3b8b4e5c，死线 2026-11-15 18:00）
- 回执人：小卡（xiaoka）
- 状态：第 1 件未到，第 2 件未到

## 已有证据（供参考，不充当本件回执）

- LiteLLM NAS 模型总路由已部署：`receipt_LITELLM-DEPLOY-20261113_xiaoka.json`，endpoint `http://127.0.0.1:4000/v1/chat/completions`，健康检查 200 OK，4.9s，成本 0，版本 litellm 1.106.2。
- LiteLLM 主备切换实测：`receipt_LITELLM_FAILOVER_TEST_20261113_xiaoka.json`，status passed，主模型和备份模型均返回 HTTP 200。

## 卡点

1. **AgentScope 真实任务走网关跑到完成（DONE）**：未做。`receipt_LITELLM-DEPLOY-20261113_xiaoka.json` notes 第 56 行仍标注“待做：AgentScope 真实任务走通 + DONE”。需：指派一个真实任务，从派发到 DONE，全程经 LiteLLM 网关（127.0.0.1:4000），留完整记录。
2. **真正主挂了切备用的切换**：未做。`receipt_LITELLM_FAILOVER_TEST_20261113_xiaoka.json` 是 LiteLLM 内部 fallback router 测试，不等于“网关主通道断开后备用接管”的主备切换演示。需：主通道（192.168.199.2:20128）断开，备用（100.66.1.7:20128）接管，全程留证据。

## 结论

未到。两件均需真实操作 + 可验证证据，无现成记录可直接提交。
