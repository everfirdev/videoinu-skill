# videoinu-skill

> [OpenClaw](https://openclaw.ai) skill for the [Videoinu](https://videoinu.com) platform — manage projects via Graphs, chat with AI Agents, and run automated Workflows.

## Features

- **Graph Management** — Create, list, and inspect Graphs (project canvases) with ViewNodes and CoreNodes
- **File Upload / Download** — Upload local files to create CoreNodes, or download assets from a Graph
- **Agent Chat** — Create sessions and chat with AI Agents in real time via WebSocket
- **Workflow Execution** — Run predefined automation pipelines and poll for results
- **Zero Dependencies** — Pure Python standard library, no third-party packages required

## Install

```bash
# Via ClawHub
npx clawhub install videoinu

# Or clone directly
git clone https://github.com/everfirdev/videoinu-skill.git
```

## Quick Start

### 1. Save your Access Key

Log in at [videoinu.com](https://videoinu.com), go to **Profile → Copy Access Key**, then:

```bash
python3 scripts/auth.py save "your-access-key"
```

### 2. List your projects

```bash
python3 scripts/list_graphs.py
```

### 3. Chat with an Agent

```bash
# Create a session on a Graph
python3 scripts/create_session.py GRAPH_ID

# Send a message
python3 scripts/agent_chat.py SESSION_ID "Analyze the structure of this project"
```

### 4. Upload a file and reference it

```bash
python3 scripts/upload_file.py /path/to/video.mp4
# → returns core_node_id

python3 scripts/agent_chat.py SESSION_ID "Edit this video {{@core_node:CORE_NODE_ID:video.mp4}}" --auto-approve
```

### 5. Run a Workflow

```bash
python3 scripts/run_workflow.py DEF_ID --graph-id GRAPH_ID \
  --inputs '{"input_image": {"type": "core_node_refs", "core_node_ids": ["NODE_ID"]}}'

python3 scripts/query_workflow.py INSTANCE_ID --poll
```

## Scripts

| Script | Description |
|--------|-------------|
| `auth.py` | Save, verify, and manage Access Keys |
| `list_graphs.py` | List user's Graphs |
| `get_graph.py` | View Graph details (ViewNodes + CoreNodes) |
| `create_graph.py` | Create a new Graph |
| `upload_file.py` | Upload a file to create a CoreNode |
| `download_file.py` | Download files from a Graph |
| `create_session.py` | Create or list Agent sessions |
| `agent_chat.py` | Chat with an AI Agent via WebSocket |
| `run_workflow.py` | Run a Workflow definition |
| `query_workflow.py` | Query Workflow execution status |

## Requirements

- Python 3.9+
- A [Videoinu](https://videoinu.com) account with an Access Key

## Configuration

| Environment Variable | Default | Description |
|---------------------|---------|-------------|
| `VIDEOINU_ACCESS_KEY` | — | Your access token (or use `auth.py save`) |
| `VIDEOINU_API_BASE` | `https://videoinu.com` | API base URL |

## License

MIT
