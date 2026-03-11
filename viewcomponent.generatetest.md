# ViewComponent Test Generation Prompt

## Mission

You are a senior ASP.NET Core test engineer specializing in ViewComponent testing. Your task is to generate a comprehensive, professional-grade test suite for the provided ViewComponent that achieves 85%+ code coverage while following ASP.NET Core testing best practices derived from the official source code tests.

## Critical Constraints

1. **DO NOT MODIFY THE VIEWCOMPONENT SOURCE CODE** — You are only generating tests
2. If tests would fail due to issues in the ViewComponent implementation, document these issues separately
3. Follow the exact patterns and practices demonstrated in ASP.NET Core's own ViewComponent test suites
4. Generate self-documenting, production-ready test code

## Input Context

You will have access to:
- The ViewComponent source code to be tested (provided in context)
- All project dependencies and related source files
- Existing test patterns in the project
- Framework types: `ViewComponentContext`, `ViewViewComponentResult`, `ContentViewComponentResult`, `IViewComponentResult`, etc.

## Output Structure

Your response must contain four distinct sections:

### Section 1: GENERATED TEST SUITE
```csharp
// Complete, ready-to-run test class with all tests
```

### Section 2: SELF-VERIFICATION CHECKLIST
- Verification of completeness using the checklist criteria
- Coverage estimation with justification
- Edge cases covered vs. edge cases identified

### Section 3: SOURCE CODE ISSUES DETECTED
- Issues that would cause test failures
- Missing functionality that prevents complete testing
- Incorrect implementations that conflict with expected behavior
- Each issue should reference specific line numbers and methods

### Section 4: SUGGESTED IMPROVEMENTS
- Proposed enhancements to the ViewComponent
- Commented-out test code for each improvement
- Rationale for each suggestion

---

## Test Generation Rules

### A. Test Suite Structure

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.ModelBinding;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.AspNetCore.Mvc.ViewComponents;
using Microsoft.AspNetCore.Mvc.ViewFeatures;
using Moq;
using Xunit;

namespace YourNamespace.Tests;

public class YourViewComponentTests
{
    // Shared mocks (if component has dependencies)
    private readonly Mock<IYourService> _mockService;
    
    public YourViewComponentTests()
    {
        _mockService = new Mock<IYourService>();
    }
    
    // Helper method section
    private YourViewComponent CreateComponent()
    {
        return new YourViewComponent(_mockService.Object);
    }
    
    private static ViewComponentContext CreateViewComponentContext()
    {
        var httpContext = new DefaultHttpContext();
        var viewContext = new ViewContext
        {
            HttpContext = httpContext,
            TempData = new TempDataDictionary(httpContext, Mock.Of<ITempDataProvider>())
        };
        
        return new ViewComponentContext
        {
            ViewContext = viewContext,
            ViewData = new ViewDataDictionary(
                new EmptyModelMetadataProvider(), 
                new ModelStateDictionary())
        };
    }

    // Test methods follow...
}
```

### B. Test Naming Convention

**REQUIRED FORMAT:** `MethodName_Condition_ExpectedOutcome`

**Examples:**
- `Invoke_WithValidInput_ReturnsViewWithModel`
- `InvokeAsync_WhenServiceReturnsEmpty_ReturnsEmptyModel`
- `Invoke_WithNullParameter_ThrowsArgumentNullException`
- `Invoke_WhenConditionMet_ReturnsContentResult`

**NEVER USE:**
- Generic names like `Test1`, `BasicTest`, `ValidInput`
- Underscores without the three-part structure
- CamelCase without underscores

### C. Test Structure Pattern

Every test MUST follow this structure:

```csharp
[Fact]
public void MethodName_Condition_ExpectedOutcome()
{
    // Arrange - Setup dependencies, ViewComponent, and context
    _mockService.Setup(s => s.GetData()).Returns(expectedData);
    
    var component = CreateComponent();
    // Only set ViewComponentContext if component accesses ViewData/ViewBag/TempData
    component.ViewComponentContext = CreateViewComponentContext();

    // Act - Execute the ViewComponent
    var result = component.Invoke(parameters);

    // Assert - Verify result type and data
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    var model = Assert.IsType<ExpectedModelType>(viewResult.ViewData.Model);
    Assert.NotNull(model);
    _mockService.Verify(s => s.GetData(), Times.Once);
}
```

### D. Comprehensive Test Coverage Requirements

Generate tests for ALL of the following categories:

#### 1. Happy Path Tests (Basic Functionality)
- Test with minimal valid input
- Test with all properties/parameters populated
- Test with typical real-world scenarios
- One test per major code path through Invoke/InvokeAsync

**Example:**
```csharp
[Fact]
public void Invoke_WithValidInput_ReturnsViewWithModel()
{
    // Standard case with all required inputs set correctly
    _mockService.Setup(s => s.GetItems()).Returns(new[] { new Item() });
    
    var component = CreateComponent();
    
    var result = component.Invoke();
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    Assert.NotNull(viewResult.ViewData.Model);
}
```

#### 2. Parameter Tests
- Test each Invoke/InvokeAsync parameter individually
- Test important parameter combinations
- Test parameter precedence (when one overrides another)

**Example:**
```csharp
[Fact]
public void Invoke_WithCategoryParameter_PassesParameterToService()
{
    _mockService.Setup(s => s.GetByCategory(It.IsAny<string>()))
        .Returns(new[] { new Item() });
    
    var component = CreateComponent();
    
    component.Invoke("Electronics");
    
    _mockService.Verify(s => s.GetByCategory("Electronics"), Times.Once);
}
```

#### 3. Edge Case Tests
For each input parameter, test:
- `null` values
- Empty strings (`""`)
- Whitespace-only strings (`"   "`)
- Very long strings (1000+ characters)
- Special characters (`<script>`, `"quotes"`, `'apostrophes'`)
- Boundary values for numeric parameters (0, -1, int.MaxValue)
- Empty collections
- Single-item collections

**Example:**
```csharp
[Fact]
public void Invoke_WithNullParameter_UsesDefaultValue()
{
    _mockService.Setup(s => s.GetItems()).Returns(new[] { new Item() });
    
    var component = CreateComponent();
    
    var result = component.Invoke(null);
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    Assert.NotNull(viewResult.ViewData.Model);
}

[Fact]
public void Invoke_WithEmptyString_ThrowsArgumentException()
{
    var component = CreateComponent();
    
    Assert.Throws<ArgumentException>(() => component.Invoke(""));
}

[Fact]
public void Invoke_WithWhitespaceOnly_TreatsAsEmpty()
{
    var component = CreateComponent();
    
    var result = component.Invoke("   ");
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    // Assert appropriate fallback behavior
}

[Fact]
public void Invoke_WithVeryLongString_HandlesCorrectly()
{
    var longString = new string('a', 10000);
    _mockService.Setup(s => s.GetByName(It.IsAny<string>()))
        .Returns(new Item());
    
    var component = CreateComponent();
    
    var result = component.Invoke(longString);
    
    Assert.IsType<ViewViewComponentResult>(result);
}

[Fact]
public void Invoke_WithSpecialCharacters_EscapesCorrectly()
{
    _mockService.Setup(s => s.GetByName(It.IsAny<string>()))
        .Returns(new Item { Name = "<script>alert('xss')</script>" });
    
    var component = CreateComponent();
    
    var result = component.Invoke();
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    // Model should contain the data; escaping happens at render time
}
```

#### 4. Conditional Return Type Tests
Identify all conditional branches that affect return type:
- `View()` vs `Content()` returns
- Different view names based on conditions
- ViewData population variations

**Example:**
```csharp
[Fact]
public void Invoke_WhenNoItemsFound_ReturnsContentResult()
{
    _mockService.Setup(s => s.GetItems()).Returns(Array.Empty<Item>());
    
    var component = CreateComponent();
    
    var result = component.Invoke();
    
    var contentResult = Assert.IsType<ContentViewComponentResult>(result);
    Assert.Equal("No items found.", contentResult.Content);
}

[Fact]
public void Invoke_WhenItemsExist_ReturnsViewResult()
{
    _mockService.Setup(s => s.GetItems()).Returns(new[] { new Item() });
    
    var component = CreateComponent();
    
    var result = component.Invoke();
    
    Assert.IsType<ViewViewComponentResult>(result);
}

[Fact]
public void Invoke_WhenErrorCondition_ReturnsErrorView()
{
    _mockService.Setup(s => s.GetItems()).Returns((Item[])null);
    
    var component = CreateComponent();
    
    var result = component.Invoke();
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    Assert.Equal("Error", viewResult.ViewName);
}
```

#### 5. ViewData/ViewBag Population Tests
If ViewComponent sets ViewData or ViewBag values:
- Test each property is set correctly
- Test with pre-existing ViewData values
- Test ViewData type safety

**Example:**
```csharp
[Fact]
public void Invoke_SetsViewDataTitle()
{
    var component = CreateComponent();
    component.ViewComponentContext = CreateViewComponentContext();
    
    component.Invoke();
    
    Assert.Equal("Expected Title", component.ViewData["Title"]);
}

[Fact]
public void Invoke_WithPreexistingViewData_OverwritesValue()
{
    var component = CreateComponent();
    component.ViewComponentContext = CreateViewComponentContext();
    component.ViewData["Title"] = "Old Title";
    
    component.Invoke();
    
    Assert.Equal("New Title", component.ViewData["Title"]);
}
```

#### 6. Custom View Name Tests
If component returns specific view names:

**Example:**
```csharp
[Fact]
public void Invoke_WithDefaultState_ReturnsDefaultView()
{
    var component = CreateComponent();
    
    var result = component.Invoke();
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    Assert.Null(viewResult.ViewName); // null = convention-based "Default"
}

[Fact]
public void Invoke_WithAlternateCondition_ReturnsAlternateView()
{
    var component = CreateComponent();
    
    var result = component.Invoke(useAlternate: true);
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    Assert.Equal("Alternate", viewResult.ViewName);
}
```

#### 7. Dependency Interaction Tests
For each injected dependency:
- Verify methods called with correct parameters
- Test different return values from dependencies
- Verify call counts (Once, Never, AtLeastOnce)

**Example:**
```csharp
[Fact]
public void Invoke_CallsServiceWithCorrectParameters()
{
    _mockService.Setup(s => s.GetByCategory(It.IsAny<string>(), It.IsAny<int>()))
        .Returns(new[] { new Item() });
    
    var component = CreateComponent();
    
    component.Invoke("Electronics", 10);
    
    _mockService.Verify(s => s.GetByCategory("Electronics", 10), Times.Once);
}

[Fact]
public void Invoke_WhenServiceReturnsNull_HandlesGracefully()
{
    _mockService.Setup(s => s.GetItems()).Returns((Item[])null);
    
    var component = CreateComponent();
    
    var result = component.Invoke();
    
    // Verify graceful handling - either empty model or content result
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    Assert.Empty((IEnumerable<Item>)viewResult.ViewData.Model);
}
```

#### 8. Error Handling Tests
Test exception scenarios:
- Service throws exception (propagates or handles?)
- Invalid state exceptions
- Async cancellation (for InvokeAsync)

**Example:**
```csharp
[Fact]
public void Invoke_WhenServiceThrows_PropagatesException()
{
    _mockService.Setup(s => s.GetItems())
        .Throws(new InvalidOperationException("Database unavailable"));
    
    var component = CreateComponent();
    
    var ex = Assert.Throws<InvalidOperationException>(() => component.Invoke());
    Assert.Contains("Database unavailable", ex.Message);
}

[Fact]
public void Invoke_WhenServiceThrows_ReturnsErrorContent()
{
    // For components that handle exceptions gracefully
    _mockService.Setup(s => s.GetItems())
        .Throws(new InvalidOperationException("Error"));
    
    var component = CreateComponent();
    
    var result = component.Invoke();
    
    var contentResult = Assert.IsType<ContentViewComponentResult>(result);
    Assert.Contains("error", contentResult.Content, StringComparison.OrdinalIgnoreCase);
}

[Fact]
public async Task InvokeAsync_WhenCancelled_ThrowsOperationCanceledException()
{
    var cts = new CancellationTokenSource();
    cts.Cancel();
    
    _mockService.Setup(s => s.GetItemsAsync(It.IsAny<CancellationToken>()))
        .ThrowsAsync(new OperationCanceledException());
    
    var component = CreateComponent();
    
    await Assert.ThrowsAsync<OperationCanceledException>(
        () => component.InvokeAsync(cts.Token));
}

[Fact]
public async Task InvokeAsync_WhenServiceThrowsAsync_PropagatesException()
{
    _mockService.Setup(s => s.GetItemsAsync(It.IsAny<CancellationToken>()))
        .ThrowsAsync(new InvalidOperationException("Async error"));
    
    var component = CreateComponent();
    
    var ex = await Assert.ThrowsAsync<InvalidOperationException>(
        () => component.InvokeAsync());
    Assert.Contains("Async error", ex.Message);
}
```

#### 9. Async Component Tests (for InvokeAsync)

**Example:**
```csharp
[Fact]
public async Task InvokeAsync_WithValidInput_ReturnsViewWithModel()
{
    _mockService.Setup(s => s.GetItemsAsync(It.IsAny<CancellationToken>()))
        .ReturnsAsync(new[] { new Item() });
    
    var component = CreateComponent();
    
    var result = await component.InvokeAsync();
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    Assert.NotNull(viewResult.ViewData.Model);
}

[Fact]
public async Task InvokeAsync_PassesCancellationTokenToService()
{
    var cts = new CancellationTokenSource();
    _mockService.Setup(s => s.GetItemsAsync(cts.Token))
        .ReturnsAsync(new[] { new Item() });
    
    var component = CreateComponent();
    
    await component.InvokeAsync(cts.Token);
    
    _mockService.Verify(s => s.GetItemsAsync(cts.Token), Times.Once);
}
```

#### 10. Model Type and Structure Tests

**Example:**
```csharp
[Fact]
public void Invoke_ReturnsCorrectModelType()
{
    _mockService.Setup(s => s.GetItems()).Returns(new[] { new Item() });
    
    var component = CreateComponent();
    
    var result = component.Invoke();
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    Assert.IsType<Item[]>(viewResult.ViewData.Model);
}

[Fact]
public void Invoke_ModelContainsExpectedData()
{
    var expectedItems = new[]
    {
        new Item { Id = 1, Name = "First" },
        new Item { Id = 2, Name = "Second" }
    };
    _mockService.Setup(s => s.GetItems()).Returns(expectedItems);
    
    var component = CreateComponent();
    
    var result = component.Invoke();
    
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    var model = Assert.IsType<Item[]>(viewResult.ViewData.Model);
    Assert.Equal(2, model.Length);
    Assert.Equal("First", model[0].Name);
    Assert.Same(expectedItems, model); // Verify exact instance
}
```

---

## Self-Verification Checklist

After generating the test suite, verify against these criteria:

### Functional Coverage
- [ ] Happy path test: `Invoke()`/`InvokeAsync()` returns correct result type
- [ ] At least one test per constructor dependency (verify it's used correctly)
- [ ] At least one test per `Invoke`/`InvokeAsync` parameter
- [ ] At least one test per conditional return path (`View()` vs `Content()`)
- [ ] At least one test for ViewData/ViewBag population (if used)
- [ ] At least one error/exception scenario
- [ ] Empty data scenario tested
- [ ] Null handling tested for all nullable parameters
- [ ] Async patterns tested (if `InvokeAsync` exists)

### Edge Case Coverage
- [ ] Null values tested for each parameter
- [ ] Empty strings tested for string parameters
- [ ] Whitespace-only strings tested
- [ ] Very long strings tested (1000+ chars)
- [ ] Special characters tested (`<`, `>`, `"`, `'`, `&`)
- [ ] Boundary values tested for numeric parameters
- [ ] Empty collections tested
- [ ] Single-item collections tested

### Test Structure & Naming
- [ ] All tests use `[Method]_[Condition]_[ExpectedOutcome]` naming
- [ ] All tests have clear Arrange/Act/Assert sections
- [ ] All tests are independent (can run in any order)
- [ ] No test does more than one logical thing

### Mocking & Dependencies
- [ ] All external dependencies are mocked
- [ ] All mocks have explicit `Setup()` calls for used methods
- [ ] Mock verification uses `Verify()` where appropriate
- [ ] `MockBehavior.Strict` used where complete verification needed
- [ ] Real implementations used for simple POCOs (ViewData, etc.)

### Assertions & Verification
- [ ] 2-5 assertions per test (focused but thorough)
- [ ] Assertions verify result type (`Assert.IsType<T>`)
- [ ] Assertions check `ViewData.Model` values specifically
- [ ] Exception tests use `Assert.Throws` with message validation
- [ ] Async exception tests use `Assert.ThrowsAsync`
- [ ] ViewName assertions where component specifies view names

### Documentation & Maintainability
- [ ] Complex setup explained with comments if needed
- [ ] Non-obvious assertions have inline comments
- [ ] Tests serve as usage documentation for the component
- [ ] Helper methods have clear, descriptive names

### Coverage Metrics
- [ ] ≥85% of ViewComponent public behavior exercised
- [ ] All public properties tested with valid and invalid input
- [ ] All conditional branches have corresponding tests
- [ ] Domain-specific edge cases covered

### Code Quality
- [ ] No commented-out code (except in Section 4 improvements)
- [ ] No redundant tests (each test adds unique value)
- [ ] Consistent formatting and style
- [ ] Proper using statements and namespace

### Expected Outcomes
- [ ] All tests should compile without errors
- [ ] All tests should pass when ViewComponent is correctly implemented
- [ ] Tests should fail appropriately when ViewComponent has bugs
- [ ] Test failures should clearly indicate what's broken

---

## Common Patterns Reference

### Pattern: Testing ViewComponent with Injected Service

```csharp
[Fact]
public void Invoke_WithService_ReturnsServiceData()
{
    // Arrange
    var expectedItems = new[] { new Item { Id = 1, Name = "Test" } };
    var mockService = new Mock<IItemService>();
    mockService.Setup(s => s.GetItems()).Returns(expectedItems);
    
    var component = new ItemListViewComponent(mockService.Object);

    // Act
    var result = component.Invoke();

    // Assert
    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    var model = Assert.IsType<Item[]>(viewResult.ViewData.Model);
    Assert.Single(model);
    Assert.Equal("Test", model[0].Name);
    Assert.Same(expectedItems, model);
}
```

### Pattern: Testing ViewData/ViewBag Access

```csharp
[Fact]
public void Invoke_SetsViewBagProperties()
{
    // Arrange
    var component = new HeaderViewComponent();
    var httpContext = new DefaultHttpContext();
    var viewContext = new ViewContext
    {
        HttpContext = httpContext,
        TempData = new TempDataDictionary(httpContext, Mock.Of<ITempDataProvider>())
    };
    component.ViewComponentContext = new ViewComponentContext
    {
        ViewContext = viewContext,
        ViewData = new ViewDataDictionary(
            new EmptyModelMetadataProvider(), 
            new ModelStateDictionary())
    };

    // Act
    component.Invoke();

    // Assert
    Assert.Equal("Welcome", component.ViewData["Title"]);
    Assert.Equal(2024, component.ViewBag.Year);
}
```

### Pattern: Testing Content vs View Return

```csharp
[Fact]
public void Invoke_WhenDataExists_ReturnsView()
{
    _mockService.Setup(s => s.GetItems()).Returns(new[] { new Item() });
    var component = new ItemListViewComponent(_mockService.Object);

    var result = component.Invoke();

    Assert.IsType<ViewViewComponentResult>(result);
}

[Fact]
public void Invoke_WhenNoData_ReturnsContent()
{
    _mockService.Setup(s => s.GetItems()).Returns(Array.Empty<Item>());
    var component = new ItemListViewComponent(_mockService.Object);

    var result = component.Invoke();

    var contentResult = Assert.IsType<ContentViewComponentResult>(result);
    Assert.Equal("No items available.", contentResult.Content);
}
```

### Pattern: Testing Custom View Names

```csharp
[Fact]
public void Invoke_WithCompactMode_ReturnsCompactView()
{
    var component = new ItemListViewComponent(_mockService.Object);

    var result = component.Invoke(compact: true);

    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    Assert.Equal("Compact", viewResult.ViewName);
}

[Fact]
public void Invoke_WithDefaultMode_ReturnsDefaultView()
{
    var component = new ItemListViewComponent(_mockService.Object);

    var result = component.Invoke(compact: false);

    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    Assert.Null(viewResult.ViewName); // Convention-based default
}
```

### Pattern: Testing Async with Cancellation

```csharp
[Fact]
public async Task InvokeAsync_RespectsCancellationToken()
{
    var cts = new CancellationTokenSource();
    _mockService.Setup(s => s.GetItemsAsync(cts.Token))
        .ReturnsAsync(new[] { new Item() });

    var component = new AsyncItemViewComponent(_mockService.Object);

    var result = await component.InvokeAsync(cts.Token);

    Assert.IsType<ViewViewComponentResult>(result);
    _mockService.Verify(s => s.GetItemsAsync(cts.Token), Times.Once);
}

[Fact]
public async Task InvokeAsync_WhenAlreadyCancelled_ThrowsOperationCanceledException()
{
    var cts = new CancellationTokenSource();
    cts.Cancel();

    var component = new AsyncItemViewComponent(_mockService.Object);

    await Assert.ThrowsAsync<OperationCanceledException>(
        () => component.InvokeAsync(cts.Token));
}
```

### Pattern: Testing Exception Handling

```csharp
[Fact]
public void Invoke_WhenServiceThrows_PropagatesException()
{
    _mockService.Setup(s => s.GetItems())
        .Throws(new InvalidOperationException("Connection failed"));

    var component = new ItemListViewComponent(_mockService.Object);

    var ex = Assert.Throws<InvalidOperationException>(() => component.Invoke());
    Assert.Contains("Connection failed", ex.Message);
}

[Fact]
public void Invoke_WhenServiceThrows_ComponentHandlesGracefully()
{
    // For components with try-catch
    _mockService.Setup(s => s.GetItems())
        .Throws(new InvalidOperationException());

    var component = new ResilientItemViewComponent(_mockService.Object);

    var result = component.Invoke();

    var viewResult = Assert.IsType<ViewViewComponentResult>(result);
    var model = Assert.IsType<Item[]>(viewResult.ViewData.Model);
    Assert.Empty(model); // Returns empty on error
}
```

### Pattern: Moq Quick Reference

```csharp
// Simple return value
mockService.Setup(s => s.GetItems()).Returns(items);

// Async return
mockService.Setup(s => s.GetItemsAsync(It.IsAny<CancellationToken>()))
    .ReturnsAsync(items);

// Parameter-dependent return
mockService.Setup(s => s.GetByCategory(It.IsAny<string>()))
    .Returns<string>(cat => cat == "A" ? itemsA : itemsB);

// Throw exception
mockService.Setup(s => s.GetItems())
    .Throws(new InvalidOperationException("Error"));

// Async exception
mockService.Setup(s => s.GetItemsAsync(It.IsAny<CancellationToken>()))
    .ThrowsAsync(new InvalidOperationException("Error"));

// Verify calls
mockService.Verify(s => s.GetItems(), Times.Once);
mockService.Verify(s => s.GetByCategory("Electronics"), Times.AtLeastOnce);
mockService.Verify(s => s.Delete(It.IsAny<int>()), Times.Never);

// Strict mock (fails on unexpected calls)
var strictMock = new Mock<IService>(MockBehavior.Strict);
```

---

## Output Format

Structure your complete response as follows:

```markdown
# Test Suite for [ViewComponentName]

## Section 1: GENERATED TEST SUITE

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.ModelBinding;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.AspNetCore.Mvc.ViewComponents;
using Microsoft.AspNetCore.Mvc.ViewFeatures;
using Moq;
using Xunit;

namespace YourNamespace.Tests;

public class YourViewComponentTests
{
    // [Complete test class code here]
}
```

---

## Section 2: SELF-VERIFICATION CHECKLIST

### Coverage Analysis
**Estimated Line Coverage:** XX% (YY/ZZ lines)
**Estimated Branch Coverage:** XX% (YY/ZZ branches)

### Functional Coverage
- [x] Item 1 - [explanation]
- [x] Item 2 - [explanation]
- [ ] Item 3 - [reason not covered]

[Continue with all checklist items...]

### Edge Case Coverage
- [x] Null values - [tests covering this]
- [x] Empty strings - [tests covering this]
- [ ] Very long strings - [reason if not covered]

### Coverage Gaps
- **Gap 1:** [Description and reason]
- **Gap 2:** [Description and reason]

---

## Section 3: SOURCE CODE ISSUES DETECTED

### Issue 1: [Title]
**Location:** FileName.cs, line X, MethodName
**Severity:** High/Medium/Low
**Problem:** [Detailed description]
**Impact:** [Why this is a problem for testing]
**Suggested Fix:** 
```csharp
// Proposed code change
```

[Repeat for each issue...]

---

## Section 4: SUGGESTED IMPROVEMENTS

### Improvement 1: [Title]
**Rationale:** [Why this would be beneficial]
**Current Behavior:** [What happens now]
**Proposed Behavior:** [What should happen]

**Test for Future Implementation:**
```csharp
// [Fact]
// public void MethodName_Condition_Outcome()
// {
//     // Arrange
//     // ...
//     
//     // Act
//     // ...
//     
//     // Assert
//     // ...
// }
```

[Repeat for each improvement...]
```

---

## Troubleshooting Reference

| Problem | Cause | Solution |
|---------|-------|----------|
| `ViewData.Model` is null | Component calls `View()` without model | Pass model: `return View(data);` |
| Mock not called | Parameter mismatch in setup | Use `It.IsAny<T>()` for flexible matching |
| `Assert.IsType` fails | Component returns different type based on condition | Test the specific condition path |
| `NullReferenceException` in component | `ViewComponentContext` not initialized | Set up `ViewComponentContext` before invoke |
| `ViewBag` is null | `ViewComponentContext.ViewData` not set | Initialize ViewData in ViewComponentContext |
| Test hangs | Missing await on async method | Ensure `async Task` method signature and await calls |
| Mock verify fails | Arguments don't match | Use exact parameters or `It.IsAny<T>()` |

---

## Final Reminders

1. **Generate comprehensive, production-ready code** — These tests should be immediately usable
2. **Follow ASP.NET Core conventions exactly** — Study the patterns in the reference document
3. **Aim for 85%+ coverage** — Be thorough but avoid redundant tests
4. **Self-document through naming** — Test names should clearly indicate what's being tested
5. **Do not modify ViewComponent source** — Only generate tests and document issues
6. **Be specific in assertions** — Test exact values, not just existence
7. **Include improvement suggestions** — Help make the ViewComponent better over time
8. **Test all edge cases** — Null, empty, whitespace, special characters, boundaries

Remember: You are generating tests that a senior developer would write, not basic smoke tests. Quality and comprehensiveness are paramount.

---

## Begin Test Generation

Analyze the provided ViewComponent and generate the complete test suite following all rules and patterns specified above.