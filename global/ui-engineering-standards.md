# Global UI Engineering Standards

Binding for every OpenCode session that designs, implements, reviews, tests,
or approves user-interface work. These are process and quality rules — never
an aesthetic. A project's `AGENTS.md` overrides them for project-specific
matters; project visual identity lives in the project's own design spec, never
here. Agent files reference sections of this file instead of restating them.

## 1. The UI delivery pipeline

For significant UI work, the orchestrator runs this sequence; each stage's
output is the next stage's input:

```
PRODUCT UNDERSTANDING → DESIGN RESEARCH → UX ARCHITECTURE → VISUAL DIRECTION
→ DESIGN SYSTEM → IMPLEMENTATION → BROWSER VALIDATION → INDEPENDENT VISUAL
REVIEW → REFINEMENT → APPROVAL
```

- **Significant** = new screens/features, redesigns, or changes affecting
  layout, hierarchy, navigation, or visual identity. Trivial fixes (copy
  typo, one-token correction inside an established system) may work directly
  in the existing design system — record why the pipeline was skipped.
- No stage is skipped because an agent "already knows what to do".
  Assumptions are not artifacts.

## 2. Design research (before visual direction)

- Ground every significant direction in 2–3 real references (live products,
  named design systems, documented patterns) with URLs and retrieval dates.
  Quality of reference beats quantity.
- For each reference state: **what to take, what NOT to copy, and why it
  works for its audience** — and whether that audience fits this product.
- Extract principles, not pixels: reproducing a layout without its content,
  constraints, and audience produces the "interchangeable SaaS" look.
- Collect the category's clichés as this product's **avoid-list** (§3).

## 3. Visual identity (per project, never global)

- Every project defines its own direction in concrete terms — named
  references, typographic character, density, radius/color posture, one
  sentence a designer could defend. "Modern and clean" is not a direction.
- Nothing in this file prescribes a house style. Never impose one aesthetic
  across unrelated products.
- The approved spec declares the direction **and** its avoid-list: the
  model-default defaults this product must not fall into (e.g. default sans
  stacks used without reason, decorative gradients, card-wall layouts).

## 4. UX architecture (before implementation of significant UI)

Define and approve before implementation:

- Target users and the journeys being served — each step's goal, friction,
  and states: entry, success, empty, loading, error, edge.
- Information architecture: what is on each screen, in what order, why.
  Hierarchy is a decision, not a default grid.
- Navigation model and interaction behavior — what every control does,
  including disabled and permission-denied cases.
- Screen-state inventory: every screen's default/loading/empty/error/success
  states with realistic content (no lorem ipsum, no invented statistics
  presented as real).

## 5. Visual composition rules

Each axis gets one coherent, audience-justified choice per screen area:

- **Layout** — explicit alignment/grid logic and content priority; not a
  default card wall.
- **Typography** — a type scale with real size jumps; line-height; measure
  (~70–80 chars max for prose); tabular numerals where numbers align; one or
  two families loaded deliberately, not by library default.
- **Spacing** — one spacing scale used consistently; off-scale spacing is a
  defect.
- **Color** — palette roles (surface/text/accent/status) with explicit light
  **and** dark values; contrast verified **computationally** against WCAG AA
  (report the ratio; never eyeball hex values); color is never the only
  carrier of meaning.
- **Information density** — match the audience (tools for experts may be
  dense; consumer flows breathable). Airy whitespace everywhere is as lazy
  as cramming.
- **Borders/shadows/elevation** — one border weight and one elevation
  language per project; elevation reserved for real layering.
- **Icons** — one icon system (source, stroke, grid); no mixed libraries;
  no emoji as UI icons unless the brand deliberately uses them.
- **Animation** — purposeful (orient, confirm, reveal); one language with
  specified duration/easing; `prefers-reduced-motion` honored; omitted when
  unjustified.
- **Responsive** — named breakpoints with defined *behavior* changes
  (navigation mode, density, content priority) — never "it adapts".

## 6. Anti-pattern challenge list

Actively challenge these in any design or implementation; keeping one
requires a product-specific reason:

- Card grids as the default layout for anything not genuinely card-like;
  card-in-card nesting.
- Gradients as decoration rather than meaning (esp. purple-on-white heroes).
- Generic dashboard compositions (4 KPI tiles + chart + table by reflex).
- Arbitrary/unexplained colors; multiple accent colors; status rainbows.
- Rounded-container proliferation — everything a pill or rounded box.
- Decorative icons next to every heading; emoji as iconography.
- Repetitive feature grids; meaningless statistics; fake "social proof".
- Generic marketing copy (Elevate / Seamless / Unlock), AI-cliché eyebrows
  and numbered markers, all-caps labels, one-word highlighted headlines.
- Inconsistent spacing, mixed radii, unexplained font defaults.
- Decorative elements with no information job (orbs, blobs, gratuitous
  fade-ups on every section).

These are **challenges, not bans**: a pricing page may legitimately be three
cards; a gradient may carry brand identity. The defect is choosing by
default. Context and usability decide — every kept pattern carries its
reason.

## 7. Implementation quality bar

- Production components in the project's idioms; semantic tokens only (no
  hardcoded values that belong to the token layer); componentize on real
  repetition, not speculation.
- Complete states everywhere — default, loading, empty, error, success,
  disabled, hover/focus — with realistic copy.
- Accessibility is part of done: semantic HTML, real controls, keyboard
  operation, visible focus, labeled fields with error announcements,
  contrast AA, reduced motion — plus automated checks where the project has
  them. Preferred a11y automation needs **no install**: inject axe-core from
  its CDN via `playwright-cli run-code` (`page.addScriptTag({url: …axe.min.js})`,
  then `axe.run` with WCAG tags) and record violations per theme/viewport —
  evidence format exemplified in `.ui-qa/axe-report.json` of the reference
  pipeline test.
- Real backend integration behind typed contracts with loading/error
  handling; unimplemented APIs isolated behind a stated adapter; no fake
  data as real, no dead controls, no TODO styling.
- Performance: no unnecessary re-renders or bundle bloat; measure with the
  project's tools, don't guess.

## 8. Visual QA workflow (mandatory before any visual approval)

1. Run the application (dev server or static file) — never approve from
   code-reading.
2. Open each relevant screen and capture screenshots with `playwright-cli`
   (load the `playwright-cli` skill) at viewports **375, 430, 768, 1280,
   1440** — for **every supported theme** (light and dark where both exist).
   Repeat the *key behavioral checks* (overflow, primary interactions,
   console) in **every browser engine the project claims to support**, not
   only the default Chromium.
3. Trigger non-default states deliberately (hover/focus by interaction,
   empty/loading/error via fixtures or route mocks). Vision sees only what
   is captured.
4. Store evidence in `.ui-qa/` at the project root (gitignore it in a repo):
   `<screen>-<viewport>-<theme>[-state].png`, keeping **before/after pairs**
   for every correction round.
5. Inspect actual pixels: hierarchy, type, spacing, overflow, alignment,
   truncation, contrast, states. Screenshots **flag** problems; computed
   styles or code **confirm** them (contrast ratios are computed, never
   estimated from an image).
6. Compare against the approved spec artifact (§9).
7. Assign severity: **blocker** (broken interaction, layout collapse,
   inaccessible primary flow) · **major** (clear quality defect) · **minor**
   (polish) · **note** (preference).
8. Correct, re-capture, iterate — **bounded to ~3–5 rounds**; still failing
   means escalate to the orchestrator, not another loop.
9. Obtain **independent review**: the `design-reviewer` agent — never the
   implementer or the spec author — reads/captures the evidence and applies
   §10.
10. No claim of visual approval without actual screenshot evidence attached
    to the report.

## 9. Spec persistence (the design contract)

The orchestrator persists the approved design spec + UX architecture to a
project file (commonly `docs/design/<feature>.md`) **before** implementation
starts. Review, refinement, and later sessions work against this artifact,
never against chat memory. The file path travels in every handoff brief.

## 10. Quality gate (before approving significant UI)

Score each dimension 0–10 (0 = absent, 10 = excellent), apply weights:

| Dimension | Weight |
|---|---|
| Visual hierarchy & composition | 20% |
| Typography & readability | 15% |
| UX & interaction design | 20% |
| Visual consistency | 15% |
| Product identity & differentiation | 10% |
| Responsive behavior | 10% |
| Accessibility | 10% |

- Approval threshold: **≥ 90/100**.
- **Blocking criteria override the score** — any of these fails the gate
  regardless of total: broken interactions; major layout problems at any
  target viewport; inaccessible primary workflows; severe responsive
  failures; missing primary states (loading/error) on implemented screens.
- Every finding carries evidence (screenshot file + viewport + element or
  measured value), severity, and a specific actionable correction.
- The reviewer is `design-reviewer` (read-only), independent of implementer
  and spec author. Re-review after corrections until the gate passes or a
  blocker is escalated to the human. Scores never launder a blocker: a 95
  with a broken primary flow is **not ready**.
