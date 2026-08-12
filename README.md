# codex-orchestration

Global Codex subagent orchestration: role roster and routing rules.

## Layout

- `AGENTS.md` — orchestrator instructions (routing, escalation, workflows). Install to `~/.codex/AGENTS.md`.
- `agents/*.toml` — role definitions (model, reasoning effort, sandbox, instructions). Install to `~/.codex/agents/`.

## Role roster

| Role | Model | Effort | Sandbox | Job |
|---|---|---|---|---|
| orchestrator (main) | gpt-5.6-sol | medium | workspace-write | synthesis, routing, ownership |
| explorer | gpt-5.6-luna | medium | read-only | investigate/research (repo + external refs) |
| worker | gpt-5.6-luna | high | workspace-write | implementation |
| reviewer | gpt-5.6-luna | max | read-only | independent verification |
| designer | gpt-5.6-sol | medium | workspace-write | visual/product judgment |
| architect | gpt-5.6-sol | medium | read-only | technical judgment |

Escalate to Sol High only for exceptionally difficult or consequential judgment.

## Relevant config.toml

The orchestration-relevant keys in `~/.codex/config.toml` (full file not tracked — it contains API keys):

```toml
model = "gpt-5.6-sol"
model_reasoning_effort = "medium"

# --- Multi-agent orchestration: Sol Medium orchestrator; roles defined in ~/.codex/agents/*.toml ---

[agents]
max_threads = 8
max_depth = 1
default_subagent_model = "gpt-5.6-luna"
default_subagent_reasoning_effort = "medium"
```

`max_depth = 1` — subagents never spawn subagents; the main thread coordinates multi-step flows.

## Setup

```sh
cp AGENTS.md ~/.codex/AGENTS.md
cp agents/*.toml ~/.codex/agents/
```

Then validate:

```sh
codex doctor
```

## Smoke test

Verify routing by spawning every role and checking child-session model/effort in `~/.codex/sessions/`:

```sh
codex exec --enable multi_agent "Spawn explorer, worker, reviewer, designer, architect each replying their role name"
```