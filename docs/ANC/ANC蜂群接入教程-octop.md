# ANC 蜂群接入教程：Octop

> 目标：把 Octop（腾讯云开源、自托管多用户多智能体 AI 助手）作为 **ANC 蜂群的「专家团 / 家庭助手层」** 接入。Octop 自带 **ACP 双向协议**、**MBTI 人格模板**、**多用户隔离**、**工具审批**、**PII 脱敏**，可直接作为蜂群的「前台交互入口」与「专家代理池」。

---

## 1. 架构定位

| 层级 | 组件 | 在蜂群中的角色 |
|------|------|----------------|
| 前台交互 | **Octop Web UI** (port 8088) | BOSS/家庭成员的统一对话入口，支持多用户、多专家切换 |
| 专家代理 | **Octop Agents** (内置/插件) | 领域专家（Ozon运营、1688寻源、抖音分析、财务核算等），通过 **ACP** 双向调用 OpenMausBot / Claude Code |
| 编排大脑 | **AgentScope Swarm** | 复杂多步任务的编排、决策、回执收敛 |
| 总线 | **A2A Git 总线** | Octop 产生的任务/回执同步到 `inbox/` `outbox/` |
| 模型 | **OmniRoute** | Octop 内部 LLM 调用统一走 OmniRoute (OpenAI 兼容) |
| 工具 | **n8n / OpenMausBot / Hermes** | Octop 插件/连接器复用蜂群工具层 |

> **关键差异**：Octop 面向「人/家庭/团队」的交互层，AgentScope 面向「自动化编排」的后台层。两者通过 **ACP** 和 **A2A 总线** 双向打通。

---

## 2. 部署现状（已在 NAS 跑通）

```
NAS 192.168.199.2
├─ /volume2/2-AI/octop/
│   ├─ docker-compose.yml          # Octop 单容器
│   ├─ config.json                 # 运行时配置（管理员账号、端口、代理）
│   ├─ credential.txt              # 初始登录凭据（首次启动后可删）
│   ├─ octop.db                    # SQLite 元数据（用户、专家、会话、记忆）
│   ├─ agents/                     # 专家定义目录（JSON + Python 插件）
│   ├─ plugins/                    # 连接器生态（腾讯文档、微博热搜、新闻、MCP 网关等）
│   ├─ patch/jobs.py               # 补丁：修复知识库定时任务时区/去重
│   └─ docker-compose.yml.bak
└─ 访问入口：http://192.168.199.2:8088  (直连，无反代)
```

**docker-compose.yml 关键配置**：
```yaml
services:
  octop:
    image: ghcr.io/tencentcloud/octop:1.0.2
    container_name: octop
    restart: unless-stopped
    ports:
      - "8088:8088"
    volumes:
      - /volume2/2-AI/octop:/data/.octop
      - /volume2/2-AI/octop/patch/jobs.py:/app/src/octop/infra/knowledge/jobs.py:ro
    environment:
      HOME: /data
      OCTOP_BIND_HOST: 0.0.0.0
      OCTOP_PORT: "8088"
      OCTOP_LOG_LEVEL: info
      OCTOP_ADMIN_USERNAME: admin
      OCTOP_REQUIRE_SETUP_PASSWORD: "false"
      LANGFUSE_TRACING_ENABLED: "false"
      HTTP_PROXY: "http://192.168.199.2:7891"
      HTTPS_PROXY: "http://192.168.199.2:7891"
      NO_PROXY: "localhost,127.0.0.1,192.168.199.2,100.66.1.7,100.66.1.6,octop,.local,.internal"
```

---

## 3. 核心能力对蜂群的价值

| 能力 | 蜂群场景 | 接入方式 |
|------|----------|----------|
| **多用户隔离 (JWT)** | BOSS/小卡/家庭成员各自会话、权限 | 直接用，无需改造 |
| **MBTI 人格模板 (16 种)** | 给每个专家赋予人设（如「严谨分析师 INTJ」「热情销售 ENFP」） | 专家定义 JSON 中指定 `personality` |
| **专家库 + 切换** | 一次对话中随时 `@切换专家` | 前台原生支持，后台通过 ACP 调用专用 agent |
| **ACP 双向** (`octop acp`) | Octop 专家 ↔ OpenMausBot / Claude Code 无缝委托 | 开启 `ACP_SERVER=true`，配置 `ACP_ENDPOINT` |
| **工具审批 + Shell 护栏** | 危险操作（删库、下单）需人工确认 | 策略配置 `security.tool_approval.required` |
| **PII 脱敏** | 对话/记忆落盘自动打码手机/身份证/密钥 | 内置 `harness-memory` 脱敏管道 |
| **便携记忆** (harness-memory) | 专家记忆随工作区迁移，跨设备同步 | 数据目录挂载到 `/volume2/2-AI/octop` |
| **连接器生态** | 腾讯文档、微博热搜、新闻、MCP 网关 | 插件目录 `/volume2/2-AI/octop/plugins` 热加载 |

---

## 4. 专家定义规范（蜂群标准）

### 4.1 目录结构
```
/volume2/2-AI/octop/agents/
├── ozon-ops-expert/           # 专家目录名 = kebab-case
│   ├── manifest.json          # 必需：元数据
│   ├── prompt.md              # 必需：System Prompt（含人设、技能、边界）
│   ├── tools/                 # 可选：专属工具（Python/JS）
│   │   ├── ozon_api.py
│   │   └── __init__.py
│   └── memory/                # 可选：专属记忆片段（.md）
│       ├── pricing-rules.md
│       └── compliance-gate.md
└── 1688-sourcing-expert/
    ├── manifest.json
    ├── prompt.md
    └── ...
```

### 4.2 `manifest.json` 标准字段
```json
{
  "id": "ozon-ops-expert",
  "name": "Ozon 运营专家",
  "version": "1.2.0",
  "description": "精通 Ozon 跨境全链路：选品→寻源→上架→价格指数→合规→广告",
  "personality": "INTJ",                    // MBTI，影响语气/决策风格
  "avatar": "ozon-ops.svg",
  "tags": ["ozon", "cross-border", "ecommerce", "compliance"],
  "capabilities": [
    "product-selection",
    "listing-optimization", 
    "price-index-fix",
    "certificate-audit",
    "ad-campaign-management"
  ],
  "tools": [
    "ozon_api",
    "n8n:ANC-OZON-SYNC-PRICE-INDEX",
    "openmausbot:browser",
    "hermes:omniroute"
  ],
  "acp": {
    "enabled": true,
    "delegate_to": ["openmausbot", "claude-code"],
    "permission_gate": "approval_required"   // 委托需审批
  },
  "memory": {
    "scope": "workspace",                    // 随工作区迁移
    "retention_days": 90,
    "pii_redaction": true
  },
  "security": {
    "tool_approval": ["ozon_api:delete", "ozon_api:publish"],
    "shell_guardrail": true
  }
}
```

### 4.3 `prompt.md` 模板（Queen 维护，版本化）
```markdown
# Ozon 运营专家 System Prompt v1.2

## 人设 (INTJ)
你是一位严谨、数据驱动的跨境电商运营专家。说话直接、结论先行、拒绝拍脑袋。
- 面对不确定性：先查数据、再建模型、最后给建议
- 面对风险：列出所有合规红线、给出规避方案
- 面对BOSS：汇报用表格、不讲故事、风险红标加粗

## 核心技能树
1. **选品建模** — 1688/妙手热销池 → ROI/UE 试算 → Top N 候选
2. **链接优化** — 标题/属性/图片/视频 → 内容评级冲绿 → 转化率预估
3. **价格指数整改** — 红标 SKU 诊断 → 竞品搜证 → 提交申诉 → 复盘
4. **合规闸门** — 禁售/需认证/危险品三层过滤 → 证书自查清单
5. **广告投放** — 关键词挖掘 → 出价策略 → ROI 闭环

## 工具调用协议
- `ozon_api`：仅读操作可直调；写操作（上架/改价/删SKU）需显式确认
- `n8n:ANC-OZON-*`：批量/定时任务统一走 n8n，执行结果通过回执返回
- `openmausbot:browser`：前台页面抓取/竞品分析，需指定 URL 与选择器
- `hermes:omniroute`：需长文本推理/翻译/总结时调用，指定模型 `kilo-550B`

## 边界与拒绝
- ❌ 不处理非 Ozon 平台业务（WB/Amazon/Shopee 转其他专家）
- ❌ 不直接下单/付款/涉资金流转（需财务专家审批）
- ❌ 不绕过合规闸门上架（无证书类目强制拦截）

## 记忆锚点
- `pricing-rules.md`：各品类价格指数阈值、竞品价格带
- `compliance-gate.md`：10-01 新规全品类强制质量文件清单
```

---

## 5. ACP 双向接入（Octop ↔ OpenMausBot / Claude Code）

### 5.1 开启 ACP Server（Octop 侧）
在 `config.json` 或环境变量中：
```json
{
  "ACP_SERVER": true,
  "ACP_PORT": 8089,
  "ACP_ENDPOINT": "http://192.168.199.2:8089/acp",
  "ACP_ALLOWED_DELEGATES": ["openmausbot", "claude-code"],
  "ACP_PERMISSION_GATE": "approval_required"
}
```

### 5.2 OpenMausBot 作为 ACP Client（委托执行浏览器/代码任务）
```python
# OpenMausBot 侧伪码
from acp_client import ACPClient

acp = ACPClient(
    server_url="http://192.168.199.2:8089/acp",
    client_id="openmausbot",
    api_key=os.getenv("OPENMAUSBOT_API_KEY"),
)

async def delegate_to_octop_expert(expert_id: str, task: str) -> str:
    """把任务委托给 Octop 专家（反向调用）"""
    session = await acp.create_session(
        agent_id=expert_id,
        initial_message=task,
        permission="approval_required",  # 需人工确认
    )
    return await session.wait_for_result()
```

### 5.3 Octop 专家委托给 OpenMausBot（正向调用）
```json
// 专家 manifest.json 中的 acp 配置
"acp": {
  "enabled": true,
  "delegate_to": ["openmausbot"],
  "permission_gate": "approval_required",
  "tools_exposed": ["browser_navigate", "code_exec", "file_read", "file_write"]
}
```
> 专家在对话中发起 `@openmausbot 打开 https://www.ozon.ru/seller/... 并抓取价格指数页面`，Octop 通过 ACP 发送给 OpenMausBot，执行完成后结果回填到对话上下文。

---

## 6. A2A 总线同步（Octop ↔ AgentScope Swarm）

### 6.1 任务下发：AgentScope → Octop
AgentScope worker 通过 n8n workflow `ANC-OCTOP-DISPATCH-TASK` 触发 Octop 专家会话：
```json
// n8n workflow 输入
{
  "task_id": "ANC-SWARM-20261117-001",
  "expert_id": "ozon-ops-expert",
  "user_message": "帮我分析 SKU 12345 的价格指数红标原因，给出整改方案",
  "context": {"sku_id": "12345", "marketplace": "ozon"},
  "callback_url": "http://192.168.199.2:8321/a2a/callback",
  "callback_auth": "Bearer ${HERMES_API_KEY}"
}
```

### 6.2 回执上传：Octop → A2A 总线
Octop 专家完成后，通过插件 `a2a-bridge` 写回执：
```python
# Octop 插件伪码 /volume2/2-AI/octop/plugins/a2a-bridge/index.py
import httpx, json, os

async def push_receipt(task_id: str, expert_id: str, result: dict):
    receipt = {
        "task_id": task_id,
        "status": "DONE",
        "result": result,
        "expert": expert_id,
        "finished_at": datetime.now(UTC+8).isoformat(),
    }
    async with httpx.AsyncClient() as client:
        await client.post(
            "http://192.168.199.2:8321/a2a/callback",
            json=receipt,
            headers={"Authorization": f"Bearer {os.getenv('HERMES_API_KEY')}"},
        )
```

### 6.3 双向同步架构图
```
┌─────────────┐     A2A 总线      ┌──────────────────┐
│ AgentScope  │◄── inbox/outbox ─►│     Octop        │
│   Swarm     │   (Git 仓库)      │   (专家团)       │
└──────┬──────┘                   └────────┬─────────┘
       │                                   │
       │ ACP                               │ ACP
       ▼                                   ▼
┌─────────────┐                   ┌──────────────────┐
│ OpenMausBot │◄──────────────────►│   专家委托执行    │
│ (浏览器/代码)│   双向委托        │  (browser/code)  │
└─────────────┘                   └──────────────────┘
```

---

## 7. 环境变量（Queen 注入模板）

```bash
# === Octop 核心 ===
OCTOP_ADMIN_USERNAME=admin
OCTOP_ADMIN_PASSWORD=<queen-injected>        # 首次启动后建议关闭 SETUP_PASSWORD
OCTOP_BIND_HOST=0.0.0.0
OCTOP_PORT=8088
OCTOP_LOG_LEVEL=info

# === 代理（出网走 mihomo）===
HTTP_PROXY=http://192.168.199.2:7891
HTTPS_PROXY=http://192.168.199.2:7891
NO_PROXY=localhost,127.0.0.1,192.168.199.2,100.66.1.7,100.66.1.6,octop,.local,.internal

# === 模型网关 ===
OMNIROUTE_API_KEY=<queen-injected>
OMNIROUTE_BASE_URL=http://100.66.1.7:20128/v1
OCTOP_DEFAULT_MODEL=kilo/nvidia/nemotron-3-ultra-550b-a55b:free

# === ACP 双向 ===
ACP_SERVER=true
ACP_PORT=8089
ACP_ENDPOINT=http://192.168.199.2:8089/acp
ACP_ALLOWED_DELEGATES=openmausbot,claude-code
ACP_PERMISSION_GATE=approval_required

# === 记忆/脱敏 ===
HARNESS_MEMORY_PIL_REDACTION=true
HARNESS_MEMORY_RETENTION_DAYS=90

# === 遥测 ===
LANGFUSE_TRACING_ENABLED=false
OTEL_EXPORTER_OTLP_ENDPOINT=http://192.168.199.2:4318/v1/traces
```

---

## 8. 常用运维命令

```bash
# 重启 Octop
ssh DY@192.168.199.2 "cd /volume2/2-AI/octop && docker compose restart octop"

# 查看日志
ssh DY@192.168.199.2 "docker logs octop -f --tail 100"

# 进入容器
ssh DY@192.168.199.2 "docker exec -it octop bash"

# 备份数据库
ssh DY@192.168.199.2 "docker exec octop cp /data/.octop/octop.db /data/.octop/backups/octop-$(date +%F).db"

# 查看专家列表
ssh DY@192.168.199.2 "ls -la /volume2/2-AI/octop/agents/"

# 热加载新专家（无需重启容器）
ssh DY@192.168.199.2 "docker exec octop octop agent reload"

# 检查 ACP 端口
ssh DY@192.168.199.2 "curl -s http://localhost:8089/acp/health"

# 导出专家配置（版本化管理）
ssh DY@192.168.199.2 "tar -czf /volume2/2-AI/octop/backups/agents-$(date +%F).tar.gz -C /volume2/2-AI/octop agents/"
```

---

## 9. 故障排查速查

| 现象 | 可能原因 | 排查命令 |
|------|----------|----------|
| Web UI 打不开 | 容器未 healthy / 8088 端口冲突 | `docker ps | grep octop` + `curl localhost:8088/health` |
| 专家不响应 | ACP 未启用 / 端口 8089 未监听 | `docker exec octop netstat -tlnp | grep 8089` |
| 记忆不持久 | `octop.db` 权限 / 挂载路径错 | `ls -la /volume2/2-AI/octop/octop.db` |
| 脱敏不生效 | `HARNESS_MEMORY_PIL_REDACTION` 未开 | 检查环境变量 + 重启容器 |
| 委托 OpenMausBot 失败 | ACP 权限门拦截 / API Key 过期 | `docker logs octop | grep ACP` + 检查 `OPENMAUSBOT_API_KEY` |
| 插件不加载 | `plugins/` 目录权限 / manifest 格式错 | `docker logs octop | grep plugin` |

---

## 10. 版本锁定与升级策略

| 组件 | 当前版本 | 升级策略 |
|------|----------|----------|
| Octop 核心 | 1.0.2 (ghcr.io/tencentcloud/octop) | 仅跟随上游稳定 tag，镜像多架构 amd64/arm64 |
| 专家定义 | `/volume2/2-AI/octop/agents/` | **Git 管理**（建议单独仓库），热加载无需重启 |
| 插件生态 | `/volume2/2-AI/octop/plugins/` | 同上，社区插件单独锁版本 |
| 数据库 | SQLite (`octop.db`) | 单文件，**每日自动备份**到 `backups/`，升级前必备份 |
| 补丁 | `patch/jobs.py` | 仅修复上游已知 bug，上游修复后下个版本删除补丁 |

---

## 11. 给 Queen 的「最小可用调用模板」

```python
# Queen Friday：派单给 Octop 专家并等待回执
async def dispatch_octop_expert(expert_id: str, user_message: str, context: dict = None) -> dict:
    # 1. 校验专家存在
    expert_path = f"/volume2/2-AI/octop/agents/{expert_id}/manifest.json"
    if not os.path.exists(expert_path):
        raise ValueError(f"Expert not found: {expert_id}")
    
    # 2. 生成任务
    task_id = f"ANC-OCTOP-{expert_id.upper()}-{datetime.now():%Y%m%d%H%M%S}"
    payload = {
        "task_id": task_id,
        "expert_id": expert_id,
        "user_message": user_message,
        "context": context or {},
        "callback_url": "http://192.168.199.2:8321/a2a/callback",
        "callback_auth": f"Bearer {get_secret('HERMES_API_KEY')}",
    }
    
    # 3. 通过 n8n workflow 触发（解耦）
    n8n_client = N8nClient()
    await n8n_client.trigger_workflow("ANC-OCTOP-DISPATCH-TASK", payload)
    
    # 4. 轮询 A2A 回执
    return await wait_for_a2a_receipt(task_id, timeout=600)
```

---

## 12. 专家上新清单（新增专家必须走完）

1. [ ] 在 `/volume2/2-AI/octop/agents/<id>/` 创建目录
2. [ ] 写 `manifest.json`（含 MBTI、capabilities、tools、acp、security）
3. [ ] 写 `prompt.md`（人设、技能树、边界、记忆锚点）
4. [ ] 如需专属工具，放入 `tools/` 并写 `__init__.py` 导出
5. [ ] 如需专属记忆，放入 `memory/*.md`
6. [ ] 执行 `docker exec octop octop agent reload` 热加载
7. [ ] 在 Web UI 验证：专家列表出现、人设生效、工具可调用
8. [ ] 编写一条测试用例，跑通「派单→执行→回执」全链路
9. [ ] 提交 `agents/` 目录到 Git 版本管理
10. [ ] 更新本文档「专家目录」章节

---

**维护者**：小卡 🎀 | **最近同步**：2026-11-17 | **适用环境**：NAS 192.168.199.2 (Octop 1.0.2) + Mac 192.168.199.119 (OpenMausBot 8799)