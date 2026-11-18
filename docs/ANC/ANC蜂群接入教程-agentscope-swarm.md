# ANC 蜂群接入教程：AgentScope Swarm

> 目标：把 AgentScope 2.x 多智能体框架作为 **ANC 蜂群的「大脑执行层」** 接入。所有 AgentScope agents 通过 **A2A 总线** 领取任务、写回执，模型调用统一走 **OmniRoute (100.66.1.7:20128)**，工具层复用 **n8n (ETL)、OpenMausBot (浏览器/代码/模型)、Hermes (NAS 统一网关 8321)**。

---

## 1. 架构定位

| 层级 | 组件 | 在蜂群中的角色 |
|------|------|----------------|
| 编排 | **AgentScope Swarm** | 5 专用 agents (Main/Sourcing/Ops/SupplyChain/Finance) 以 `TeamPipeline` 协作 |
| 总线 | **A2A Git 总线** (`/volume1/2-AI/A2A`) | 任务下发 `inbox/task_*.json`、回执收集 `outbox/receipt_*.json` |
| 模型 | **OmniRoute** | 统一网关，免费池首选 `kilo/nvidia/nemotron-3-ultra-550b-a55b:free` |
| 工具 | **n8n / OpenMausBot / Hermes** | 确定性 ETL / 浏览器&代码执行 / 本地统一网关 |
| 存储 | **PGVector + Neo4j + Redis + NATS** | 共享记忆、向量检索、图谱、事件总线、对象存储 |

---

## 2. 部署现状（已在 NAS 跑通）

```
NAS 192.168.199.2
├─ /home/DY/agentscope_deploy/docker-compose.yml   # 5 服务栈
│   ├─ agentscope-swarm     (image: agentscope-swarm:2.0.11, port 8789)
│   ├─ agentscope-postgres  (pgvector/pgvector:pg16, port 5435)
│   ├─ agentscope-redis     (redis:7-alpine, port 6380)
│   ├─ agentscope-neo4j     (neo4j:5.15, ports 7474/7687)
│   └─ agentscope-nats      (nats:2.10-alpine, ports 4222/8222)
└─ network: agentscope_deploy_swarm-net (bridge)
```

**关键挂载**：
- `/volume2/2-AI` → `/workspace/data:rw`          # 共享知识库、文件
- `/volume1/2-AI/A2A` → `/opt/a2a:rw`             # **A2A 总线（核心）**
- `/volume2/hermes/data/agentscope_swarm` → `/workspace/code:ro`  # 代码只读层
- `/volume2/hermes/data/.venv-browser` → 同路径 `ro`  # 浏览器 venv 复用

**启动命令**（生产守护模式）：
```yaml
command: ["python3", "/opt/a2a/scripts/swarm_a2a_daemon.py"]
```
> `swarm_a2a_daemon.py` 轮询 `/opt/a2a/inbox`，仅认领 `assigned_to` 命中本 worker 的任务，调用 `main.py --goal` 执行，写 `outbox` 回执并移入 `processed/`。

---

## 3. 环境变量（Queen Friday 注入模板）

Queen 在派单时把下列变量写入任务 `env` 字段，AgentScope 容器启动时自动加载（来自 `.env` + 任务级覆盖）：

```bash
# === OmniRoute（统一模型网关）===
OMNIROUTE_API_KEY=<queen-injected>          # OmniRoute Bearer Token
OMNIROUTE_BASE_URL=http://100.66.1.7:20128/v1

# === 飞书网关（群内 @bot、文件透传）===
FEISHU_APP_ID=<queen-injected>
FEISHU_APP_SECRET=<queen-injected>
FEISHU_VERIFICATION_TOKEN=<queen-injected>
FEISHU_ENCRYPT_KEY=<queen-injected>

# === 共享存储 ===
POSTGRES_HOST=agentscope-postgres
POSTGRES_PORT=5432
POSTGRES_DB=agentscope
POSTGRES_USER=swarm
POSTGRES_PASSWORD=<queen-injected>

REDIS_HOST=agentscope-redis
REDIS_PORT=6379
REDIS_PASSWORD=<queen-injected>
REDIS_DB=0

NEO4J_URI=bolt://agentscope-neo4j:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=<queen-injected>

# === 工具层网关 ===
N8N_API_KEY=<queen-injected>
N8N_BASE_URL=http://192.168.199.2:5679        # n8n 反代后的公网/内网地址
OPENMAUSBOT_API_KEY=<queen-injected>
OPENMAUSBOT_BASE_URL=http://192.168.199.119:8799  # Mac OpenMausBot Web UI
HERMES_API_KEY=<queen-injected>
HERMES_BASE_URL=http://192.168.199.2:8321      # NAS Hermes 统一网关

# === 遥测 ===
OTEL_EXPORTER_OTLP_ENDPOINT=http://192.168.199.2:4318/v1/traces

# === A2A 总线根（容器内）===
A2A_ROOT=/opt/a2a
```

> **安全**：所有 `<queen-injected>` 由 Queen Friday 在任务下发时通过 `env` 字段注入，**不写死在镜像、不进 Git、不落盘明文**。

---

## 4. AgentScope 代码侧接入要点（给 Queen 调用的 SDK 模式）

### 4.1 模型创建 —— 必须用 `OpenAICredential` 包装
```python
from agentscope.models import OpenAIChatWrapper
from agentscope.credential import OpenAICredential

cred = OpenAICredential(
    api_key=os.getenv("OMNIROUTE_API_KEY"),
    base_url=os.getenv("OMNIROUTE_BASE_URL"),
)
model = OpenAIChatWrapper(
    model_name="kilo/nvidia/nemotron-3-ultra-550b-a55b:free",  # free-pool 首选
    credential=cred,
    max_tokens=4096,              # 推理型模型 <200 会空正文
    stream=True,
)
```

### 4.2 Tool 注册 —— `FunctionTool + Toolkit`（非 `@tool` 装饰器）
```python
from agentscope.tool import FunctionTool, Toolkit

def my_tool(arg1: str, arg2: int) -> dict:
    """工具说明（给模型看）"""
    return {"result": f"{arg1}-{arg2}"}

ft = FunctionTool(
    func=my_tool,
    input_schema={"type": "object", "properties": {
        "arg1": {"type": "string"}, "arg2": {"type": "integer"}
    }, "required": ["arg1", "arg2"]},
)
toolkit = Toolkit(tools=[ft])
agent = ReactAgent(model=model, toolkit=toolkit, ...)
```

### 4.3 Pipeline 运行 —— 必须用 `TeamPipeline.reply_stream`（async generator）
```python
from agentscope.pipeline import TeamPipeline, TeamMember
from agentscope.message import Msg

pipeline = TeamPipeline(
    members=[
        TeamMember(agent=main_agent, name="Main"),
        TeamMember(agent=sourcing_agent, name="Sourcing"),
        TeamMember(agent=ops_agent, name="Ops"),
        TeamMember(agent=supply_agent, name="SupplyChain"),
        TeamMember(agent=finance_agent, name="Finance"),
    ],
    leader="Main",
)

# 任务输入：Msg.content 必须是 list[{"type": "text", "text": "..."}]
msg = Msg(name="user", role="user", content=[{"type": "text", "text": goal}])

async for chunk in pipeline.reply_stream(msg):
    # chunk 是 Msg 对象，逐块处理/落盘
    await write_receipt(chunk)
```

### 4.4 A2A 总线守护进程（已内置在 `swarm_a2a_daemon.py`）
```python
# 伪码逻辑（实际见 /opt/a2a/scripts/swarm_a2a_daemon.py）
while True:
    for task_file in sorted(glob("/opt/a2a/inbox/task_*.json")):
        task = json.load(open(task_file))
        if task.get("assigned_to") != MY_WORKER_ID:
            continue
        goal = task["goal"]
        # 执行 pipeline
        result = await run_pipeline(goal)
        # 写回执
        receipt = {"task_id": task["task_id"], "status": "DONE", "result": result, ...}
        atomic_write(f"/opt/a2a/outbox/receipt_{task['task_id']}_xiaoka.json", receipt)
        # 移入 processed/
        shutil.move(task_file, f"/opt/a2a/processed/{os.path.basename(task_file)}")
    time.sleep(30)  # SWARM_POLL_INTERVAL
```

---

## 5. Queen Friday 调用协议（A2A 任务格式）

Queen 下发任务到 `inbox/`，AgentScope worker 自动拾取：

```json
{
  "task_id": "ANC-SWARM-20261117-001",
  "from": "friday",
  "to": "xiaoka",
  "assigned_to": "agentscope-swarm-worker-1",
  "goal": "对比 1688 与 妙手 热销池前 50 SKU 的 ROI，输出 Top 10 候选",
  "env": {
    "OMNIROUTE_API_KEY": "sk-...",
    "N8N_API_KEY": "n8n-...",
    "OPENMAUSBOT_API_KEY": "omb-..."
  },
  "tools_allowed": ["n8n", "openmausbot", "hermes"],
  "timeout_sec": 1800,
  "created_at": "2026-11-17T10:00:00+08:00"
}
```

Worker 完成后写回执到 `outbox/`：
```json
{
  "task_id": "ANC-SWARM-20261117-001",
  "status": "DONE",
  "result": {"top_10": [...], "roi_table": "..."},
  "worker": "agentscope-swarm-worker-1",
  "finished_at": "2026-11-17T10:15:32+08:00"
}
```

---

## 6. 常用运维命令

```bash
# 启动/重启全栈
ssh DY@192.168.199.2 "cd /home/DY/agentscope_deploy && docker compose up -d"

# 查看 swarm worker 日志
ssh DY@192.168.199.2 "docker logs agentscope-swarm -f --tail 100"

# 进入容器调试
ssh DY@192.168.199.2 "docker exec -it agentscope-swarm bash"

# 手动跑一次 pipeline（调试用）
ssh DY@192.168.199.2 "docker exec agentscope-swarm python3 -c '
import asyncio, os
os.chdir(\"/workspace/code\")
from main import run_pipeline
asyncio.run(run_pipeline(\"测试目标\"))
'"

# 检查 A2A 总线挂载
ssh DY@192.168.199.2 "docker exec agentscope-swarm ls -la /opt/a2a/"

# 检查共享存储
ssh DY@192.168.199.2 "docker exec agentscope-swarm ls -la /workspace/data/"
```

---

## 7. 版本锁定与升级策略

| 组件 | 当前版本 | 升级策略 |
|------|----------|----------|
| AgentScope 框架 | 2.0.11 (image tag) | 仅跟随上游 `agentscope-ai/agentscope` main 分支发布的稳定 tag，镜像由 CI 自动构建推送 GHCR |
| PGVector | pg16 | 按需，数据卷 `postgres_data` 持久化，升级前 `pg_dump` |
| Neo4j | 5.15 | 同理，`neo4j_data` 持久化 |
| NATS | 2.10-alpine | JetStream 存储在 `/data`（容器内），建议迁移到宿主卷避免重启丢数据 |

> ⚠️ **NATS 存储当前在容器内 `/tmp/nats/jetstream`（临时），重启丢数据** — 后续需挂载到 `/volume2/2-AI/nats-jetstream`。

---

## 8. 故障排查速查

| 现象 | 可能原因 | 排查命令 |
|------|----------|----------|
| worker 不领任务 | `assigned_to` 不匹配 / A2A 挂载缺失 | `docker exec agentscope-swarm ls /opt/a2a/inbox/` |
| 模型调用 401/429 | OmniRoute token 过期 / 配额耗尽 | `curl -H "Authorization: Bearer $OMNIROUTE_API_KEY" $OMNIROUTE_BASE_URL/models` |
| n8n 工具调用失败 | `N8N_BASE_URL` 端口错 / Basic Auth 过期 | `curl -u n8n-admin:$N8N_BASIC_AUTH_PASSWORD $N8N_BASE_URL/api/v1/workflows` |
| OpenMausBot 无响应 | Mac 睡眠 / 8799 端口未暴露 | `ssh xiaoka@192.168.199.119 "curl -s localhost:8799/health"` |
| PGVector 连接拒绝 | postgres 容器未 healthy | `docker exec agentscope-postgres pg_isready -U swarm -d agentscope` |

---

## 9. 给 Queen 的「最小可用调用模板」

```python
# Queen Friday 内部调用示例
async def dispatch_agentscope_task(goal: str, assigned_to: str = "agentscope-swarm-worker-1"):
    task_id = f"ANC-SWARM-{datetime.now():%Y%m%d-%H%M%S}"
    task = {
        "task_id": task_id,
        "from": "friday",
        "to": "xiaoka",
        "assigned_to": assigned_to,
        "goal": goal,
        "env": inject_secrets(["OMNIROUTE_API_KEY", "N8N_API_KEY", "OPENMAUSBOT_API_KEY", "HERMES_API_KEY"]),
        "tools_allowed": ["n8n", "openmausbot", "hermes"],
        "timeout_sec": 1800,
        "created_at": datetime.now(UTC+8).isoformat(),
    }
    atomic_write(f"/opt/nas/volume2/2-AI/A2A/inbox/task_{task_id}_xiaoka.json", task)
    # 轮询 outbox/ 等回执...
    return await wait_for_receipt(task_id)
```

---

**维护者**：小卡 🎀 | **最近同步**：2026-11-17 | **适用环境**：NAS 192.168.199.2 + Mac 192.168.199.119 + Windows 192.168.199.70