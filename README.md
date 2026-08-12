# codex-orchestration

Global Codex subagent orchestration: a small, opinionated role roster with
heterogeneous model routing, installable in a few minutes.

## What this solves

Out of the box, Codex subagents tend to inherit the main thread's model. If you
run everything on a high-end model, every delegation is expensive; if you run
everything on a cheap model, the judgment-heavy steps underdeliver. This setup
routes work by *kind of thinking needed*:

> **Luna = investigate / implement / verify** — cheap enough to use freely
> **Sol = design / architect / decide** — spend it where judgment is the bottleneck

Escalate by difficulty within a tier (Luna High → Luna Max) before crossing to
Sol, and escalate to **Sol High** only for exceptionally difficult or
consequential judgment — it is an escalation option, not a default role.

## Roles

| Role | Model | Effort | Sandbox | Job |
|---|---|---|---|---|
| orchestrator/main | Sol Medium | medium | workspace-write | synthesis, routing, ownership |
| explorer | Luna Medium | medium | read-only | investigate/research: repo + external references |
| worker | Luna High | high | workspace-write | implementation: fixes, features, tests, refactors |
| reviewer | Luna Max | max | read-only | independent verification: correctness, regressions, security, test adequacy |
| designer | Sol Medium | medium | workspace-write | visual/product judgment: UI/UX, game presentation, art direction, polish |
| architect | Sol Medium | medium | read-only | technical judgment: architecture, migrations, tradeoffs |

Designer and architect share a model but not a job: designer owns visual taste
end-to-end (and may implement directly — never delegate away the part of
implementation where taste is exercised), architect decides architecture and
stays read-only.

## Layout

```
codex-orchestration/
├── README.md
├── AGENTS.md              # orchestrator instructions (routing, escalation, workflows)
├── config.example.toml    # minimal orchestration config — merge, don't replace
└── agents/
    ├── explorer.toml
    ├── worker.toml
    ├── reviewer.toml
    ├── designer.toml
    └── architect.toml
```

## Install

```sh
cp AGENTS.md ~/.codex/AGENTS.md
cp agents/*.toml ~/.codex/agents/
```

Then merge the relevant keys from `config.example.toml` into your existing
`~/.codex/config.toml` (do not replace it — it contains your API keys and
machine-specific settings). Validate:

```sh
codex doctor
```

`~/.codex/AGENTS.md` is the **global** instruction file: it applies to every
project. Project-level `AGENTS.md` files in a repo's root (or subdirectories)
are loaded **in addition to** the global one, so project-specific rules —
framework conventions, build commands, a game engine's quirks — belong in the
project file, while role routing and escalation stay global.

## Optional: Context7

`explorer` prefers the Context7 MCP server for authoritative library/framework
documentation when it is available, and degrades gracefully to web search when
it is not. Install it (`npx -y @upstash/context7-mcp`) and register it in your
`config.toml` under `[mcp_servers]` to get higher-quality external API answers.

## Smoke test

Verify routing by spawning every role and checking each child session's
model/effort in `~/.codex/sessions/`:

```sh
codex exec --enable multi_agent "Spawn explorer, worker, reviewer, designer, architect each replying their role name"
```

Expect: explorer luna/medium · worker luna/high · reviewer luna/max ·
designer sol/medium · architect sol/medium.