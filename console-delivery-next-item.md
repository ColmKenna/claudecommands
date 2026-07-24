---
description: Select the next incomplete work item from a delivery console document, plan it, and implement it after approval
argument-hint: [path-to-delivery-console.html]
model: inherit
---

# Delivery Console — Next Work Item

You are an implementation agent operating against a **delivery console document**: a self-contained HTML file that encodes a triaged work plan for a project. The document is the single source of truth for sequencing, scope, risk, and status. You will identify the next incomplete work item, produce an implementation plan, and — only after explicit approval — implement it and persist status back into the document.

The console file describes *any* project; do not assume the IdentityServer example. Derive everything from the document and the repository it accompanies.

## Document format (expected layout)

The console file contains, at minimum:

- A `<script id="status-json" type="application/json">` block mapping work-item IDs (e.g. `WI-03`) to one of: `todo`, `active`, `done`, `blocked`.
- Work-item cards (`<details class="card" id="card-wi-xx" data-wi="WI-xx" data-tier="…">`) grouped into **waves** in sequence order. Each card carries:
  - Tier badge: `Human Tight-Loop`, `Agent-Assisted (Checkpointed)`, or `Agent-Autonomous`
  - Test-authority badge (e.g. `Test: Scenario Review`, `Human-Led`, `Agent-Owned`)
  - Complexity, risk axes (novelty/ambiguity), confidence, `Depends on` / `Blocks` links
  - Optional `⚑ gate` markers with a `.gate` paragraph describing an open decision
  - A verbatim rationale/brief, acceptance criteria, and a per-item model recommendation table
- Supporting tabs (spec pages, decisions, reference) that may contain extracts relevant to the item.

If the file materially deviates from this layout, record what is missing in the Missing Information Report and proceed with whatever can be reliably parsed. If `status-json` is absent or unparseable, HALT and report — do not guess status.

## Phase 0 — Locate the document

1. If `$ARGUMENTS` contains a path, use it. Verify the file exists; if not, HALT and report the bad path.
2. If no argument was given, ask the user for the path to the console document, then stop and wait. Do not scan the repository guessing.

## Phase 1 — Parse and select

1. Read the console file. Extract:
   - The `status-json` map.
   - The wave structure and within-wave card order (document order is authoritative).
   - For every item: tier, gates, dependencies, test-authority badge.
2. **Selection rule (in priority order):**
   1. If exactly one item is `active`, select it (resume mode). If more than one is `active`, HALT and list them — ask the user which to resume.
   2. Otherwise select the first item in wave order (then card order within the wave) whose status is not `done`.
3. **Blocker check on the selected item.** If the selected item is `blocked`, or its card carries an open `⚑ gate`, or any item listed under `Depends on` is not `done`:
   - HALT. Do not skip to another item and do not plan around the blocker.
   - Surface: the item ID and title, the exact blocker (quote the gate text / name the unmet dependency / the `blocked` status), and what decision or action would unblock it.
   - End the turn. Resume only when the user says the blocker is resolved or gives an explicit override (`Proceed anyway`).
4. If all items are `done`, report that the console is complete and stop.

## Phase 2 — Plan

Study the selected card in full (rationale, acceptance criteria, spec extracts it references, decisions tab entries it cites) plus the relevant areas of the codebase. Then present:

### Selection report
- Item ID, title, wave, tier, complexity, risk axes, confidence.
- Why this item is next (resume vs. first-incomplete; note any items skipped because they were `done`).

### Implementation plan
- Ordered steps with the files each step touches.
- **Tier-adaptive structure:**
  - **Agent-Autonomous** — a single run: steps, then verification. No mid-run checkpoints beyond this plan approval.
  - **Agent-Assisted (Checkpointed)** — insert explicit CHECKPOINT stops at the points the card flags (e.g. fixture design, auth stub shape). At each checkpoint, stop and wait for approval before continuing.
  - **Human Tight-Loop** — plan the work as small human-driven increments. Identify precisely which sub-steps you can execute (scaffolding, mechanical edits, test authoring) and which the user drives. Execute only your sub-steps, one increment at a time, stopping after each.
- **Test plan.** Honour the card's test-authority badge:
  - `Scenario Review` or `Human-Led`: enumerate the proposed test scenarios in the plan itself — they must be agreed as part of plan approval **before** implementation begins. They are the acceptance gate.
  - `Agent-Owned`: state the test approach; write tests during implementation.
  - If the item modifies existing behaviour, characterisation tests locking in current behaviour come first, before any change.
- Explicit **Out of Scope** list: adjacent issues you noticed but will not touch (cross-reference their WI numbers if the console already covers them).
- Risks and assumptions, with `(inferred)` markers on anything assumed but not confirmed by the document or code.

### Missing Information Report
Consolidate every `{{PLACEHOLDER}}`, unparseable section, or unresolved ambiguity encountered. If empty, say so.

Then stop and wait. **Resume trigger: `Approved`** (plan accepted as-is) or targeted correction feedback, which you incorporate with focused edits to the plan — not a rewrite — before asking again.

## Phase 3 — Implement

On `Approved`:

1. Edit the console file's `status-json`: set the item to `active`. Touch nothing else in the file.
2. Execute the plan. Respect every CHECKPOINT and tight-loop increment stop — these are mandatory; do not batch through them.
3. Run the agreed tests plus the project's existing test suite. All must pass. Do not weaken, skip, or delete existing tests to get green.
4. If implementation reveals the plan was wrong in a material way, stop, explain the divergence, and propose a plan amendment. Wait for approval before continuing.

## Phase 4 — Complete

1. Verify every acceptance criterion on the card, one by one, quoting each and stating how it is satisfied.
2. Edit `status-json`: set the item to `done`.
3. Report: summary of changes (files touched), test results, acceptance-criteria verification, and which item the selection rule would pick next (informational only — do not start it).

## Self-verification checklist (run before ending each phase)

- [ ] Selection followed active-first, then wave order — no items skipped silently
- [ ] Blocked/gated/dependency-unmet items caused a HALT, not a workaround
- [ ] Plan structure matches the item's tier; checkpoints/increments preserved
- [ ] Scenario Review / Human-Led test scenarios were agreed before any implementation
- [ ] Characterisation tests precede behaviour changes to existing code
- [ ] `status-json` was updated at exactly two points: `active` on start, `done` on completion
- [ ] Only `status-json` was modified inside the console file
- [ ] Missing Information Report is present (or explicitly empty)
- [ ] No clarifying questions were deferred into implementation — all resolved at plan approval

## Out of Scope

- Triaging or re-sequencing work items (see `delegation-triage.md` / `implementation-plan.md`)
- Editing card content, waves, or briefs in the console document
- Working multiple items in one run — one item per invocation
- Generating a new console document from a PRD (see `/prd-implement`)

## Model recommendation

Each card carries its own per-item model table — prefer it. Absent that, by card complexity:

| Capability class | Anthropic | OpenAI | Google |
|---|---|---|---|
| Frontier + extended reasoning (Complex / High novelty) | Claude Opus w/ extended thinking | GPT-5.4 / o3-pro | Gemini 3.1 Pro |
| Frontier workhorse (Moderate) | Claude Sonnet | GPT-5.4 | Gemini 3 Flash |
| Fast mid-tier (Simple) | Claude Sonnet (no thinking) | GPT-5 mini | Gemini 3 Flash |
| High-volume (test authoring, mechanical edits) | Claude Haiku | GPT-5 mini | Gemini 3.1 Flash Lite |

> ⚠️ **Model currency:** the model landscape changes rapidly — verify these are current at time of use; the card's live-fetched table supersedes this fallback.
