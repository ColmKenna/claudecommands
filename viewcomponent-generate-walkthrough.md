---
description: ASP.NET Core ViewComponent Testing Walkthrough Generator
---
## PROMPT PURPOSE & OBJECTIVES

**Primary Objective:**
Generate a comprehensive, step-by-step reference walkthrough that teaches developers how to systematically test any ASP.NET Core ViewComponent using both unit and integration testing approaches. The output is a living guide developers use while writing their own tests, grounded in patterns from actual ASP.NET Core source code examples.

**Secondary Objective:**
Establish a repeatable, reusable framework for testing ViewComponents that developers can apply to any ViewComponent implementation they encounter, covering both C# logic testing (unit) and HTML rendering verification (integration).

---

## TARGET AUDIENCE & CONSTRAINTS

**Target Audience:**
- Developers with 1+ years of xUnit experience
- Understand basic test structure (Arrange-Act-Assert)
- May be new to ViewComponent-specific testing patterns
- Building tests for custom ViewComponents
- Familiar with ASP.NET Core Razor Pages or MVC

**Experience Level Assumption:**
✓ Know how to write xUnit tests, use Assert methods
✓ Familiar with C# and dependency injection patterns
✓ Understand mocking concepts and Moq basics
✓ Have written integration tests before
✗ Don't assume: ViewComponent lifecycle details, complex view rendering engines, advanced Razor syntax beyond basics

**Content Scope - INCLUDE:**
- Unit testing patterns (ViewComponent invoke methods)
- Integration testing patterns (rendered HTML verification)
- Progressive examples from simple to complex
- Mocking strategies for ViewComponent dependencies
- HTML assertion patterns using AngleSharp (DOM parsing + CSS selectors)
- Troubleshooting for ViewComponent-specific test failures
- Real patterns from ASP.NET Core source tests

**Content Scope - EXCLUDE (HARD BOUNDARIES):**
- General xUnit fundamentals (test classes, [Fact] attribute, Assert patterns)
- How to structure a test project or folder organization
- How to configure NuGet packages or MSBuild
- Full ASP.NET Core dependency injection setup
- How to mock unrelated ASP.NET Core types (Controllers, Services, DbContext) beyond what's needed for ViewComponent tests
- Performance or load testing
- End-to-end UI testing (Selenium, Playwright)

---

## PHASE 1: SOURCE CODE ANALYSIS

### 1.1 Repository Navigation

**Action:** Locate and examine ASP.NET Core ViewComponent tests in the official repository.

**File Locations:**
```
https://github.com/dotnet/aspnetcore
Paths:
- src/Microsoft.AspNetCore.Mvc/test/ViewComponentTests.cs
- src/Microsoft.AspNetCore.Mvc.ViewFeatures/test/ViewComponentResultExecutorTests.cs
- src/Microsoft.AspNetCore.Mvc.TagHelpers/test/ (integration test examples)
```

**Files to Examine:**
Look for test classes with these naming patterns:
- `ViewComponentTests.cs` (unit tests for invocation logic)
- `ViewComponentResultExecutorTests.cs` (view rendering tests)
- Example ViewComponent implementations used in tests

### 1.2 Source Pattern Selection Criteria

**Your Task:** Select test examples that represent three complexity levels for unit testing and three for integration testing.

**UNIT TESTING - Category Selection:**

**Category 1: Simple ViewComponent (No Dependencies, Direct Return)**
- File: `ViewComponentTests.cs`
- Select: A test like `Execute_InvokesViewComponent_WithDefaultViewName()`
- Criteria: ViewComponent with no constructor dependencies, returns ViewViewComponentResult, minimal setup
- Document: Full test method showing basic unit test anatomy

**Category 2: Intermediate ViewComponent (Injected Dependency, Conditional Logic)**
- File: `ViewComponentTests.cs`
- Select: A test that uses mocked IViewComponentContext or injected service
- Criteria: Tests behavior that depends on injected dependency, uses ViewData or ViewBag
- Document: Full test method + mocking patterns

**Category 3: Complex ViewComponent (Multiple Dependencies, Async, State Management)**
- File: `ViewComponentTests.cs` or integration tests
- Select: A test with InvokeAsync() and multiple mocked services
- Criteria: Tests async behavior, multiple dependencies, conditional returns
- Document: Full test method + all Moq setup

**INTEGRATION TESTING - Category Selection:**

**Category 4: Simple Integration (Component Renders Basic HTML)**
- File: `ViewComponentResultExecutorTests.cs`
- Select: Test showing view rendering to HTML output
- Criteria: Simple component that renders static or basic model data
- Document: Full test with view rendering and HTML assertion

**Category 5: Intermediate Integration (Model Data in Rendered Output)**
- File: Same as above or example ViewComponent
- Select: Test that verifies component's model appears in rendered HTML
- Criteria: Shows rendering of data-bound elements, list rendering
- Document: Full test with model-based assertions

**Category 6: Complex Integration (Conditional Rendering, Special Cases)**
- File: Same as above
- Select: Test showing different HTML output based on component state
- Criteria: Tests multiple rendering paths, error states
- Document: Full test with conditional assertion patterns

### 1.3 Critical: Mocking Framework & Rendering Pattern Check

**Action 1: Identify Mocking Framework in Source**
```
Search ViewComponentTests.cs for:
- new Mock<...>()       ← Moq syntax
- Substitute.For<...>() ← NSubstitute syntax
- Hand-rolled test doubles (custom fake classes)
```

**Action 2: Identify HTML Assertion/Rendering Pattern**
```
Search for:
- ViewViewComponentResult assertions
- How tests render views to HTML (if at all)
- What HTML parsing approach is used (if any)
```

**Outcome Handling:**

**If you find Moq patterns:**
→ Proceed normally. Use examples as-is. All mocking examples should be Moq syntax.

**If you find NSubstitute patterns:**
→ Adapt language but keep examples grounded in source.
→ When extracting examples, note: [Using NSubstitute from source]
→ Add a Note section: "Source tests use NSubstitute for mocking. All examples below use Moq, which provides equivalent functionality."

**If you find hand-rolled test doubles:**
→ Show the actual pattern from source.
→ In Section E (Mocking), explain the pattern and provide a Moq equivalent.
→ Note: [Pattern from source - Moq equivalent below]

**If source doesn't show integration/rendering tests:**
→ Document finding: "Source unit tests focus on invocation logic and return types. For integration testing patterns (rendering + HTML verification), this guide provides established best practices with AngleSharp DOM parsing."

**If source doesn't use AngleSharp for HTML:**
→ Note what they use (if anything), then explain: "For rendered HTML assertions, this guide uses AngleSharp, a modern, well-maintained HTML5 parser. Alternative: HtmlAgilityPack."

### 1.4 Document Extracted Patterns

For each selected test, extract and document:

**1. ViewComponent Signature**
```
Example from source:
- Constructor parameters and dependencies
- Invoke or InvokeAsync method signature
- Return type (IViewComponentResult)
- Any parameters passed to Invoke
```

**2. Unit Test Method Structure**
```
Extract the setup into logical groups:
Group A: Dependency instantiation/mocking (2-3 lines)
Group B: ViewComponent instantiation (1-2 lines)
Group C: ViewComponentContext setup (if needed) (2-5 lines)
Group D: Invoke() call and result capture (1-2 lines)
Group E: Assertions on result type and model (3-5 lines)
Group F: Mock verification (if applicable) (1-2 lines)

Document what each group does and why it's needed.
```

**3. Execution & Assertion Structure**
```
Extract:
- The Invoke/InvokeAsync() call
- Result type assertion (Assert.IsType<ViewViewComponentResult>)
- Model/ViewData assertions
- Mock verification calls
- What's being verified at the C# logic level
```

**4. Integration Test Method Structure (NEW)**
```
Extract (if found in source):
- Component instantiation
- ViewComponentContext creation for rendering
- Invoke/InvokeAsync call
- View rendering to HTML string
- HTML parsing with selector queries
- Assertions on rendered HTML elements/content
```

**5. Mocking Patterns Identified**
```
Document:
- Which types are mocked (IViewComponentContext, IService, etc.)
- Mock setup code (exact lines)
- What isolation is being achieved
- Which dependencies are NOT mocked (and why)
- Verify call patterns
```

**6. HTML Assertion Patterns (NEW)**
```
If integration tests found, document:
- How HTML output is captured
- What HTML parsing approach is used
- CSS selector usage
- Assertion patterns on rendered elements
```

---

## PHASE 2: WALKTHROUGH GENERATION

### 2.1 Document Structure & Organization

Generate output as a **markdown document** with this exact structure:

```
# ASP.NET Core ViewComponent Testing Walkthrough

## PART 1: UNIT TESTING
  ### Section A: Foundation & Setup (Unit Testing)
  ### Section B: Progressive Complexity - Happy Path (Unit Tests)
  ### Section C: Edge Cases & State Variations (Unit Tests)
  ### Section D: Error Handling & Validation (Unit Tests)
  ### Section E: Mocking Strategy & Dependencies (Unit Tests)
  ### Section F: Troubleshooting Guide (Unit Tests)

## PART 2: INTEGRATION TESTING
  ### Section G: Foundation & Setup (Integration Testing)
  ### Section H: Progressive Complexity - Happy Path (Integration Tests)
  ### Section I: Edge Cases & Rendering Variations (Integration Tests)
  ### Section J: Error Handling & Fallback Rendering (Integration Tests)
  ### Section K: HTML Assertion Patterns & Strategies (CRITICAL)
  ### Section L: Troubleshooting Guide (Integration Tests)

## FINAL VERIFICATION CHECKLIST (Unit + Integration Combined)
```

**Formatting Standards:**

| Element | Rule |
|---------|------|
| **Main Sections** | Use `## ` (H2 markdown) |
| **Subsections** | Use `### ` (H3 markdown) |
| **Code Examples** | Use ` ```csharp ` code blocks |
| **Callout/Rationale** | Use `> ` (blockquote) for "Why" explanations |
| **Key Terms** | Use `**bold**` for emphasis, NOT ALL_CAPS |
| **Checklists** | Use `- [ ]` markdown checkbox format |
| **Spacing** | One blank line between sections, before/after code blocks |
| **Line Length** | Code blocks max 20 lines (excerpt longer tests) |
| **Tables** | Use markdown table format for decision matrices |

---

## PART 1: UNIT TESTING SECTIONS

### 2.2 SECTION A: Foundation & Setup (Unit Testing)

**Purpose:**
Establish what a basic ViewComponent unit test looks like by showing the anatomy and breaking it into conceptual components.

**Content Structure:**

**Subsection A.1: Understanding ViewComponent Unit Tests**

Write 2-3 paragraphs explaining: A ViewComponent unit test verifies the C# logic (what data the component prepares) without rendering HTML. The test has three functional parts. Explain why each is necessary.

**Subsection A.2: Part 1 - Dependency Setup & Component Instantiation**

Show 2-3 lines of code creating mocked dependencies and instantiating the component.

```markdown
### Part 1: Dependency Setup & Component Instantiation

\`\`\`csharp
var mockService = new Mock<IAlertService>();
var component = new AlertViewComponent(mockService.Object);
\`\`\`

**What this does:** Creates a mocked dependency (IAlertService) and injects it into the ViewComponent's constructor.

**Why:** Unit tests isolate the component's logic from external dependencies. Mocking lets you control what the dependency returns and verify the component calls it correctly.
```

**Subsection A.3: Part 2 - Invoke the Component & Capture Result**

```markdown
### Part 2: Invoke the Component & Capture Result

\`\`\`csharp
var result = component.Invoke("warning");
\`\`\`

**What this does:** Calls the ViewComponent's Invoke method with parameters.

**Why:** This is what you're testing—the component's logic, not the view rendering. The result tells you what data the component prepared.
```

**Subsection A.4: Part 3 - Assert on the Result**

```markdown
### Part 3: Assert on Result Type & Data

\`\`\`csharp
var viewResult = Assert.IsType<ViewViewComponentResult>(result);
var model = Assert.IsType<Alert[]>(viewResult.ViewData.Model);
Assert.Equal(1, model.Length);

mockService.Verify(s => s.GetAlerts("warning"), Times.Once);
\`\`\`

**What this does:**
- Verifies the result is ViewViewComponentResult (not ContentViewComponentResult or other type)
- Casts and verifies the model is correct type and contains expected data
- Verifies the mocked service was called with the correct parameter

**Why:** Unit tests verify behavior at the C# level. You're checking: Does the component call the right service method? Does it prepare the right data for the view?
```

**Subsection A.5: Complete Working Example**

Show a full test method from source (ViewComponentTests.cs), keeping it to 15-20 lines maximum.

```markdown
### Complete Example: Basic ViewComponent Unit Test

**Source:**
[Link to GitHub: src/Microsoft.AspNetCore.Mvc/test/ViewComponentTests.cs]

\`\`\`csharp
[Actual or realistic test method from source, 15-20 lines]
\`\`\`

**This test demonstrates:**
- Mock dependency creation
- Component instantiation with injected mock
- Invoke() call and result type assertion
- Model/data assertion
```

**Subsection A.6: Foundation Checklist**

```markdown
### Before Moving to Complexity: Verify You Can...

- [ ] Create mock dependencies using `new Mock<IService>()`
- [ ] Instantiate the ViewComponent with mocked dependencies
- [ ] Call Invoke() or InvokeAsync() and capture the result
- [ ] Assert the result is the expected type (ViewViewComponentResult, etc.)
- [ ] Access ViewData.Model and assert on its contents
- [ ] Understand why each setup step is needed
```

---

### 2.3 SECTION B: Progressive Complexity - Happy Path (Unit Tests)

**Purpose:**
Show how unit tests evolve in complexity as you test more sophisticated ViewComponent behavior.

**Use this Template for each progression:**

```markdown
## Progression [Level]: [Scenario Name]

**What you're testing:** [1-2 sentences on the behavior]

**Example from source:** [ViewComponent name and source file]

### The Test

\`\`\`csharp
[5-15 lines of test code]
\`\`\`

### Breaking It Down

#### Setup (Arrange)
[2-3 sentences on dependencies and component setup]

#### Execution (Act)
[1-2 sentences on what Invoke/InvokeAsync does]

#### Assertion (Assert)
[2-3 sentences on what's verified and why]

### Key Insight
[1 paragraph on the pattern to apply to your own tests]
```

**Progression 1: Simple Invoke - No Dependencies**

```csharp
[Fact]
public void Invoke_NoParameters_ReturnsDefaultContent()
{
    var component = new SimpleComponent();
    
    var result = component.Invoke();
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    Assert.Null(viewResult.ViewData.Model);  // No model
}
```

**Breaking it down:**
- **Setup:** Component has no constructor parameters, so no mocks needed.
- **Execution:** Invoke() returns a ViewViewComponentResult.
- **Assertion:** Verify result type and model state (null or empty).

**Key insight:** This is the simplest unit test pattern. It verifies a ViewComponent without dependencies returns the expected view reference.

---

**Progression 2: Invoke with Injected Service**

```csharp
[Fact]
public void Invoke_WithService_ReturnsPreparedData()
{
    var mockService = new Mock<IProductService>();
    mockService.Setup(s => s.GetProducts())
        .Returns(new[] { new Product { Id = 1, Name = "Widget" } });
    
    var component = new ProductListComponent(mockService.Object);
    var result = component.Invoke();
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    var model = Assert.IsType<Product[]>(viewResult.ViewData.Model);
    Assert.Single(model);
    Assert.Equal("Widget", model[0].Name);
}
```

**Breaking it down:**
- **Setup:** Mock IProductService and configure it to return test data.
- **Execution:** Invoke() the component, which calls the service internally.
- **Assertion:** Verify the component passed service data to the view.

**Key insight:** Mock external dependencies to isolate the component's logic. Verify that the component receives data from the service and passes it to the view correctly.

---

**Progression 3: Invoke with Parameters & Service Call Verification**

```csharp
[Fact]
public void Invoke_WithCategoryParameter_CallsServiceWithParameter()
{
    var mockService = new Mock<IProductService>();
    mockService.Setup(s => s.GetProductsByCategory(It.IsAny<string>()))
        .Returns(new[] { new Product { Category = "Electronics" } });
    
    var component = new ProductListComponent(mockService.Object);
    var result = component.Invoke("Electronics");
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    var model = Assert.IsType<Product[]>(viewResult.ViewData.Model);
    Assert.Single(model);
    
    // Verify the component called the service with the correct parameter
    mockService.Verify(
        s => s.GetProductsByCategory("Electronics"), 
        Times.Once);
}
```

**Breaking it down:**
- **Setup:** Mock service with parameter-specific behavior.
- **Execution:** Invoke with parameter—component should pass it to the service.
- **Assertion:** Verify result AND verify the service was called with the correct parameter.

**Key insight:** When a ViewComponent has parameters, test that it passes them correctly to dependencies. This ensures the parameter handling logic works as expected.

---

### 2.4 SECTION C: Edge Cases & State Variations (Unit Tests)

**Purpose:**
Show what happens when inputs deviate from the happy path but shouldn't break.

**Category 1: Null or Missing Dependencies**

```csharp
[Fact]
public void Constructor_WithNullDependency_ThrowsArgumentNullException()
{
    var ex = Assert.Throws<ArgumentNullException>(() =>
        new ProductListComponent(null));
    
    Assert.Contains("IProductService", ex.Message);
}
```

**Why test:** Clarify whether null dependencies should fail fast (exception) or be handled gracefully.

---

**Category 2: Empty Data from Dependency**

```csharp
[Fact]
public void Invoke_WhenServiceReturnsEmpty_ReturnsEmptyModel()
{
    var mockService = new Mock<IProductService>();
    mockService.Setup(s => s.GetProducts())
        .Returns(Array.Empty<Product>());
    
    var component = new ProductListComponent(mockService.Object);
    var result = component.Invoke();
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    var model = Assert.IsType<Product[]>(viewResult.ViewData.Model);
    Assert.Empty(model);
}
```

**Why test:** Components should handle empty data gracefully without exceptions.

---

**Category 3: Conditional Return Types (View vs. Content)**

```csharp
[Fact]
public void Invoke_WhenUnauthorized_ReturnsContentWithError()
{
    var mockAuth = new Mock<IAuthService>();
    mockAuth.Setup(s => s.IsAuthorized()).Returns(false);
    
    var component = new ProtectedComponent(mockAuth.Object);
    var result = component.Invoke();
    
    var contentResult = Assert.IsType<ContentViewComponentResult>(result);
    Assert.Contains("Unauthorized", contentResult.Content);
}
```

**Why test:** Components may return different result types (View vs. Content) based on state. Test both paths.

---

**Category 4: ViewData/ViewBag Population**

```csharp
[Fact]
public void Invoke_SetsViewDataVariables()
{
    var component = new HeaderComponent();
    component.ViewComponentContext = new ViewComponentContext
    {
        ViewData = new ViewDataDictionary(
            new EmptyModelMetadataProvider(),
            new ModelStateDictionary())
    };
    
    var result = component.Invoke();
    
    Assert.NotNull(component.ViewComponentContext.ViewData["Title"]);
    Assert.Equal("Page Title", component.ViewComponentContext.ViewData["Title"]);
}
```

**Why test:** Components often set ViewData for use in their views. Test these side effects.

---

### 2.5 SECTION D: Error Handling & Validation (Unit Tests)

**Scenario 1: Service Throws Exception**

```csharp
[Fact]
public void Invoke_WhenServiceThrows_ThrowsException()
{
    var mockService = new Mock<IProductService>();
    mockService.Setup(s => s.GetProducts())
        .Throws(new InvalidOperationException("DB unavailable"));
    
    var component = new ProductListComponent(mockService.Object);
    
    var ex = Assert.Throws<InvalidOperationException>(() =>
        component.Invoke());
    
    Assert.Contains("DB unavailable", ex.Message);
}
```

**Or, if the component handles exceptions:**

```csharp
[Fact]
public void Invoke_WhenServiceThrows_ReturnsEmptyData()
{
    var mockService = new Mock<IProductService>();
    mockService.Setup(s => s.GetProducts())
        .Throws<InvalidOperationException>();
    
    var component = new ProductListComponent(mockService.Object);
    var result = component.Invoke();
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    var model = Assert.IsType<Product[]>(viewResult.ViewData.Model);
    Assert.Empty(model);  // Gracefully handled
}
```

---

**Scenario 2: Async Component Timeout or Cancellation**

```csharp
[Fact]
public async Task InvokeAsync_WhenCancelled_ThrowsOperationCanceledException()
{
    var mockService = new Mock<IProductService>();
    mockService.Setup(s => s.GetProductsAsync(It.IsAny<CancellationToken>()))
        .Throws<OperationCanceledException>();
    
    var component = new ProductListComponent(mockService.Object);
    
    await Assert.ThrowsAsync<OperationCanceledException>(() =>
        component.InvokeAsync(CancellationToken.None));
}
```

---

### 2.6 SECTION E: Mocking Strategy & Dependencies (Unit Tests)

**Provide a decision matrix:**

| Dependency | Mock or Real? | Why |
|---|---|---|
| IViewComponentContext | Use Real | Part of setup, not what you're testing |
| ViewDataDictionary | Use Real | Simple POCO, deterministic |
| IRepository / IService | **Mock** | External dependency; isolate component logic |
| HttpContext (if used) | Mock (if accessed) | Complex; only mock if actually used |
| ILogger | Mock or Null | Not testing logging |
| IOptions<TConfig> | Real (if simple) | Provide test config; usually no side effects |
| IMapper / AutoMapper | Can be Real | Usually deterministic |

---

**Subsection E.1: Creating ViewComponentContext for Unit Tests**

```csharp
// Minimal Setup (if ViewComponent doesn't access context)
var component = new SimpleComponent();
// No need to set ViewComponentContext

// Full Setup (if ViewComponent accesses ViewData/ViewBag)
var context = new ViewComponentContext
{
    ViewData = new ViewDataDictionary(
        new EmptyModelMetadataProvider(),
        new ModelStateDictionary()),
    ViewBag = new DynamicViewData()
};

component.ViewComponentContext = context;
```

---

**Subsection E.2: Moq Patterns for Services**

```csharp
// Pattern 1: Simple Return Value
var mockService = new Mock<IProductService>();
mockService.Setup(s => s.GetProducts())
    .Returns(new[] { new Product { Id = 1 } });

// Pattern 2: Parameter-Specific Setup
mockService.Setup(s => s.GetProductsByCategory(It.IsAny<string>()))
    .Returns<string>(category =>
        category == "Electronics"
            ? new[] { new Product { Category = "Electronics" } }
            : Array.Empty<Product>());

// Pattern 3: Async Methods
mockService.Setup(s => s.GetProductsAsync(It.IsAny<CancellationToken>()))
    .ReturnsAsync(new[] { new Product { Id = 1 } });

// Pattern 4: Verify Calls Were Made
mockService.Verify(s => s.GetProducts(), Times.Once);
mockService.Verify(s => s.GetProducts(), Times.Never);
mockService.Verify(s => s.GetProductsByCategory("Electronics"), Times.AtLeastOnce());
```

---

### 2.7 SECTION F: Troubleshooting Guide (Unit Tests)

**Only include ViewComponent-specific issues:**

**Problem 1: "Invoke() returns null, but I expected ViewViewComponentResult"**

**Symptom:** `component.Invoke()` returns null.
**Root cause:** ViewComponent's Invoke method isn't returning anything.
**Solution:** Check the component code for missing return statement.
**Prevention:** Unit tests should always assert the result type; null results catch logic errors immediately.

---

**Problem 2: "ViewData.Model is null even though component should have data"**

**Symptom:** `viewResult.ViewData.Model` is null.
**Root cause:** Component calls `return View();` instead of `return View(model);`.
**Solution:** Check component code—ensure model is passed to View().
**Prevention:** Always assert on ViewData.Model in unit tests.

---

**Problem 3: "Mock service isn't being called (parameter mismatch)"**

**Symptom:** Mock setup configured for `GetProducts()`, but component calls `GetProducts(filter)`.
**Root cause:** Parameter mismatch in mock setup.
**Solution:** Use `It.IsAny<T>()` to match any parameters.

```csharp
// Wrong:
mockService.Setup(s => s.GetProducts()).Returns(...);

// Right:
mockService.Setup(s => s.GetProducts(It.IsAny<string>())).Returns(...);
```

---

**Problem 4: "How do I test InvokeAsync()?"**

**Solution:** Use async/await:

```csharp
[Fact]
public async Task InvokeAsync_ReturnsView()
{
    var component = new MyComponent();
    var result = await component.InvokeAsync();
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    // ...
}
```

---

**Problem 5: "ViewComponentContext is required but I don't know how to set it up"**

**Solution:** Only set it if your component accesses ViewData or ViewBag:

```csharp
var context = new ViewComponentContext
{
    ViewData = new ViewDataDictionary(
        new EmptyModelMetadataProvider(),
        new ModelStateDictionary())
};

component.ViewComponentContext = context;
```

---

## PART 2: INTEGRATION TESTING SECTIONS

### 2.8 SECTION G: Foundation & Setup (Integration Testing)

**Purpose:**
Establish the difference between unit and integration testing, and show how to render a ViewComponent to HTML.

**Subsection G.1: Integration Testing vs. Unit Testing**

```
Unit Testing:        Component logic only (Invoke → ViewViewComponentResult with model)
Integration Testing: Component logic + View rendering (HTML output with data)
```

For integration tests, you:
1. Invoke the component (same as unit test)
2. Get the ViewViewComponentResult
3. Render its view to an HTML string
4. Parse and assert on the rendered HTML

---

**Subsection G.2: The Three Parts of an Integration Test**

**Part 1: Setup Component & Context**

```csharp
var component = new AlertComponent(new TestAlertService());
component.ViewComponentContext = CreateViewComponentContext();
```

**What this does:** Creates the component with test dependencies and provides rendering context.
**Why:** Views need ViewComponentContext and services to render HTML.

---

**Part 2: Invoke & Render the View**

```csharp
var result = await component.InvokeAsync();
var viewResult = result as ViewViewComponentResult;
var html = await RenderViewToString(component, viewResult);
```

**What this does:** Invokes component and renders its view to HTML.
**Why:** You need actual HTML to test rendering behavior.

---

**Part 3: Parse HTML & Assert on Elements**

```csharp
var doc = new HtmlParser().ParseDocument(html);
var alerts = doc.QuerySelectorAll(".alert");
Assert.NotEmpty(alerts);
Assert.Equal("Warning", alerts.First().TextContent);
```

**What this does:** Parses HTML as DOM and uses CSS selectors to verify elements.
**Why:** Tests verify specific elements exist with correct content and attributes.

---

**Subsection G.3: Helper Method - RenderViewToString**

**You'll need a helper to render the ViewComponentResult to HTML string. Here's a working implementation:**

```csharp
private async Task<string> RenderViewToString(
    ViewComponent component,
    ViewViewComponentResult viewResult)
{
    var httpContext = new DefaultHttpContext();
    httpContext.RequestServices = CreateServiceProvider();
    
    var viewComponentContext = new ViewComponentContext
    {
        ViewContext = new ViewContext
        {
            HttpContext = httpContext,
            RouteData = new RouteData(),
            ActionDescriptor = new ActionDescriptor()
        },
        ViewData = viewResult.ViewData ?? new ViewDataDictionary(
            new EmptyModelMetadataProvider(),
            new ModelStateDictionary()),
        ViewBag = new DynamicViewData()
    };
    
    component.ViewComponentContext = viewComponentContext;
    
    using (var writer = new StringWriter())
    {
        var view = viewResult.ResolveAndExecuteView(viewComponentContext);
        await view.RenderAsync(new ViewContext
        {
            Writer = writer,
            HttpContext = httpContext,
            RouteData = new RouteData(),
            ActionDescriptor = new ActionDescriptor(),
            ViewData = viewComponentContext.ViewData
        });
        
        return writer.ToString();
    }
}

private IServiceProvider CreateServiceProvider()
{
    var services = new ServiceCollection();
    services.AddSingleton<IViewComponentHelper, DefaultViewComponentHelper>();
    services.AddSingleton<IHtmlHelper, HtmlHelper>();
    // Add other services as needed for your views
    return services.BuildServiceProvider();
}
```

> **Note:** This implementation requires `using Microsoft.AspNetCore.Mvc.ViewFeatures;` and related namespaces. Adjust based on your ASP.NET Core version.

---

**Subsection G.4: Complete Working Integration Test Example**

```csharp
[Fact]
public async Task Invoke_RendersAlertWithMessage()
{
    // Setup
    var alerts = new[] { new Alert { Message = "Warning", Type = "warning" } };
    var service = new TestAlertService(alerts);
    var component = new AlertComponent(service);
    component.ViewComponentContext = CreateViewComponentContext();
    
    // Invoke & Render
    var result = await component.InvokeAsync();
    var html = await RenderViewToString(component, result as ViewViewComponentResult);
    
    // Assert on rendered HTML
    Assert.Contains("Warning", html);
    Assert.Contains("class=\"alert-warning\"", html);
    
    // Or use DOM parsing for more precise verification
    var doc = new HtmlParser().ParseDocument(html);
    var alertDiv = doc.QuerySelector("div.alert-warning");
    Assert.NotNull(alertDiv);
    Assert.Equal("Warning", alertDiv.TextContent);
}
```

---

**Subsection G.5: Integration Testing Checklist**

```markdown
### Before Moving to Complexity: Verify You Can...

- [ ] Create a ViewComponent instance with test dependencies
- [ ] Set up ViewComponentContext properly
- [ ] Invoke/InvokeAsync and capture the ViewViewComponentResult
- [ ] Render the view result to HTML string
- [ ] Parse HTML using AngleSharp HtmlParser
- [ ] Use CSS selectors to find elements (QuerySelector, QuerySelectorAll)
- [ ] Assert on HTML content (TextContent) and attributes (GetAttribute)
```

---

### 2.9 SECTION H: Progressive Complexity - Happy Path (Integration Tests)

**Mirror unit test progressions, but verify rendered HTML:**

**Progression 1: Simple Component - Renders Expected HTML**

```csharp
[Fact]
public async Task Invoke_RendersHeaderWithTitle()
{
    var component = new HeaderComponent();
    component.ViewComponentContext = CreateViewComponentContext();
    
    var result = await component.InvokeAsync();
    var html = await RenderViewToString(component, result as ViewViewComponentResult);
    
    Assert.Contains("<h1>My Site</h1>", html);
}
```

**Breaking it down:**
- **Setup:** Create component and set context.
- **Invoke & Render:** Get HTML.
- **Assert:** Verify expected HTML fragment exists.

**Key insight:** For quick HTML checks, use simple string contains. Only use DOM parsing for complex assertions.

---

**Progression 2: Component with Model - Renders Data in HTML**

```csharp
[Fact]
public async Task Invoke_WithProducts_RendersListItems()
{
    var products = new[]
    {
        new Product { Id = 1, Name = "Widget" },
        new Product { Id = 2, Name = "Gadget" }
    };
    
    var service = new TestProductService(products);
    var component = new ProductListComponent(service);
    component.ViewComponentContext = CreateViewComponentContext();
    
    var result = await component.InvokeAsync();
    var html = await RenderViewToString(component, result as ViewViewComponentResult);
    
    // Parse HTML and verify structure
    var doc = new HtmlParser().ParseDocument(html);
    var items = doc.QuerySelectorAll("li.product-item");
    
    Assert.Equal(2, items.Length);
    Assert.Contains("Widget", html);
    Assert.Contains("Gadget", html);
    
    // Verify specific data in specific elements
    Assert.Equal("Widget", items[0].TextContent);
    Assert.Equal("Gadget", items[1].TextContent);
}
```

**Breaking it down:**
- **Setup:** Create test service with data.
- **Invoke & Render:** Get HTML.
- **Assert:** Parse HTML, count elements, verify data content.

**Key insight:** When rendering model data, use CSS selectors to verify correct number of elements and correct data in each.

---

**Progression 3: Conditional Rendering - Different HTML Based on State**

```csharp
[Fact]
public async Task Invoke_WhenAuthorized_RendersDashboard()
{
    var mockAuth = new Mock<IAuthService>();
    mockAuth.Setup(s => s.IsAuthorized()).Returns(true);
    
    var component = new DashboardComponent(mockAuth.Object);
    component.ViewComponentContext = CreateViewComponentContext();
    
    var result = await component.InvokeAsync();
    var html = await RenderViewToString(component, result as ViewViewComponentResult);
    
    var doc = new HtmlParser().ParseDocument(html);
    var dashboard = doc.QuerySelector("div.dashboard");
    
    Assert.NotNull(dashboard);
    Assert.Contains("Welcome back", html);
}

[Fact]
public async Task Invoke_WhenNotAuthorized_RendersLoginPrompt()
{
    var mockAuth = new Mock<IAuthService>();
    mockAuth.Setup(s => s.IsAuthorized()).Returns(false);
    
    var component = new DashboardComponent(mockAuth.Object);
    component.ViewComponentContext = CreateViewComponentContext();
    
    var result = await component.InvokeAsync();
    var html = await RenderViewToString(component, result as ViewViewComponentResult);
    
    var doc = new HtmlParser().ParseDocument(html);
    var loginPrompt = doc.QuerySelector("div.login-prompt");
    
    Assert.NotNull(loginPrompt);
    Assert.DoesNotContain("Welcome back", html);
}
```

**Breaking it down:**
- **Setup:** Create component with different states.
- **Invoke & Render:** Get HTML for each state.
- **Assert:** Verify different elements render based on state.

**Key insight:** Test conditional rendering by invoking with different setup and verifying different HTML output.

---

### 2.10 SECTION K: HTML Assertion Patterns & Strategies (CRITICAL)

**This section is unique to integration testing and most important for rendering verification.**

---

**Pattern 1: Simple String Contains (Fastest)**

```csharp
var html = await RenderViewToString(component, viewResult);
Assert.Contains("Success message", html);
Assert.DoesNotContain("<script>", html);  // Verify malicious code not present
```

**When to use:**
- Quick verification that specific text exists
- Checking for simple HTML snippets
- Rough spot checks for presence/absence

**When NOT to use:**
- When HTML formatting varies (whitespace breaks exact matching)
- When you need to verify structure or count elements
- When you need to check attributes precisely

**Pitfall - Formatting Changes:**
```csharp
// Expected:
Assert.Contains("<button>Submit</button>", html);

// But HTML might be formatted as:
"<button>\n  Submit\n</button>"

// Test fails because exact string doesn't match!
```

---

**Pattern 2: DOM Parsing with CSS Selectors (Recommended)**

```csharp
var doc = new HtmlParser().ParseDocument(html);

// Find single element
var alertDiv = doc.QuerySelector("div.alert");
Assert.NotNull(alertDiv);

// Find multiple elements
var buttons = doc.QuerySelectorAll("button.btn-primary");
Assert.Equal(2, buttons.Length);

// Verify content
Assert.Equal("Submit", buttons.First().TextContent);

// Verify attributes
Assert.Equal("btn-primary", buttons.First().ClassName);
Assert.True(buttons.First().HasAttribute("disabled"));
```

**When to use:**
- Finding specific elements by selector
- Counting elements (e.g., list items)
- Verifying element content or attributes
- Checking nested structures
- **This is the recommended approach for most integration tests**

**When NOT to use:**
- For simple "does this text exist" checks (use Pattern 1)
- When performance is critical (marginally slower, but negligible in tests)

**AngleSharp CSS Selector Guide:**

```csharp
// Basic selectors
doc.QuerySelector("div");                    // First div
doc.QuerySelector(".alert");                 // First element with class "alert"
doc.QuerySelector("#header");                // Element with id "header"
doc.QuerySelector("button[type='submit']");  // Button with attribute type="submit"

// Combination selectors
doc.QuerySelector("div.alert");              // Div with class "alert"
doc.QuerySelector("form button.submit");     // Button with class "submit" inside form
doc.QuerySelector("li:first-child");         // First list item

// Get all matches
doc.QuerySelectorAll(".alert");              // All elements with class "alert"
doc.QuerySelectorAll("button");              // All buttons

// Work with results
var items = doc.QuerySelectorAll("li.item");
foreach (var item in items)
{
    var text = item.TextContent;
    var id = item.GetAttribute("data-id");
}
```

---

**Pattern 3: Attribute-Specific Checks (Most Precise)**

```csharp
var doc = new HtmlParser().ParseDocument(html);

// Form field attributes
var emailInput = doc.QuerySelector("input[name='email']");
Assert.NotNull(emailInput);
Assert.Equal("email", emailInput.GetAttribute("type"));
Assert.True(emailInput.HasAttribute("required"));
Assert.Equal("user@example.com", emailInput.GetAttribute("value"));

// CSS class verification
var button = doc.QuerySelector("button");
Assert.True(button.ClassList.Contains("btn"));
Assert.True(button.ClassList.Contains("btn-primary"));

// Data attributes
var dataDiv = doc.QuerySelector("div[data-alert-id]");
Assert.Equal("123", dataDiv.GetAttribute("data-alert-id"));

// ARIA attributes (accessibility)
var closeBtn = doc.QuerySelector("button[aria-label]");
Assert.Equal("Close dialog", closeBtn.GetAttribute("aria-label"));
Assert.Equal("true", closeBtn.GetAttribute("aria-pressed"));
```

**When to use:**
- Testing form field attributes (type, required, value, name)
- Verifying CSS classes applied correctly
- Checking data-* attributes
- Testing accessibility (ARIA) attributes
- When precise attribute matching is critical

---

**Subsection K.1: Decision Tree**

```
Your assertion goal:
├─ "Verify this text exists in the output"
│  └─ Use Pattern 1: Assert.Contains("text", html)
│
├─ "Find specific elements and verify their count/content/structure"
│  └─ Use Pattern 2: doc.QuerySelector(...)
│
└─ "Verify specific attributes (class, type, data-*, aria-*)"
   └─ Use Pattern 3: element.GetAttribute(...)
```

---

**Subsection K.2: AngleSharp Cheat Sheet**

```csharp
// Create parser and parse HTML
var parser = new HtmlParser();
var doc = parser.ParseDocument(html);

// Select elements
var single = doc.QuerySelector("selector");           // First match
var multiple = doc.QuerySelectorAll("selector");      // All matches

// Element properties
var text = element.TextContent;                       // Inner text (no tags)
var html = element.InnerHtml;                         // Inner HTML (with tags)
var className = element.ClassName;                    // All classes as string
var classList = element.ClassList;                    // ITokenList (for Contains, etc.)

// Attributes
var value = element.GetAttribute("name");             // Get attribute value
var exists = element.HasAttribute("required");        // Check if attribute exists
element.SetAttribute("data-test", "value");           // Set attribute

// Class operations
element.ClassList.Add("new-class");                   // Add class
element.ClassList.Remove("old-class");                // Remove class
element.ClassList.Contains("class-name");             // Check if has class
element.ClassList.Toggle("toggled");                  // Toggle class

// Navigation
var parent = element.ParentElement;                   // Parent element
var children = element.Children;                      // Child elements
var firstChild = element.FirstElementChild;           // First child element
```

---

**Subsection K.3: Safe HTML Assertion Best Practices**

```
- [ ] Use CSS selectors for element finding (more robust than string matching)
- [ ] Always check element exists (null check) before accessing properties
- [ ] Use TextContent for text verification (handles whitespace automatically)
- [ ] Use GetAttribute() for attributes, check with HasAttribute() first
- [ ] For classes, use ClassList.Contains() or ClassName (handles multiple classes)
- [ ] Test both positive (element exists) and negative (element absent) cases
- [ ] Don't assume specific HTML formatting or whitespace
- [ ] Use specific selectors (e.g., "input.email" instead of "input") to avoid false matches
```

---

### 2.11 SECTION I: Edge Cases & Rendering Variations (Integration Tests)

**Focus on how ViewComponent handles edge cases when rendering:**

**Category 1: Empty or Missing Data**

```csharp
[Fact]
public async Task Invoke_WithNoProducts_RendersEmptyState()
{
    var service = new TestProductService(Array.Empty<Product>());
    var component = new ProductListComponent(service);
    component.ViewComponentContext = CreateViewComponentContext();
    
    var result = await component.InvokeAsync();
    var html = await RenderViewToString(component, result as ViewViewComponentResult);
    
    var doc = new HtmlParser().ParseDocument(html);
    var items = doc.QuerySelectorAll("li.product-item");
    
    Assert.Empty(items);
    Assert.Contains("No products available", html);
}
```

---

**Category 2: Large Content / Truncation**

```csharp
[Fact]
public async Task Invoke_WithLongText_TruncatesInDisplay()
{
    var longText = "A" + new string('b', 200);
    var service = new TestService { LongText = longText };
    var component = new DisplayComponent(service);
    component.ViewComponentContext = CreateViewComponentContext();
    
    var result = await component.InvokeAsync();
    var html = await RenderViewToString(component, result as ViewViewComponentResult);
    
    var doc = new HtmlParser().ParseDocument(html);
    var display = doc.QuerySelector(".text-display");
    
    // Verify truncation applied
    Assert.True(display.TextContent.Length <= 100);
    Assert.Contains("...", display.TextContent);
}
```

---

**Category 3: Special Characters & HTML Encoding**

```csharp
[Fact]
public async Task Invoke_WithSpecialCharacters_RendersHtmlEncoded()
{
    var alerts = new[] { new Alert { Message = "<script>alert('xss')</script>" } };
    var service = new TestAlertService(alerts);
    var component = new AlertComponent(service);
    component.ViewComponentContext = CreateViewComponentContext();
    
    var result = await component.InvokeAsync();
    var html = await RenderViewToString(component, result as ViewViewComponentResult);
    
    // Verify dangerous code is encoded
    Assert.DoesNotContain("<script>", html);
    Assert.Contains("&lt;script&gt;", html);
}
```

---

### 2.12 SECTION J: Error Handling & Fallback Rendering (Integration Tests)

**Focus on what the user sees when things go wrong:**

**Scenario 1: Service Error - Renders Error Message**

```csharp
[Fact]
public async Task Invoke_WhenServiceThrows_RendersErrorMessage()
{
    var mockService = new Mock<IProductService>();
    mockService.Setup(s => s.GetProductsAsync(It.IsAny<CancellationToken>()))
        .ThrowsAsync(new InvalidOperationException("Database error"));
    
    var component = new ProductListComponent(mockService.Object);
    component.ViewComponentContext = CreateViewComponentContext();
    
    var result = await component.InvokeAsync();
    var html = await RenderViewToString(component, result as ViewViewComponentResult);
    
    var doc = new HtmlParser().ParseDocument(html);
    var errorDiv = doc.QuerySelector("div.error-message");
    
    Assert.NotNull(errorDiv);
    Assert.Contains("Unable to load products", html);
}
```

---

**Scenario 2: Null Model - Graceful Fallback**

```csharp
[Fact]
public async Task Invoke_WithNullModel_RendersWithDefaults()
{
    var service = new TestService { ReturnNull = true };
    var component = new WidgetComponent(service);
    component.ViewComponentContext = CreateViewComponentContext();
    
    var result = await component.InvokeAsync();
    var html = await RenderViewToString(component, result as ViewViewComponentResult);
    
    var doc = new HtmlParser().ParseDocument(html);
    var widget = doc.QuerySelector("div.widget");
    
    Assert.NotNull(widget);  // Component still renders
    Assert.Contains("Default Widget", html);  // Shows default content
}
```

---

### 2.13 SECTION L: Troubleshooting Guide (Integration Tests)

**Integration-specific issues:**

**Problem 1: "HTML output is empty even though component rendered"**

**Symptom:** `RenderViewToString()` returns empty or whitespace-only string.
**Root cause:** ViewComponentContext isn't properly set up, or view file can't be found.
**Solution:**
```csharp
// Verify context is created
Assert.NotNull(component.ViewComponentContext);

// Verify view result has view name
var viewResult = result as ViewViewComponentResult;
Assert.NotNull(viewResult);
Assert.NotNull(viewResult.ViewName);  // "Default" or specific view name
```

---

**Problem 2: "CSS selector doesn't find element I can see in HTML"**

**Symptom:** `doc.QuerySelector(".my-class")` returns null, but element is visible in HTML.
**Root cause:** Class name differs, or selector syntax is wrong.
**Solution:**
```csharp
// Debug: Print HTML to see actual structure
Console.WriteLine(html);

// Verify selector syntax
var element = doc.QuerySelector("div.my-class");  // element.class
var element = doc.QuerySelector(".my-class");     // Shorthand
var element = doc.QuerySelector("#my-id");        // ID selector

// Verify class values
var classList = element.ClassList;
var hasClass = classList.Contains("my-class");
```

---

**Problem 3: "Assert.Contains fails due to whitespace differences"**

**Symptom:** Expected `"<button>Submit</button>"` but HTML has newlines/indentation.
**Root cause:** HTML formatting varies (normal).
**Solution:** Use DOM parsing instead:

```csharp
// Don't:
Assert.Contains("<button>Submit</button>", html);

// Do:
var button = doc.QuerySelector("button");
Assert.Equal("Submit", button.TextContent);
```

---

**Problem 4: "Special characters/HTML entities not rendering correctly"**

**Symptom:** Ampersand (&) appears as text instead of &amp; entity.
**Root cause:** View template uses `@Html.Raw()` instead of auto-encoding.
**Solution:** Check view template. Use `@Model.Property` (auto-encoded) instead of `@Html.Raw()`.

---

**Problem 5: "How do I test asynchronous ViewComponent rendering?"**

**Solution:** Use async/await:

```csharp
[Fact]
public async Task InvokeAsync_RendersAsyncContent()
{
    var component = new AsyncComponent();
    component.ViewComponentContext = CreateViewComponentContext();
    
    var result = await component.InvokeAsync();  // await the InvokeAsync
    var html = await RenderViewToString(component, result as ViewViewComponentResult);
    
    // Assert on HTML...
}
```

---

**Problem 6: "RenderViewToString helper is too complex or not working"**

**Solution:** Simplify by using a test host or mock renderer. If using the helper provided above, verify:
- `IServiceProvider` includes `IViewComponentHelper` and `IHtmlHelper`
- `ViewComponentContext` has valid `ViewContext`
- View file exists in the project with correct name ("Default.cshtml")

---

## PHASE 3: OUTPUT QUALITY STANDARDS

### 3.1 Annotation Standards

**Principle:** Annotate at the **conceptual block level**, not line-by-line.

**Correct Approach:**

Each code example should be grouped into 2-4 conceptual blocks. Each block should be 2-8 lines of related code, followed by brief explanations of **What** and **Why**.

**Example:**
```markdown
### Part 1: Create Mock Service

\`\`\`csharp
var mockService = new Mock<IProductService>();
mockService.Setup(s => s.GetProducts())
    .Returns(new[] { new Product { Id = 1 } });
\`\`\`

**What this does:** Creates a mocked dependency with configured return value.
**Why:** Isolates the component's logic from external services.
```

---

### 3.2 Code Snippet Formatting Standards

- **Max line length:** 20 lines per block (excerpt longer tests with `[... additional setup ...]`)
- **Language specification:** Always use ` ```csharp `
- **Source attribution:** Include comment with source file path if from Microsoft tests
- **Synthetic examples:** Mark as `[Generalized Example]` if created for clarity

**Example:**
```csharp
// From: src/Microsoft.AspNetCore.Mvc/test/ViewComponentTests.cs
[Fact]
public void Invoke_WithParameter_CallsService()
{
    var mockService = new Mock<IProductService>();
    mockService.Setup(s => s.GetProducts(It.IsAny<string>()))
        .Returns(new[] { new Product { Id = 1 } });
    
    var component = new ProductListComponent(mockService.Object);
    var result = component.Invoke("Electronics");
    
    // [... assertions ...]
}
```

---

### 3.3 Target Output Length

**2,500 - 4,500 words** (slightly longer than TagHelper due to two testing paradigms)

- If under 2,500 words: Add more specific examples or expand troubleshooting
- If over 4,500 words: Consolidate progressions or remove redundant patterns

---

### 3.4 Section Completeness Checklist

**UNIT TESTING (Sections A-F):**
- [ ] Section A: 3 parts of anatomy + 1 complete working example
- [ ] Section B: 3 progressions (simple → with params → with verification)
- [ ] Section C: 4-5 edge case categories with ≥1 example each
- [ ] Section D: 3-4 error handling scenarios
- [ ] Section E: Dependency matrix + Moq pattern cheat sheet
- [ ] Section F: 5-6 troubleshooting issues specific to unit testing

**INTEGRATION TESTING (Sections G-L):**
- [ ] Section G: 3 parts of anatomy + RenderViewToString helper + 1 complete example
- [ ] Section H: 3 progressions (simple → with data → conditional)
- [ ] Section I: 3-4 edge cases (empty data, long content, special chars)
- [ ] Section J: 2-3 error/fallback scenarios
- [ ] Section K: ALL 3 PATTERNS (String contains, DOM selectors, attributes) + decision tree + AngleSharp cheat sheet
- [ ] Section L: 5-6 integration-specific troubleshooting issues

**FINAL CHECKLIST:**
- [ ] Final Verification Checklist combines unit + integration criteria

---

### 3.5 Code Quality Standards

- [ ] Every code example ≤ 20 lines (excerpt with `[... setup ...]` if needed)
- [ ] Conceptual block annotations (not line-by-line explanations)
- [ ] All examples from source include file path in comment
- [ ] All synthetic examples marked `[Generalized Example]`
- [ ] Moq syntax consistent throughout (fluent style)
- [ ] AngleSharp syntax consistent (QuerySelector, GetAttribute, etc.)
- [ ] No incomplete code snippets (all compilable examples)

---

### 3.6 Formatting Standards

| Element | Standard |
|---------|----------|
| **Main Sections** | `## ` (H2) |
| **Subsections** | `### ` (H3) |
| **Code blocks** | ` ```csharp ` with language specified |
| **Callouts** | `> ` (blockquote) for rationale/why explanations |
| **Emphasis** | `**bold**` for key terms, not ALL_CAPS |
| **Lists** | Use `- [ ]` checkboxes for verification checklists |
| **Tables** | Markdown tables for decision matrices |
| **Spacing** | One blank line between sections and before/after code blocks |

---

### 3.7 Self-Verification Checklist (For Claude Code Agent)

Before generating final output, verify:

**Completeness:**
- [ ] All 12 sections present (A-L + Final Checklist)
- [ ] Each section has 2+ examples where applicable
- [ ] Progressions build from simple to complex
- [ ] Edge cases are realistic and actionable
- [ ] Troubleshooting covers top issues for each testing type

**Code Quality:**
- [ ] Every code example ≤ 20 lines
- [ ] All examples fully annotated at conceptual level
- [ ] Examples from source include file path
- [ ] Consistent use of Moq syntax
- [ ] Consistent use of AngleSharp syntax
- [ ] No pseudocode (all examples are compilable)

**Clarity:**
- [ ] No general xUnit fundamentals (assumes baseline knowledge)
- [ ] No general ASP.NET Core setup content
- [ ] All "What" explanations present (task description)
- [ ] All "Why" explanations present (reasoning)
- [ ] ViewComponent-specific terminology used correctly
- [ ] Section flow is logical (progressions build on foundation)

**Output Format:**
- [ ] Markdown structure correct (H2 sections, H3 subsections)
- [ ] All code blocks specify ` ```csharp `
- [ ] All blockquotes (>) used for rationale/why
- [ ] All checklists use markdown checkbox format
- [ ] One blank line between sections
- [ ] Tables properly formatted
- [ ] No orphaned code (all code has context)

**Coverage:**
- [ ] Both unit and integration testing covered equally
- [ ] Happy path, edge cases, and error handling included
- [ ] All three HTML assertion patterns explained with examples
- [ ] Helper methods provided (RenderViewToString, CreateViewComponentContext)
- [ ] Decision trees provided (when to use which pattern)
- [ ] Cheat sheets provided (Moq, AngleSharp)

**Usability:**
- [ ] Developer can use this while writing tests in parallel
- [ ] Examples are copy-paste ready (realistic)
- [ ] Troubleshooting references patterns from earlier sections
- [ ] Final checklist gives clear stopping point
- [ ] Cross-references between sections (e.g., "see Section X for...")

---

## FINAL SECTION: Verification Checklist (Unit + Integration Combined)

```markdown
## Verification Checklist: Is Your Test Suite Complete?

Use this checklist when you've finished writing your ViewComponent test suite
to ensure comprehensive coverage of both unit and integration testing.

### Functional Coverage

**Unit Testing:**
- [ ] Happy path test: Component.Invoke() returns correct ViewViewComponentResult
- [ ] At least one test per constructor dependency (verify it's used)
- [ ] At least one test per Invoke parameter
- [ ] At least one test per conditional return path (View vs. Content vs. Default)
- [ ] At least one test for ViewData/ViewBag population (if used)
- [ ] At least one error scenario (exception handling or graceful degradation)

**Integration Testing:**
- [ ] Component renders basic HTML without errors
- [ ] Component renders model data correctly in HTML
- [ ] At least one test per conditional rendering path (different HTML output)
- [ ] Empty data scenario (no items, null values)
- [ ] Special characters are properly HTML-encoded
- [ ] Async rendering (if component uses InvokeAsync)

### Test Structure & Quality

**Naming & Organization:**
- [ ] Unit test names follow pattern: `[Method]_[Condition]_[Outcome]`
  - Example: `Invoke_WithProducts_ReturnsViewWithModel`
- [ ] Integration test names follow pattern: `[Method]_[Condition]_[Renders|Verifies]`
  - Example: `InvokeAsync_WithProducts_RendersListItems`
- [ ] All tests are independent (can run in any order)
- [ ] Tests grouped by testing type (unit section, then integration section)

**Arrange-Act-Assert Structure:**
- [ ] Each unit test has clear setup, invoke, assert
- [ ] Each integration test has setup, invoke/render, assert
- [ ] No test does more than one logical thing (single responsibility)
- [ ] No hardcoded magic strings (use meaningful variable names)

### Mocking & Dependencies

**Unit Tests:**
- [ ] All external dependencies are mocked (IRepository, IService, etc.)
- [ ] All mocks have explicit Setup() calls (not left empty)
- [ ] Assertions test behavior (return values, ViewData), not mock calls (few exceptions)
- [ ] No over-mocking: only mock what's actually needed for isolation

**Integration Tests:**
- [ ] ViewComponentContext properly configured
- [ ] ServiceProvider includes necessary services (IViewComponentHelper, etc.)
- [ ] View files exist and can be found
- [ ] Rendering doesn't throw exceptions

### Assertions & Verification

**Unit Tests:**
- [ ] Every test has 1-3 assertions (minimum 1)
- [ ] Assertions verify ViewViewComponentResult or other result type
- [ ] Assertions check ViewData.Model or ViewBag values
- [ ] Assertions verify service calls (Verify() for mocks)
- [ ] Tests for exceptions use Assert.Throws or Assert.ThrowsAsync

**Integration Tests:**
- [ ] Every test has 1-3 assertions (minimum 1)
- [ ] Assertions use appropriate HTML pattern:
  - [ ] Simple string contains for quick checks
  - [ ] CSS selectors (QuerySelector) for element verification
  - [ ] Attribute checks (GetAttribute) for precise verification
- [ ] Tests verify positive case (element exists with correct data)
- [ ] Tests verify negative case (element missing when it should be)

### Coverage Metrics

- [ ] Unit tests exercise at least 80% of component's C# logic
- [ ] Integration tests exercise all rendering paths
- [ ] Both happy path and error paths are tested
- [ ] Edge cases specific to your component are covered
- [ ] ViewComponent-specific features are tested (conditional returns, etc.)

### Code Quality & Maintainability

- [ ] Complex test setup is commented (explaining why)
- [ ] Non-obvious assertions are annotated
- [ ] Tests serve as documentation for using the component
- [ ] No duplicate test code (shared setup or helpers)
- [ ] HTML parsing uses consistent AngleSharp patterns

### Self-Test: Run & Verify

- [ ] All tests pass individually
- [ ] All tests pass as a suite
- [ ] Tests pass on a clean checkout
- [ ] Tests complete in under 10 seconds total (good CI feedback)
- [ ] Test names clearly describe what they verify

---

## SUCCESS CRITERIA

The output is successful if a developer can:

1. ✓ Understand the anatomy of a unit test for ViewComponents (Section A)
2. ✓ Write a unit test for their own ViewComponent (Section B)
3. ✓ Know what unit test edge cases to cover (Section C)
4. ✓ Test error handling in the component logic (Section D)
5. ✓ Set up mocking correctly for dependencies (Section E)
6. ✓ Debug failing unit tests (Section F)
7. ✓ Understand the anatomy of an integration test (Section G)
8. ✓ Write an integration test that renders and verifies HTML (Section H)
9. ✓ Know what integration test edge cases to cover (Section I)
10. ✓ Test error states in rendered output (Section J)
11. ✓ Choose and use the right HTML assertion pattern (Section K)
12. ✓ Debug failing integration tests (Section L)
13. ✓ Verify their test suite is comprehensive (Final Checklist)

The output is high-quality if it:
- Uses real examples from ASP.NET Core source (not invented)
- Explains **Why**, not just **What**
- Avoids general xUnit/ASP.NET Core content
- Is scannable as a reference while writing tests
- Provides working code examples (copy-paste ready)
- Includes helper methods developers can reuse
- Covers both unit and integration testing equally
- Makes clear distinctions between the two testing types

---

## EXECUTION INSTRUCTIONS FOR CLAUDE CODE

### Step 1: Repository Access & Analysis
1. Access https://github.com/dotnet/aspnetcore
2. Navigate to `src/Microsoft.AspNetCore.Mvc/test/ViewComponentTests.cs`
3. Follow Section 1.3 (Mocking Framework Check) to identify patterns
4. Document findings before proceeding

### Step 2: Source Code Extraction
1. Extract patterns for Category 1 (simple), 2 (intermediate), 3 (complex) per Section 1.2
2. Note mocking framework used
3. Document test naming conventions
4. Look for integration test examples in ViewComponentResultExecutorTests.cs

### Step 3: Content Generation
Generate each section in order (A → Final Checklist):
- Generate unit testing sections (A-F) first
- Then generate integration testing sections (G-L)
- Verify each section against standards in Phase 3 before outputting

### Step 4: Code Quality Pass
Before returning final output:
- Run through Self-Verification Checklist (Section 3.7)
- Verify all code examples are ≤ 20 lines
- Verify all conceptual block annotations are present
- Confirm word count is 2,500-4,500 words
- Check markdown formatting

### Step 5: Output Delivery
Generate as a single markdown document.
Title: `ASP.NET Core ViewComponent Testing Walkthrough: A Step-by-Step Guide`

---

## NOTES FOR COLM

**Key Adaptations from TagHelper Prompt:**
- Unit testing mirrors TagHelper structure (familiar foundation for you)
- Integration testing is NEW section (unique to ViewComponents)
- Section K (HTML Assertions) is critical differentiator
- RenderViewToString helper is provided for developer convenience
- Two testing types run in separate sections (Option A organization)

**Emphasis Points:**
- AngleSharp for modern HTML parsing (vs. regex or string matching)
- Clear progression from simple to complex in both testing types
- Specific troubleshooting for integration issues (rendering, HTML assertion failures)
- Helper methods included (RenderViewToString, CreateViewComponentContext)

**Expected Output Quality:**
- 2,500-4,500 words of high-quality, prescriptive content
- 12+ complete code examples (extraction from source + synthesis)
- Clear, actionable decision trees
- Practical troubleshooting rooted in real problems
- Copy-paste ready code developers can use immediately
```