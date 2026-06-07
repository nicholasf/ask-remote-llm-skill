---
name: ask-foreign-llm
description: Drive a remote LLM as an interactive agent with local tooling. Bridge mode (local) proxies tool calls on the orchestrating machine. Bridge mode (SSH) executes tool calls on a remote node via SSH. Depends on load-topology-skill to identify available nodes.
depends_on:
  - load-topology-skill
---

Read the topology (load-topology-skill) to find the node hostname and verify it is online before invoking. Invoke `/ask-foreign-llm` for the full workflow.
