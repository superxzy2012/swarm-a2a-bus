# ANC 蜂群接入教程：n8n

> 目标：把 n8n 作为 **ANC 蜂群的「确定性 ETL / 工作流编排层」** 接入。所有 AgentScope agents、OpenMausBot、Hermes 通过 **n8n REST API** 调用预置工作流，执行结果通过 **A2A 总线回执** 回传。n8n 自身不直接消费 A2A 任务，而是作为**工具层**被上层 agents 调用。

---

## 1. 架构定位

| 层级 | 组件 | 在蜂群中的角色 |
|------|------|----------------|
| 编排 | **AgentScope Swarm** | 决策大脑，决定「何时调用哪个 n8n workflow」 |
| 工具层 | **n8n** (port 5679 反代) | 400+ 集成的可视化工作流引擎，承担：数据抓取/清洗/入库、API 聚合、定时同步 |
| 总线 | **A2A Git 总线** | n8n 执行完成 → 通过 webhook / REST 回调 → 写 `outbox/receipt_*.json` |
| 模型 | **OmniRoute** | n8n 内部节点若需 LLM，统一走 OmniRoute (OpenAI 兼容) |
| 认证 | **Basic Auth + API Key** | `X-N8N-API-KEY` header + Basic Auth 双重保护 |

---

## 2. 部署现状（已在 NAS 跑通）

```
NAS 192.168.199.2
├─ /volume2/2-AI/n8n/
│   ├─ docker-compose.yml          # n8n 单容器
│   ├─ data/                       # /home/node/.n8n (工作流、凭据、加密键)
│   ├─ pool-health/                # /files (文件节点读写沙箱)
│   └─ .env                        # N8N_BASIC_AUTH_PASSWORD, N8N_ENCRYPTION_KEY
└─ 反代入口：http://192.168.199.2:5679/  (nginx/mihomo 旁路由转发 5677→5679)
```

**docker-compose.yml 关键配置**：
```yaml
services:
  n8n:
    image: n8nio/n8n:2.43.5
    container_name: n8n
    restart: unless-stopped
    ports:
      - "127.0.0.1:5677:5678"          # 仅本机回环，外部走反代 5679
    volumes:
      - /volume2/2-AI/n8n/data:/home/node/.n8n
      - /volume2/2-AI/n8n/pool-health:/files
    environment:
      N8N_HOST: "192.168.199.2"
      N8N_PORT: "5678"
      N8N_PROTOCOL: "http"
      N8N_WEBHOOK_URL: "http://192.168.199.2:5679/"
      N8N_EDITOR_BASE_URL: "http://192.168.199.2:5679/"
      N8N_BASIC_AUTH_ACTIVE: "true"
      N8N_BASIC_AUTH_USER: "n8n-admin"
      N8N_BASIC_AUTH_PASSWORD: "${N8N_BASIC_AUTH_PASSWORD}"
      N8N_ENCRYPTION_KEY: "${N8N_ENCRYPTION_KEY}"
      N8N_SECURE_COOKIE: "false"
      N8N_TRUST_PROXY: "true"
      N8N_UNVERIFIED_PACKAGES_ENABLED: "true"
      N8N_RESTRICT_FILE_ACCESS_TO: "/files"
    mem_limit: 512m
    memswap_limit: 512m
```

---

## 3. 访问方式

| 场景 | 地址 | 认证 |
|------|------|------|
| **编辑器 (UI)** | http://192.168.199.2:5679/ | Basic Auth: `n8n-admin` / `${N8N_BASIC_AUTH_PASSWORD}` |
| **REST API** | http://192.168.199.2:5679/api/v1/ | Header `X-N8N-API-KEY: ${N8N_API_KEY}` + Basic Auth |
| **Webhook 入口** | http://192.168.199.2:5679/webhook/xxx | 无需认证（由 workflow 决定） |
| **内网直连** | http://192.168.199.2:5677/ | 仅 NAS 本机可达 |

> **API Key 获取**：编辑器右上角用户菜单 → API Keys → Create → 复制（仅显示一次，落 1Password / Queen 密钥库）。

---

## 4. 蜂群标准工作流模板（Queen 预置，AgentScope 调用）

### 4.1 命名规范
```
ANC-<域>-<动词>-<对象>[-<版本>]
例：ANC-OZON-SYNC-PRICE-INDEX-v2
    ANC-1688-SOURCE-SKU-DETAILS
    ANC-DOUYIN-INGEST-VIDEO-BATCH
```

### 4.2 必有节点结构（Queen 强制模板）

```
Start (Webhook / Schedule / Manual)
    ↓
Set (参数校验、注入 env 变量、生成 run_id)
    ↓
[核心节点链] → HTTP Request / Code / Database / 等
    ↓
IF (成功/失败分支)
    ├── True → Set (标准化输出) → Respond to Webhook / HTTP Request (回调 A2A)
    └── False → Set (错误结构化) → Respond to Webhook / HTTP Request (回调 A2A)
```

### 4.3 标准输入（Webhook Body / Start 参数）
```json
{
  "task_id": "ANC-OZON-SYNC-20261117-001",
  "run_id": "wf-20261117-100000-abc123",
  "params": { "sku_ids": ["123", "456"], "force": true },
  "callback_url": "http://192.168.199.2:8321/a2a/callback",  # Hermes 接收回执
  "callback_auth": "Bearer ${HERMES_API_KEY}"
}
```

### 4.4 标准输出（回调 A2A / Hermes）
```json
POST ${callback_url}
Headers: Authorization: ${callback_auth}, Content-Type: application/json
Body:
{
  "task_id": "ANC-OZON-SYNC-20261117-001",
  "run_id": "wf-20261117-100000-abc123",
  "status": "DONE",           // DONE / FAILED / PARTIAL
  "output": { "synced": 42, "failed": 3, "details": [...] },
  "error": null,              // 失败时填入错误码+堆栈
  "finished_at": "2026-11-17T10:05:23+08:00"
}
```

---

## 5. AgentScope / Hermes 调用 n8n 的 SDK 模式

### 5.1 Python 客户端（内置于 AgentScope `tools/n8n_adapter.py`）
```python
import httpx
from dataclasses import dataclass

@dataclass
class N8nClient:
    base_url: str = "http://192.168.199.2:5679"  # 反代后的公网地址
    api_key: str = os.getenv("N8N_API_KEY")
    basic_user: str = "n8n-admin"
    basic_pass: str = os.getenv("N8N_BASIC_AUTH_PASSWORD")
    timeout: float = 300.0

    def _headers(self) -> dict:
        return {"X-N8N-API-KEY": self.api_key, "Content-Type": "application/json"}

    def _auth(self) -> tuple:
        return (self.basic_user, self.basic_pass)

    async def trigger_workflow(self, workflow_id: str, payload: dict) -> dict:
        """触发 workflow 执行（异步，立即返回 execution_id）"""
        async with httpx.AsyncClient(timeout=self.timeout) as client:
            r = await client.post(
                f"{self.base_url}/api/v1/workflows/{workflow_id}/execute",
                json=payload,
                headers=self._headers(),
                auth=self._auth(),
            )
            r.raise_for_status()
            return r.json()  # {"executionId": "...", "finished": false, ...}

    async def wait_execution(self, execution_id: str, poll_interval: int = 5) -> dict:
        """轮询执行结果直到完成"""
        async with httpx.AsyncClient(timeout=self.timeout) as client:
            while True:
                r = await client.get(
                    f"{self.base_url}/api/v1/executions/{execution_id}",
                    headers=self._headers(),
                    auth=self._auth(),
                )
                r.raise_for_status()
                data = r.json()
                if data.get("finished"):
                    return data
                await asyncio.sleep(poll_interval)

    async def run_and_wait(self, workflow_id: str, payload: dict) -> dict:
        """触发 + 等待完成，返回标准化结果"""
        exec_info = await self.trigger_workflow(workflow_id, payload)
        result = await self.wait_execution(exec_info["executionId"])
        return {
            "status": "DONE" if result.get("status") == "success" else "FAILED",
            "output": result.get("data", {}).get("resultData", {}),
            "error": result.get("data", {}).get("error"),
        }
```

### 5.2 FunctionTool 包装（给 AgentScope agents 用）
```python
from agentscope.tool import FunctionTool

n8n_client = N8nClient()

async def run_n8n_workflow(workflow_name: str, params: dict) -> dict:
    """调用 ANC 标准 n8n workflow
    Args:
        workflow_name: ANC-xxx 模板名（会自动解析为 workflow_id）
        params: 业务参数，会注入 task_id/run_id/callback
    Returns:
        标准化输出 {status, output, error}
    """
    # 解析 workflow_id（可维护 name→id 映射表，或按 tag 筛选）
    workflow_id = await resolve_workflow_id(workflow_name)
    payload = {
        "task_id": f"ANC-{workflow_name}-{datetime.now():%Y%m%d%H%M%S}",
        "run_id": f"wf-{uuid.uuid4().hex[:8]}",
        "params": params,
        "callback_url": "http://192.168.199.2:8321/a2a/callback",
        "callback_auth": f"Bearer {os.getenv('HERMES_API_KEY')}",
    }
    return await n8n_client.run_and_wait(workflow_id, payload)

n8n_tool = FunctionTool(
    func=run_n8n_workflow,
    input_schema={
        "type": "object",
        "properties": {
            "workflow_name": {"type": "string", "description": "ANC 模板名，如 OZON-SYNC-PRICE-INDEX"},
            "params": {"type": "object", "description": "业务参数 JSON"},
        },
        "required": ["workflow_name", "params"],
    },
)
toolkit = Toolkit(tools=[n8n_tool])
```

---

## 6. Queen Friday 维护的 Workflow 目录（动态注册）

Queen 启动时扫描 n8n 所有 workflow，按 `tags` 中的 `anc:` 前缀建立索引：

```python
# Queen 侧伪码
async def refresh_n8n_catalog():
    workflows = await n8n_client.list_workflows(active_only=True)
    catalog = {}
    for wf in workflows:
        for tag in wf.get("tags", []):
            if tag.startswith("anc:"):
                key = tag[4:]  # "anc:OZON-SYNC-PRICE-INDEX" → "OZON-SYNC-PRICE-INDEX"
                catalog[key] = wf["id"]
    await redis.hset("anc:n8n:catalog", mapping=catalog)
```

AgentScope 调用时只需传 `workflow_name`，SDK 自动解析 `workflow_id`。

---

## 7. 常用运维命令

```bash
# 重启 n8n
ssh DY@192.168.199.2 "cd /volume2/2-AI/n8n && docker compose restart n8n"

# 查看日志
ssh DY@192.168.199.2 "docker logs n8n -f --tail 100"

# 进入容器
ssh DY@192.168.199.2 "docker exec -it n8n bash"

# 导出所有 workflow（备份）
ssh DY@192.168.199.2 "docker exec n8n n8n export:workflow --all --output=/files/workflows-backup-$(date +%F).json"

# 导入 workflow
ssh DY@192.168.199.2 "docker exec n8n n8n import:workflow --input=/files/ANC-OZON-SYNC-PRICE-INDEX.json"

# 检查 API Key 是否有效
curl -H "X-N8N-API-KEY: $N8N_API_KEY" -u n8n-admin:$N8N_BASIC_AUTH_PASSWORD \
  http://192.168.199.2:5679/api/v1/workflows | jq '.data | length'

# 查看执行历史
curl -H "X-N8N-API-KEY: $N8N_API_KEY" -u n8n-admin:$N8N_BASIC_AUTH_PASSWORD \
  "http://192.168.199.2:5679/api/v1/executions?limit=20&status=failed" | jq
```

---

## 8. 故障排查速查

| 现象 | 可能原因 | 排查命令 |
|------|----------|----------|
| 401 Unauthorized | API Key 过期 / Basic Auth 密码错 | `curl -v -H "X-N8N-API-KEY: $KEY" -u n8n-admin:$PASS http://192.168.199.2:5679/api/v1/workflows` |
| 502 Bad Gateway | 反代 5679→5677 挂了 / n8n 容器未 healthy | `docker ps | grep n8n` + `curl localhost:5677/healthz` |
| Workflow 执行卡住 | 节点死循环 / 外部 API 超时 / 内存超限 | `docker stats n8n` + 查看执行日志 `docker logs n8n` |
| 加密凭据解密失败 | `N8N_ENCRYPTION_KEY` 变更 / 数据卷损坏 | 检查 `.env` 与 `data/config` 一致性，必要时重建容器保留 `data/` |
| 文件节点读写失败 | `N8N_RESTRICT_FILE_ACCESS_TO` 路径不对 | 确认 `/files` 挂载到 `/volume2/2-AI/n8n/pool-health` |

---

## 9. 版本锁定与升级策略

| 组件 | 当前版本 | 升级策略 |
|------|----------|----------|
| n8n 核心 | 2.43.5 | 仅跟随 LTS tag，升级前在测试环境跑全量 workflow 回归 |
| 节点包 | n8n 默认捆绑 | `N8N_UNVERIFIED_PACKAGES_ENABLED=true` 允许社区节点，社区节点单独锁版本 |
| 数据卷 | `/volume2/2-AI/n8n/data` | **绝不删除**，升级只换镜像 tag，数据向后兼容 |

---

## 10. 给 Queen 的「最小可用调用模板」

```python
# Queen Friday 内部：触发 n8n workflow 并等待结果
async def dispatch_n8n_workflow(workflow_name: str, params: dict, timeout: int = 300) -> dict:
    client = N8nClient()
    workflow_id = await redis.hget("anc:n8n:catalog", workflow_name)
    if not workflow_id:
        raise ValueError(f"Unknown workflow: {workflow_name}")
    
    payload = {
        "task_id": f"ANC-N8N-{workflow_name}-{datetime.now():%Y%m%d%H%M%S}",
        "run_id": f"wf-{uuid.uuid4().hex[:8]}",
        "params": params,
        "callback_url": "http://192.168.199.2:8321/a2a/callback",
        "callback_auth": f"Bearer {get_secret('HERMES_API_KEY')}",
    }
    return await client.run_and_wait(workflow_id, payload)
```

---

**维护者**：小卡 🎀 | **最近同步**：2026-11-17 | **适用环境**：NAS 192.168.199.2 (n8n 2.43.5) + 反代 5679