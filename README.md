# opencode-agent-system

A portable global OpenCode agent system for professional full-stack software
engineering, AI development, SaaS, and product engineering — built for
efficient use of an OpenCode Go subscription.

Validated against **OpenCode v2.2.6** (V2 configuration schema; the V1 keys
`permission`/`bash`/`task` are *not* used).

## Architecture overview

One accountable **orchestrator** coordinates **20 specialists** across six
departments. Specialists never delegate to each other (enforced by
`subagent: deny`); only the orchestrator launches them, at most **two
concurrently**, and it evaluates every result before integrating it.

```
orchestrator (primary, default agent — accountable owner)
├── PRODUCT        product-manager · ux-researcher
├── RESEARCH       researcher · design-intelligence · technical-seo-engineer
├── DESIGN         ui-design ──────────────┐
├── BUILD          ui-engineer · backend-engineer · ai-engineer ·
│                  database-engineer · technical-writer
├── ASSURE         qa-engineer · test-automation-engineer ·
│                  cybersecurity-engineer · reviewer · debugger
└── OPERATE        devops-sre-engineer · performance-capacity-engineer ·
                   release-manager
```

Workflow routing (who runs for what) is encoded in the orchestrator prompt as
six default routes: product development, bug fix, UI development, AI feature,
security, operational. Routes are defaults, not checklists — the orchestrator
invokes only the specialists a task actually needs.

## Agent directory (capability matrix)

| Agent | Primary responsibility | Model | Mode | Permissions | Delegates to |
|---|---|---|---|---|---|
| `orchestrator` | Task ownership: plan, route, integrate, verify, deliver | `opencode-go/mimo-v2.6-flash` | primary | edit ✓, shell ask, subagent deny→21 allows | all specialists + `explore` |
| `product-manager` | Requirements, scope, stories, acceptance criteria, MVP, metrics | `opencode-go/glm-5.3-flash` | all | read-only (edit deny), subagent deny | — |
| `ux-researcher` | User problems, JTBD, journeys, usability/IA research, a11y needs | `opencode/glm-5.3-flash-free` | all | read-only, subagent deny | — |
| `researcher` | External facts: official docs, alternatives, comparisons | `opencode/mimo-v2.6-flash-free` | all | read-only, subagent deny | — |
| `architect` | System/DB/API design, trade-offs, security boundaries, decision records | `opencode-go/mimo-v2.6-flash` | all | read-only, subagent deny | — |
| `design-intelligence` | Design systems, interaction patterns, usability problems, differentiation | `opencode/longcat-2.5-free` | all | read-only, subagent deny | — |
| `technical-seo-engineer` | Crawlability, metadata, structured data, CWV (web-surface products only) | `opencode/glm-5.3-flash-free` | all | read-only, subagent deny | — |
| `ui-design` | Art direction, IA, tokens, component/motion specs, handoff | `opencode-go/glm-5.3-flash` | all | read-only, subagent deny | — |
| `ui-engineer` | Token-exact implementation, states, a11y, screenshot-driven refinement | `opencode-go/mimo-v2.6-flash` | all | edit ✓, subagent deny | — |
| `backend-engineer` | NestJS/FastAPI, PostgreSQL/MongoDB, Prisma, Redis, authN/authZ, data integrity | `opencode-go/deepseek-v4.1-flash` | all | edit ✓, subagent deny | — |
| `ai-engineer` | LLM integrations, tool calling, RAG, evaluation, cost/latency budgets | `opencode-go/deepseek-v4.1-flash` | all | edit ✓, subagent deny | — |
| `database-engineer` | Schema, indexing, query optimization, migrations, pooling, lifecycle | `opencode-go/deepseek-v4.1-flash` | all | edit ✓, subagent deny | — |
| `technical-writer` | Architecture/API docs, onboarding, runbooks, doc maintenance | `opencode/mimo-v2.6-flash-free` | all | edit ✓ (docs), subagent deny | — |
| `qa-engineer` | Test strategy, AC traceability, functional testing, release verdict | `opencode-go/deepseek-v4.1-flash` | all | read-only, subagent deny | — |
| `test-automation-engineer` | Test architecture, Playwright/API tests, fixtures, CI, flaky tests | `opencode-go/mimo-v2.6-flash` | all | edit ✓, subagent deny | — |
| `cybersecurity-engineer` | Threat modelling, OWASP assessment, severity-rated findings + remediation specs | `opencode-go/glm-5.3-flash` | all | read-only (separation of duties), subagent deny | — |
| `debugger` | Root cause, reproduction, minimal safe fix, regression identification | `opencode/deepseek-v4.1-flash-free` | all | edit ✓, subagent deny | — |
| `reviewer` | Read-only quality gate: security, correctness, coverage, readiness verdict | `opencode-go/glm-5.3-flash` | all | read-only, subagent deny | — |
| `devops-sre-engineer` | CI/CD, Docker, infra, observability, backup/rollback, incident readiness | `opencode-go/deepseek-v4.1-flash` | all | edit ✓, subagent deny | — |
| `performance-capacity-engineer` | Load/stress testing, p50/p95/p99 evidence, bottlenecks, capacity | `opencode-go/mimo-v2.6-flash` | all | edit ✓, subagent deny | — |
| `release-manager` | Release checklist, versioning, rollback readiness, risk sign-off | `opencode-go/mimo-v2.6-flash` | all | edit ✓ (artifacts), subagent deny | — |

**Read-only** = `edit * deny` (can still run shell with approval). Implementers
keep shell `ask` and env/deny baselines from the global rules.

## Workflow matrix (which specialists for common tasks)

| Task | Required (in order) | Optional |
|---|---|---|
| New product/feature | product-manager → ux-researcher/researcher → architect → ui-design → ui-engineer/backend-engineer → qa-engineer → reviewer → release-manager | design-intelligence (first, if visual direction undecided), cybersecurity (auth/data), devops-sre + performance (shipping) |
| Bug fix | reproduce → debugger → responsible engineer → test-automation-engineer → qa-engineer → reviewer | — |
| UI development | product requirements → design-intelligence → ui-design → ui-engineer → screenshot/browser checks → a11y (test-automation) → qa-engineer → reviewer | — |
| AI feature | researcher → architect → ai-engineer → backend-engineer → evaluation evidence → cybersecurity → qa-engineer | — |
| Security issue | cybersecurity-engineer → responsible engineer (fix) → test-automation (regression tests) → independent verification | — |
| Operational change | architect → devops-sre-engineer → performance-capacity-engineer → qa-engineer → release-manager | — |

## Collaboration contract

- **Brief in:** objective · context · files in scope · constraints · expected
  output · acceptance criteria · verification requirements · dependencies ·
  whether changes are allowed.
- **Report out:** work completed · files changed/inspected · decisions · tests
  performed · results · remaining issues · risks/blockers · next action.
- The orchestrator never accepts a specialist report blindly: claims are
  checked against files and re-verified with local commands. Failed specialist
  deliverables are rejected with one corrected re-brief.

## Delegation policy

- No delegation of trivial tasks — the orchestrator does those itself.
- Max **two** concurrent specialists; parallel only for independent work;
  dependent stages run sequentially.
- No delegation chains (specialists are `subagent: deny`); the orchestrator
  re-briefs the next specialist itself.
- One focused brief per specialist; findings are reused, never re-discovered.
- Reviews run once per delivery — re-reviews only after changes.

## Model allocation and escalation

Allocation is hybrid: free-tier models for research/analysis, economical
`opencode-go` flash models for engineering (see the matrix above). All model
IDs were verified against `opencode models` on v2.2.6 before assignment.

Escalation models (never assigned permanently; per-run override only, with a
one-sentence reason recorded in the delivery summary):

- `opencode-go/mimo-v2.6-pro` · `opencode-go/deepseek-v4-pro` ·
  `opencode-go/kimi-k2.7-code` · `opencode-go/glm-5.3`

Triggers: repeated bounded-task failure · high-impact architecture decisions ·
concurrency/distributed problems · security-sensitive changes · persistent
debugging failure · unresolved critical reviewer findings.

**Excluded models:** Muse Spark contributor models (`muse-spark*` on both
providers) are disabled in `opencode.jsonc` via
`providers.<id>.models.<id>.disabled` because they train on request data.
Re-enabling requires deleting those lines — a deliberate user action. (Note:
`provider.use` policies can only deny whole providers, which would disable
`opencode`/`opencode-go` entirely; per-model disable is the correct mechanism.
`opencode models` still prints disabled models in its raw catalog listing —
selection is what is blocked, verified with a live `provider.no-route` test.)

## Permissions and security model (27 baseline rules)

- Shell: **ask** by default; read-only git inspection allowed; `git push`
  always asks; `git reset --hard` and `rm -rf` variants **denied** for all.
- `.env`/`.env.*` edits denied (`.env.example` editable); env reads ask.
- `external_directory: ask` — anything outside the active workspace prompts.
- Research/design/review/QA/security agents are edit-denied; only orchestrator
  and implementers may write; nobody but the orchestrator may launch
  subagents, and only the 21 named specialists + built-in `explore`.
- Rules are ordered (global first, agent last, last match wins — V2 semantics
  per official docs). Policies: none required beyond the model exclusions.

## Install and update

```sh
# backup first
set -l BK ~/Backups/opencode-global-(date +%Y%m%d-%H%M%S)
cp -a ~/.config/opencode $BK
# from a clone of this repo:
cp -r global/. ~/.config/opencode/
opencode reload
```

`service.json` is machine-specific, gitignored, and never touched by installs.

## Configuration validation

```sh
opencode debug agents     # expect 28 entries: 21 custom + 7 built-in
opencode debug config     # parses without error; shows global opencode.jsonc
opencode models           # escalation models present (disabled models still list)
```

Check the essentials: `default_agent = orchestrator`, every custom agent's
`model` resolves, modes are `primary`/`all`, specialists carry
`subagent: deny`, the orchestrator's allow-list covers every specialist, and
the 27 baseline rules appear in every merged ruleset.

## Smoke testing

Bounded pattern per agent (runs in an isolated temp directory):

```sh
mkdir -p /tmp/oc-smoke && cd /tmp/oc-smoke
opencode run --agent <id> --auto --format json "<small task with clear acceptance criteria>"
```

Verify: role/report-format adherence · correct model (from `opencode debug
agents`) · no `subagent` tool calls (specialists) · no files created outside
the task's intended scope · claims re-run locally (e.g., `node --test`).

## Troubleshooting

- **Config not picked up:** `opencode reload`, then `opencode debug config`
  (checks the file parses) and `opencode debug agents`.
- **Agent missing from `@`/runs:** check frontmatter validity and that the
  file lives in `~/.config/opencode/agents/*.md`; `opencode debug agents` is
  the source of truth.
- **Model unavailable (`provider.no-route`):** model disabled in
  `providers…disabled`, provider not in plan, or region/privacy setting —
  check the workspace region (Global) and the model-exclusion block.
- **Approval prompts on shell:** expected (`shell: *` → ask); answer "once"
  to avoid saving durable allow patterns.
- **Permission rule seems ignored:** rules are ordered — last match wins; put
  exceptions after broad rules (agent frontmatter is evaluated last).
- **Run hangs with no output:** provider stream stall — kill and retry; if
  persistent, check `opencode` server logs (`opencode debug paths` → log).

## Known limitations

- Benchmark/smoke evidence is bounded-task evidence, not a guarantee for
  large or open-ended work; free-tier models are rate-limit and
  tool-call-reliability variable.
- Browser automation for agents is CLI-native: `playwright-cli` (global) +
  bundled Chromium, driven through the normal `shell` permission (approval
  prompts apply). The built-in `browser` Code Mode namespace stays removed
  until a desktop app/browser client is attached to the service — allow rules
  are already staged in `opencode.jsonc`.
- k6, axe-core, and security scanners are not installed (report-only
  decision); load testing and a11y/dependency scanning need per-project
  tooling.
- `external_directory: ask` means cross-directory reads in sessions prompt
  unless approved.
- SEO specialist is dormant by design — invoke only for products with a web
  surface; it never promises rankings.
