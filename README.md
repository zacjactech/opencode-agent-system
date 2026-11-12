# opencode-agent-system

A portable global OpenCode agent system for professional full-stack software
engineering, AI development, SaaS, and product engineering — built for
efficient use of an OpenCode Go subscription.

Validated against **OpenCode v2.2.7** (V2 configuration schema; the V1 keys
`permission`/`bash`/`task` are *not* used).

Upgraded **2026-11-12** with the UI Engineering System: global UI standards
bound via the `instructions` field, an independent `design-reviewer` agent,
the §10 quality gate, and a `playwright-cli`-scoped shell allow rule (details
below; validation evidence in the session that produced this release).

## Architecture overview

One accountable **orchestrator** coordinates **21 specialists** across six
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
│                  cybersecurity-engineer · reviewer · design-reviewer ·
│                  debugger
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
| `orchestrator` | Task ownership: plan, route, integrate, verify, deliver | `opencode-go/mimo-v2.6-flash` | primary | edit ✓, shell ask, subagent deny→22 allows | all specialists + `explore` |
| `product-manager` | Requirements, scope, stories, acceptance criteria, MVP, metrics | `opencode-go/glm-5.3-flash` | all | read-only (edit deny), subagent deny | — |
| `ux-researcher` | User problems, JTBD, journeys, usability/IA research, a11y needs | `opencode/glm-5.3-flash-free` | all | read-only, subagent deny | — |
| `researcher` | External facts: official docs, alternatives, comparisons | `opencode/mimo-v2.6-flash-free` | all | read-only, subagent deny | — |
| `architect` | System/DB/API design, trade-offs, security boundaries, decision records | `opencode-go/mimo-v2.6-flash` | all | read-only, subagent deny | — |
| `design-intelligence` | Design systems, interaction patterns, usability problems, differentiation | `opencode/longcat-2.5-free` | all | read-only, subagent deny | — |
| `technical-seo-engineer` | Crawlability, metadata, structured data, CWV (web-surface products only) | `opencode/glm-5.3-flash-free` | all | read-only, subagent deny | — |
| `ui-design` | Art direction, IA, tokens, component/motion specs, handoff | `opencode-go/glm-5.3-flash` | all | read-only, subagent deny | — |
| `ui-engineer` | Token-exact implementation, states, a11y, screenshot-driven refinement | `opencode-go/mimo-v2.6-flash` | all | edit ✓, subagent deny | — |
| `design-reviewer` | Independent rendered-UI gate: screenshot inspection, §10 rubric scoring, severity-ranked findings | `opencode-go/mimo-v2.6-flash` | all | read-only (edit deny), subagent deny | — |
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
| New product/feature | product-manager → ux-researcher/researcher → architect → ui-design → ui-engineer/backend-engineer → design-reviewer (UI changed) → qa-engineer → reviewer → release-manager | design-intelligence (first, if visual direction undecided), cybersecurity (auth/data), devops-sre + performance (shipping) |
| Bug fix | reproduce → debugger → responsible engineer → test-automation-engineer → qa-engineer → reviewer | — |
| UI development | product requirements → design-intelligence (references) → ux-researcher (journeys/IA, significant work) → ui-design (direction + tokens, spec persisted §9) → ui-engineer (implements + §8 evidence) → design-reviewer (independent §10 gate) → bounded correction loop (3–5 rounds) → qa-engineer (functional/a11y) → reviewer | — |
| AI feature | researcher → architect → ai-engineer → backend-engineer → evaluation evidence → cybersecurity → qa-engineer | — |
| Security issue | cybersecurity-engineer → responsible engineer (fix) → test-automation (regression tests) → independent verification | — |
| Operational change | architect → devops-sre-engineer → performance-capacity-engineer → qa-engineer → release-manager | — |

## UI engineering standards and visual quality gate

A single reusable standards file — `global/ui-engineering-standards.md`,
installed to `~/.config/opencode/` and bound into **every session** via the
V2 `instructions` field in `opencode.jsonc` — governs all UI work. Agent
files reference its sections instead of duplicating rules:

- **§1 Pipeline**: PRODUCT UNDERSTANDING → DESIGN RESEARCH → UX ARCHITECTURE
  → VISUAL DIRECTION → DESIGN SYSTEM → IMPLEMENTATION → BROWSER VALIDATION →
  INDEPENDENT VISUAL REVIEW → REFINEMENT → APPROVAL (trivial fixes may skip
  with a recorded reason).
- **§2–§6 Design rules**: reference-driven research (2–3 references, what to
  take / NOT to copy), per-project visual identity + avoid-list, UX
  architecture before implementation, composition rules per axis, and an
  anti-pattern *challenge list* (not a ban list — kept patterns carry a
  product-specific reason).
- **§8 Visual QA**: run the app, screenshot **375/430/768/1280/1440 × all
  themes** into `.ui-qa/` (before/after pairs), trigger non-default states
  deliberately, severity labels, bounded 3–5 correction rounds.
- **§9 Spec persistence**: the approved design spec is persisted as a
  project file (e.g. `docs/design/<feature>.md`) before implementation —
  the review contract, not chat memory.
- **§10 Quality gate**: 7-dimension weighted rubric (hierarchy 20 / type 15 /
  UX 20 / consistency 15 / identity 10 / responsive 10 / a11y 10), **≥90 to
  pass**, blocking criteria (broken interactions, layout collapse,
  inaccessible primary flows, severe responsive failure, missing primary
  states) override any score. Enforced by `design-reviewer` — independent of
  implementer and spec author — never the code `reviewer` or `qa-engineer`.

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

## Permissions and security model (30 baseline rules)

- Shell: **ask** by default; read-only git inspection allowed; **`playwright-cli *`
  allowed** (scoped browser-QA de-friction added 2026-11-11: the shell
  scanner extracts and checks every compound subcommand separately, so this
  cannot smuggle a denied command; risk boundary = JS inside bundled
  headless Chromium — trusted briefs only); `git push`
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
opencode debug agents     # expect 29 entries: 22 custom + 7 built-in
opencode debug config     # parses without error; shows global opencode.jsonc
                          # (instructions array + playwright-cli rule present)
opencode models           # escalation models present (disabled models still list)
```

Check the essentials: `default_agent = orchestrator`, every custom agent's
`model` resolves, modes are `primary`/`all`, specialists carry
`subagent: deny`, the orchestrator's allow-list covers every specialist, and
the 30 baseline rules appear in every merged ruleset, and the
`instructions` array resolves to `ui-engineering-standards.md` (a live
session shows it as an instruction update on save).

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
- **Long agent run hits the execution window mid-task:** capture the
  `sessionID` from `--format json` output, then resume that exact session
  with `opencode run --session <sessionID> --auto "…resolve open items in ≤3
  tool calls each, then deliver the full report now"` — prior tool results
  persist, so nothing verified is lost. Use this before re-running a review
  from scratch; pairs with design-reviewer's bounded-investigation rule.

## Known limitations

- Benchmark/smoke evidence is bounded-task evidence, not a guarantee for
  large or open-ended work; free-tier models are rate-limit and
  tool-call-reliability variable.
- Browser automation for agents is CLI-native: `playwright-cli` (global);
  **Chromium + Firefox installed** (Firefox matrix validated 2026-11-12:
  5-viewport overflow, billing/theme toggles, FAQ, 0 console errors).
  **WebKit is an accepted limitation on this machine (Arch):** Playwright's
  build targets Ubuntu 24.04 sonames — 12 libs missing incl. AUR-only
  ICU74-era versioned libs and a probable dead-end (`libbacktrace.so.0`),
  plus a permanent host-validation skip on an unsupported distro. Full
  `ldd` evidence: `.ui-qa/limitation-fixes.md` in the reference pipeline
  test. If a project claims WebKit/Safari support, prefer the official
  Playwright container over machine-global legacy libs; on
  Ubuntu/Debian `npx playwright install-deps webkit` is the one-step path.
  `playwright-cli *` is allow-listed in the shell baseline
  (2026-11-11), so visual-QA sweeps run without approval fatigue; other
  shell use stays `ask`. The built-in `browser` Code Mode namespace has allow
  rules active but needs a browser/desktop client attached to the service to
  have a target. Note: `file://` URLs are blocked by playwright-cli — serve
  static pages over HTTP (e.g. `python3 -m http.server`).
- `opencode run` child sessions hit intermittent provider stream stalls
  (observed 2026-11-12 on `opencode-go/mimo-v2.6-flash`: active seconds then
  minutes of silence). Recovery: kill + retry; if it recurs on the same task,
  per-run `--model` override with a recorded reason (escalation policy).
- No axe/k6/scanner is *installed* (report-only decision): a11y gating is
  **standardized in the global standards §7** as a zero-install method —
  axe-core injected from CDN via `playwright-cli run-code` (evidence format:
  `.ui-qa/axe-report.json` in the reference pipeline test). A *CI* a11y
  gate remains a per-project decision; load testing and dependency scanning
  need per-project tooling.
- The UI standards load ambiently via `instructions` (approved trade-off:
  every session — including backend-only — carries ~190 lines). V2 has no
  per-agent instructions; if token cost ever outweighs the load guarantee,
  swap the `instructions` entry for a `skills/` entry in one config line
  (weaker: loading becomes agent-discretion).
- `external_directory: ask` means cross-directory reads in sessions prompt
  unless approved.
- SEO specialist is dormant by design — invoke only for products with a web
  surface; it never promises rankings.
