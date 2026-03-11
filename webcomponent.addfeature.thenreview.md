# TDD Web Component Feature Implementation + Code Review

## Objective

Enhance an existing JavaScript web component by adding new features using strict Test-Driven Development (TDD), then perform a comprehensive code review of the changes and generate an actionable remediation plan for any issues discovered.

This prompt executes in **two phases**:
1. **Phase A: Feature Implementation** — TDD-driven feature development
2. **Phase B: Code Review & Remediation Plan** — Quality assurance and improvement planning

---

## Constraints (All Phases)

- **No frameworks:** React, Vue, Angular, Lit, Svelte, etc.
- **No build tools:** Webpack, Vite, Babel, Rollup, etc.
- **Target environment:** Modern ES modules in evergreen browsers
- **APIs:** Custom Elements v1 and Shadow DOM only

---

## Agent Execution Instructions

**AUTONOMOUS EXECUTION REQUIREMENT**: Proceed without user confirmation until both phases are complete. Only pause for user feedback if a blocking ambiguity requires clarification.

---

# PHASE A: Feature Implementation (TDD)

## A.1 Baseline Verification

Complete these tasks before making code changes:

1. **Project Reconnaissance**: Inspect the current project structure, existing component implementation(s), and tooling configuration.
2. **Test Suite Assessment**: Run the full test suite to establish the green baseline; document any failures you did not cause.
3. **Documentation Review**: Read the current docs (`docs/*.md`, `README.md`, `readme.technical.md`) to understand existing capabilities and constraints.
4. **Feature Requirements Extraction**: Derive the exact feature(s) to implement from the task context. If ambiguity remains, enumerate open questions before coding.
5. **Change Journal Entry**: Create or append to `docs/steps.md` with a baseline entry summarizing current state.
6. **Checkpoint Creation**: Generate `docs/checkpoint-YYYY-MM-DD.md` with comprehensive summary of pre-implementation state.

## A.2 TDD Development Cycle

> **🚨 MANDATORY TEST-FIRST RULE 🚨**  
> **TESTS MUST BE WRITTEN BEFORE CODE - NO EXCEPTIONS**

For EVERY feature:

1. **RED** — Write the test that fails for the new behavior.
2. **GREEN** — Write the minimal production code to satisfy the test.
3. **REFACTOR** — Improve design while keeping tests green.

### Cycle Requirements

- Never write implementation code before its corresponding failing test.
- Run the entire test suite after each GREEN phase to detect regressions immediately.
- Document each cycle (Red/Green/Refactor, key decisions, test names) in `docs/steps.md`.
- Keep `docs/spec.md` focused on the current, authoritative specification.
- Update `docs/README.md` and `docs/readme.technical.md` with user-facing and technical insights.
- Refresh `examples/demo.html` so the new feature is discoverable and verifiable.

### Feature Completion Criteria

Before proceeding to Phase B, ensure:

- [ ] All new tests pass (GREEN achieved)
- [ ] All existing tests still pass (no regressions)
- [ ] Code has been refactored for clarity and consistency
- [ ] Documentation reflects the new feature
- [ ] Demo showcases the new functionality
- [ ] Styles use Constructable Stylesheets (or existing pattern) without regressions

---

# PHASE B: Code Review & Remediation Plan

After feature implementation is complete, perform a comprehensive code review of the **entire component** (not just the new code) and generate an actionable improvement plan.

## B.1 Code Review Execution

Analyze the component against the following categories:

### Review Categories

| Priority | Category | Focus Areas |
|----------|----------|-------------|
| High | **Security** | XSS via innerHTML/outerHTML, unsanitized user input, eval usage, unsafe URL handling, slot content injection |
| High | **Accessibility** | ARIA roles/attributes, keyboard navigation, focus management, screen reader compatibility, semantic HTML |
| High | **Performance** | Layout thrashing, excessive DOM manipulation, memory leaks, inefficient event handlers, unnecessary re-renders |
| Medium | **Lifecycle** | Proper connectedCallback/disconnectedCallback usage, attribute observation, upgrade handling |
| Medium | **API Design** | Consistent naming, attribute reflection, event patterns, slot usage, CSS custom properties |
| Medium | **Error Handling** | Graceful degradation, input validation, meaningful error messages |
| Low | **Code Quality** | DRY violations, dead code, naming conventions, documentation gaps |
| Low | **Maintainability** | Complex conditionals, magic numbers, coupling issues |

### Review Output Format

Generate a structured review document with:

```markdown
## Code Review Summary

**Component:** [Component name]
**Review Date:** [Date]
**Lines of Code:** [Count]
**Test Coverage:** [Percentage if available]

### Health Assessment

[2-3 sentence overall assessment of component health]

### Issues Found

| Priority | Category | Code Location | Explanation & Suggestion |
|----------|----------|---------------|--------------------------|
| High/Medium/Low | [Category] | [Method/line] | [What's wrong and how to fix it] |

### Positive Observations

[List 2-3 things done well to maintain context]

### New Feature Assessment

[Specific review of the just-implemented feature - any concerns or improvements?]
```

## B.2 Implementation Plan Generation

Based on the code review, generate a prioritized remediation plan:

### Executive Summary

Provide a brief (3–5 sentence) overview:
- Total number of issues by priority (High / Medium / Low)
- Key risk areas requiring immediate attention
- Estimated complexity for full remediation
- Recommended approach (single PR, phased rollout, etc.)

### Implementation Phases

Organize work into logical phases. Each phase should be independently deployable and testable.

#### Phase Structure Template

```markdown
## Phase [N]: [Phase Title]

**Goal:** [One-sentence description]
**Risk Level:** Low / Medium / High
**Dependencies:** [List phases that must complete first, or "None"]
**Estimated Effort:** XS / S / M / L / XL

### Tasks

| Task # | Issue Reference | Task Description | Acceptance Criteria | Existing Tests | New Tests Required |
|--------|-----------------|------------------|---------------------|----------------|-------------------|
| [N.1]  | [Priority-Category-Location] | [Specific action] | [Verification method] | [Tests that must pass] | [New tests to add] |
```

#### Phase Guidelines

- **Phase 1**: High-priority Security issues
- **Phase 2**: Remaining High-priority issues (Accessibility, Performance, Lifecycle)
- **Phase 3+**: Medium-priority issues, grouped by related code or patterns
- **Final Phase**: Low-priority issues and enhancements
- Each phase: 3–7 tasks

### Test Plan

For each issue, specify:

| Issue Reference | Test Type | Test Description | Test File Location |
|-----------------|-----------|------------------|-------------------|
| [From review] | Regression | [Test that would have caught the bug] | [path] |
| [From review] | Validation | [Test that confirms the fix works] | [path] |
| [From review] | Edge Case | [Test for boundary conditions] | [path] |

### Risk Assessment

| Risk | Likelihood | Impact | Mitigation Strategy |
|------|------------|--------|---------------------|
| [Description] | Low/Medium/High | Low/Medium/High | [How to reduce/avoid] |

### Code Change Previews

For High-priority issues, provide before/after code snippets:

```markdown
#### Issue: [Reference]

**Before:**
```javascript
// Problematic code
```

**After:**
```javascript
// Fixed code
```

**Explanation:** [Why this fixes the issue]
```

---

## B.3 Final Deliverables Checklist

Before completing, ensure all outputs are generated:

### Feature Implementation Outputs
- [ ] Feature code implemented and working
- [ ] All tests passing (new + existing)
- [ ] `docs/steps.md` updated with TDD cycle log
- [ ] `docs/spec.md` updated with feature specification
- [ ] `docs/README.md` updated with usage documentation
- [ ] `docs/readme.technical.md` updated with implementation details
- [ ] `examples/demo.html` updated with feature demonstration
- [ ] `docs/checkpoint-YYYY-MM-DD.md` created

### Code Review Outputs
- [ ] `docs/code-review-YYYY-MM-DD.md` — Full review document
- [ ] `docs/remediation-plan-YYYY-MM-DD.md` — Implementation plan for issues

---

## Output File Templates

### docs/code-review-YYYY-MM-DD.md

```markdown
# Code Review: [Component Name]

**Date:** [YYYY-MM-DD]
**Reviewer:** Claude (Automated)
**Trigger:** Post-feature implementation review

## Summary

[Health assessment]

## Metrics

- **Lines of Code:** [N]
- **Test Count:** [N]
- **Test Coverage:** [X%]
- **Issues Found:** [N High, N Medium, N Low]

## Issues

| # | Priority | Category | Location | Issue | Suggestion |
|---|----------|----------|----------|-------|------------|
| 1 | High | Security | render():42 | innerHTML with user input | Use textContent or DOM APIs |

## Positive Observations

1. [Good practice observed]
2. [Good practice observed]

## New Feature Assessment

[Review of just-implemented feature]
```

### docs/remediation-plan-YYYY-MM-DD.md

```markdown
# Remediation Plan: [Component Name]

**Generated:** [YYYY-MM-DD]
**Based on:** code-review-YYYY-MM-DD.md

## Executive Summary

[3-5 sentences]

## Phase 1: [Title]

**Goal:** [Description]
**Risk:** [Level]
**Effort:** [Size]

| Task | Issue | Description | Acceptance | Tests |
|------|-------|-------------|------------|-------|
| 1.1 | H-Sec-render | Replace innerHTML | No XSS vectors | +2 new |

[Continue for all phases...]

## Risk Assessment

[Table of risks]

## Test Plan

[Table of required tests]

## Code Previews

[Before/after for high-priority items]
```

---

## Validation Checklist

Before finalizing, verify:

- [ ] Feature works as specified
- [ ] All tests pass (existing + new)
- [ ] TDD cycle was followed (RED → GREEN → REFACTOR)
- [ ] Code review covers all categories
- [ ] All High-priority issues addressed in Phases 1-2 of remediation plan
- [ ] Each remediation task has clear acceptance criteria
- [ ] Each task specifies existing tests that must pass
- [ ] Each task defines new regression and validation tests
- [ ] No frameworks or build tools suggested anywhere
- [ ] All code examples are valid Vanilla JS + Web Components
- [ ] Documentation is complete and accurate

---

## Success Criteria

1. **Feature Complete**: New functionality works as specified with full test coverage
2. **No Regressions**: All existing tests continue to pass
3. **Quality Assessed**: Comprehensive code review identifies improvement opportunities
4. **Actionable Plan**: Clear, prioritized remediation roadmap with test requirements
5. **Documentation Current**: All docs reflect the enhanced component state
6. **Demo Updated**: New feature is visible and testable in the demo

**Remember**: Deliver value incrementally, maintain the TDD discipline, ensure quality through review, and leave the project healthier than you found it.