# opencode-agent-system

A portable global OpenCode agent system for professional full-stack software
engineering, AI development, SaaS, and product engineering — built for
efficient use of an OpenCode Go subscription.

Validated against **OpenCode v2.2.6** (V2 configuration schema).

## Architecture

One orchestrator coordinates nine specialists. The orchestrator owns the final
deliverable and delegates sparingly; specialists never delegate to each other.

```
orchestrator (primary, default agent)
├── researcher          external docs, alternatives, comparisons (read-only)
├── architect           system/DB/API design, trade-offs, decision records (read-only)
├── design-intelligence UX/competitor research, journeys, a11y (read-only)
├── ui-design           visual systems, tokens, component specs (read-only)
├── ui-engineer         React/Next.js implementation, a11y, performance
├── backend-engineer    NestJS/FastAPI, PostgreSQL/MongoDB, Prisma, Redis
├── ai-engineer         LLM integrations, RAG, agents, evaluation
├── debugger            root-cause analysis, minimal safe fixes
└── reviewer            security/quality/performance gate (read-only)
```

### Orchestration rules (encoded in the orchestrator prompt)

- Workflow: Understand → Inspect → Research → Plan → Delegate → Implement →
  Test → Review → Deliver.
- Maximum **two** concurrent specialists; one specialist per independent task.
- Parallelize only genuinely independent work; never launch all specialists for
  one feature; no trivial delegation.
- Reuse findings — no repeated repository-wide exploration or duplicate research.
- Prefer deterministic commands and tests before further AI investigation;
  review once per delivery unless changes warrant another cycle.
- Each specialist returns one concise structured result with evidence.

## Model allocation (hybrid free + OpenCode Go)

| Agent | Model | Provider |
| --- | --- | --- |
| orchestrator | `opencode-go/mimo-v2.6-flash` | Go |
| researcher | `opencode/mimo-v2.6-flash-free` | Free |
| architect | `opencode-go/mimo-v2.6-flash` | Go |
| design-intelligence | `opencode/longcat-2.5-free` | Free |
| ui-design | `opencode-go/glm-5.3-flash` | Go |
| ui-engineer | `opencode-go/mimo-v2.6-flash` | Go |
| backend-engineer | `opencode-go/deepseek-v4.1-flash` | Go |
| ai-engineer | `opencode-go/deepseek-v4.1-flash` | Go |
| debugger | `opencode/deepseek-v4.1-flash-free` | Free |
| reviewer | `opencode-go/glm-5.3-flash` | Go |

**3 free-provider agents, 7 OpenCode Go agents.**

### Escalation policy

Premium models are never assigned permanently. Use only for tasks that
demonstrably exceed the default model, with a stated one-sentence reason:

- `opencode-go/mimo-v2.6-pro`
- `opencode-go/deepseek-v4-pro`
- `opencode-go/kimi-k2.7-code`
- `opencode-go/glm-5.3`

Triggers: repeated implementation failure · complex architectural decisions ·
concurrency/distributed-system problems · security-sensitive changes ·
persistent debugging failures · unresolved critical reviewer findings.

Escalation is applied as a per-run model override, then the default resumes.

## Security baseline (opencode.jsonc, 27 rules)

- Shell commands **ask** for approval by default.
- Read-only git inspection (`status`, `diff`, `log`, `show`, branch listing,
  `remote -v`, `rev-parse`, `ls-files`, `blame`, `stash list`) allowed.
- `git push` always **asks** — never silently authorized. (Note: "Allow once"
  keeps it a per-time decision; "Allow always" saves a durable per-project
  allow pattern.)
- Hard-denied for every agent: `git reset --hard`, `rm -rf` (all flag variants).
- Edits to `.env` / `.env.*` denied (`.env.example` stays editable); env reads
  ask.
- Anything outside the active project asks (`external_directory`).
- Reviewer, researcher, architect, ui-design, design-intelligence: **edit denied**.
- Only the orchestrator may launch subagents, and only the nine specialists
  plus built-in `explore`; specialists cannot delegate further.

## Install

Back up first, then copy `global/` over your global config directory:

```sh
BACKUP=~/.config/opencode.backup-$(date +%Y%m%d-%H%M%S)
cp -a ~/.config/opencode "$BACKUP"
cp -r global/. ~/.config/opencode/
```

Then reload OpenCode (`opencode reload`, or `/reload` in the TUI) and verify:

```sh
opencode debug agents     # expect 10 custom agents
opencode models           # expect the allocated model IDs present
```

Set your own `service.json` stays untouched — it is machine-specific and
intentionally excluded from this repo.

## Repository layout

```
global/                 mirrors ~/.config/opencode/
├── opencode.jsonc      root model, default agent, permission baseline
├── AGENTS.md           shared engineering + security instructions
└── agents/*.md         the 10 agent definitions
```

Config lives under `global/` (not the repo root) so cloning this repository
into a project directory cannot accidentally activate it as project config.

## Notes

- No credentials or machine-specific files are stored here (`service.json`,
  `.env*` are gitignored).
- Model availability was verified against the live `opencode models` CLI and
  provider metadata before assignment; free-tier behavior (rate limits,
  tool-calling reliability) can only be confirmed through actual usage.
- Model assignments set defaults, not consumption caps.
