---
description: "Guidelines for reviewing a JavaScript web component and producing an actionable remediation plan."
---

# Web Component Code Review Prompt

## Role

Act as a Senior Front-End Architect specializing in Security, Performance, Accessibility, and JavaScript Best Practices for production-ready Vanilla JavaScript Web Components.

### Constraints

- **No frameworks:** React, Vue, Angular, Lit, Svelte, etc.
- **No build tools:** Webpack, Vite, Babel, Rollup, etc.
- **Target environment:** Modern ES modules in evergreen browsers
- **APIs:** Custom Elements v1 and Shadow DOM only
- Do not propose framework migrations or build tool adoption as fixes

---

## Task

Perform a rigorous, production-grade code review of the provided Web Component code and generate an actionable remediation plan.

---

## Agent Execution Instructions

AUTONOMOUS EXECUTION REQUIREMENT: Proceed without user confirmation until the review and remediation plan are complete. Only pause for user feedback if a blocking ambiguity requires clarification (for example, multiple components are present and the target is unclear).

---

# Phase 1: Baseline Assessment

Complete these tasks before writing the review:

1. Project Reconnaissance
   - Inspect the project structure and locate the web component implementation(s)
   - Identify the main component under review
2. Documentation Review
   - Read current docs (docs/*.md, README.md, readme.technical.md)
   - Note intended behaviors, public APIs, and constraints
3. Test Suite Snapshot (if available)
   - Identify existing tests and coverage information
   - Run tests if feasible; document any failures you did not cause

---

# Phase 2: Code Review

Perform a rigorous, production-grade review of the provided Web Component code. Your review must cover all categories below.

## 1. Security and Web Component Best Practices

### XSS and Injection Risks

- Flag dangerous HTML injection patterns:
  - `innerHTML`, `outerHTML`, `insertAdjacentHTML`
  - Template string injection into DOM without sanitization
- Flag dynamic attribute assignments that could execute scripts:
  - `setAttribute('onclick', ...)`, `el.onclick = userInput`, `el.href = 'javascript:...'`
- Evaluate safety of slotted content handling - ensure slots cannot inject malicious markup into component logic

### Shadow DOM Encapsulation

- Verify correct use of `this.attachShadow({ mode: 'open' | 'closed' })`
- Confirm scoped DOM queries (no leaking selectors to light DOM)
- Verify style isolation is maintained

### CSP and Trusted Types

- Evaluate compatibility with Content Security Policy
- Recommend Trusted Types API usage where applicable for innerHTML-like operations

### Custom Element Lifecycle

- Review correct usage of lifecycle callbacks:
  - `constructor` - no DOM access, no attributes, no children
  - `connectedCallback` - setup, event listeners, observers
  - `disconnectedCallback` - full cleanup of listeners, observers, timers, subscriptions, AbortControllers
  - `attributeChangedCallback` - attribute handling, property reflection
  - `adoptedCallback` - document transition handling (if applicable)
- Confirm no DOM manipulation occurs in the constructor

### Anti-Patterns

- Identify web component anti-patterns:
  - Synchronous heavy work in lifecycle hooks
  - Reading layout properties immediately after DOM writes (forced reflow)
  - Failing to clean up resources on disconnect
  - Using `document.querySelector` instead of `this.shadowRoot.querySelector`

---

## 2. Performance and Efficiency

### DOM Manipulation

- Detect inefficient patterns:
  - Repeated `querySelector` / `querySelectorAll` calls for the same elements
  - Layout thrashing (interleaved reads and writes)
  - Unnecessary re-renders or redundant DOM updates

### Optimization Techniques

- Recommend where applicable:
  - Element caching (store references after initial query)
  - `DocumentFragment` for batch insertions
  - `requestAnimationFrame` for visual updates
  - Microtask scheduling (`queueMicrotask`) for batched state updates
  - Debouncing/throttling for high-frequency events

### Style Performance

- Recommend `adoptedStyleSheets` over repeated inline `<style>` elements for style reuse
- Flag dynamically created `<style>` tags on every render

### Attribute Handling

- Verify efficient handling of `observedAttributes`
- Check attribute-to-property reflection is not causing unnecessary updates
- Ensure `attributeChangedCallback` guards against no-op changes

### Memory Management

- Identify potential memory leaks:
  - Retained references to removed DOM nodes
  - Orphaned closures holding component references
  - Event listeners not removed on disconnect
  - Uncleared timers or intervals

### Lazy Initialization

- Suggest lazy initialization or deferred rendering for:
  - Components not immediately visible (below fold, collapsed sections)
  - Heavy sub-components that can be loaded on demand

### Blocking Operations

- Flag synchronous or heavy computations that may block the main thread
- Recommend Web Workers or chunked processing for CPU-intensive tasks

---

## 3. Accessibility (a11y)

### Semantic HTML

- Verify correct semantic HTML within Shadow DOM boundary
- Ensure ARIA supplements (not replaces) native semantics
- Flag div/span soup where semantic elements should be used

### Keyboard Accessibility

- Verify logical focus order and tabbing
- Check arrow-key navigation for composite widgets (menus, listboxes, tabs, grids)
- Ensure no mouse-only interactions exist
- Flag missing `tabindex` on custom interactive elements

### Focus Management

- Evaluate focus trapping for modal/overlay components
- Verify focus restoration when dialogs close
- Check programmatic focus calls (`focus()`) are appropriate

### Labeling and Descriptions

- Confirm proper use of:
  - `aria-label`
  - `aria-labelledby`
  - `aria-describedby`
- Verify Shadow DOM does not obstruct accessible name computation
- Check that labels are associated correctly across shadow boundaries

### Form Participation

- For form-associated components, verify:
  - Use of `ElementInternals` API
  - `static formAssociated = true` declaration
  - Proper form validation and submission participation

### Dynamic Content

- Evaluate live regions (`aria-live`, `role="status"`, `role="alert"`) for dynamic announcements
- Ensure state changes are communicated to assistive technology

### Motion and Animation

- Check for `prefers-reduced-motion` media query support
- Flag animations that cannot be disabled

### Visual Considerations

- Note contrast concerns when determinable from code
- Consider `forced-colors` / high-contrast mode support for critical UI indicators

---

## 4. General JavaScript Best Practices

### Code Quality

- Flag overuse of global variables or state leaking outside the component
- Identify poor variable naming or unclear function purpose
- Detect side effects in unexpected places (getters, constructors)

### Maintainability

- Check for modular structure with clear separation of concerns
- Identify repeated logic that should be extracted to helper functions
- Flag complex or deeply nested branching that should be simplified
- Recommend private fields (`#privateField`) for internal state

### Correctness

- Check for incorrect or inconsistent use of `this`
- Flag missing `const` / `let` or unintended `var` usage
- Identify misuse of Promises, `async/await`, or event handling
- Flag error-prone patterns:
  - Mutation of shared references
  - Incorrect array/object copying (shallow vs. deep)
  - Missing `await` on async operations

### Error Handling

- Verify `try/catch` around async operations
- Check for graceful fallbacks when APIs are unavailable
- Ensure errors do not silently fail or break component state

### Input Validation

- Validate and sanitize all external inputs:
  - Attributes
  - Slot content
  - User-provided data
  - URL parameters

### Modern JS Practices

- Encourage:
  - `const` for immutable references, `let` only when reassignment is needed
  - Minimal mutation; prefer creating new objects/arrays
  - Template literals over string concatenation
  - `Map` / `Set` when appropriate over plain objects/arrays
  - Optional chaining (`?.`) and nullish coalescing (`??`)
  - Private class fields (`#field`) for encapsulation

### Documentation

- Recommend JSDoc type annotations for public API methods
- Verify public interface (attributes, properties, methods, events) is documented

---

## 5. Code Clarity and Readability

### Self-Documenting Code

- Verify the component's purpose is immediately apparent from its class name, file structure, and public API
- Flag methods or logic blocks that require mental gymnastics to understand
- Identify magic numbers, cryptic variable names, or unexplained boolean flags
- Check that the component's public interface (attributes, properties, methods, events, slots) clearly communicates its intended usage

### Structural Clarity

- Evaluate whether the code follows a logical, predictable organization:
  - static properties -> constructor -> lifecycle -> public methods -> private methods -> event handlers
- Flag deeply nested conditionals or callbacks that obscure control flow
- Identify overly long methods that should be decomposed into smaller, named functions
- Check that related functionality is grouped together, not scattered throughout the file

### Naming Conventions

- Verify method names clearly describe their action (verbs for methods, nouns for properties)
- Flag ambiguous names: `handle()`, `process()`, `doIt()`, `temp`, `data`, `value`
- Check consistency in naming patterns throughout the component

---

## 6. Dead Code and Unreachable Paths

### Unreachable Code

- Identify code after `return`, `throw`, or `break` statements that can never execute
- Flag conditional branches that can never be true based on type or logic analysis
- Detect methods or properties that are defined but never called or accessed
- Identify event listeners registered for events that are never dispatched

### Redundant Code

- Flag duplicate logic that could be consolidated into shared helper methods
- Identify variables that are assigned but never read
- Detect unnecessary re-declarations or re-assignments
- Flag `else` blocks that mirror the `if` block or are otherwise superfluous

### Obsolete Code

- Identify commented-out code blocks that should be removed
- Flag TODO/FIXME comments referencing completed or abandoned work
- Detect polyfills or workarounds for browser features now universally supported in evergreen browsers

---

## 7. Comment Quality

### Redundant Comments

- Flag comments that merely restate what the code already expresses:
  - `// increment counter` above `counter++`
  - `// constructor` above `constructor()`
  - `// returns the value` above `return this.value`
- Identify excessive inline comments that clutter readable code

### Missing Comments

- Flag complex algorithms, regex patterns, or non-obvious logic lacking explanatory comments
- Identify workarounds or browser-specific hacks without documentation of why they exist
- Check that public API methods have clear descriptions of parameters, return values, and side effects

### Outdated Comments

- Flag comments that contradict or no longer match the code they describe
- Identify version-specific comments referencing outdated browser support

### Comment Style

- Recommend JSDoc for public API documentation when absent
- Flag informal or unprofessional comment language inappropriate for production code

---

## Review Principles

- **Accuracy over volume:** Only report genuine issues. Do not fabricate or exaggerate problems to fill the table.
- **Actionable specificity:** Every suggestion must include concrete code patterns or techniques the developer can immediately apply.
- **Proportional severity:** Reserve High priority for issues that could cause security vulnerabilities, data loss, accessibility barriers, or significant performance degradation in production.

---

## Output Format

### 1. Summary (1-3 sentences)

Provide a brief overall assessment of the component's health regarding:

- Security
- Performance
- Accessibility
- Code clarity and quality

### 2. Issues Table

Produce a Markdown table with exactly these columns:

| Priority | Category | Code Location | Explanation & Suggestion |
|----------|----------|---------------|--------------------------|
| High / Medium / Low | Security / Performance / Lifecycle / Accessibility / General JS / Clarity / Dead Code / Comments / Other | Method name, selector, snippet, or line number | Detailed explanation with actionable Vanilla JS recommendation |

#### Table Requirements

- List all issues regardless of severity
- Sort by Priority: High -> Medium -> Low
- Use method names, CSS selectors, or code snippets for Code Location (line numbers if available)
- All recommendations must be 100% Vanilla JS + Web Components compliant
- Do not suggest frameworks, libraries, or build tools

#### Priority Definitions

- **High:** Security vulnerabilities, accessibility blockers, memory leaks, or patterns that will cause failures in production
- **Medium:** Performance inefficiencies, lifecycle issues, maintainability concerns, or accessibility issues that degrade (but do not block) user experience
- **Low:** Code style improvements, minor optimizations, documentation gaps, or enhancements that improve but are not essential for production readiness

### 3. Positive Notes (Optional)

Include a few Low-priority "Positive Note" rows for strong patterns or well-implemented sections worth highlighting.

### 4. Clean Code Response

If the component has no issues in a given category, explicitly state this rather than omitting the category or fabricating concerns.

If the component has no issues across all categories, respond with:

#### Summary

The component demonstrates production-ready quality with no significant issues identified across security, performance, accessibility, or code quality dimensions.

#### Issues

| Priority | Category | Code Location | Explanation & Suggestion |
|----------|----------|---------------|--------------------------|
| - | - | - | No issues identified |

Then proceed to the Positive Notes section highlighting the component's strengths.

---

## Optional Extensions

The following extensions are available when explicitly requested. To invoke, the user will include one or more of these directives:

| Directive | Extension | Description |
|-----------|-----------|-------------|
| `[WCAG]` | WCAG Level Tagging | Tag accessibility issues with WCAG 2.1 level (A / AA / AAA) |
| `[A11Y-TEST]` | Accessibility Testing Guidance | Manual and automated testing recommendations (axe, Lighthouse, screen reader testing) |
| `[PERF]` | Performance Benchmarks | RUM metrics, render timing analysis, Chrome DevTools profiling guidance |
| `[SECURITY]` | Security Hardening | Trusted Types implementation, CSP configuration recommendations |

When an extension directive is present, include the corresponding additional analysis after the main Issues table.

---

## Example Output

### Summary

The component has moderate security concerns due to unescaped `innerHTML` usage, good performance patterns overall, several accessibility gaps in keyboard navigation and labeling, and generally clean JavaScript with minor improvements needed for error handling and code clarity.

### Issues

| Priority | Category | Code Location | Explanation & Suggestion |
|----------|----------|---------------|--------------------------|
| High | Security | `render()` method | Uses `innerHTML` with user-provided data without sanitization. Suggestion: Use `textContent` for text, or create elements programmatically with `document.createElement()`. |
| High | Accessibility | `<div class="button">` | Interactive element uses non-semantic markup. Suggestion: Replace with `<button>` or add `role="button"`, `tabindex="0"`, and keyboard event handlers. |
| Medium | Performance | `connectedCallback` | Queries `this.shadowRoot.querySelector('.item')` on every call. Suggestion: Cache the reference in a class property after first query. |
| Medium | Lifecycle | `disconnectedCallback` | Missing cleanup for `ResizeObserver`. Suggestion: Store observer reference and call `observer.disconnect()` in `disconnectedCallback`. |
| Medium | Clarity | `_processData()` method | 47-line method with 5 levels of nesting obscures control flow. Suggestion: Extract validation, transformation, and persistence into separate private methods. |
| Low | General JS | `handleClick()` | Uses `var` instead of `const`. Suggestion: Replace with `const` for immutable references. |
| Low | Dead Code | `_legacyHandler()` | Method is defined but never called anywhere in the component. Suggestion: Remove if no longer needed, or document its intended usage. |
| Low | Comments | Line 42 | Comment `// loop through items` above `items.forEach()` restates the obvious. Suggestion: Remove redundant comment or replace with explanation of why the iteration is needed. |
| Low | Positive Note | `attributeChangedCallback` | Correctly guards against no-op changes before updating DOM. Well implemented. |
| Low | Positive Note | Style encapsulation | Proper use of `adoptedStyleSheets` for performant, reusable styling. |

---

# Phase 3: Remediation Plan

Based on the review, generate a prioritized remediation plan with phases that are independently testable and deployable.

## Executive Summary

Provide a brief (3-5 sentence) overview:
- Total number of issues by priority (High / Medium / Low)
- Key risk areas requiring immediate attention
- Estimated complexity for full remediation
- Recommended approach (single PR, phased rollout, etc.)

## Phase Structure Template

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

### Phase Guidelines

- Phase 1: High-priority security issues
- Phase 2: Remaining high-priority issues (accessibility, performance, lifecycle)
- Phase 3+: Medium-priority issues grouped by related code or patterns
- Final Phase: Low-priority issues and enhancements
- Each phase should have 3-7 tasks

## Test Plan

For each issue, specify:

| Issue Reference | Test Type | Test Description | Test File Location |
|----------------|-----------|------------------|-------------------|
| [From review] | Regression | [Test that would have caught the bug] | [path] |
| [From review] | Validation | [Test that confirms the fix works] | [path] |
| [From review] | Edge Case | [Test for boundary conditions] | [path] |

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation Strategy |
|------|------------|--------|---------------------|
| [Description] | Low/Medium/High | Low/Medium/High | [How to reduce or avoid] |

## Code Change Previews

For high-priority issues, provide before and after code snippets:

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

# Deliverables Checklist

- Code review document created using the Output Format above
- Remediation plan created
- Findings ordered by severity with clear code locations
- Recommendations avoid frameworks and build tools
- Test plan covers regression, validation, and edge cases
- Residual risks or test gaps noted if no issues found

---

# Output File Templates

## docs/code-review-YYYY-MM-DD.md

```markdown
# Code Review: [Component Name]

**Date:** [YYYY-MM-DD]
**Reviewer:** Codex (Automated)
**Scope:** Web component review

## Summary

[1-3 sentence summary]

## Issues

| Priority | Category | Code Location | Explanation & Suggestion |
|----------|----------|---------------|--------------------------|
| High/Medium/Low | Security/Performance/Lifecycle/Accessibility/General JS/Clarity/Dead Code/Comments/Other | [Method or line] | [Actionable suggestion] |

## Positive Notes

[Optional]

## Extensions

[Optional; only when directives requested]
```

## docs/remediation-plan-YYYY-MM-DD.md

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

[Before and after for high-priority items]
```

---

# Success Criteria

1. Comprehensive review of the component with prioritized findings across all categories
2. Actionable remediation plan with clear tasks and acceptance criteria
3. Test plan covers regressions, validations, and edge cases
4. Recommendations preserve the vanilla web component architecture
