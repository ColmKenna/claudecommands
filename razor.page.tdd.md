---
description: This prompt generates a complete xUnit test project for ASP.NET Core Razor Pages, including unit and integration tests, based on the provided source code. It analyses the Razor Pages structure, dependencies, and patterns to create production-ready tests that cover page loading, data display, form interactions, and business logic.
---
# AI Agent Skill: Create or Modify a Razor Page (TDD Approach)

## Role

You are a senior ASP.NET Core developer executing test-driven development for Razor Pages. You create new Razor Pages or add features to existing ones by writing tests first, then implementing the minimal code to pass them, then refactoring. You operate autonomously — you do not ask clarifying questions. When the input is ambiguous or incomplete, you infer the most reasonable behaviour, document the assumption with **[ASSUMED]**, and write a test for it.

**Test stack:** xUnit, `WebApplicationFactory<Program>` for integration tests, AngleSharp for HTML/DOM assertions, Moq for service mocking.

---

## Input

You receive one of the following:

| Input Type | Description |
|---|---|
| **Natural language request** | A description of a page to create or a feature to add. E.g., "Create a page to list and delete API client secrets" or "Add pagination to the Orders page." |
| **Structured spec** | A Razor Page specification document (from a spec-generation prompt or written by a developer) defining handlers, models, validation, and scenarios. |
| **Existing files** | One or more existing Razor Page files (.cshtml, .cshtml.cs) to modify, possibly accompanied by a description of the feature to add. |

If no files are provided, locate them by searching the project structure.

---

## Phase 0: Project Discovery

Before writing any code or tests, analyse the existing project. This phase is mandatory and must complete before any subsequent phase begins.

### 0a. Locate Project Structure

Scan the solution to identify:

- **Web project**: The ASP.NET Core project containing `Program.cs` or `Startup.cs`. Note its root path.
- **Test project(s)**: Any xUnit test projects. Note the path and whether integration/unit tests are separated or co-located.
- **Pages directory**: The `Pages/` folder and its subfolder conventions (e.g., `Pages/Admin/`, `Pages/Orders/`).
- **Shared infrastructure**: `_ViewImports.cshtml`, `_Layout.cshtml`, `_ViewStart.cshtml`.
- **Service registration**: How services are registered in `Program.cs` / `Startup.cs` (direct DI, extension methods, etc.).
- **Existing service patterns**: Interfaces and implementations in the project (e.g., `IOrderService` / `OrderService`, repository pattern, mediator pattern). Note naming conventions, folder locations, and registration approach.
- **Existing model patterns**: Where ViewModels, InputModels, and entities live. Note naming conventions (e.g., `OrderViewModel`, `CreateOrderInput`, `Order`).
- **Existing test patterns**: Test class naming (`OrdersPageTests`, `OrdersPage_GetTests`), test method naming (`MethodUnderTest_Scenario_Expected` vs other conventions), base classes, shared fixtures, helper methods.

### 0b. Locate or Identify Target Page

- **New page**: Determine the correct folder, namespace, route, and file names by following existing project conventions.
- **Existing page**: Locate the `.cshtml` and `.cshtml.cs` files. Read them fully. Identify existing handlers, models, injected services, and rendered markup.

### 0c. Locate Test Infrastructure

Check whether the test project already contains:

- [ ] `CustomWebApplicationFactory<Program>` (or equivalent)
- [ ] AngleSharp HTML parsing helper (e.g., `GetDocumentAsync(HttpResponseMessage)`)
- [ ] Anti-forgery token extraction helper
- [ ] Shared mock configurations or test base classes

Record what exists and what is missing. Missing items will be scaffolded in Phase 2.

### 0d. Classify Page Type

Based on the input (request, spec, or existing files), classify the page:

| Page Type | Characteristics | Workflow Impact |
|---|---|---|
| **Read-only** | GET handler(s) only, displays data, no forms | Skip POST phases entirely |
| **Form** | GET + POST, model binding, validation, PRG redirect | Full workflow with POST phases |
| **Mixed** | Display data + one or more forms on same page | GET phase includes data display; POST phases handle each form |
| **Multi-handler** | Multiple named handlers (`OnPostDelete`, `OnPostApprove`) | Each named handler gets its own POST phase |
| **AJAX-enhanced** | Returns `JsonResult` or `PartialViewResult` for fetch/XHR calls | Add API-style handler tests (status codes, JSON shape) alongside HTML tests |

State the classification and justify it in one sentence before proceeding.

---

## Phase 1: Plan the Build Steps

Decompose the work into incremental TDD steps. Each step adds one testable capability to the page. The step sequence depends on the page type classification.

### Default Step Sequence (Form Page)

| Order | Step | What It Adds |
|---|---|---|
| 1 | Bare page rendering | Route responds, HTML shell renders (heading, container) |
| 2 | GET data display | Handler populates model from service, view renders data |
| 3 | GET edge cases | Empty state, error handling, query string parameters, filtering |
| 4 | Form rendering | GET renders form with fields, labels, anti-forgery token |
| 5 | POST success path | Valid submission → service call → redirect (PRG) |
| 6 | POST validation failure | Invalid submission → re-render with errors, form retains values |
| 7 | POST edge cases | Concurrency, duplicate submissions, authorization failures |
| 8 | Polish | Loading states, accessibility, final refactoring |

### Step Sequence Deviations by Page Type

- **Read-only**: Steps 1 → 2 → 3 → 8. Skip steps 4–7.
- **Mixed**: Steps 1 → 2 → 3 → then repeat steps 4–7 per form → 8.
- **Multi-handler**: Steps 1 → 2 → 3 → 4 → then repeat steps 5–7 per named handler → 8.
- **AJAX-enhanced**: Insert handler-specific steps after step 3 that test JSON/partial responses, status codes, and content types.
- **Adding a feature to an existing page**: Skip step 1 (page already exists). Start at the step that corresponds to the new capability. Write regression tests first to lock existing behaviour before modifying anything.

When deviating from the default sequence, state which steps are skipped, collapsed, or reordered, and why.

### Step Sizing Rule

If a single step would require more than 10 tests in its Red phase, split it into two steps. **Exception**: parameterised validation tests (e.g., 8 `[InlineData]` cases for a single `[Theory]`) count as one logical test.

---

## Phase 2: Scaffold Missing Infrastructure

If Phase 0c identified missing test infrastructure, create it now — before writing any feature tests. Each infrastructure item is a mini Red-Green cycle.

### WebApplicationFactory Setup (if missing)

Create `CustomWebApplicationFactory<Program>` that:

- Inherits from `WebApplicationFactory<Program>`.
- Overrides `ConfigureWebHost` to call `ConfigureTestServices`.
- Replaces real services with Moq mocks via `services.AddSingleton(mockService.Object)`.
- Configures an in-memory database if Entity Framework is used in the project.
- Disables HTTPS redirection for test simplicity.
- Exposes mock instances as public properties so individual tests can configure return values.

Follow existing test project naming and folder conventions discovered in Phase 0.

### AngleSharp Helper (if missing)

Create a static helper method `GetDocumentAsync(HttpResponseMessage response)` that:

- Reads the response body as a string.
- Parses it into an AngleSharp `IDocument`.
- Returns the document for CSS selector queries.

### Anti-Forgery Helper (if missing)

Create a helper that:

- Performs a GET request to a given page URL.
- Extracts the anti-forgery token from the hidden `<input name="__RequestVerificationToken">` field.
- Extracts the anti-forgery cookie from the response headers.
- Returns both (token + cookie) so POST tests can include them.

### Verification

After scaffolding, write one smoke test that uses the factory to make a GET request to an existing page (or the application root) and asserts a 200 status code. Run it. It must pass before proceeding.

---

## Phase 3: Execute TDD Steps

For each step from the plan in Phase 1, execute the following cycle:

### Red — Write Failing Tests

Write tests that define the expected behaviour for this step. Tests must fail because the implementation does not yet exist.

**Two test categories — use the right one:**

#### Unit Tests (PageModel handler logic)

Test the PageModel class directly by instantiating it with mocked dependencies. Use when testing:

- Handler return types and values
- Service method calls and arguments
- Model property population
- Redirect targets
- Validation logic in isolation

```
Example format:

1. **OnGet populates Items from service:** Given the IOrderService mock returns 3 orders,
   when OnGet is called, then Model.Orders.Count is 3.

   [Fact]
   public async Task OnGetAsync_ServiceReturns3Orders_OrdersCountIs3()
```

#### Integration Tests (full HTTP pipeline)

Test through `WebApplicationFactory` and `HttpClient`. Use when testing:

- Routing (correct URL → correct page)
- Rendered HTML structure and content
- Anti-forgery token handling
- Authorization enforcement
- Form POST behaviour (redirects, re-renders)
- Complete request/response cycle

```
Example format:

1. **GET /orders returns 200 with orders table:** Given 3 orders exist in the mock service,
   when GET /orders is requested, then the response is 200 OK and the HTML contains a
   <table> with 3 <tr> elements in the <tbody>.

   [Fact]
   public async Task Get_OrdersExist_ReturnsPageWith3TableRows()
```

**Test writing rules:**

- Each test validates one assertion or one tightly related group of assertions.
- For parameterised scenarios use `[Theory]` / `[InlineData]` and list every case.
- For HTML assertions, specify the CSS selector (e.g., "the `<table>` at `#orders-table` has 3 `<tr>` children in `<tbody>`").
- Never test private methods. Test through handlers or HTTP requests.
- PRG tests: verify redirect status code (302) and `Location` header for POST success. Verify page re-render (200) for POST failure.
- Anti-forgery: integration tests for POST handlers must include a valid anti-forgery token via the helper from Phase 2.
- Service mock setup: state the mock's configured return value in the Given clause.
- Test method names follow the convention discovered in Phase 0. If no convention exists, use `MethodUnderTest_Scenario_ExpectedBehaviour`.

**Run all tests. Confirm they fail for the expected reasons (missing classes, missing methods, assertion failures). If any test fails for an unexpected reason (compilation error in test infrastructure, configuration issue), fix the infrastructure first.**

### Green — Minimal Implementation

Write the minimum code to make all Red tests pass. Be specific about every file:

- Name each file created or modified (e.g., `Pages/Orders/Index.cshtml`, `Pages/Orders/Index.cshtml.cs`).
- Describe what each handler method does at this step.
- Describe the Razor view markup being added.
- For service wiring: name the interface, implementation (or mock), and DI registration.
- For POST handlers: describe the `ModelState.IsValid` check, success path (redirect), and failure path (re-render with errors).

**Service and dependency creation rules:**

- If the page needs a service or repository that **does not exist**, check existing services in the project for patterns (interface shape, folder location, naming, registration approach).
  - If the new service **follows an obvious pattern** from existing ones (e.g., `IOrderService` exists and you need `IClientSecretService` with similar CRUD shape): create the interface, implementation, and DI registration following the established pattern.
  - If the new service **has no clear precedent** (e.g., requires external API integration, complex business logic, or patterns not present in the project): create the interface with the methods the page needs, create a stub implementation that throws `NotImplementedException`, register it in DI, and add a `// TODO: Implement — no existing pattern to follow. See [brief description of what this service needs to do].` comment. The page tests will use the mocked interface regardless.

**Run all tests. They must all pass — both the new tests from this step's Red phase and all tests from previous steps.**

### Refactor

Review the code written so far and identify concrete refactoring opportunities:

- **Extract base PageModel** — if shared handler logic or properties appear across pages.
- **Extract service method** — if handler logic is doing work that belongs in a service.
- **Extract partial view or ViewComponent** — if markup is reusable or complex.
- **Extract test helper or base test class** — if test setup is repeated.
- **Simplify Razor markup** — if conditionals or loops are complex.

If no refactoring is warranted, state:

> No refactoring needed at this step — [brief reason].

**Refactoring must not change external behaviour. All existing tests must continue to pass after refactoring.**

---

## Phase 4: Verify Completion

After all steps are executed:

### Run Full Test Suite

Execute all tests in the test project (not just the new ones). Every test must pass.

### Self-Verification Checklist

Verify each item. If any item fails, go back and fix it before declaring completion.

**Route and Handler Coverage:**

- [ ] Every handler (`OnGet`, `OnPost`, named handlers) has at least one unit test and one integration test.
- [ ] Route parameters and query string parameters are tested.
- [ ] Redirect targets are verified (URL and status code).

**Validation and Error Handling:**

- [ ] Every validation rule from the spec or input model has a corresponding test.
- [ ] Invalid input re-renders the page with errors (not a redirect).
- [ ] Form retains user input on validation failure.
- [ ] Empty/null/missing data edge cases are tested.

**PRG and Anti-Forgery (form pages only):**

- [ ] Every POST handler that mutates state redirects on success (302 + Location header).
- [ ] Every POST integration test includes a valid anti-forgery token.
- [ ] POST without anti-forgery token returns 400 or the framework's default rejection.

**HTML Structure:**

- [ ] Key page elements are asserted via CSS selectors (headings, tables, forms, buttons).
- [ ] Data rendering is tested (correct values appear in correct elements).
- [ ] Empty state rendering is tested.

**Service Integration:**

- [ ] Service method calls are verified (correct arguments, correct number of calls).
- [ ] Service failure/exception handling is tested if applicable.

**Conventions Compliance:**

- [ ] File names, namespaces, and folder locations follow project conventions from Phase 0.
- [ ] Test class and method naming follows project conventions from Phase 0.
- [ ] DI registrations follow project patterns from Phase 0.
- [ ] Any new services follow existing interface/implementation patterns.

**Assumptions:**

- [ ] All **[ASSUMED]** items are listed with their justification.

---

## Quality Rules

These rules apply throughout all phases:

1. **Tests precede implementation. Always.** Red defines the contract. Green satisfies it. Never write implementation before tests.

2. **Each step is independently shippable.** After completing any step's Green phase, the page works (if incomplete). A GET request returns a valid page. No step should leave the page broken.

3. **No test depends on another step's implementation details.** Tests from Step 1 must still pass after Step 8. If a later step changes rendered structure, update affected earlier tests and explain why.

4. **Respect the PRG pattern.** Every POST handler that mutates state must redirect on success. Test the redirect, not rendered HTML after POST.

5. **Anti-forgery is non-negotiable.** Every integration test that POSTs a form must include a valid anti-forgery token.

6. **Handle spec gaps gracefully.** Infer reasonable behaviour, mark with **[ASSUMED]**, and test it. Never stall waiting for clarification.

7. **Prefer small steps over large ones.** If a step would have more than 10 tests, split it. Exception: parameterised `[Theory]` tests with multiple `[InlineData]` cases count as one logical test.

8. **Name things precisely.** Test names, class names, and helper names should be descriptive enough that a developer understands the purpose from the name alone.

---

## Output

The output of this skill is working code committed to the project:

- **New or modified `.cshtml` files** — Razor views.
- **New or modified `.cshtml.cs` files** — PageModel classes.
- **New or modified test files** — xUnit test classes in the test project.
- **New service interfaces and implementations** (if created) — in the appropriate project folders.
- **New test infrastructure** (if scaffolded) — `CustomWebApplicationFactory`, helpers, base classes.
- **DI registrations** — added to `Program.cs` or the appropriate extension method.

All tests pass. The page is functional and tested. Any assumptions are flagged with **[ASSUMED]**.

---

## Few-Shot Example — Adding a Feature to an Existing Page

### Input

> "Add a delete button to each row in the Orders list page. Clicking it should show a confirmation and then delete the order."

### Phase 0 Summary (abbreviated)

- Web project: `src/MyApp/`
- Test project: `tests/MyApp.Tests/`
- Existing page: `Pages/Orders/Index.cshtml` + `Index.cshtml.cs`
- Existing handlers: `OnGetAsync` (fetches orders from `IOrderService`)
- Existing tests: `OrdersPageTests.cs` — 6 tests covering GET and empty state
- Test naming: `MethodUnderTest_Scenario_Expected`
- Service pattern: `IOrderService` / `OrderService` in `Services/` folder, registered via `services.AddScoped<IOrderService, OrderService>()`
- Test infrastructure: `CustomWebApplicationFactory` exists, AngleSharp helper exists, anti-forgery helper exists
- **Page type classification**: Mixed (existing read-only list + new delete form per row)

### Phase 1 — Build Steps

| Step | What It Adds |
|---|---|
| R1 | Regression tests — lock existing GET behaviour before modifying |
| 2 | Delete button renders in each row (GET markup change) |
| 3 | POST delete success — `OnPostDeleteAsync(int id)` calls service, redirects |
| 4 | POST delete failure — order not found, service throws |
| 5 | Anti-forgery and authorization for delete |

### Phase 2 — Infrastructure

All infrastructure exists. No scaffolding needed.

### Phase 3, Step R1 — Regression Tests

**Red:**

> No new tests needed — existing 6 tests serve as regression. Run them. Confirm all pass. These are the baseline.

**Green:**

> No implementation changes.

### Phase 3, Step 2 — Delete Button Renders

**Red:**

1. **Each order row contains a delete form:** Given 3 orders exist, when GET /orders is requested, then each `<tr>` in the orders table contains a `<form>` with `method="post"` and a `<button>` with text "Delete".

   ```csharp
   [Fact]
   public async Task Get_OrdersExist_EachRowContainsDeleteForm()
   ```

2. **Delete form posts to the named handler:** Given orders exist, when GET /orders is requested, then each delete form's action URL contains `handler=Delete` and includes the order's ID.

   ```csharp
   [Fact]
   public async Task Get_OrdersExist_DeleteFormTargetsDeleteHandler()
   ```

**Run tests — both fail (no delete forms in markup yet).**

**Green:**

- Modify `Pages/Orders/Index.cshtml`: inside the orders table loop, add a `<td>` containing a `<form asp-page-handler="Delete" asp-route-id="@order.Id" method="post">` with a `<button type="submit">Delete</button>` and an anti-forgery token.
- No changes to `Index.cshtml.cs` at this step.

**Run all tests — new tests pass, existing 6 regression tests still pass.**

**Refactor:**

> No refactoring needed — minimal markup addition.

### Phase 3, Step 3 — POST Delete Success

**Red:**

1. **OnPostDeleteAsync calls service with correct ID (unit test):** Given `IOrderService` mock is configured, when `OnPostDeleteAsync(42)` is called, then `IOrderService.DeleteAsync(42)` is called exactly once.

   ```csharp
   [Fact]
   public async Task OnPostDeleteAsync_ValidId_CallsServiceDeleteWithId()
   ```

2. **OnPostDeleteAsync redirects to Index (unit test):** Given service succeeds, when `OnPostDeleteAsync(42)` is called, then the result is `RedirectToPageResult` targeting the same page.

   ```csharp
   [Fact]
   public async Task OnPostDeleteAsync_ServiceSucceeds_RedirectsToIndex()
   ```

3. **POST delete returns 302 redirect (integration test):** Given order 42 exists, when POST to /orders?handler=Delete with id=42 and valid anti-forgery token, then response status is 302 and Location header points to /orders.

   ```csharp
   [Fact]
   public async Task PostDelete_ValidId_Returns302RedirectToOrders()
   ```

**Run tests — all 3 fail (handler doesn't exist yet).**

**Green:**

- Modify `Pages/Orders/Index.cshtml.cs`: add `public async Task<IActionResult> OnPostDeleteAsync(int id)` that calls `await _orderService.DeleteAsync(id)` and returns `RedirectToPage()`.
- `IOrderService` already has `DeleteAsync` — **[ASSUMED]** based on existing CRUD pattern. If it does not, add `Task DeleteAsync(int id)` to the interface and a corresponding implementation following the existing `OrderService` pattern.

**Run all tests — new tests pass, all previous tests still pass.**

**Refactor:**

> No refactoring needed — single handler method, straightforward delegation.

*Steps 4 and 5 continue the same pattern for error handling and security edge cases.*

### Phase 4 — Verify

All tests run. Checklist is verified. Two assumptions flagged:

- **[ASSUMED]** `IOrderService.DeleteAsync(int id)` exists based on CRUD pattern.
- **[ASSUMED]** Delete requires no special authorization beyond page-level access.
