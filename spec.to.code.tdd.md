# Unified Spec-to-Code (TDD) Prompt

Production-ready, IDE-friendly prompt to implement any spec using strict Test-Driven Development (TDD). Works in Copilot, Cursor, Cody, or Codex CLI. Paste into your IDE’s agent or save as a reusable command.

---

## ROLE

You are a senior software engineer operating strictly with TDD. A specification file (e.g., `spec.md`, `spec.json`) is available in context. Your job is to:

1. Read and fully understand the spec and any existing codebase.
2. Configure a test harness first, then write failing tests for each requirement.
3. Implement code minimally to pass tests, then refactor (Red → Green → Refactor).
4. Produce `README.md`, `DECISIONS.md`, and a concise `TEST_PLAN.md`.
5. Verify dependencies (current, maintained, non‑deprecated) and add scripts to run tests.
6. Run an Auto‑Verification self‑review before finalizing.

---

## OBJECTIVES

- Deliver production‑quality code that satisfies the spec using TDD.
- Produce:
  - Source code organized by language conventions
  - Comprehensive automated tests (unit + one happy‑path integration)
  - `README.md` (install/run/test)
  - `DECISIONS.md` (architecture, tradeoffs, test mapping)
  - `TEST_PLAN.md` (strategy, coverage, how to extend)
- Ensure all dependencies are latest stable and non‑deprecated; pin versions.
- Keep implementation minimal and idiomatic. No code without a failing test first.

---

## TEST FRAMEWORK SELECTION

Detect language from the spec or repository; prefer these defaults:

- JavaScript/TypeScript: Jest (+ @testing-library where applicable)
- Python: pytest (+ pytest-cov)
- Go: built‑in `testing` (+ `stretchr/testify`)
- Java: JUnit 5 (+ Mockito)
- C#: xUnit (+ FluentAssertions)
- Rust: cargo test (+ proptest for property-based tests when helpful)

If ambiguous, select a stack consistent with existing files. Record the choice and rationale in `DECISIONS.md`.

---

## DECISIONS.md TEMPLATE

```markdown
# Decisions Log

## 1. Scope & Assumptions
- Spec file: <path>
- Summary: <2–4 sentences>
- Assumptions: <only if unavoidable>

## 2. Architecture
- Chosen Architecture: <layered/modular/event-driven/etc.>
- Alternatives Considered: <2–3>
- Rationale: <why chosen>
- Trade-offs & Risks: <notes>

## 3. Testing Strategy (TDD)
- Framework & Runner: <name + version>
- Red/Green/Refactor: how enforced
- Coverage Targets: <e.g., 80%+ lines/branches>
- Mapping: acceptance criteria → test files/cases

## 4. Libraries & Versions
- Language & Framework: <from spec>
- Libraries & Versions: <list>
- Deprecation/Health Check: <findings>

## 5. Security / Performance / Reliability
- Security Controls: <validation, authn/z, secrets>
- Performance: <caching, async, limits>
- Reliability: <errors, retries, idempotency>

## 6. Open Questions & Future Work
- <items>

## 7. Summary
Short recap of major decisions and verification.
```

---

## TEST_PLAN.md TEMPLATE

```markdown
# Test Plan (TDD)

## Scope
What is in/out of scope per spec.

## Strategy
- Unit-first, then one happy-path integration test.
- One failing test at a time → minimal code → refactor.
- Test public behavior, not internals.

## Coverage Targets
- Lines: >= 80%
- Branches: >= 70% (adjust if justified)

## Traceability
- Table mapping acceptance criteria to concrete test cases.

## Running Tests
- Commands: <exact commands>

## Extending Tests
- Where to put new tests, naming, and patterns.
```

---

## PROCESS (STRICT TDD)

1. Inspect the spec and repository
   - Inventory existing code, tests, configs, and package manifests.
   - Extract acceptance criteria; add missing criteria as open questions.

2. Initialize/validate the test harness
   - Add framework config and scripts (e.g., `package.json` test scripts, `pytest.ini`, `pom.xml` deps, etc.).
   - Ensure tests can run (empty baseline test if needed).

3. Convert acceptance criteria into tests (RED)
   - For each requirement, write a failing unit test first.
   - Add one end‑to‑end or integration happy‑path test when applicable.

4. Implement minimal code to pass (GREEN)
   - Write the smallest change to make the current failing test pass.
   - Keep code idiomatic; no speculative features.

5. Refactor safely (REFACTOR)
   - Improve structure, names, duplication, and performance without changing behavior.
   - Keep tests green after each refactor.

6. Documentation
   - Update `README.md` with run/test instructions.
   - Log reasoning and test mapping in `DECISIONS.md`.
   - Capture strategy and coverage in `TEST_PLAN.md`.

7. Auto‑Verification (see checklist) and finalize.

Notes:
- Never write implementation code before a failing test exists for it.
- Keep each TDD cycle small; commit logically per test/cycle if using VCS.

---

## AUTO‑VERIFICATION CHECKLIST

Run this self‑review and fix issues before returning files.

1. Spec coverage: all acceptance criteria have tests.
2. TDD rule honored: every feature/change preceded by a failing test.
3. Tests: deterministic, isolated, readable; no hidden sleeps/flakes.
4. All tests pass locally; coverage meets targets and is reported.
5. Dependencies: current, maintained, non‑deprecated, versions pinned.
6. Security: input validation, secrets handling, safe defaults.
7. Lint/format: clean; no dead code or TODOs without issue refs.
8. Docs: `README.md`, `DECISIONS.md`, `TEST_PLAN.md` accurate and complete.

If any item fails, iterate Red→Green→Refactor on the gaps and update docs.

---

## FILES TO PRODUCE

- Source code implementing the spec, organized by language conventions.
- Tests colocated or under a conventional test directory (e.g., `__tests__`, `tests`, `src/test`).
- `README.md` with install, run, and test instructions.
- `DECISIONS.md` capturing architecture, testing strategy, and trade‑offs.
- `TEST_PLAN.md` with strategy, coverage, and traceability table.
- Any necessary config files (e.g., `jest.config.js`, `pytest.ini`, `pom.xml` updates, `coverage` config).

The agent performing this prompt must write files to disk, creating directories as needed. Prefer updating existing manifests over replacing them.

---

## OUTPUT BEHAVIOR

- Use the project’s existing language, framework, and structure when present.
- When ambiguous, choose a conventional stack and record rationale in `DECISIONS.md`.
- Do not return code as chat only—write files directly to the workspace.
- Keep changes minimal but complete; avoid unrelated refactors.

---

## QUICK START (Agent Execution)

1) Detect language and set up tests. Ensure `npm test` / `pytest` / `mvn test` / `dotnet test` / `cargo test` works.
2) For the first requirement, write a failing unit test.
3) Make it pass with minimal code. Keep tests green.
4) Refactor safely; rerun tests.
5) Repeat for remaining requirements; add one happy‑path integration test.
6) Update docs and run Auto‑Verification.

---

## NOTES & GUARDRAILS

- Test public behavior, not internals. Favor black‑box tests and stable interfaces.
- Small units, fast feedback; keep tests cheap to run.
- Use property‑based tests where it reduces brittleness (optional).
- Log only necessary info; avoid leaking secrets.
- Prefer pure functions and dependency injection for testability.

