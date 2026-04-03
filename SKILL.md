---
name: videoinu
version: "1.0.0"
description: "Videoinu 平台技能 — 通过 Graph (画布) 系统管理项目、文件和 AI Agent 会话。Use when: user mentions Videoinu, Graph management, uploading files to Videoinu, agent chat on Videoinu, or running Videoinu workflows."
metadata:
  {
    "openclaw":
      {
        "emoji": "🎬",
        "requires": { "bins": ["python3"] },
      },
  }
---

# videoinu-skill

> Videoinu 平台技能 — 通过 Videoinu 的 Graph (画布) 系统管理项目、文件和 AI Agent 会话

## 重要：使用方式

**必须通过本 skill 提供的 Python 脚本与 Videoinu 交互。不要使用 mcporter、MCP、curl 或任何其他方式直接调用 API。**

所有脚本位于本 skill 的 `scripts/` 目录下。Token 已保存在 `~/.videoinu/credentials.json`，脚本会自动读取。

示例：列出项目请运行 `python3 <skill_scripts_dir>/list_graphs.py`，不要自行构造 HTTP 请求。

## 概述

videoinu-skill 提供一组 Python 脚本，用于与 Videoinu 平台交互。覆盖以下核心能力：

1. **Graph 管理** — 创建、列出、查看 Graph（项目画布）及其中的 ViewNode / CoreNode
2. **文件上传/下载** — 上传本地文件到平台（创建 CoreNode），或下载 Graph 中的文件到本地
3. **Agent 会话** — 创建 Agent 会话，通过 WebSocket 与 AI Agent 对话
4. **Workflow 执行** — 运行 Workflow 定义，查询执行状态

### 必需环境

- **二进制**: `python3` (3.9+)
- **环境变量**: `VIDEOINU_ACCESS_KEY`（必填）
- **可选环境变量**: `VIDEOINU_API_BASE`（默认 `https://videoinu.com`）
- **无第三方依赖**: 所有脚本仅使用 Python 标准库

### 鉴权

所有请求通过 Cookie 鉴权: `Cookie: token=<VIDEOINU_ACCESS_KEY>`

### 获取和保存 Access Key

用户获取 Access Key 的方式:
1. 登录 https://videoinu.com
2. 进入 Profile 页面 → 点击 **Copy Access Key**

保存 Access Key（二选一）:

**方式 A: 使用 auth.py 保存到本地（推荐）**
```bash
python3 auth.py save "your-access-key"
# Token 保存到 ~/.videoinu/credentials.json (仅当前用户可读)
# 后续所有脚本自动读取，无需环境变量
```

**方式 B: 环境变量**
```bash
export VIDEOINU_ACCESS_KEY="your-access-key"
```

Token 读取优先级: 环境变量 > `~/.videoinu/credentials.json`

验证和管理:
```bash
python3 auth.py status   # 查看当前认证状态
python3 auth.py verify   # 验证 token 是否有效
python3 auth.py logout   # 删除保存的 token
```

若用户尚未登录，引导其前往 https://videoinu.com/login 注册/登录后再获取 Key。

**安全警告**: 绝对不要将 Access Key 硬编码到脚本文件中。Access Key 是 JWT token，包含用户身份信息，泄露会导致账号被盗用。请使用 `auth.py save` 或环境变量。

---

## 脚本清单

| 脚本 | 功能 | 输入 | 输出 |
|------|------|------|------|
| `auth.py` | 保存/验证/管理 Access Key | `save`/`status`/`verify`/`logout` | 认证状态 |
| `list_graphs.py` | 列出用户的 Graph | `--page-size`, `--tag` | Graph 列表 |
| `get_graph.py` | 查看 Graph 详情（ViewNode + CoreNode） | `GRAPH_ID` | 过滤后的节点信息 |
| `create_graph.py` | 创建新 Graph | `NAME`, `--tag` | Graph ID + URL |
| `upload_file.py` | 上传文件创建 CoreNode | 文件路径 | CoreNode ID + URL |
| `download_file.py` | 下载 Graph 中的文件 | `GRAPH_ID` 或 `--urls` | 本地文件路径 |
| `create_session.py` | 创建 Agent 会话 | `GRAPH_ID`, `--list` | Session ID |
| `agent_chat.py` | 与 Agent 对话 | `SESSION_ID`, 消息 | Agent 回复 |
| `run_workflow.py` | 运行 Workflow | `DEFINITION_ID`, inputs | Instance ID |
| `query_workflow.py` | 查询 Workflow 状态 | `INSTANCE_ID`, `--poll` | 执行状态 |

---

## 核心概念

### Graph（画布/项目）
Graph 是 Videoinu 的项目容器，包含:
- **ViewNode**: 画布上的可视节点，有位置、标签、连接关系
- **CoreNode**: 底层数据节点，代表实际的资产（图片、视频、音频、文本）或操作（Workflow 产物）
- **Connection**: ViewNode 之间的连线，表示数据流
- **Group**: ViewNode 的分组

每个 ViewNode 可以引用一个或多个 CoreNode（`core_refs`），通过 `selected_core_id` 指定当前选中的版本。

### CoreNode 类型
- `asset`: 资产节点
  - `asset_type`: `image` | `video` | `audio` | `text` | `json` | `file`
  - `source_type`: `upload` | `import` | `generated`
  - 有 `url`（媒体文件）或 `content`（文本内容）
- `operation`: 操作节点（Workflow 执行产物）
  - `status`: `pending` | `completed` | `failed`

### Agent 会话
Agent 是绑定在 Graph 上的 AI 助手。通过 WebSocket 连接进行实时对话。
- 一个 Graph 对应一个 Agent Project
- 一个 Project 可以有多个 Session（会话）
- Agent 可以调用工具（Tool）操作 Graph 中的节点

### Workflow
预定义的自动化流程，接收输入（CoreNode 引用）并产出新的 CoreNode。

---

## 典型工作流

### 场景 1: 浏览用户项目

```bash
# 1. 列出所有 Graph
python3 list_graphs.py

# 2. 查看某个 Graph 的详情
python3 get_graph.py GRAPH_ID
```

### 场景 2: 创建新项目并上传文件

```bash
# 1. 创建新 Graph（自动带 free-mode tag，用户才能在界面上看见）
python3 create_graph.py "My New Project"
# → 得到 graph_id

# 2. 上传参考文件
python3 upload_file.py /path/to/reference.png
# → 得到 core_node_id, file_url

# 3. 查看 Graph 确认
python3 get_graph.py GRAPH_ID
```

### 场景 3: 与 Agent 对话

```bash
# 1. 创建会话
python3 create_session.py GRAPH_ID
# → 得到 session_id

# 2. 发送消息
python3 agent_chat.py SESSION_ID "帮我分析一下这个项目的结构"

# 3. 发送带文件引用的消息
python3 agent_chat.py SESSION_ID "看看这个图片 {{@core_node:CORE_NODE_ID:image.png}}"

# 4. 查看已有会话
python3 create_session.py GRAPH_ID --list
```

### 场景 4: 上传文件后让 Agent 处理

```bash
# 1. 上传文件
python3 upload_file.py /path/to/video.mp4
# → core_node_id = "abc123"

# 2. 创建会话（如果还没有）
python3 create_session.py GRAPH_ID
# → session_id = "sess456"

# 3. 让 Agent 处理这个文件
python3 agent_chat.py sess456 "请帮我剪辑这个视频 {{@core_node:abc123:video.mp4}}" --auto-approve
```

### 场景 5: 运行 Workflow

```bash
# 1. 查看可用的 Workflow 定义
python3 run_workflow.py --list

# 2. 在现有 Graph 中运行
python3 run_workflow.py DEF_ID --graph-id GRAPH_ID \
  --inputs '{"input_image": {"type": "core_node_refs", "core_node_ids": ["NODE_ID"]}}'
# → 得到 instance_id

# 3. 轮询执行状态直到完成
python3 query_workflow.py INSTANCE_ID --poll
```

### 场景 6: 下载 Graph 中的生成结果

```bash
# 下载 Graph 中所有图片
python3 download_file.py GRAPH_ID --type image --output-dir ./results

# 下载所有视频
python3 download_file.py GRAPH_ID --type video

# 直接下载指定 URL
python3 download_file.py --urls "https://..." "https://..." --output-dir ./output
```

---

## Agent 引用格式

在发送给 Agent 的消息中，可以引用 Graph 中的节点:

```
{{@core_node:CORE_NODE_ID:display_name}}
{{@view_node:VIEW_NODE_ID:display_name}}
```

例如:
```
请分析这张图片 {{@core_node:a1b2c3d4:sunset.png}}
```

Agent 会根据引用获取对应的 CoreNode 内容。

---

## 输出格式

所有脚本输出 JSON 到 stdout，错误输出到 stderr。

**成功**:
```json
{
  "graphs": [...],
  "has_more": false
}
```

**错误**:
```json
{
  "error": "VIDEOINU_ACCESS_KEY is not set. Run: export VIDEOINU_ACCESS_KEY=\"your-access-key\""
}
```

---

## 核心原则

1. **忠实传达用户意图**: 将用户的需求原样传递给 Agent，不要擅自扩写、翻译或润色 prompt
2. **先查后做**: 操作前先用 `get_graph.py` 了解 Graph 的当前状态
3. **引用而非描述**: 涉及已有文件时，使用 `{{@core_node:ID:name}}` 引用而非文字描述
4. **上传在先**: 如果用户提供了本地文件，先用 `upload_file.py` 上传，再在消息中引用
5. **轮询有度**: Workflow 和 Agent 响应都有超时限制，不要无限等待

---

## API 端点参考

所有 HTTP 端点基于 `VIDEOINU_API_BASE`（默认 `https://videoinu.com`）。

### Go Backend (`/api/backend/`)
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/graph/list` | 列出 Graph |
| POST | `/graph` | 创建 Graph |
| GET | `/graph/:id` | 获取 Graph 详情 |
| DELETE | `/graph/:id` | 删除 Graph |
| POST | `/core_nodes/upload/presign` | 获取上传预签名 URL |
| POST | `/core_nodes` | 创建 CoreNode（批量） |
| GET | `/core_nodes/assets_v2` | 列出资产 CoreNode |
| POST | `/wf/instance/run_in_graph` | 在 Graph 中运行 Workflow |
| POST | `/wf/instance/run_create_graph` | 运行 Workflow 并创建 Graph |
| GET | `/wf/instance/:id/status_sse` | Workflow 状态 SSE |
| GET | `/wf/definition/list` | 列出 Workflow 定义 |

### Agent Service (`/api/agent/`)
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/projects/by-graph/:graphId` | 按 Graph ID 查询 Agent Project |
| POST | `/projects/` | 创建 Agent Project |
| GET | `/sessions/by-project/:projectId` | 列出 Project 的会话 |
| POST | `/sessions/` | 创建会话 |
| DELETE | `/sessions/:sessionId` | 删除会话 |
| WS | `/sessions/:sessionId/stream` | WebSocket Agent 会话流 |

### WebSocket 消息格式 (JSON-RPC 2.0)

**发送 prompt**:
```json
{"jsonrpc": "2.0", "method": "prompt", "id": "uuid", "params": {"user_input": "message"}}
```

**心跳**:
```json
{"jsonrpc": "2.0", "method": "heartbeat", "id": "hb-uuid", "params": {"heartbeat_id": "uuid"}}
```

**批准工具调用**:
```json
{"jsonrpc": "2.0", "id": "rpc-id-from-request", "result": {"request_id": "req-id", "response": "approve"}}
```

**接收事件类型**: `TurnBegin`, `ContentPart`, `ToolCall`, `ToolCallPart`, `ToolResult`, `ApprovalRequest`, `StatusUpdate`, `SessionNotice`, `ReplayComplete`
