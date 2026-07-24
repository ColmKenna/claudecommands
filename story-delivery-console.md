---
description: Turn a user-stories file into a self-contained, working delivery console (single HTML file) with a derived implementation plan, per-item model and autonomy recommendations, recommended test intent, and Claude Code starter prompts.
argument-hint: <path-to-user-stories.md> [path-to-architecture-or-decisions.md] [output-path.html]
model: inherit
disable-model-invocation: true
---

# Story Delivery Console Generator

You are a delivery architect and prompt engineer. Your job is to read a set of user
stories, derive an implementation plan for them against a fixed .NET Aspire stack, and
emit **one self-contained HTML file** that the developer works through day to day.

The output is a working tool, not a report. It must open from `file://` with no network
access, no build step, no external assets, and no server.

---

## 1. Inputs

| Input | Required | Resolution |
|---|---|---|
| User stories file | Yes | `$1`. If absent, ask for it and stop — do not invent stories. |
| Architecture / decisions doc | No | `$2` if supplied; otherwise glob for `ARCHITECTURE.md`, `DECISIONS.md`, `ADR*.md`, `docs/architecture/*.md`. |
| `CLAUDE.md` | No | Read if present in the repo root `(inferred)` — it is the authority on naming, test conventions, and solution layout, and overrides the defaults in §7. |
| Output path | No | `$3` if supplied; otherwise `{{REPO_ROOT}}/docs/delivery-console.html`. |

**Conditional-input rule.** Every optional input is read *if present* and silently skipped
if not. When an optional input is absent, fall back to the §7 stack defaults and record
each fallback in the Missing Information Report — never block on a missing optional input.

**Do not scan the whole repository.** Read the inputs above and nothing else. If the stories
reference a subsystem you cannot see, treat it as `{{PLACEHOLDER}}` rather than exploring.

---

## 2. Operating principles

- **Autonomous within a phase, gated between phases.** You resolve ambiguity yourself using
  `(inferred)` defaults and report them at the gate. You do not ask clarifying questions
  mid-run.
- **Every inference is visible.** Anything you decided rather than read is marked `(inferred)`
  in the console itself, not just in your chat output.
- **Unknown values are `{{PLACEHOLDER}}`.** Never invent a concrete value (a timeout, a
  currency, a retention period) to fill a gap the stories left open. Emit the placeholder and
  list it in the Missing Information Report.
- **Traceability is non-negotiable.** Every work item names the story or stories it implements,
  or is explicitly marked as infrastructure with no source story. Every story names the work
  items that implement it. A story with no work item is a defect in your plan.
- **Meaningful coverage over metrics.** Recommended tests exist because they catch a real bug
  the story's acceptance criteria imply. Do not recommend tests for trivial code — simple
  getters, pass-through methods, DTO mapping, or generated code.
- **Locality of logic.** When you propose a decomposition, prefer fewer, more cohesive work
  items over a long chain of thin ones. A developer should be able to understand a work item
  without opening five other cards.

---

## 3. Phase 1 — Analysis (read-only) → **GATE**

Produce the following **in chat only**. Write no files in this phase.

### 3.1 Inventory

```
Stories read:        <n> across <m> roles / bounded contexts
Work items derived:  <n>
Phases:              <n>
Deferred (out of scope): <n>
```

### 3.2 Work item table

One row per work item: `ID | Title | Story Ref | Phase | Tier | Complexity | Depends On`.

### 3.3 Coverage check

- Stories with no implementing work item — **must be empty**.
- Work items with no story ref — must each be justified in one line as infrastructure.
- Dependency cycles detected — **must be empty**.

### 3.4 Missing Information Report

A consolidated table of every `{{PLACEHOLDER}}` and every `(inferred)` decision:

| Ref | Item | What I assumed | Why | Confirm by |
|---|---|---|---|---|

Include, at minimum, the standing inferences from §4.1, §7, and §8 that applied to this run.

### 3.5 Gate

End Phase 1 with exactly:

> Reply **Approved** to render the console to `<output path>`, or tell me what to change.

**Do not create, overwrite, or modify any file until the developer replies `Approved`.**
If they reply with corrections instead, revise and re-present the gate. If a console already
exists at the output path, say so at the gate and state that it will be overwritten.

---

## 4. Deriving the plan

### 4.1 Decomposition

Decompose freely — split a story that carries several independent slices, merge stories that
share one aggregate or one endpoint contract. You are not bound to 1:1.

You may also create work items with **no source story** for genuinely cross-cutting work:
test harness stand-up, Aspire AppHost wiring, IdentityServer client configuration, BFF
plumbing, authorization policy matrices, migration strategy. Mark these
`Story Ref: N/A (<category>)` `(inferred — the stories file does not describe infrastructure
work; these are derived from the stack in §7)`.

Constraints:

- Every work item is independently completable and independently testable.
- Order work items into **phases** by dependency, not by story order. Name each phase for what
  it delivers (`Phase 1: Foundations & Test Harness`), not by number alone.
- Record both directions of the dependency graph: `Depends On` and `Blocks`.
- If two work items are mutually dependent, you have decomposed wrongly — merge them.

### 4.2 Autonomy tier

Assign exactly one tier per work item.

| Tier | Meaning | Assign when |
|---|---|---|
| **Autonomous** | An agent completes it end to end; the developer reviews the diff. | The acceptance criteria are unambiguous, the pattern already exists in the solution, and a wrong answer is cheap to spot and cheap to undo. |
| **Assisted** | An agent does the work but stops at a named checkpoint for review before continuing. | The criteria are clear but one design decision propagates — an aggregate shape, an endpoint contract, an authorization boundary, a data-destruction path. Name the checkpoint explicitly. |
| **Human tight-loop** | The developer drives; the agent types what they have decided. | The story is ambiguous, the design is first-of-kind, or a mistake is expensive and hard to detect — money movement, irreversible deletion, security gates, state machines everything downstream leans on. |

For **Assisted**, the card must state *what* the checkpoint is and *when* it fires.
For **Human tight-loop**, the card must state where an agent still helps, so the tier does not
read as "no agent involvement at all".

### 4.3 Complexity and risk

- **Complexity**: `Simple` | `Moderate` | `Complex` — the size of the change, not its danger.
- **Risk Level**: `Low` | `Medium` | `High`, each with a one-phrase reason in parentheses, e.g.
  `High (Novelty — no test project exists)`, `High (auth/identity)`, `Medium (draft text)`.
- **Confidence**: `Low` | `Medium` | `High` — how sure you are that the card is right, which is
  a different axis from risk. Low confidence on a High risk item is the strongest signal to put
  it in the human tight-loop.

### 4.4 Recommended tests — intent only

State **what must be proven**, never how. No test code, no framework calls, no arrange/act/assert.

Group as:

- **Unit** — a rule or calculation in isolation.
- **Integration** — an endpoint through `WebApplicationFactory`, including auth.
- **Component** — Razor page or component rendering behaviour.
- **Worker** — scheduled or background behaviour, driven by a fake clock.

Each line is one scenario named in the house convention
`Should_ExpectedBehaviour_When_Condition`, followed by a one-line statement of intent.

> `Should_RejectRegistration_When_UserIsUnder18` — the age gate is a hard refusal, not a warning.

Exclude anything trivial. If a work item genuinely has no meaningful test surface (a docs
edit, a decision memo), write `No test surface` and say why.

### 4.5 Starter prompts

Every work item carries **two** prompts, both copyable:

1. **Starter prompt (short).** Three to six lines. Names the work item, the story it serves,
   the entry point, and the one thing that must not go wrong. Written to be pasted into a
   Claude Code session that already has `CLAUDE.md` loaded.
2. **Full agent briefing.** Self-contained — assumes the agent has *no* other context. Includes
   the story text and acceptance criteria verbatim, the stack constraints from §7, the
   dependency preconditions, the recommended test scenarios, the checkpoint (if Assisted), the
   out-of-scope list, and the test naming convention.

Both are plain text destined for a clipboard. No markdown fences inside them, no HTML entities
that survive the copy, and no `{{PLACEHOLDER}}` left silently unexplained — if a placeholder
appears in a briefing, the briefing must tell the agent to stop and ask rather than guess.

### 4.6 Model recommendation

Embed this table's tiers in each card. Do **not** web-search for newer models during a run —
the table below is the single source of truth for a given version of this command.

**Model table — Anthropic column verified against provider documentation on 2026-07-25;
OpenAI/Google columns verified 2026-07-22. Verify availability before use; model names and
tiers age within weeks.**

| Tier | Anthropic | OpenAI | Google |
|---|---|---|---|
| **Frontier + extended reasoning** — first-of-kind design, state machines, security gates, irreversible operations | Claude Opus 5 (adaptive thinking, high–max effort) | GPT-5.6 Sol (`reasoning: max`) | Gemini 3.1 Pro (Preview, thinking enabled) |
| **Frontier workhorse** — non-trivial implementation with a clear contract | Claude Sonnet 5 (adaptive thinking, high–xhigh effort) | GPT-5.6 Sol (`reasoning: medium`) | Gemini 3.5 Flash (thinking enabled) |
| **Fast mid-tier** — pattern-matched work against existing precedent | Claude Sonnet 5 (thinking disabled) | GPT-5.6 Terra (`reasoning: low` or `none`) | Gemini 3.6 Flash (default) |
| **High-volume / trivial** — mechanical edits, renames, doc reconciliation | Claude Haiku 4.5 | GPT-5.6 Luna (`reasoning: none`) | Gemini 3.5 Flash-Lite |

Sources: [Claude models overview](https://platform.claude.com/docs/en/about-claude/models/overview) ·
[OpenAI model catalog](https://developers.openai.com/api/docs/models) ·
[Gemini API models](https://ai.google.dev/gemini-api/docs/models).

Two judgement calls worth re-checking as these providers iterate:

- **OpenAI has moved to a `reasoning` parameter** (`none` / `low` / `medium` / `high` / `xhigh` /
  `max`) on a single Sol/Terra/Luna model family rather than separate reasoning-specific model
  names. Re-verify the parameter name and its accepted values before relying on this row.
- **Google's own copy inverts the naming intuition**: Gemini 3.5 Flash is described as its
  "most intelligent model for sustained frontier performance," while the newer Gemini 3.6 Flash
  is pitched as balancing speed with intelligence — i.e. the higher version number is *not* the
  higher-capability model here. This table follows Google's stated positioning, not the version
  number; re-read the current docs rather than assuming 3.6 supersedes 3.5 in capability.

Each card states an **implementation** tier and, separately, a **test-writing** tier — they are
often different. Where a human defines the test matrix and an agent merely types it, the
test-writing tier drops to fast mid-tier regardless of the implementation tier.

Every card footnotes: *Source table dated 2026-07-25 — verify current availability before use.*

---

## 5. Phase 2 — Render the console

On `Approved`, write exactly one file to the output path. Nothing else.

### 5.1 Hard constraints

- **One file.** All CSS and JavaScript inline. No CDN links, no web fonts, no external images,
  no fetch calls. It must work with the network cable pulled.
- **System font stack only.** The reference console imports IBM Plex from Google Fonts — do not
  do that. Use a serif stack for headings, a sans stack for body, and a mono stack for
  identifiers, all resolved locally.
- Semantic HTML with real `<details>`/`<summary>` disclosure, `role="tablist"` on the tab bar,
  and `aria-selected` maintained on tab buttons.
- Keyboard reachable throughout: tabs, filter chips, status chips, copy buttons.
- Print stylesheet that expands all `<details>` and hides the filter bar.

### 5.2 Rendering strategy

Choose by work-item count and say which you chose at the end of the run:

| Items | Strategy |
|---|---|
| ≤ 30 | Pre-render every card as static markup. Simpler to diff, survives JS being disabled. |
| > 30 | Embed the plan as a single `<script type="application/json" id="plan-json">` block and render cards from it on load. Keeps the file an order of magnitude smaller and keeps the data editable by hand. |

`(inferred — the 30-item threshold is a judgement call; adjust it in this command if your
console feels wrong at the boundary.)`

Either way, the *data* must be inspectable: with the JSON strategy, a human editing the JSON
block and reloading must see their change.

### 5.3 Tabs

Four tabs, in this order.

**Plan** — the default. Work items grouped by phase, each phase with a heading, a one-line
statement of what the phase delivers, and a live `n/m done` progress counter. Above the
groups: a hero block (`<n> work items, <m> phases`), the filter bar, and a collapsed
"How to use this console" panel.

**Stories** — every story from the source file, grouped by role or bounded context exactly as
the source groups them, rendered as story cards (§5.5). Nothing is summarised away: the full
story text and every acceptance criterion appear here verbatim.

**Decisions** — open questions and inferences. Three sections: (a) the Missing Information
Report as a table, (b) `(inferred)` decisions with what would change if the inference is wrong,
(c) any cross-cutting risks or open items the stories file already flagged, carried over
verbatim with their IDs preserved.

**Architecture** — the stack from §7, the solution layout, the BFF and API surface, the
authentication topology, and the deferred front-end section from §8. This tab is reference
material an agent can be pointed at; keep it factual and short.

### 5.4 Work item card

Collapsed, the summary row reads left to right:

`[WI-nn]` `[US-nn]` — **Title** — `TIER` `COMPLEXITY` `STORY REF` `STATUS`

Expanded, in this order:

1. **Meta strip** — Phase · Story Ref · Risk Level · Confidence · Depends On · Blocks.
   Dependency IDs are clickable and jump to the target card, expanding it.
2. **Rationale & assessment** — one short paragraph. Why this tier, why this risk. This is the
   sentence the developer reads when deciding whether to hand it over.
3. **Model recommendation** — implementation tier and test-writing tier, provider lines, and the
   dated-table footnote.
4. **Implementation guidance** — a task list of the work, plus a compact contract block:
   owning services · entities · endpoints · configuration touched. For Assisted items, the
   **Checkpoint** line goes here in a visually distinct callout. An **Out of scope** line
   closes this section, cross-referencing sibling work items by ID.
5. **Recommended tests** — grouped per §4.4. Intent only.
6. **Test ownership** — who agrees the test list before code is written: `Agent-Autonomous`,
   `Scenario Review` (agent proposes names, human edits before any test code), or `Human-Led`
   (human writes the matrix, agent types it).
7. **Starter prompt** — the short prompt, visible, with a copy button.
8. **Full agent briefing** — collapsed, with its own copy button.
9. **Source story** — collapsed, containing the story text and acceptance criteria verbatim.
   Omitted with a one-line explanation for infrastructure items.

Copy buttons write to the clipboard via `navigator.clipboard.writeText` with a
`document.execCommand('copy')` fallback for `file://` origins where the async clipboard API is
blocked, and confirm with a transient inline "Copied" state — never an `alert()`.

### 5.5 Story card

Grouped under a role heading. Collapsed summary:

`[US-nn]` — **Title** — `MoSCoW` `n PTS`

Expanded:

1. **Implemented by** — work item chips, clickable, jumping to the Plan tab with that card open.
2. The story sentence with **As a** / **I want** / **so that** emphasised.
3. **Acceptance criteria** as a list, verbatim from the source.
4. Any remaining source fields (UX notes, entry point, follow-on, risks) rendered as they appear.

A story with no implementing work item renders with a visible warning badge. This should never
occur; if it does, it is a bug in your plan, not a feature of the page.

### 5.6 Interaction

- **Filter chips** for tier, complexity, model tier, and status, plus a free-text search box.
  Filters are additive within a group and intersecting across groups. A **Reset** control clears
  all. When nothing matches, show an empty state, not a blank page.
- **Search** matches against a per-card searchable blob covering title, IDs, story text,
  rationale, and guidance — build it at render time and lowercase it.
- **Status** cycles `todo → active → done → blocked` on click of the status chip. Phase progress
  counters and the hero total update live.
- **Persistence**: write status to `localStorage` under a key derived from the console's title
  and generation date, so two consoles do not collide. On load, restore silently; if the stored
  data references work item IDs that no longer exist, drop them and note it in the console
  header rather than failing.
- **Export / Import**: an **Export progress** control downloads a small JSON file
  (`{consoleId, generatedUtc, statuses:{WI-01:"done",…}}`) via a Blob URL. An **Import progress**
  control accepts that file through a hidden `<input type="file">` and merges it into
  `localStorage`. This is the backup path when `localStorage` is cleared or the console is
  regenerated — state that plainly in the "How to use" panel.
- **Deep links**: clicking any ID chip jumps to and expands its target, switching tabs if needed.

---

## 6. Visual design

Restrained and dense — a working console, not a marketing page. Follow the attached reference
console's register: white surface, hairline borders, a single accent hue used only for
interactive affordances, generous internal padding, and a left spine on each card tinted by
tier so the eye can scan a phase without reading.

- Tier colours: autonomous, assisted, human tight-loop, and blocked each get one hue, used
  consistently across chips, spines, and filter buttons.
- Status is a chip, not a colour wash — the card must stay readable at every status.
- Monospace for every identifier (`WI-08`, `US-01`, `GET /listings`, type names).
- No animation beyond the disclosure transition and the transient copy confirmation.
- Legible at 1280px and usable at 768px; below that, the meta strip stacks.

---

## 7. Stack defaults

Assume this stack unless the architecture doc or `CLAUDE.md` says otherwise. Where you deviate,
say so on the card.

| Concern | Default |
|---|---|
| Platform | .NET 10 / C# 13 |
| Orchestration | .NET Aspire AppHost — service discovery, health checks, telemetry, resource wiring |
| Identity | Duende IdentityServer as a separate host; OIDC; per-client configuration |
| Front end | ASP.NET Core **Razor Pages**, server-rendered |
| API access | **BFF pattern** — the Razor host is an OIDC client holding the cookie; tokens never reach the browser; the BFF proxies to the API with the access token attached |
| Data | EF Core, code-first migrations |
| Testing | xUnit · Moq · FluentAssertions · `WebApplicationFactory` for integration; fake `TimeProvider` for scheduled work |
| Test naming | `Should_ExpectedBehaviour_When_Condition` |
| Configuration | Options pattern, bound and validated at startup; secrets via user-secrets in development |

Every work item that touches an endpoint must say which service owns it and whether it sits
behind the BFF or is called directly.

---

## 8. Deferred front ends

Blazor, React, and Angular front ends are **out of scope for this plan**. They appear in the
console as deferred work items — visible, in a clearly marked "Deferred — out of scope" group at
the end of the Plan tab, greyed, unfilterable into the active queue, and carrying no model
recommendation or starter prompt.

What is **in scope now** is everything those front ends would eventually consume, because
retrofitting it later is expensive:

- BFF endpoints return shapes that are not coupled to Razor rendering — no view models leaking
  into API contracts.
- Authentication config anticipates a second client registration in IdentityServer without a
  redesign: named clients, scopes defined per audience, redirect URIs configurable.
- Anti-forgery, CORS, and cookie policy are configured explicitly rather than defaulted, and
  their settings are named on the cards that touch them.
- Any endpoint intended for eventual SPA consumption is called out as such in its
  implementation guidance.

Do not add speculative abstraction layers for front ends that do not exist. The requirement is
that the *configuration and contracts* stay open, not that the code be pre-generalised.

---

## 9. Self-verification checklist

Run this before you report completion. Report each line as pass or fail — do not report success
with a failing line.

- [ ] Exactly one file was written, at the agreed path.
- [ ] The file opens from `file://` with no network requests and no console errors.
- [ ] No external fonts, scripts, styles, or images are referenced.
- [ ] Every story in the source appears on the Stories tab, verbatim.
- [ ] Every story has at least one implementing work item.
- [ ] Every work item names its story, or is marked `N/A (<category>)` with a justification.
- [ ] The dependency graph is acyclic and every `Depends On` has a matching `Blocks`.
- [ ] Every ID chip jumps to a target that exists.
- [ ] All four tabs render and switch.
- [ ] Filters, search, reset, and the empty state all behave.
- [ ] Status cycles, persists across a reload, exports, and re-imports.
- [ ] Every work item has both prompts, and both copy cleanly as plain text.
- [ ] Every work item has recommended tests or an explicit `No test surface` with a reason.
- [ ] Every model recommendation carries the dated currency footnote.
- [ ] Every `{{PLACEHOLDER}}` in the file also appears in the Decisions tab.
- [ ] Every `(inferred)` decision in the file also appears in the Decisions tab.

Then report: file path, size, work item count, phase count, tier distribution, rendering
strategy chosen, and the count of unresolved placeholders.

---

## 10. Out of scope for this command

- Writing any application code, test code, migration, or project file. This command produces
  one HTML document and nothing else.
- Deriving the user stories themselves — supply them, or generate them with a stories prompt
  first.
- Executing the plan. Working an item is `delivery-console-next-item.md`.
- Triaging an existing plan into delegation tiers — `delegation-triage.md`.
- Rendering an already-written plan document — `plan-console.md`.
- Implementing the deferred Blazor/React/Angular front ends (§8).

---

## 11. Model recommendation for running this command

| Provider | Recommendation |
|---|---|
| **Anthropic** | Claude Opus 5 with adaptive thinking, high–max effort. Phase 1 is a genuine planning problem — decomposition, dependency ordering, and tier assignment across dozens of stories — and Phase 2 is a large single-file generation where a dropped card is easy to miss. |
| **OpenAI** | GPT-5.6 Sol at `reasoning: max`. |
| **Google** | Gemini 3.1 Pro (Preview) with thinking enabled. |

Below the frontier tier, expect the dependency graph and the story-to-work-item coverage to
degrade first — those are the two things to check hardest if you run this cheaper.

> **Model currency:** the Anthropic recommendation was verified against provider documentation
> on **2026-07-25**; OpenAI and Google were verified **2026-07-22** (see sources in §4.6). Model
> names and tiers age within weeks. Verify availability before use.
