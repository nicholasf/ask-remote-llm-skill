# Ask Foreign LLM

Drive a remote LLM as an interactive agent. The LLM runs on the remote node;
tool calls execute either locally (bridge local) or on the remote node via SSH
(bridge SSH). All output is prefixed with `[node-name]`.

## Agent naming convention

Refer to agents as `<machine>-<llm>-agent`, e.g. `dtv-claude-agent`,
`pond-qwen-agent`. This makes it clear which machine and model is acting.

## Before invoking

Read the topology (load-topology-skill) to find the node hostname and verify
its inference server is active:

```bash
curl -s http://<hostname>:9337/v1/models
```

Set `$FOREIGN_AGENT_URL` and `$FOREIGN_AGENT_MODEL` to target the node.

---

## Bridge mode — local (default)

The LLM runs on the remote node; tool calls execute on the orchestrating
machine. Use when working against the local codebase.

```bash
"${SKILLS_HOME:-$HOME/.agents/skills}/ask-foreign-llm-skill/.venv/bin/python3" \
  "${SKILLS_HOME:-$HOME/.agents/skills}/ask-foreign-llm-skill/agent.py" \
  --cwd <working directory> \
  "<message>"
```

### Toolset

| Tool | Description |
|---|---|
| `read_file` | Read a file by path |
| `write_file` | Write content to a file |
| `edit_file` | Replace an exact string in a file |
| `bash` | Run a bash command in the working directory |
| `find_files` | Find files by name pattern |
| `grep` | Search for a pattern across files |
| `list_directory` | List directory tree |
| `git_diff` | Show unstaged and staged changes |

---

## Bridge mode — SSH

The LLM runs on the remote node; tool calls execute on a different remote node
via SSH. Use when the target node has the repo and toolchain but no agent
runtime. `$AGENT_SSH_USER` must be set.

```bash
"${SKILLS_HOME:-$HOME/.agents/skills}/ask-foreign-llm-skill/.venv/bin/python3" \
  "${SKILLS_HOME:-$HOME/.agents/skills}/ask-foreign-llm-skill/agent.py" \
  --ssh-node <hostname> \
  --ssh-cwd <remote working directory> \
  "<message>"
```

In SSH mode only `bash` is exposed — commands execute on the remote node via SSH.

---

## Output format

- `[node-name] ...` — text response
- `[node-name:tool] ...` — tool call
- `[node-name:result] ...` — tool result (truncated if long)

## Triggers

Invoke when the user says "ask [node]", "what does [node] think", or
"let [node] look at this" and no agent runtime (Hermes/Goose) is available
on that node. For autonomous agent delegation use ask-foreign-agent-skill.
