---
description:  delivery-console 

disable-model-invocation: true
model: inherit
allowed-tools: Read, Grep, Glob, Write
---

description: Generate an interactive single-file HTML delivery console from files added in the context
/delivery-console — build a delivery console from planning docs
Build a single self-contained HTML file ("delivery console") from the planning documents
This encodes a proven pipeline; follow it
rather than improvising.

1. Discover and classify the source docs
Glob for *.md and classify each by content, not filename. Expect some subset of:

Backlog / user stories — US-xx headings with priorities (MoSCoW or similar) and points.
Implementation plan — architecture + per-story breakdowns (owning services, tasks,
entities, endpoints, test plan, dependencies), usually grouped into iterations/milestones.
Delegation triage — work items (WI-xx or similar) with tiers (agent-autonomous /
agent-assisted / human tight-loop), test ownership, model recommendations, sequencing notes,
missing-information report.
Execution/scaffold record — what was actually delivered, deltas vs the plan.
Not all will exist. Build with what's there and say plainly what's missing (e.g. "no triage doc
→ no delegation lanes"). Detect the ID families in use (US-, WI-, BR-, G-, NFR-,D-, OI-, or project-specific equivalents) by grepping; do not assume mine.

2. Extraction — write Python, parse structurally, never paraphrase
Work in a scratch dir (e.g. _console_build/). Write small extraction scripts
(markdown + beautifulsoup4; pip install --break-system-packages if needed) producing JSON,
then renderers, then an assembler. Rules learned the hard way:

Regex the document's own structure (heading patterns, **Field:** lines, table rows,- [ ] bullets). Print counts after every extraction and reconcile against totals the docs
state about themselves ("44 work items", "43 stories / 206 points"). A mismatch is a parser
bug until proven otherwise.
Strip parenthetical qualifiers before extracting dependency edges — "US-17 (US-32 hook
stubbed)" contains exactly one real dependency. A stubbed/optional mention parsed as an edge
will create phantom cycles.
Cross-document joins (WI→US, US→iteration) must be verified, not guessed: trace each via
the text (titles, acceptance criteria) and assert the join is total; items that genuinely
map to nothing get an honest "cross-cutting / no source story" label, never an invented one.
Provenance discipline: everything rendered is either (a) verbatim/structural from a source
doc, or (b) clearly labelled authored synthesis ("Authored guidance — synthesised from this
item's triage entry"). Never silently blend the two.
{{PLACEHOLDER}} tokens and (inferred …) markers are semantically load-bearing — surface
them, never resolve them.
3. Model-tier lookup — fetch each build, never hardcode from memory
Coding-model lineups turn over in weeks, and this command may run long after it was last edited.
Before rendering any model recommendation, fetch the current model docs and rebuild the tier map
from what's actually there — do not trust a remembered model name or ID:

Anthropic: https://platform.claude.com/docs/en/about-claude/models/overview
OpenAI: https://developers.openai.com/api/docs/models
Gemini: https://ai.google.dev/gemini-api/docs/models (andhttps://ai.google.dev/gemini-api/docs/thinking for each model's thinking-level options and
default)
From each, extract: model name, API model ID, and its reasoning/effort/thinking-level parameter
— the option values it accepts and its default. Build a three-tier coding map that reuses the
delegation-tier vocabulary already in play (human tight-loop / agent-assisted /
agent-autonomous), so one badge drives both the delegation lane and the model choice:
TierForPick per providerHuman tight-loop → heaviest reasoningArchitecture calls, security-sensitive or highly ambiguous work, hard bugsEach provider's flagship reasoning model, effort/thinking set to its highest levelAgent-assisted → balancedMost stories: standard business logic, typical endpoint/entity work, test authoringEach provider's mid-tier model, effort/thinking at default or mediumAgent-autonomous → fast/cheapBoilerplate, scaffolding, mechanical/low-risk, high-volume itemsEach provider's fastest/cheapest model, effort/thinking off or lowAs of the last fetch behind this spec (verify this is still current — re-fetch the three URLs
above and replace this table if it's stale, dated 2026-07-18), that resolved to:
TierAnthropicOpenAIGeminiHeaviest reasoningClaude Opus 4.8 (claude-opus-4-8) — effort: highGPT-5.6 Sol (gpt-5.6-sol) — reasoning: high–xhighGemini 3.1 Pro (gemini-3.1-pro-preview) — thinking_level: highBalancedClaude Sonnet 5 (claude-sonnet-5) — effort: high (medium for lighter items)GPT-5.6 Terra (gpt-5.6-terra) — reasoning: mediumGemini 3.5 Flash (gemini-3.5-flash) — thinking_level: mediumFast/cheapClaude Haiku 4.5 (claude-haiku-4-5) — extended thinking: off/lowGPT-5.6 Luna (gpt-5.6-luna) — reasoning: low–noneGemini 3.1 Flash-Lite (gemini-3.1-flash-lite) — thinking_level: low–minimalFor the rare item that genuinely needs more than the top tier — sustained multi-file agentic
work spanning many sessions — note Claude Fable 5 (claude-fable-5, adaptive thinking always
on) as the ceiling option rather than inventing a fourth tier; mention it only when the item's
own triage entry calls for that kind of long-horizon autonomy.
Treat the table above as a fallback only. If the live fetch succeeds, use what was actually
retrieved (model names, IDs, and effort/thinking options change independently of each other,
and new tiers get added) and say so in the console's footer. If the fetch fails (no network
access at build time), use this fallback table and mark every model badge it produced
"unverified — provider docs unreachable at build time."

4. The console — structure and features
One HTML file, zero external dependencies except Google-Fonts @import (must degrade
gracefully). Vanilla JS. Never use localStorage/sessionStorage — state lives in-memory plus a
hand-editable JSON block (below).
Tabs: Plan (work-item cards grouped by iteration) · Stories · Triage overflow (doc sections
not belonging to one item) · Decisions/record · Reference. Sticky header: brand, global search
(/ focuses), tab nav. Sticky filter bar on Plan: Owner tier, Size, Model tier, Status chips +
Reset + visible-count.
Hero tile board: one small numbered tile per work item, columns = iterations, colour =
delegation tier, click = jump to card. Legend + headline stats.
Work-item cards (<details>): summary row = spine (tier colour), ID chips, title, badges
(tier, test tier, complexity, model tier, gated flag), status pill. Body =
meta grid (group, risk, confidence, Depends on / Blocks with transitive downstream count,
critical-path marker) → Rationale → Model & thinking (implementation + test-writing tiers,
with the source table's ageing caveat) → Model recommendation (cross-provider equivalents +
effort/thinking level, per the model-tier map) → Implementation notes (triage acceptance
criteria / checkpoint / out-of-scope) → Implementation recommendations (the plan's
per-story task checklist, owning services, entities, endpoints, test plan, dependencies) →
Test ownership → Source story (nested collapsible with full spec text).
Implementation recommendations — required on every work item. Each card MUST carry an
"Implementation recommendations" section telling the implementer how to approach the item:

Primary source: the plan's own per-story breakdown (task checklist with done-state,
owning services, new/changed entities, new/changed endpoints, test coverage plan,
dependencies). If an item spans multiple stories, render each scope. Footnote it as coming
from the plan.
Fallback: items with no plan block (cross-cutting/process/decision items) get a short
authored note synthesised strictly from that item's own triage entry — sequencing, what to
decide, what to hand an agent — explicitly labelled as authored guidance so it can't be
mistaken for approved plan content.
No card ships without one; make the recommendation text searchable so entity/endpoint names
are findable from global search.
Model recommendation — required on every work item. Each card MUST also carry a "Model
recommendation" block, driven by the item's own complexity/risk/confidence fields from the
model-tier lookup above:

Start from the item's delegation tier, but bump it up one level when the item's own
complexity or risk fields warrant it (e.g. an agent-autonomous item flagged high-risk gets
the balanced-tier pick, not the fast/cheap one) — note the bump and the field that triggered
it; never bump down.
Render all three provider picks — provider — model name (model id) — effort/thinking setting — never just one, since implementers vary in which API they hold a key for.
One line of rationale drawn from the item's own fields (complexity, risk, test tier) —
never invented.
A footnote citing the model-tier lookup's fetch date and the three source URLs, so a stale
console is self-evidently stale.
Keep model names and IDs as plain searchable text (not buried only inside a collapsed
detail) so global search can answer "which items need Opus / Sol / Pro."
Cross-linking: every ID mention anywhere (outside code/pre/a) becomes a jump chip.
Jumps are script-driven (data-jump + a goTo() that switches tab, opens ancestor <details>,
scrolls, flashes) — never bare href="#anchor", which breaks in sandboxed previews.
Status tracking: a commented, hand-editable <script id="status-json" type="application/json"> block near the top of the file is the database
(todo/active/done/blocked; seed blocked from the docs' own hard-block flags). Status pill on
each card cycles states; tiles dim on done; iteration headers show "n/m done"; hero shows a
Done stat; a "Copy status JSON" button exports current state for pasting back into the block.
Copy-briefing button per card: assembles a self-contained agent packet to the clipboard —
global constraints (stack, naming decisions, invariants, placeholder/inference handling,
checkpoint etiquette; derive these from the docs) + work package (tiers, model, risk, deps) +
model recommendation (all three provider picks + effort/thinking settings) + triage
instructions + plan breakdown + full source-story text. Store packets in a<script id="briefings-json" type="application/json"> block (escape </ as <\/).
Clipboard: navigator.clipboard.writeText with a hidden-textarea execCommand fallback and
visible "✓ Copied" feedback.
Reference tab: traceability matrix (story × implementing-WI, flag uncovered stories,
points per iteration) · dependency view (critical path = longest chain weighted by points,
topological; widest-blockers table from transitive downstream sets) · a single "needs a human
decision" rollup (open items, gates, placeholder tokens → affected WIs) · model tier legend
(the three-tier × three-provider table from the model-tier lookup, plus its fetch date and the
three source URLs, so staleness is auditable in one place) · ID index for the non-US/WI
families — if the docs contain no canonical definitions, say so explicitly and build a
usage-derived index (every citation with doc:line, linked to affected items) · glossary derived
from usage, labelled as such.
Footer: generation date + sha256/12 of each source file, with a note that changed hashes
mean the console is stale, and that status persists only via the status-json block.
Also include: a collapsible "How to use this console" explaining the two-lane workflow,
briefing button, and status persistence model.

5. Design system
Print-adjacent editorial console: light grey paper #e8ebee, white cards, navy chrome#16324f; IBM Plex Serif (headings) / Sans (body) / Mono (chips, labels, metadata); 2–3px
radii; thin borders over shadows; 4px tier-coloured spines on cards. Tier palette: human#8c2f3d, assisted #9a5e14, autonomous #12655a, gate/blocked #5b3f8c — reuse this same
palette for model-tier badges (heaviest reasoning = human colour, balanced = assisted colour,
fast/cheap = autonomous colour) so the two axes read as one visual language. Uppercase
letter-spaced mono micro-labels. Responsive to ~380px (filters unstick, badges wrap).

6. Verify before delivering
Structural: details count == summary count; card count == source work-item count;
briefings/status entries == card count; every data-jump target id exists.
Model coverage: every card carries a Model recommendation with all three provider picks;
the fetch-date footnote is present; if the live fetch failed, every affected badge is marked
unverified rather than silently using the fallback table.
Functional (if a headless browser is available — try python3 -c "import playwright"):
no console errors; a tile click opens the right card; a cross-tab ID jump lands and opens;
filters + search + status cycle + both copy buttons work; screenshot desktop and ~390px
widths and actually look at them.
Honesty pass: re-read rendered rationale/criteria for a random 3 items against the source
markdown — verbatim or labelled-authored, nothing else.
Deliver the single HTML file named <project>-delivery-console.html and summarise: doc
classification, join coverage, anything unparseable, and any honesty labels applied.