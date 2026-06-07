# ask-foreign-llm-skill

Drive a remote LLM as an interactive agent. The LLM runs on a node in your
topology; tool calls execute either on the orchestrating machine (bridge local)
or on a remote node via SSH (bridge SSH).

For fully autonomous agent delegation — where the remote node runs its own
agent runtime like Hermes or Goose — use
[ask-foreign-agent-skill](https://github.com/nicholasf/ask-foreign-agent-skill)
instead.

---

## How it works

The remote LLM receives a task and can call tools in a loop until it is done.
In bridge local mode, every tool call is proxied back to the orchestrating
machine and executed there. In bridge SSH mode, tool calls are forwarded to a
remote node via SSH.

```
dtv-claude-agent
  │
  ├─ sends prompt ──────────────► pond-qwen-agent (LLM on port 9337)
  │                                      │
  │◄── tool call (read_file, bash…) ─────┘
  │
  ├─ executes tool locally (bridge local)
  │  or via SSH on target node (bridge SSH)
  │
  └─ returns result ────────────► pond-qwen-agent (continues)
```

---

## Dependency on load-topology-skill

[load-topology-skill](https://github.com/nicholasf/load-topology-skill) is the
source of truth for which nodes are available and what models they are running.
Before invoking, read the topology to confirm the target node is online and its
inference server is active.

Set these environment variables to target a node:

```bash
export FOREIGN_AGENT_URL=http://<hostname>:9337/v1
export FOREIGN_AGENT_MODEL=<model-name>
```

---

## Modes

### Bridge mode — local

The LLM runs on the remote node. Tool calls execute on the orchestrating
machine. Use this when the task is against the local codebase.

```bash
python3 agent.py --cwd /path/to/project "Summarise how authentication works"
```

### Bridge mode — SSH

The LLM runs on the remote node. Tool calls execute on a different remote node
via SSH. Use this when the target node has the repo and toolchain but no agent
runtime. Set `$AGENT_SSH_USER` to the SSH username.

```bash
export AGENT_SSH_USER=nicholasf

python3 agent.py \
  --ssh-node <hostname> \
  --ssh-cwd /path/to/project \
  "Run the test suite and report failures"
```

In SSH mode only `bash` is available — the remote shell handles everything.

---

## Toolset (bridge local mode)

| Tool | Description |
|---|---|
| `bash` | Run a shell command in the working directory |
| `read_file` | Read a file by path |
| `write_file` | Write content to a file |
| `edit_file` | Replace an exact string in a file |
| `find_files` | Find files matching a name pattern |
| `grep` | Search for a pattern across files |
| `list_directory` | List directory tree |
| `git_diff` | Show unstaged and staged changes |

---

## Setup

```bash
uv sync
# or: pip install langchain-core langchain-openai
```

---

## Security

**Bridge SSH mode grants the remote LLM shell access to the target node.**

- The LLM is not sandboxed. Adversarial content in files or prompts could
  trigger destructive bash commands on the remote node.
- Use a dedicated SSH user with restricted permissions where possible.
- Prefer nodes that are not shared with other workloads.
- Review all changes before committing or merging.

Bridge local mode does not have this exposure — tool calls execute on the
orchestrating machine under the user's own account.
