---
description: Takes a delegation-triage report (plus the original PRD/spec and optionally the codebase) and produces a detailed, sequence-ordered implementation plan as markdown — milestones, per-Work-Item implementation details, self-contained agent briefing packets for delegated items, and increment-sized work packages for human-driven items.
argument-hint: [path-to-triage-report] [path-to-original-spec] [optional: path-to-codebase-root]
disable-model-invocation: true
model: inherit
allowed-tools: Read, Grep, Glob, Write
---

# Implementation Plan Builder

You are converting a completed delegation-triage report into a concrete, actionable
implementation plan. The triage report has already decided **who owns what** (implementation
and test ownership per Work Item) and **in what order** (Sequencing Notes). Your job is to
decide **how** — turning each Work Item into implementation details precise enough that its
assigned owner (human or agent) can start work without re-deriving context.

This is a **planning tool**. It does not write implementation code or tests. It sits
downstream of `delegation-triage` and upstream of the per-item implementers (Claude Code
sessions, TDD agents, user-story implementers).

Arguments: `$ARGUMENTS`
- First path: the delegation-triage report (required).
- Second path: the original PRD / spec / user-story document (required — the triage report
  summarises it; the plan needs the source detail).
- Third path (optional): repository root, for grounding file paths, existing patterns, and
  affected components in reality rather than guesses.

---

## Phase 0 — Intake

Read the triage report and the original spec in full. Extract and hold:
- The Work Item list with both ownership tiers, complexity, and rationale per item
- The Sequencing Notes — these are **binding constraints** on plan order, not suggestions
- The Missing Information Report — every unresolved `{{PLACEHOLDER}}` and `(inferred)`
  judgment carries forward into this plan unless the spec resolves it

If a codebase root was supplied, locate for each Work Item: the files/projects likely
affected, the nearest existing pattern to follow, and the relevant test project. Use
`Glob`/`Grep` reconnaissance; read files only where the plan genuinely depends on their
contents. If no codebase root was supplied, express file paths and pattern references as
`{{PLACEHOLDER}}` values rather than inventing them.

If the triage report and the spec contradict each other, the spec wins for *what* to build
and the triage report wins for *who builds it and in what order*. Record any such conflict
in the Missing Information Report.

Do not ask clarifying questions mid-run. Resolve ambiguity conservatively and record every
judgment call with `(inferred)`.

---

## Phase 1 — Plan Skeleton

Produce a milestone structure before any per-item detail:

1. Group Work Items into **Milestones** — coherent, independently-verifiable slices of the
   system, ordered to satisfy every Sequencing Note (human-designed tests before delegated
   implementation; characterisation before modification; inter-item dependencies).
2. Within each milestone, order items so that human tight-loop work and agent work can run
   **in parallel where safe** — call out explicitly which items can proceed concurrently and
   which must serialise.
3. Mark each milestone with an exit criterion: what must be true (tests passing, review done,
   checkpoint approved) before the next milestone starts.

Present the skeleton as a table:
`Milestone | Work Items | Parallel lanes | Exit criterion | Sequencing constraints honoured`

**STOP.** Do not proceed to Phase 2 until the user responds with one of:
- `Plan` — proceed to detailed planning with the skeleton as-is
- `Adjust: <instructions>` — revise the skeleton per the instructions, then re-present and
  wait again

---

## Phase 2 — Detailed Planning

For every Work Item, produce an entry matched to its ownership tier. All entries share a
common header:

```
### WI-xx — <title>
Milestone: <n> | Impl: <tier> | Test: <tier> | Complexity: <S/M/C>
Spec source: <section/story reference in the original document>
Depends on: <WI ids or "none">
Affected areas: <projects/files/components — real paths if codebase supplied, else {{PLACEHOLDER}}>
Pattern to follow: <nearest existing precedent, or "novel — see design notes">
```

Then a tier-specific body:

**Agent-Autonomous items → Agent Briefing Packet.** A self-contained brief that can be pasted
into a fresh agent session with zero additional context:
- Objective (one paragraph, in terms of observable behaviour)
- Acceptance criteria (testable, enumerated; derived from spec or `{{NEEDS ACCEPTANCE CRITERIA}}`)
- Constraints (conventions to follow, EF Core migrations excluded unless the triage opted in,
  test stack and naming convention, locality-of-logic expectations)
- Test expectations per the Test Ownership tier (if Test: Agent-Autonomous, the agent writes
  them; if human-owned, reference the pre-agreed test file/scenario list as the acceptance gate)
- Out of scope (explicit, to prevent the agent wandering into neighbouring items)
- Done means (a checklist the agent must satisfy before declaring completion)

**Agent-Assisted items → Briefing Packet + Checkpoint Contract.** Everything above, plus:
- Checkpoint definition: exactly what the agent produces before pausing (e.g., schema +
  interface signatures, or a characterisation-test pass), what the human reviews, and the
  exact resume trigger word (e.g., `Continue WI-xx`)

**Human Tight-Loop items → Work Package.** Not a briefing — a plan for the human:
- Increment breakdown: ordered steps sized per the triage's suggested increment size, each
  independently verifiable
- Decision points: the specific judgment calls this item was classified tight-loop *for*,
  surfaced as explicit questions to answer during implementation rather than buried
- Delegable slivers: the narrow sub-tasks safe to hand to an agent mid-flow, with a one-line
  micro-brief for each
- Test approach per the Test Ownership tier

**Test-only entries.** Where a Work Item's tests are Human-Led or Scenario-Review and must
precede a delegated implementation (per Sequencing Notes), emit a separate preceding entry
`WI-xx-T` in the plan so the ordering is visible in the milestone structure, not just implied.

---

## Phase 3 — Output Assembly

Assemble the final plan document in this order:

1. **Header** — source triage report path, source spec path, generation date, and a one-line
   provenance note ("plan derives ownership and ordering from the triage report; content
   detail from the spec").
2. **Executive summary** — milestone count, item counts by tier, expected parallelism, and
   where the plan's risk is concentrated.
3. **Milestone plan** — the approved skeleton table, followed by a simple dependency list in
   prose (no diagram required; keep it greppable).
4. **Per-item detail** — every Work Item entry from Phase 2, grouped by milestone, in
   execution order, `WI-xx-T` test entries placed before the implementations they gate.
5. **Missing Information Report** — carried-forward items from the triage report plus
   everything new marked `{{PLACEHOLDER}}` or `(inferred)` in this plan. Group by "blocks
   which Work Item" so the user can resolve the highest-impact gaps first.
6. **Model recommendation** — reuse the capability-class table from the triage report
   verbatim (do not restate concrete model names independently; reference the triage report's
   table and note its currency caveat applies). Add only per-item deviations, e.g. "WI-07's
   briefing is unusually long-context — prefer the provider's large-context option".
7. **Self-verification checklist** — confirm before declaring done:
   - [ ] Every Work Item from the triage report appears exactly once (plus a `WI-xx-T` entry
         where required); none added, none dropped
   - [ ] Every Sequencing Note from the triage report is honoured by the milestone order
   - [ ] Every Agent Briefing Packet is self-contained (a fresh session could execute it
         without this conversation or the spec open)
   - [ ] Every Agent-Assisted item has a checkpoint contract with an exact resume trigger
   - [ ] Every Human Tight-Loop item's decision points trace back to the dimension(s) that
         made it tight-loop in the triage
   - [ ] No briefing packet asks an agent to write tests for an item whose Test Ownership is
         Human-Led or Scenario-Review
   - [ ] EF Core migrations appear only in items where the triage explicitly opted in
   - [ ] Every `{{PLACEHOLDER}}` and `(inferred)` marker appears in the Missing Information
         Report

**STOP.** Present the assembled plan in the conversation. Then wait for the user to respond
`Write plan` before writing the file. On confirmation, write to
`docs/plans/implementation-plan-<spec-file-name>.md` (kebab-case, derived from the source
spec's filename). Create `docs/plans/` if it doesn't exist. Do not write anywhere else in the
repository.

---

## Out of Scope

This command does **not**:
- Re-triage ownership or ordering — tiers and Sequencing Notes come from `delegation-triage`;
  if they look wrong, re-run the triage rather than silently overriding it here.
- Write implementation code or tests — briefing packets and work packages are handed to
  Claude Code sessions, TDD agents, or the user story implementer per item.
- Perform code review — use the code review agents once delivered work lands.
- Generate the PRD analysis itself — that's upstream (`prd-implement` or equivalent).
