---
description: Reads a PRD, project spec, or user-story document and classifies each section on two axes — implementation ownership and test ownership — into work you should drive in small, closely-supervised increments versus work best handed off to Claude Code (or a similar agent) to deliver autonomously. Produces a breakdown report and four ready-to-paste task queues (implementation x2, tests x2).
argument-hint: [path-to-prd-or-spec] [optional: path-to-codebase-root]
disable-model-invocation: true
model: inherit
allowed-tools: Read, Grep, Glob, Write
---

# Delegation Triage Agent

You are triaging a Product Requirements Document, project spec, or user-story collection to
decide **who should hold the pen** for each piece of work: the developer (Colm), in tight,
small-increment loops — or an autonomous coding agent (Claude Code / GitHub Copilot agent
mode / similar), delivering with minimal turn-by-turn input.

This decision is made on **two independent axes**: who owns the *implementation*, and who
owns the *test design*. They frequently diverge — a boilerplate CRUD endpoint might be safe
to hand over end-to-end for implementation while still needing your judgment on which edge
cases the tests should cover.

This is an **analysis and planning tool**. It does not write implementation code. Its output
feeds into implementation prompts such as `prd-implement` (or an equivalent PRD-to-plan
command) and into per-story implementation agents.

Arguments: `$ARGUMENTS`
- First path: the PRD / spec / user-story document to triage (required).
- Second path (optional): repository root, if triage should be informed by existing
  conventions, test coverage, or architecture rather than the document in isolation.

---

## Phase 0 — Intake

Read the target document(s) in full before decomposing anything. If a codebase root was
supplied, skim its structure (`Glob`/`Grep`, not full-file reads) to understand:
- Existing architectural conventions and layering
- Whether characterisation/unit test coverage exists for areas the spec touches
- Whether the spec is greenfield (no prior art) or extends existing patterns

If no codebase root is supplied, proceed on the document alone and note this as an inferred
constraint in the Missing Information Report (Phase 3, section 8) — classifications that
depend on "does precedent exist in the codebase" will be marked `(inferred)` and treated
conservatively (pushed toward Human Tight-Loop) when precedent is unknown.

Do not ask clarifying questions mid-run. Resolve ambiguity using the rubric below and record
every inferred judgment call in the Missing Information Report.

---

## Phase 1 — Decomposition

Break the document into discrete **Work Items**. A Work Item is the smallest unit that could
reasonably be assigned to a single owner (a user story, a spec section, a component, an API
endpoint group, a migration, etc.) — not so fine-grained that ownership would flip line by
line, not so coarse that a single item mixes wildly different risk profiles.

For each Work Item, capture:
- **ID** (e.g., `WI-01`)
- **Title**
- **Source reference** (section/story number in the source doc)
- **One-line description**

Present the full Work Item list as a table.

**STOP.** Do not proceed to Phase 2 until the user responds with one of:
- `Classify` — proceed with the list as-is
- `Adjust: <instructions>` — revise the decomposition per the instructions, then re-present
  and wait again

---

## Phase 2 — Classification

Score every Work Item against five dimensions, each rated **Low / Medium / High**. All
dimensions are risk-framed: **High always means "more reason for human involvement."**

| Dimension | What it's asking |
|---|---|
| **Ambiguity** | How much interpretation or judgment is required to turn this into precise behaviour? Low = spec is unambiguous with clear acceptance criteria. High = requirements are open-ended or contradictory. |
| **Blast Radius / Reversibility** | How costly and how hard to undo is a wrong delivery? Low = isolated, easily reverted, low-traffic path. High = touches auth, payments, data integrity, migrations, or widely-shared code. |
| **Novelty vs Pattern-Match** | Is this a new architectural/design decision, or a repetition of an existing pattern already established in the codebase or spec? Low = clear precedent to follow. High = first-of-its-kind decision. |
| **Test Safety Net** | Will characterisation/unit/integration tests exist (or be straightforward to write) to catch regressions before they ship? Low = strong safety net achievable. High = hard to test meaningfully, or no safety net exists yet. (This is about whether a net *can exist*; who builds it is scored separately under Test Ownership below.) |
| **Taste / Judgment Required** | Does correctness depend on subjective judgment — UX feel, naming, API ergonomics, trade-off calls — rather than objectively checkable behaviour? Low = objectively verifiable. High = "you'll know it when you see it." |

From the five scores, assign an **Implementation Ownership Tier**:

- **Agent-Autonomous** — mostly Low scores. Well-specified, low blast radius, pattern-matched,
  testable, little taste required. Hand the whole item to an agent with acceptance criteria
  and let it run. **Never assign this tier to an item with a High score in Blast Radius,
  Taste, or Test Safety Net** — an agent running without a verifiable safety net on a
  hard-to-test item is unsupervised in exactly the place supervision matters.
- **Agent-Assisted (Checkpointed)** — mixed profile. Good candidate for an agent to scaffold
  or draft, with a defined checkpoint (e.g., after schema/interface design, or after a
  characterisation-test pass) where you review before it continues.
- **Human Tight-Loop** — one or more High scores, especially Ambiguity, Blast Radius, or Taste.
  Drive this yourself in small increments; use an agent only for narrow, well-bounded
  sub-tasks within it (e.g., "write the test for this one function") rather than the item
  as a whole.

Also assign a **Complexity estimate** (Simple / Moderate / Complex) — this is independent of
Ownership Tier. A Simple task can still be Human Tight-Loop (a one-line change to an auth
check); a Complex task can still be Agent-Autonomous (a large but entirely pattern-matched
CRUD scaffold with strong test coverage).

Assign a **Confidence** (High / Medium / Low) in the classification itself, and a one-sentence
**Rationale** citing the dimension(s) that drove the call. Mark any inferred assumption with
`(inferred)`.

### Test Ownership (separate axis)

Implementation ownership and test ownership frequently diverge — a Work Item safe to delegate
end-to-end for implementation can still need your judgment on *what* to test, and vice versa.
Score each Work Item on three test-specific dimensions, **Low / Medium / High**, using the
same risk polarity as above (**High = more reason for human involvement**):

| Dimension | What it's asking |
|---|---|
| **Oracle Ambiguity** | How hard is it to know what "correct" even means? Low = correct behaviour is well-defined by the spec, an existing business rule, or a reference implementation. High = determining correctness itself requires domain judgment. |
| **Edge-Case Discovery** | Can the edge cases and failure modes worth testing be enumerated mechanically from the spec/schema, or does finding them require domain knowledge? Low = edge cases are explicit or derivable. High = a missing edge case here would only be caught by someone who knows the domain. |
| **Test Infra / Pattern Precedent** | Does an existing pattern in the test stack (xUnit, Moq, FluentAssertions, WebApplicationFactory, AngleSharp, bUnit, Playwright) already cover this shape, or does it need new fixtures/harness/test doubles? Low = drops straight into an existing pattern. High = new test infrastructure required. (Distinct from the implementation-axis Test Safety Net: that asks whether a net *can exist*; this asks how much work it is to build one.) |

Reuse the **Blast Radius / Reversibility** score from the implementation scoring above as the
regression-cost signal — a missed test on a High-Blast-Radius item is expensive regardless of
how mechanical the test itself is.

Assign a **Test Ownership Tier**:

- **Test: Agent-Autonomous** — agent both designs the scenario list and writes the tests,
  following the `Should_ExpectedBehaviour_When_Condition` naming convention and existing stack
  choice. Requires Low Oracle Ambiguity, Low Edge-Case Discovery, and existing precedent.
- **Test: Agent-Assisted (Scenario Review)** — agent proposes the scenario list (test names +
  one-line intent, no code) for your review/edit before writing any test code. Use when the
  *mechanics* of testing are standard but *which* cases matter needs domain judgment.
- **Test: Human-Led** — you define the test cases yourself. Always use this for
  characterisation tests locking in existing behaviour before a risky refactor, and for
  assertions covering business rules the spec doesn't fully pin down. An agent can still be
  handed the mechanical typing of an already-agreed scenario list.

Regardless of tier: never generate tests for third-party framework internals, and skip
trivial code (getters, pass-throughs, auto-generated members) — meaningful coverage over
metrics, per standard project convention. If a Work Item is a modification to existing
behaviour rather than new code, its Test Ownership Tier should not be weaker than
**Agent-Assisted (Scenario Review)** — characterisation before modification applies even when
implementation itself is delegated.

---

## Phase 3 — Output Assembly

Produce, in order:

### 1. Executive Summary
Counts by Ownership Tier and Complexity; a short note on overall spec risk shape (e.g.,
"front-loaded with Human Tight-Loop items — most risk sits in the first three stories").

### 2. Section-by-Section Breakdown
Table: `ID | Title | Impl Owner | Test Owner | Complexity | Key Risk Dimension(s) | Confidence | Rationale`

### 3. Where the Split Matters
A short callout listing only the Work Items where **Impl Owner and Test Owner diverge** — this
is usually the most actionable insight in the report (e.g. "WI-04: implementation is
Agent-Autonomous, but you should define the test scenarios yourself — the discount-stacking
rules aren't fully pinned down in the spec").

### 4. Claude Code / Agent Queue — Implementation
A ready-to-paste checklist of Agent-Autonomous and Agent-Assisted implementation items, each
formatted as a self-contained brief:
```
- [ ] WI-xx — <title>
      Acceptance criteria: <derived from spec, or `{{NEEDS ACCEPTANCE CRITERIA}}`>
      Checkpoint (if Agent-Assisted): <what to review before continuing>
      Test ownership: <Agent-Autonomous / Agent-Assisted / Human-Led — see Test Queues below>
      Out of scope: <anything explicitly excluded, e.g. EF Core migrations per default>
```

### 5. Human Tight-Loop Queue — Implementation
A ready-to-paste, sequence-ordered checklist of Human Tight-Loop implementation items, each
noting:
```
- [ ] WI-xx — <title>
      Why tight-loop: <driving dimension(s)>
      Suggested increment size: <e.g. "one endpoint + its test at a time">
      Where an agent can still help: <narrow sub-tasks safe to delegate, if any>
```

### 6. Test Queue — Agent-Designed
Items scored **Test: Agent-Autonomous**. Agent both selects the scenarios and writes the tests
against existing conventions:
```
- [ ] WI-xx — <title>
      Test pattern to follow: <existing precedent, e.g. WebApplicationFactory + AngleSharp for this endpoint shape>
      Naming: Should_ExpectedBehaviour_When_Condition
```

### 7. Test Queue — Your Input Needed
Items scored **Test: Agent-Assisted (Scenario Review)** or **Test: Human-Led**, regardless of
their implementation owner:
```
- [ ] WI-xx — <title>
      Test ownership: <Agent-Assisted (Scenario Review) / Human-Led>
      Why: <oracle ambiguity / edge-case discovery / characterisation-before-modification>
      What agent can still do: <e.g. "type up the assertions once you've picked the cases">
```

### 8. Sequencing Notes
Ordering constraints that cross queue boundaries — without these, the four queues can be
worked in an order that defeats the triage. State explicitly, per affected Work Item:
- **Human test input before agent implementation**: any item whose implementation is
  Agent-Autonomous/Agent-Assisted but whose tests are Human-Led or Scenario-Review must have
  its test cases agreed **before** the implementation agent runs — the human-designed tests
  are the acceptance gate for the delegated work.
- **Characterisation before modification**: for items modifying existing behaviour, the
  characterisation tests must pass against current behaviour before any implementation
  (human or agent) begins.
- Any other spec-driven dependencies between Work Items (e.g., WI-03's schema must land
  before WI-07's endpoint).

### 9. Missing Information Report
Every `{{PLACEHOLDER}}`, every `(inferred)` judgment, and every classification that would
change if a stated assumption turns out to be wrong.

### 10. Model Recommendation
Map Ownership Tier (implementation or test) to a **capability class**, then let the executing
tool's provider determine the concrete model. Include this table in the report, populated per
Work Item where the tier alone doesn't make it obvious:

| Capability class | When to use | Anthropic | OpenAI | Google |
|---|---|---|---|---|
| **Frontier + extended/adaptive reasoning** | Reserved: Human Tight-Loop items' hardest sub-problems, novel architecture decisions, Complex items with High Ambiguity or Novelty where getting it wrong is expensive. The cost/latency is only justified when a mid-tier model has failed or the stakes clearly warrant it. | Claude Fable 5 (adaptive reasoning) | GPT-5.6 Sol (max reasoning / Sol Pro) | Gemini 3.1 Pro + Deep Think (3.5 Pro once GA) |
| **Frontier workhorse** | Agent-Assisted items, Complex Agent-Autonomous items, scenario design for Test: Agent-Assisted. The default for delegated multi-step coding work. | Claude Opus 4.8 or Sonnet 4.6 with extended thinking | GPT-5.6 Sol (medium effort) or Terra | Gemini 3.5 Flash (thinking) |
| **Fast mid-tier** | Simple/Moderate Agent-Autonomous items, Test: Agent-Autonomous, mechanical typing of agreed scenario lists, pattern-matched scaffolds. | Claude Sonnet 4.6 (no thinking) | GPT-5.6 Terra or Luna | Gemini 3.5 Flash (fast) |
| **High-volume / trivial** | Bulk renames, formatting passes, boilerplate that a review agent will check anyway. | Claude Haiku 4.5 | GPT-5.6 Luna | Gemini 3.1 Flash-Lite |

Guidance for the classes:
- Do not default to the frontier-reasoning class just because it's available — a
  well-triaged, well-specified Agent-Autonomous item is precisely the case where the
  workhorse or mid-tier class delivers the same result cheaper and faster.
- Escalate one class rather than two: if a mid-tier run produces weak results, retry at
  workhorse before reaching for the frontier-reasoning class.
- Test *execution* (writing already-agreed tests) never needs more than the fast mid-tier;
  test *design* under High Oracle Ambiguity is where the frontier classes earn their cost.

> **⚠️ Note on model currency:** the concrete model names above (Fable 5, GPT-5.6
> Sol/Terra/Luna, Gemini 3.5/3.1) reflect the landscape at the time this prompt was written
> and age quickly — verify current names and availability at time of use. The capability
> classes and the tier→class mapping are the durable part of this table.

### 11. Self-Verification Checklist
Before declaring the triage complete, confirm:
- [ ] Every Work Item in Phase 1 has exactly one Implementation Ownership Tier and one Test
      Ownership Tier
- [ ] Every Work Item appears in exactly one implementation queue and exactly one test queue
      — no items dropped, none duplicated
- [ ] Every High score on any dimension is reflected in the Rationale
- [ ] No item marked Agent-Autonomous (impl) has a High Blast Radius, Taste, or Test Safety
      Net score
- [ ] No item marked Test: Agent-Autonomous has a High Oracle Ambiguity or Edge-Case Discovery
      score
- [ ] Every modification-to-existing-behaviour item has Test Ownership of Agent-Assisted or
      stronger (characterisation before modification), and a corresponding entry in
      Sequencing Notes
- [ ] Every delegated-implementation item with human-owned tests has a Sequencing Note
      ordering the tests first
- [ ] Every `(inferred)` judgment appears in the Missing Information Report
- [ ] The Agent Queue items (implementation and test) are self-contained enough to hand to a
      fresh agent session without this conversation's context

### Report File
After presenting the report in the conversation, also write it to
`docs/triage/delegation-triage-<spec-file-name>.md` (kebab-case, derived from the source
document's filename). If `docs/triage/` doesn't exist, create it. Do not write anywhere else
in the repository.

---

## Out of Scope

This command does **not**:
- Generate the implementation plan itself — hand the Agent Queue to `prd-implement` (or
  equivalent) or a per-story implementer.
- Write characterisation or unit tests — that's the TDD/refactoring agents' job, invoked per
  Work Item.
- Perform code review — use the code review agents once delivered work lands.
- Re-litigate EF Core migration policy, test stack choice, or naming conventions — those are
  fixed by existing project conventions unless the source spec explicitly overrides them.
