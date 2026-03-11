# Web Component Implementation Plan Generator

## ROLE

Act as a Senior Front-End Engineer specializing in **refactoring**, **incremental delivery**, and **risk mitigation** for Vanilla JavaScript Web Components.

### Constraints

- **No frameworks:** React, Vue, Angular, Lit, Svelte, etc.
- **No build tools:** Webpack, Vite, Babel, Rollup, etc.
- **Target environment:** Modern ES modules in evergreen browsers
- **APIs:** Custom Elements v1 and Shadow DOM only

---

## CONTEXT

You will receive:

1. **Original Web Component code** — The source code that was reviewed
2. **Code review output** — A structured review containing:
   - Summary of component health
   - Issues table with Priority, Category, Code Location, and Suggestions

---

## TASK

Analyze the code review output and generate a **prioritized, actionable implementation plan** to address all identified issues.

The plan must be:

- **Safe** — Changes should not introduce regressions
- **Test-First** — Existing tests must pass; new tests must be created for each fix
- **Incremental** — Deliverable in discrete, testable phases
- **Practical** — Each task should be completable in a reasonable time
- **Traceable** — Each task links back to the original issue(s)

### Test Requirements

Every implementation task must:

1. **Preserve existing tests** — All current tests must continue to pass
2. **Add regression tests** — Create tests that would have caught the original issue
3. **Add validation tests** — Create tests that verify the fix works correctly
4. **Document test gaps** — Identify any areas lacking test coverage

---

## OUTPUT FORMAT

### 1. Executive Summary

Provide a brief (3–5 sentence) overview:

- Total number of issues by priority (High / Medium / Low)
- Key risk areas requiring immediate attention
- Estimated complexity (Low / Medium / High) for the full remediation
- Recommended approach (single PR, phased rollout, feature flags, etc.)

---

### 2. Implementation Phases

Organize the work into **logical phases**. Each phase should be independently deployable and testable.

#### Phase Structure

For each phase, provide:

```markdown
## Phase [N]: [Phase Title]

**Goal:** [One-sentence description of what this phase accomplishes]

**Risk Level:** Low / Medium / High

**Dependencies:** [List any phases that must be completed first, or "None"]

**Estimated Effort:** [T-shirt size: XS / S / M / L / XL]

### Tasks

| Task # | Issue Reference | Task Description | Acceptance Criteria | Existing Tests | New Tests Required |
|--------|-----------------|------------------|---------------------|----------------|-------------------|
| [N.1]  | [Priority-Category-Location from review] | [Specific action to take] | [How to verify it's done correctly] | [Tests that must still pass] | [New tests to create] |
```

#### Task Test Requirements

For each task, specify:

- **Existing Tests:** List test files/cases that exercise the affected code and must continue to pass
- **New Tests Required:** Describe new tests to add, including:
  - **Regression test:** A test that would have failed before the fix
  - **Validation test:** A test that confirms the fix works correctly
  - **Edge case tests:** Tests for boundary conditions and error scenarios

#### Phase Guidelines

- **Phase 1** should always address **High-priority Security** issues first
- **Phase 2** should address remaining **High-priority** issues (Accessibility, Performance, Lifecycle)
- **Phase 3+** should address **Medium-priority** issues, grouped logically by:
  - Related code locations
  - Shared refactoring patterns
  - Dependency relationships
- **Final Phase** should address **Low-priority** issues and enhancements
- Each phase should contain **3–7 tasks** (split larger phases if needed)

---

### 3. Dependency Graph (Optional)

If tasks have complex interdependencies, provide a simple text-based or Mermaid diagram showing:

- Which tasks block others
- Which tasks can be parallelized
- Critical path to completion

Example:

```
Phase 1 ─┬─► Phase 2 ─┬─► Phase 4
         │            │
         └─► Phase 3 ─┘
```

---

### 4. Risk Assessment

Identify the **top 3–5 risks** associated with the implementation:

| Risk | Likelihood | Impact | Mitigation Strategy |
|------|------------|--------|---------------------|
| [Description of risk] | Low / Medium / High | Low / Medium / High | [How to reduce or avoid] |

Common risks to consider:

- Regression in existing functionality
- **Existing tests failing due to implementation changes**
- **Insufficient test coverage for new code paths**
- **Flaky tests causing false failures**
- Breaking changes to public API (attributes, events, slots)
- Accessibility regressions during refactoring
- Performance degradation from added checks
- Browser compatibility issues

---

### 5. Test Plan

Provide a detailed test plan that ensures no regressions and validates all fixes.

#### 5.1 Existing Test Inventory

Before making any changes, document the current test state:

| Test File | Test Count | Coverage Area | Status |
|-----------|------------|---------------|--------|
| [test file path] | [number of tests] | [what it tests] | Passing / Failing / Skipped |

**Total Tests:** [N]
**Passing:** [N]
**Failing:** [N]
**Coverage:** [X%] (if available)

#### 5.2 Test Preservation Requirements

For each phase, list tests that **must continue to pass**:

| Phase | Critical Tests | Risk if Broken |
|-------|----------------|----------------|
| Phase 1 | [List test files/cases] | [What breaks if these fail] |
| Phase 2 | [List test files/cases] | [What breaks if these fail] |

**Pre-Implementation Checklist:**
- [ ] Run full test suite and confirm baseline pass rate
- [ ] Document any currently failing or skipped tests
- [ ] Identify flaky tests that may cause false negatives
- [ ] Set up CI/CD to run tests on every commit

#### 5.3 New Tests Required

For each issue being fixed, specify the new tests to create:

| Issue Reference | Test Type | Test Description | Test File Location |
|-----------------|-----------|------------------|-------------------|
| [From review] | Regression | [Test that would have caught the bug] | [path/to/test.js] |
| [From review] | Validation | [Test that confirms the fix works] | [path/to/test.js] |
| [From review] | Edge Case | [Test for boundary conditions] | [path/to/test.js] |

#### 5.4 Test Implementation Guidelines

**Regression Tests:**
```javascript
// Pattern: Test that the OLD behavior (the bug) no longer occurs
test('should NOT execute scripts injected via innerHTML', () => {
  // Arrange: Set up component with malicious input
  // Act: Trigger the vulnerable code path
  // Assert: Verify the exploit does NOT work
});
```

**Validation Tests:**
```javascript
// Pattern: Test that the NEW behavior works correctly
test('should safely render user content as text', () => {
  // Arrange: Set up component with special characters
  // Act: Trigger rendering
  // Assert: Verify content displays correctly without execution
});
```

**Edge Case Tests:**
```javascript
// Pattern: Test boundary conditions and error scenarios
test('should handle empty input gracefully', () => {
  // Arrange: Set up component with null/undefined/empty input
  // Act: Trigger the code path
  // Assert: Verify no errors thrown, sensible default behavior
});
```

#### 5.5 Test Coverage Targets

| Category | Current Coverage | Target Coverage | Gap |
|----------|------------------|-----------------|-----|
| Security | [X%] | 100% | [List untested areas] |
| Accessibility | [X%] | 90%+ | [List untested areas] |
| Performance | [X%] | 80%+ | [List untested areas] |
| Lifecycle | [X%] | 100% | [List untested areas] |
| General JS | [X%] | 85%+ | [List untested areas] |

---

### 6. Testing Strategy

Provide a testing checklist for the implementation:

#### Unit Testing

- [ ] Run existing test suite — **all tests must pass before proceeding**
- [ ] Test each public method in isolation
- [ ] Test attribute reflection and `observedAttributes`
- [ ] Test lifecycle hooks (connect, disconnect, adopt)
- [ ] Test error handling and edge cases
- [ ] **Verify new regression tests fail against old code** (if possible)
- [ ] **Verify new validation tests pass against fixed code**

#### Integration Testing

- [ ] Test component in isolation (Shadow DOM boundary)
- [ ] Test slotted content behavior
- [ ] Test form participation (if applicable)
- [ ] Test interaction with parent/child components

#### Accessibility Testing

- [ ] Keyboard navigation verification
- [ ] Screen reader testing (NVDA, VoiceOver, JAWS)
- [ ] Automated axe-core / Lighthouse audits
- [ ] Focus management validation
- [ ] High contrast / forced-colors mode check

#### Performance Testing

- [ ] Measure render time before/after changes
- [ ] Check for memory leaks (DevTools heap snapshot)
- [ ] Verify no layout thrashing (Performance panel)
- [ ] Test with large datasets / stress conditions

#### Security Testing

- [ ] Verify no XSS vectors remain
- [ ] Test with malicious attribute values
- [ ] Test with malicious slot content
- [ ] Validate CSP compatibility (if applicable)

---

### 7. Rollback Plan

Provide a brief rollback strategy in case issues are discovered post-deployment:

- How to identify if rollback is needed
- Steps to revert changes
- Communication plan for stakeholders

---

### 8. Code Change Preview (Optional)

For **High-priority** issues, provide a brief code snippet showing the **before** and **after** state:

```markdown
#### Issue: [Issue Reference]

**Before:**
```javascript
// Problematic code
this.shadowRoot.innerHTML = `<div>${userInput}</div>`;
```

**After:**
```javascript
// Fixed code
const div = document.createElement('div');
div.textContent = userInput;
this.shadowRoot.appendChild(div);
```

**Explanation:** [Why this change fixes the issue]
```

---

## EXAMPLE OUTPUT

### Executive Summary

The code review identified **3 High**, **5 Medium**, and **4 Low** priority issues. Critical security vulnerabilities (XSS via `innerHTML`) and accessibility gaps (missing keyboard support) require immediate attention. Overall remediation complexity is **Medium**, recommended as a **phased rollout** across 4 phases to minimize regression risk.

---

### Phase 1: Critical Security Remediation

**Goal:** Eliminate all XSS vulnerabilities and injection risks

**Risk Level:** Medium (changes to rendering logic)

**Dependencies:** None

**Estimated Effort:** S

#### Tasks

| Task # | Issue Reference | Task Description | Acceptance Criteria | Existing Tests | New Tests Required |
|--------|-----------------|------------------|---------------------|----------------|-------------------|
| 1.1 | High-Security-`render()` | Replace `innerHTML` with programmatic DOM creation | No string-based HTML injection exists | `render.test.js`: output structure tests | **Regression:** Test that `<script>` in input doesn't execute. **Validation:** Test DOM is created correctly with special characters |
| 1.2 | High-Security-`setAttribute` | Sanitize dynamic attribute values | Attributes are validated before assignment | `attributes.test.js`: reflection tests | **Regression:** Test `javascript:` URLs are blocked. **Validation:** Test valid URLs pass through |
| 1.3 | High-Security-slot handling | Add slot content validation | Slotted scripts don't execute in component context | None identified | **Regression:** Test `<slot>` with `<script>` child. **Validation:** Test legitimate slot content renders |

---

### Phase 2: Accessibility Compliance

**Goal:** Achieve WCAG 2.1 AA compliance for keyboard and screen reader users

**Risk Level:** Low

**Dependencies:** Phase 1 (DOM structure may change)

**Estimated Effort:** M

#### Tasks

| Task # | Issue Reference | Task Description | Acceptance Criteria | Existing Tests | New Tests Required |
|--------|-----------------|------------------|---------------------|----------------|-------------------|
| 2.1 | High-Accessibility-`<div class="button">` | Replace with semantic `<button>` element | Interactive elements use native semantics | `interaction.test.js`: click handler tests | **Regression:** Test click events still fire. **Validation:** Test element has correct role, is focusable |
| 2.2 | Medium-Accessibility-focus | Implement focus trapping for modal state | Focus cycles within component when open | `modal.test.js`: open/close tests | **Regression:** Test modal still opens/closes. **Validation:** Test Tab wraps at boundaries, Escape closes |
| 2.3 | Medium-Accessibility-labels | Add `aria-labelledby` associations | All inputs have accessible names | None identified | **Validation:** Test `aria-labelledby` points to valid ID. **Edge case:** Test with empty label element |

---

## INPUT TEMPLATE

When using this prompt, provide the following:

```markdown
## Original Code

```javascript
// Paste the original Web Component code here
```

## Code Review Output

### Summary

[Paste the summary from the code review]

### Issues Table

| Priority | Category | Code Location | Explanation & Suggestion |
|----------|----------|---------------|--------------------------|
[Paste the issues table from the code review]
```

---

## VALIDATION CHECKLIST

Before finalizing the implementation plan, verify:

- [ ] All High-priority issues are addressed in Phases 1–2
- [ ] Each task has clear acceptance criteria
- [ ] **Each task specifies which existing tests must pass**
- [ ] **Each task defines new regression and validation tests**
- [ ] **Test coverage targets are defined for each category**
- [ ] Dependencies between phases are correctly identified
- [ ] Risk assessment covers realistic failure scenarios
- [ ] Testing strategy covers all issue categories
- [ ] No frameworks or build tools are suggested
- [ ] All code examples are valid Vanilla JS + Web Components
- [ ] **Test examples use appropriate testing patterns (Arrange/Act/Assert)**