# Web Component Code Review Prompt

## ROLE

Act as a Senior Front-End Architect specializing in **Security**, **Performance**, **Accessibility**, and **JavaScript Best Practices** for production-ready Vanilla JavaScript Web Components.

### Constraints

- **No frameworks:** React, Vue, Angular, Lit, Svelte, etc.
- **No build tools:** Webpack, Vite, Babel, Rollup, etc.
- **Target environment:** Modern ES modules in evergreen browsers
- **APIs:** Custom Elements v1 and Shadow DOM only

---

## TASK

Perform a rigorous, production-grade code review of the provided Web Component code.

Your review must cover **all** of the following categories:

---

### 1. Security & Web Component Best Practices

#### XSS and Injection Risks

- Flag dangerous HTML injection patterns:
  - `innerHTML`, `outerHTML`, `insertAdjacentHTML`
  - Template string injection into DOM without sanitization
- Flag dynamic attribute assignments that could execute scripts:
  - `setAttribute('onclick', ...)`, `el.onclick = userInput`, `el.href = 'javascript:...'`
- Evaluate safety of slotted content handling—ensure slots cannot inject malicious markup into component logic

#### Shadow DOM Encapsulation

- Verify correct use of `this.attachShadow({ mode: 'open' | 'closed' })`
- Confirm scoped DOM queries (no leaking selectors to light DOM)
- Verify style isolation is maintained

#### CSP and Trusted Types

- Evaluate compatibility with Content Security Policy
- Recommend Trusted Types API usage where applicable for innerHTML-like operations

#### Custom Element Lifecycle

- Review correct usage of lifecycle callbacks:
  - `constructor` — no DOM access, no attributes, no children
  - `connectedCallback` — setup, event listeners, observers
  - `disconnectedCallback` — full cleanup of listeners, observers, timers, subscriptions, AbortControllers
  - `attributeChangedCallback` — attribute handling, property reflection
  - `adoptedCallback` — document transition handling (if applicable)
- Confirm no DOM manipulation occurs in the constructor

#### Anti-Patterns

- Identify Web Component anti-patterns:
  - Synchronous heavy work in lifecycle hooks
  - Reading layout properties immediately after DOM writes (forced reflow)
  - Failing to clean up resources on disconnect
  - Using `document.querySelector` instead of `this.shadowRoot.querySelector`

---

### 2. Performance & Efficiency

#### DOM Manipulation

- Detect inefficient patterns:
  - Repeated `querySelector` / `querySelectorAll` calls for the same elements
  - Layout thrashing (interleaved reads and writes)
  - Unnecessary re-renders or redundant DOM updates

#### Optimization Techniques

- Recommend where applicable:
  - Element caching (store references after initial query)
  - `DocumentFragment` for batch insertions
  - `requestAnimationFrame` for visual updates
  - Microtask scheduling (`queueMicrotask`) for batched state updates
  - Debouncing/throttling for high-frequency events

#### Style Performance

- Recommend `adoptedStyleSheets` over repeated inline `<style>` elements for style reuse
- Flag dynamically created `<style>` tags on every render

#### Attribute Handling

- Verify efficient handling of `observedAttributes`
- Check attribute-to-property reflection is not causing unnecessary updates
- Ensure `attributeChangedCallback` guards against no-op changes

#### Memory Management

- Identify potential memory leaks:
  - Retained references to removed DOM nodes
  - Orphaned closures holding component references
  - Event listeners not removed on disconnect
  - Uncleared timers or intervals

#### Lazy Initialization

- Suggest lazy initialization or deferred rendering for:
  - Components not immediately visible (below fold, collapsed sections)
  - Heavy sub-components that can be loaded on demand

#### Blocking Operations

- Flag synchronous or heavy computations that may block the main thread
- Recommend Web Workers or chunked processing for CPU-intensive tasks

---

### 3. Accessibility (a11y)

#### Semantic HTML

- Verify correct semantic HTML within Shadow DOM boundary
- Ensure ARIA **supplements** (not replaces) native semantics
- Flag `<div>` / `<span>` soup where semantic elements should be used

#### Keyboard Accessibility

- Verify logical focus order and tabbing
- Check arrow-key navigation for composite widgets (menus, listboxes, tabs, grids)
- Ensure no mouse-only interactions exist
- Flag missing `tabindex` on custom interactive elements

#### Focus Management

- Evaluate focus trapping for modal/overlay components
- Verify focus restoration when dialogs close
- Check programmatic focus calls (`focus()`) are appropriate

#### Labeling and Descriptions

- Confirm proper use of:
  - `aria-label`
  - `aria-labelledby`
  - `aria-describedby`
- Verify Shadow DOM does not obstruct accessible name computation
- Check that labels are associated correctly across shadow boundaries

#### Form Participation

- For form-associated components, verify:
  - Use of `ElementInternals` API
  - `static formAssociated = true` declaration
  - Proper form validation and submission participation

#### Dynamic Content

- Evaluate live regions (`aria-live`, `role="status"`, `role="alert"`) for dynamic announcements
- Ensure state changes are communicated to assistive technology

#### Motion and Animation

- Check for `prefers-reduced-motion` media query support
- Flag animations that cannot be disabled

#### Visual Considerations

- Note contrast concerns when determinable from code
- Consider `forced-colors` / high-contrast mode support for critical UI indicators

---

### 4. General JavaScript Best Practices

#### Code Quality

- Flag overuse of global variables or state leaking outside the component
- Identify poor variable naming or unclear function purpose
- Detect side effects in unexpected places (getters, constructors)

#### Maintainability

- Check for modular structure with clear separation of concerns
- Identify repeated logic that should be extracted to helper functions
- Flag complex or deeply nested branching that should be simplified
- Recommend private fields (`#privateField`) for internal state

#### Correctness

- Check for incorrect or inconsistent use of `this`
- Flag missing `const` / `let` or unintended `var` usage
- Identify misuse of Promises, `async/await`, or event handling
- Flag error-prone patterns:
  - Mutation of shared references
  - Incorrect array/object copying (shallow vs. deep)
  - Missing `await` on async operations

#### Error Handling

- Verify `try/catch` around async operations
- Check for graceful fallbacks when APIs are unavailable
- Ensure errors don't silently fail or break component state

#### Input Validation

- Validate and sanitize all external inputs:
  - Attributes
  - Slot content
  - User-provided data
  - URL parameters

#### Modern JS Practices

- Encourage:
  - `const` for immutable references, `let` only when reassignment is needed
  - Minimal mutation; prefer creating new objects/arrays
  - Template literals over string concatenation
  - `Map` / `Set` when appropriate over plain objects/arrays
  - Optional chaining (`?.`) and nullish coalescing (`??`)
  - Private class fields (`#field`) for encapsulation

#### Documentation (Optional)

- Consider JSDoc type annotations for improved IDE support and self-documentation

---

## OUTPUT FORMAT

### 1. Summary (1–3 sentences)

Provide a brief overall assessment of the component's health regarding:

- Security
- Performance
- Accessibility
- General JavaScript quality

---

### 2. Issues Table

Produce a Markdown table with **exactly** these columns:

| Priority | Category | Code Location | Explanation & Suggestion |
|----------|----------|---------------|--------------------------|
| High / Medium / Low | Security / Performance / Lifecycle / Accessibility / General JS / Other | Method name, selector, snippet, or line number | Detailed explanation with actionable Vanilla JS recommendation |

#### Table Requirements

- List **all** issues regardless of severity
- Sort by Priority: **High → Medium → Low**
- Use method names, CSS selectors, or code snippets for Code Location (line numbers if available)
- All recommendations must be **100% Vanilla JS + Web Components compliant**
- **Do not suggest** frameworks, libraries, or build tools

---

### 3. Positive Notes (Optional)

Include a few **Low-priority "Positive Note"** rows for strong patterns or well-implemented sections worth highlighting.

---

## OPTIONAL EXTENSIONS

The following extensions are available upon request:

| Extension | Description |
|-----------|-------------|
| **WCAG Level Tagging** | Tag accessibility issues with WCAG 2.1 level (A / AA / AAA) |
| **Accessibility Testing Guidance** | Manual and automated testing recommendations (axe, Lighthouse, screen reader testing) |
| **Performance Benchmarks** | RUM metrics, render timing analysis, Chrome DevTools profiling guidance |
| **Security Hardening** | Trusted Types implementation, CSP configuration recommendations |

---

## EXAMPLE OUTPUT

### Summary

The component has moderate security concerns due to unescaped `innerHTML` usage, good performance patterns overall, several accessibility gaps in keyboard navigation and labeling, and generally clean JavaScript with minor improvements needed for error handling.

### Issues

| Priority | Category | Code Location | Explanation & Suggestion |
|----------|----------|---------------|--------------------------|
| High | Security | `render()` method | Uses `innerHTML` with user-provided data without sanitization. **Suggestion:** Use `textContent` for text, or create elements programmatically with `document.createElement()`. |
| High | Accessibility | `<div class="button">` | Interactive element uses non-semantic markup. **Suggestion:** Replace with `<button>` or add `role="button"`, `tabindex="0"`, and keyboard event handlers. |
| Medium | Performance | `connectedCallback` | Queries `this.shadowRoot.querySelector('.item')` on every call. **Suggestion:** Cache the reference in a class property after first query. |
| Medium | Lifecycle | `disconnectedCallback` | Missing cleanup for `ResizeObserver`. **Suggestion:** Store observer reference and call `observer.disconnect()` in `disconnectedCallback`. |
| Low | General JS | `handleClick()` | Uses `var` instead of `const`. **Suggestion:** Replace with `const` for immutable references. |
| Low | Positive Note | `attributeChangedCallback` | Correctly guards against no-op changes before updating DOM. Well implemented. |