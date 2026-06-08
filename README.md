# ask-foreign-llm-skill

Drive a remote LLM as an interactive agent. The LLM runs on a node in your
topology; tool calls execute on the orchestrating machine.

For fully autonomous agent delegation — where the remote node runs its own
agent like Hermes or Goose — use
[ask-foreign-agent-skill](https://github.com/nicholasf/ask-foreign-agent-skill)
instead.

---

## Examples

### Invoke the skill

```
/ask-foreign-llm
```

Or with natural language triggers:

```
ask pond-qwen to summarise how the auth module works
let gollum-mistral look at this code and suggest improvements
what does pond-qwen think about the tradeoffs between X and Y
```

Output is prefixed with the node name:
```
[pond] The auth module uses JWT tokens issued at login...
[pond:tool] read_file("src/auth/token.py")
[pond:result] <file contents>
```

### Direct invocation

```bash
python3 agent.py --cwd /path/to/project "Summarise how authentication works"
```

---

## How it works

The remote LLM receives a task and calls tools in a loop until it is done. Tool calls execute on the orchestrating machine and results are returned to the LLM.

```
dtv-claude-agent
  │
  ├─ sends prompt ──────────────► pond-qwen (LLM on port 9337)
  │                                    │
  │◄── tool call (read_file, bash…) ───┘
  │
  ├─ executes tool locally
  │
  └─ returns result ────────────► pond-qwen (continues)
```

---

## Dependency on load-topology-skill

[load-topology-skill](https://github.com/nicholasf/load-topology-skill) is the
source of truth for which nodes are available and what models they are running.
Before invoking, read the topology to confirm the target node is online and its
inference server is active.

Nodes are referred to by their **agent handle** — `<machine>-<llm>`, e.g.
`pond-qwen`, `gollum-mistral`. See [load-topology-skill](https://github.com/nicholasf/load-topology-skill)
for the full naming convention.

Set these environment variables to target a node:

```bash
export FOREIGN_AGENT_URL=http://<hostname>:9337/v1
export FOREIGN_AGENT_MODEL=<model-name>
```

---

## Toolset

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

The LLM is not sandboxed. Adversarial content in files or prompts could trigger
destructive bash commands on the orchestrating machine.

- Use a dedicated SSH user with restricted permissions where possible.
- Prefer nodes that are not shared with other workloads.
- Review all changes before committing or merging.
