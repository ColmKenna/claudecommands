---
description:  TagHelper Test Generation Prompt
---
# TagHelper Test Generation Prompt

## Mission

You are a senior ASP.NET Core test engineer specializing in TagHelper testing. Your task is to generate a comprehensive, professional-grade unit test suite for the provided TagHelper that achieves 85%+ code coverage while following ASP.NET Core testing best practices.

## Critical Constraints

1. **DO NOT MODIFY THE TAGHELPER SOURCE CODE** - You are only generating tests
2. If tests would fail due to issues in the TagHelper implementation, document these issues separately
3. Follow the exact patterns and practices demonstrated in ASP.NET Core's own TagHelper test suites
4. Generate self-documenting, production-ready test code

## Input Context

You will have access to:
- The TagHelper source code to be tested (provided in context)
- All project dependencies and related source files
- Existing test patterns in the project
- Framework types: `TagHelperContext`, `TagHelperOutput`, `IHtmlGenerator`, etc.

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
- Proposed enhancements to the TagHelper
- Commented-out test code for each improvement
- Rationale for each suggestion

---

## Test Generation Rules

### A. Test Suite Structure

```csharp
using Microsoft.AspNetCore.Razor.TagHelpers;
using Microsoft.AspNetCore.Mvc.ViewFeatures;
using Microsoft.AspNetCore.Mvc.Rendering;
using Moq;
using Xunit;

namespace YourNamespace.Tests;

public class YourTagHelperTests
{
    // Helper method section
    private YourTagHelper CreateTagHelper(/* parameters */)
    {
        // Reusable instantiation logic
    }

    private static TagHelperContext CreateContext(/* parameters */)
    {
        // Reusable context creation
    }

    private static TagHelperOutput CreateOutput(/* parameters */)
    {
        // Reusable output creation
    }

    // Test methods follow...
}
```

### B. Test Naming Convention

**REQUIRED FORMAT:** `MethodName_Condition_ExpectedOutcome`

**Examples:**
- `Process_WithValidName_SetsNameAttribute`
- `ProcessAsync_WhenForIsEmpty_ThrowsArgumentException`
- `Process_WithCheckboxType_GeneratesHiddenInput`
- `ProcessAsync_InEndOfFormMode_AddsToFormContext`

**NEVER USE:**
- Generic names like `Test1`, `BasicTest`, `ValidInput`
- Underscores without the three-part structure
- CamelCase without underscores

### C. Test Structure Pattern

Every test MUST follow this structure:

```csharp
[Fact]
public async Task MethodName_Condition_ExpectedOutcome()
{
    // Arrange - Setup dependencies, TagHelper, Context, Output
    var metadataProvider = new EmptyModelMetadataProvider();
    var htmlGenerator = new TestableHtmlGenerator(metadataProvider);
    var tagHelper = new YourTagHelper(htmlGenerator)
    {
        PropertyName = value
    };
    
    var context = new TagHelperContext(
        tagName: "element",
        allAttributes: new TagHelperAttributeList { { "name", "value" } },
        items: new Dictionary<object, object>(),
        uniqueId: "test");
    
    var output = new TagHelperOutput(
        tagName: "element",
        attributes: new TagHelperAttributeList(),
        getChildContentAsync: (useCached, encoder) => 
            Task.FromResult<TagHelperContent>(new DefaultTagHelperContent()))
    {
        TagMode = TagMode.StartTagAndEndTag
    };

    // Act - Execute the TagHelper
    await tagHelper.ProcessAsync(context, output);

    // Assert - Verify output mutations
    Assert.Equal("expected", output.TagName);
    Assert.Equal("value", output.Attributes["name"].Value);
}
```

### D. Comprehensive Test Coverage Requirements

Generate tests for ALL of the following categories:

#### 1. Happy Path Tests (Basic Functionality)
- Test with minimal valid input
- Test with all properties populated
- Test with typical real-world scenarios
- One test per major code path through Process/ProcessAsync

**Example:**
```csharp
[Fact]
public async Task Process_WithValidInput_GeneratesExpectedOutput()
{
    // Standard case with all required properties set correctly
}
```

#### 2. Property Combination Tests
- Test each public property individually
- Test important property combinations
- Test property precedence (when one overrides another)

**Example:**
```csharp
[Fact]
public void Process_WithNameAndFor_NameTakesPrecedence()
{
    // When both Name and For.Name are set, Name wins
}
```

#### 3. Edge Case Tests
For each input property, test:
- `null` values
- Empty strings (`""`)
- Whitespace-only strings
- Very long strings
- Special characters
- Boundary values for numeric properties

**Example:**
```csharp
[Fact]
public void Process_WithNullName_UsesForName()
{
    // Fallback behavior
}

[Fact]
public void Process_WithEmptyName_ThrowsArgumentException()
{
    // Or whatever the correct behavior is
}
```

#### 4. Conditional Rendering Tests
Identify all conditional branches in Process/ProcessAsync:
- `if` statements that affect output
- Different rendering based on property values
- ViewContext settings that change behavior

**Example:**
```csharp
[Fact]
public async Task ProcessAsync_WhenCheckboxInlineMode_AddsHiddenInputToPostElement()
{
    // ViewContext.CheckBoxHiddenInputRenderMode = CheckBoxHiddenInputRenderMode.Inline
}

[Fact]
public async Task ProcessAsync_WhenCheckboxEndOfFormMode_AddsHiddenInputToFormContext()
{
    // ViewContext.CheckBoxHiddenInputRenderMode = CheckBoxHiddenInputRenderMode.EndOfForm
}
```

#### 5. State Management Tests (context.Items)
If TagHelper reads or writes to `context.Items`:
- Test with empty Items dictionary
- Test with required items present
- Test with unexpected items
- Test with null values in Items

**Example:**
```csharp
[Fact]
public void Process_WithoutFormContext_ThrowsInvalidOperationException()
{
    // Missing required parent state
}

[Fact]
public void Process_WithFormContext_AddsStateToFormContext()
{
    // Validates state sharing between TagHelpers
}
```

#### 6. Interaction Tests with Mocks
For TagHelpers using `IHtmlGenerator`, `IUrlHelper`, etc.:
- Mock the interface
- Verify methods called with correct parameters
- Test different return values from mocked methods

**Example:**
```csharp
[Fact]
public async Task ProcessAsync_CallsGenerateCheckBox_WithExpectedParameters()
{
    var htmlGenerator = new Mock<IHtmlGenerator>(MockBehavior.Strict);
    var tagBuilder = new TagBuilder("input") { TagRenderMode = TagRenderMode.SelfClosing };
    
    htmlGenerator
        .Setup(g => g.GenerateCheckBox(
            It.IsAny<ViewContext>(),
            It.IsAny<ModelExplorer>(),
            "expectedName",
            null,
            It.IsAny<object>()))
        .Returns(tagBuilder)
        .Verifiable();
    
    var tagHelper = new InputTagHelper(htmlGenerator.Object)
    {
        Name = "expectedName",
        InputTypeName = "checkbox"
    };
    
    await tagHelper.ProcessAsync(context, output);
    
    htmlGenerator.Verify();
}
```

#### 7. Validation and Error Tests
- Test invalid input combinations
- Test exceptions with `Assert.Throws<T>`
- Test error messages contain expected information

**Example:**
```csharp
[Fact]
public void Process_WithInvalidConfiguration_ThrowsInvalidOperationException()
{
    var exception = Assert.Throws<InvalidOperationException>(() =>
    {
        tagHelper.Process(context, output);
    });
    
    Assert.Contains("expected error message fragment", exception.Message);
}
```

#### 8. Output Buffer Tests
Test ALL output buffers when TagHelper writes to them:
- `output.TagName`
- `output.TagMode`
- `output.Attributes`
- `output.PreContent`
- `output.Content`
- `output.PostContent`
- `output.PreElement`
- `output.PostElement`

**Example:**
```csharp
[Fact]
public async Task ProcessAsync_WithCheckbox_WritesToPostElement()
{
    await tagHelper.ProcessAsync(context, output);
    
    var postElementContent = output.PostElement.GetContent();
    Assert.Contains("<input type=\"hidden\"", postElementContent);
}
```

#### 9. Theory-Based Tests (Data-Driven)
Use `[Theory]` with `[InlineData]` for testing multiple similar scenarios:

**Example:**
```csharp
[Theory]
[InlineData("text", "text")]
[InlineData("password", "password")]
[InlineData("email", "email")]
[InlineData("number", "number")]
public void Process_WithInputType_SetsTypeAttribute(string inputType, string expectedType)
{
    tagHelper.InputTypeName = inputType;
    
    tagHelper.Process(context, output);
    
    Assert.Equal(expectedType, output.Attributes["type"].Value);
}
```

### E. Dependency Handling Rules

#### When to Use Real Implementations
Use real framework types for:
- `TagHelperContext`
- `TagHelperOutput`
- `ViewContext`
- `ModelMetadataProvider` (use `EmptyModelMetadataProvider` or `TestModelMetadataProvider`)
- `ModelExpression`

**Example:**
```csharp
var metadataProvider = new EmptyModelMetadataProvider();
var htmlGenerator = new TestableHtmlGenerator(metadataProvider);
var viewContext = TestableHtmlGenerator.GetViewContext(
    model: false,
    htmlGenerator,
    metadataProvider);
```

#### When to Use Mocks
Mock these interfaces using Moq:
- `IHtmlGenerator` (when testing interaction, not output)
- `IUrlHelper`
- `IUrlHelperFactory`
- Custom service interfaces

**Moq Pattern:**
```csharp
var mockGenerator = new Mock<IHtmlGenerator>(MockBehavior.Strict);
mockGenerator
    .Setup(g => g.MethodName(It.IsAny<ViewContext>(), /* parameters */))
    .Returns(new TagBuilder("tag"))
    .Verifiable();

// After test execution
mockGenerator.Verify();
mockGenerator.VerifyNoOtherCalls(); // Optional: ensure no unexpected interactions
```

#### NEVER Mock
- `TagHelperContext`
- `TagHelperOutput`
- `ViewContext`
- Simple data structures

### F. ModelExpression Setup Patterns

Many TagHelpers use `For` property of type `ModelExpression`. Use these patterns:

#### Pattern 1: Simple Type
```csharp
var metadataProvider = new EmptyModelMetadataProvider();
var modelExplorer = metadataProvider.GetModelExplorerForType(typeof(string), "value");
var modelExpression = new ModelExpression("PropertyName", modelExplorer);

tagHelper.For = modelExpression;
```

#### Pattern 2: Complex Model Property
```csharp
var metadataProvider = new TestModelMetadataProvider();
metadataProvider.ForProperty<Model>(nameof(Model.PropertyName))
    .DisplayDetails(dd => dd.DisplayName = "Display Name");

var modelExplorer = metadataProvider
    .GetModelExplorerForType(typeof(Model), new Model())
    .GetExplorerForProperty(nameof(Model.PropertyName));

var modelExpression = new ModelExpression(nameof(Model.PropertyName), modelExplorer);
```

#### Pattern 3: Empty/Null For.Name
```csharp
var modelExplorer = metadataProvider.GetModelExplorerForType(typeof(bool), false);
var modelExpression = new ModelExpression(string.Empty, modelExplorer);

tagHelper.For = modelExpression;
```

### G. ViewContext Configuration

Configure ViewContext for different scenarios:

#### Basic ViewContext
```csharp
var viewContext = TestableHtmlGenerator.GetViewContext(
    model: new Model(),
    htmlGenerator,
    metadataProvider);

tagHelper.ViewContext = viewContext;
```

#### With FormContext
```csharp
viewContext.FormContext = new FormContext
{
    CanRenderAtEndOfForm = true
};
```

#### With CheckBox Hidden Input Mode
```csharp
viewContext.CheckBoxHiddenInputRenderMode = CheckBoxHiddenInputRenderMode.Inline;
// or
viewContext.CheckBoxHiddenInputRenderMode = CheckBoxHiddenInputRenderMode.EndOfForm;
```

#### With Client Validation
```csharp
viewContext.ClientValidationEnabled = true;
viewContext.Html5DateRenderingMode = Html5DateRenderingMode.Rfc3339;
```

### H. Assertion Best Practices

#### 1. Be Specific
```csharp
// GOOD
Assert.Equal("input", output.TagName);
Assert.Equal("text", output.Attributes["type"].Value);
Assert.Equal(TagMode.SelfClosing, output.TagMode);

// BAD
Assert.NotNull(output.TagName);
Assert.NotEmpty(output.Attributes);
```

#### 2. Test Content, Not Just Existence
```csharp
// GOOD
var content = output.PostElement.GetContent();
Assert.Contains("<input type=\"hidden\" name=\"fieldName\"", content);
Assert.Contains("value=\"false\"", content);

// BAD
Assert.NotEmpty(output.PostElement.GetContent());
```

#### 3. Multiple Assertions Per Test (When Logical)
```csharp
[Fact]
public void Process_WithCompleteInput_GeneratesFullOutput()
{
    tagHelper.Process(context, output);
    
    Assert.Equal("input", output.TagName);
    Assert.Equal(TagMode.SelfClosing, output.TagMode);
    Assert.Equal("text", output.Attributes["type"].Value);
    Assert.Equal("fieldName", output.Attributes["name"].Value);
    Assert.Equal("myValue", output.Attributes["value"].Value);
}
```

#### 4. Test Exception Messages
```csharp
var exception = Assert.Throws<ArgumentNullException>(() =>
{
    tagHelper.Process(context, output);
});

Assert.Equal("For", exception.ParamName);
```

### I. Helper Method Patterns

Create reusable helper methods to reduce duplication:

```csharp
private YourTagHelper CreateTagHelper(
    IHtmlGenerator htmlGenerator = null,
    ModelExpression forExpression = null,
    string name = null)
{
    htmlGenerator ??= CreateHtmlGenerator();
    
    return new YourTagHelper(htmlGenerator)
    {
        For = forExpression,
        Name = name,
        ViewContext = CreateViewContext()
    };
}

private IHtmlGenerator CreateHtmlGenerator()
{
    var metadataProvider = new EmptyModelMetadataProvider();
    return new TestableHtmlGenerator(metadataProvider);
}

private ViewContext CreateViewContext()
{
    var metadataProvider = new EmptyModelMetadataProvider();
    var htmlGenerator = new TestableHtmlGenerator(metadataProvider);
    return TestableHtmlGenerator.GetViewContext(false, htmlGenerator, metadataProvider);
}

private static TagHelperContext CreateContext(
    string tagName = "input",
    TagHelperAttributeList attributes = null)
{
    return new TagHelperContext(
        tagName: tagName,
        allAttributes: attributes ?? new TagHelperAttributeList(),
        items: new Dictionary<object, object>(),
        uniqueId: "test");
}

private static TagHelperOutput CreateOutput(
    string tagName = "input",
    TagMode tagMode = TagMode.SelfClosing)
{
    return new TagHelperOutput(
        tagName: tagName,
        attributes: new TagHelperAttributeList(),
        getChildContentAsync: (useCached, encoder) => 
            Task.FromResult<TagHelperContent>(new DefaultTagHelperContent()))
    {
        TagMode = tagMode
    };
}
```

---

## Generation Workflow

Follow this systematic approach:

### Step 1: Analyze TagHelper Source Code

Examine the TagHelper and document:

1. **Public Properties:**
   - List all settable properties
   - Note required vs. optional properties
   - Identify property dependencies or precedence rules

2. **Process/ProcessAsync Logic:**
   - Map all conditional branches
   - Identify state reads from `context.Items`
   - Document state writes to `context.Items` or `ViewContext.FormContext`
   - Note all output buffer modifications

3. **Dependencies:**
   - List constructor parameters
   - Identify injected services (IHtmlGenerator, IUrlHelper, etc.)
   - Note ViewContext usage patterns

4. **Edge Cases:**
   - Null handling for each property
   - Empty string handling
   - Boundary conditions
   - Error conditions and exceptions thrown

### Step 2: Plan Test Coverage

Create a coverage map:

```markdown
## Test Coverage Plan

### Properties to Test:
- [ ] PropertyName1 (null, empty, valid values)
- [ ] PropertyName2 (boundary conditions)
- [ ] PropertyName3 (interaction with PropertyName1)

### Code Paths:
- [ ] Path 1: When condition A is true
- [ ] Path 2: When condition B is true
- [ ] Path 3: Default path

### Interactions:
- [ ] IHtmlGenerator.GenerateMethod called correctly
- [ ] ViewContext.FormContext modified as expected

### Edge Cases:
- [ ] Null For property
- [ ] Empty Name
- [ ] Invalid ViewContext state
```

### Step 3: Generate Tests Systematically

Generate tests in this order:

1. **Helper methods** (CreateTagHelper, CreateContext, CreateOutput)
2. **Happy path tests** (1-3 tests for basic functionality)
3. **Property tests** (one test per property, plus combinations)
4. **Edge case tests** (null, empty, boundaries)
5. **Conditional rendering tests** (one per branch)
6. **State management tests** (context.Items scenarios)
7. **Interaction tests** (mocked dependencies)
8. **Error tests** (exceptions and validation)

### Step 4: Verify Coverage

After generating tests, verify:

1. **Line Coverage:**
   - Count lines in Process/ProcessAsync
   - Count lines exercised by tests
   - Calculate percentage
   - Identify untested lines

2. **Branch Coverage:**
   - Count conditional branches
   - Verify each branch has a test
   - Document uncovered branches

3. **Property Coverage:**
   - Ensure each property tested with valid input
   - Ensure each property tested with edge cases
   - Verify property combinations tested

4. **Completeness:**
   - Run through self-verification checklist (provided below)
   - Identify any missing test categories
   - Add missing tests or document why they're not possible

### Step 5: Document Issues and Suggestions

#### Document Source Code Issues

For each issue detected:
```markdown
### Issue: [Brief Description]
**Location:** ClassName.cs, line X, MethodName
**Problem:** Detailed description of the issue
**Impact:** Why this prevents testing or causes test failures
**Suggested Fix:** How to correct the issue in source code

**Failed Test Example:**
```csharp
[Fact]
public void TestName_Condition_Outcome()
{
    // This test would fail because...
}
```
```

#### Suggest Improvements

For each improvement:
```markdown
### Improvement: [Brief Description]
**Rationale:** Why this would improve the TagHelper
**Current Behavior:** What happens now
**Proposed Behavior:** What should happen

**Test for Future Implementation:**
```csharp
// [Fact]
// public void TestName_Condition_Outcome()
// {
//     // Test code for the improvement
//     // Uncomment when improvement is implemented
// }
```
```

---

## Self-Verification Checklist

After generating the test suite, verify against these criteria:

### Functional Coverage
- [ ] At least one happy path test with valid input
- [ ] Edge case tests for each configurable property
- [ ] All conditional rendering paths covered
- [ ] Error/validation tests for expected exceptions
- [ ] State management via context.Items tested (if applicable)
- [ ] Estimated line coverage ≥85%
- [ ] Estimated branch coverage ≥80%

### Test Structure & Quality
- [ ] All test names follow `Method_Condition_Outcome` format
- [ ] Clear Arrange-Act-Assert structure in every test
- [ ] Single responsibility per test method
- [ ] Tests are independent (no ordering dependencies)
- [ ] Meaningful variable names; no magic strings
- [ ] Helper methods reduce duplication

### Mocking & Dependencies
- [ ] External dependencies (IHtmlGenerator) mocked or real, consistently
- [ ] Explicit Setup() for each mock interaction
- [ ] Assertions target output behavior, not just mock calls
- [ ] Real implementations used for TagHelperContext/Output
- [ ] ViewContext properly configured for each test scenario

### Assertions & Verification
- [ ] 2-5 assertions per test (focused but thorough)
- [ ] Verify specific attribute values, not just existence
- [ ] Exception tests use Assert.Throws with message validation
- [ ] HTML content checks are specific (not just "contains some text")
- [ ] All output buffers tested when TagHelper writes to them

### Documentation & Maintainability
- [ ] Complex setup explained if needed
- [ ] Non-obvious assertions have inline comments
- [ ] Tests serve as usage documentation
- [ ] Helper methods have clear parameter names

### Coverage Metrics
- [ ] ≥85% of TagHelper public behavior exercised
- [ ] All public properties tested with valid and invalid input
- [ ] All conditional branches have corresponding tests
- [ ] Domain-specific edge cases covered

### Code Quality
- [ ] No commented-out code (except improvement suggestions)
- [ ] No redundant tests (each test adds value)
- [ ] Consistent formatting and style
- [ ] Proper using statements and namespace

### Expected Outcomes
- [ ] All tests should compile without errors
- [ ] All tests should pass when TagHelper is correctly implemented
- [ ] Tests should fail appropriately when TagHelper has bugs
- [ ] Test failures should clearly indicate what's broken

---

## Common Patterns Reference

### Pattern: Testing Checkbox with Hidden Input

```csharp
[Fact]
public async Task ProcessAsync_WithCheckboxInline_AddsHiddenToPostElement()
{
    var metadataProvider = new EmptyModelMetadataProvider();
    var htmlGenerator = new TestableHtmlGenerator(metadataProvider);
    var viewContext = TestableHtmlGenerator.GetViewContext(false, htmlGenerator, metadataProvider);
    viewContext.FormContext.CanRenderAtEndOfForm = true;
    viewContext.CheckBoxHiddenInputRenderMode = CheckBoxHiddenInputRenderMode.Inline;
    
    var tagHelper = new InputTagHelper(htmlGenerator)
    {
        Name = "field",
        InputTypeName = "checkbox",
        ViewContext = viewContext
    };
    
    var context = new TagHelperContext(
        tagName: "input",
        allAttributes: new TagHelperAttributeList { { "name", "field" }, { "type", "checkbox" } },
        items: new Dictionary<object, object>(),
        uniqueId: "test");
    
    var output = new TagHelperOutput(
        tagName: "input",
        attributes: new TagHelperAttributeList(),
        getChildContentAsync: (cached, enc) => Task.FromResult<TagHelperContent>(new DefaultTagHelperContent()))
    {
        TagMode = TagMode.SelfClosing
    };
    
    await tagHelper.ProcessAsync(context, output);
    
    Assert.NotEmpty(output.PostElement.GetContent());
    Assert.Contains("type=\"hidden\"", output.PostElement.GetContent());
}
```

### Pattern: Testing Form Tag with Antiforgery

```csharp
[Fact]
public async Task ProcessAsync_WithPost_GeneratesAntiforgeryToken()
{
    var viewContext = CreateViewContext();
    var context = new TagHelperContext(
        tagName: "form",
        allAttributes: new TagHelperAttributeList(),
        items: new Dictionary<object, object>(),
        uniqueId: "test");
    
    var output = new TagHelperOutput(
        tagName: "form",
        attributes: new TagHelperAttributeList(),
        getChildContentAsync: (cached, enc) => Task.FromResult<TagHelperContent>(new DefaultTagHelperContent()));
    
    var htmlGenerator = new Mock<IHtmlGenerator>();
    htmlGenerator
        .Setup(g => g.GenerateAntiforgery(It.IsAny<ViewContext>()))
        .Returns(new TagBuilder("input"));
    
    var tagHelper = new FormTagHelper(htmlGenerator.Object)
    {
        Method = "post",
        ViewContext = viewContext
    };
    
    await tagHelper.ProcessAsync(context, output);
    
    Assert.NotEmpty(output.PostContent.GetContent());
}
```

### Pattern: Testing with ModelExpression

```csharp
[Fact]
public void Process_WithFor_UsesModelPropertyName()
{
    var metadataProvider = new EmptyModelMetadataProvider();
    var htmlGenerator = new TestableHtmlGenerator(metadataProvider);
    var modelExplorer = metadataProvider.GetModelExplorerForType(typeof(string), "value");
    var modelExpression = new ModelExpression("PropertyName", modelExplorer);
    
    var tagHelper = new InputTagHelper(htmlGenerator)
    {
        For = modelExpression,
        ViewContext = TestableHtmlGenerator.GetViewContext(false, htmlGenerator, metadataProvider)
    };
    
    var context = CreateContext();
    var output = CreateOutput();
    
    tagHelper.Process(context, output);
    
    Assert.Equal("PropertyName", output.Attributes["name"].Value);
}
```

### Pattern: Testing Context.Items State Management

```csharp
[Fact]
public void Process_WithParentFormContext_UsesFormData()
{
    var tagHelper = CreateTagHelper();
    var formContext = new FormContext();
    
    var context = new TagHelperContext(
        tagName: "input",
        allAttributes: new TagHelperAttributeList(),
        items: new Dictionary<object, object>
        {
            { typeof(FormContext), formContext }
        },
        uniqueId: "test");
    
    var output = CreateOutput();
    
    tagHelper.Process(context, output);
    
    // Verify TagHelper used or modified form context
    Assert.NotNull(formContext.EndOfFormContent);
}
```

---

## Output Format

Structure your complete response as follows:

```markdown
# Test Suite for [TagHelperName]

## Section 1: GENERATED TEST SUITE

```csharp
using Microsoft.AspNetCore.Razor.TagHelpers;
using Microsoft.AspNetCore.Mvc.ViewFeatures;
using Microsoft.AspNetCore.Mvc.Rendering;
using Moq;
using Xunit;

namespace YourNamespace.Tests;

public class YourTagHelperTests
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

## Final Reminders

1. **Generate comprehensive, production-ready code** - These tests should be immediately usable
2. **Follow ASP.NET Core conventions exactly** - Study the patterns in the reference document
3. **Aim for 85%+ coverage** - Be thorough but avoid redundant tests
4. **Self-document through naming** - Test names should clearly indicate what's being tested
5. **Do not modify TagHelper source** - Only generate tests and document issues
6. **Be specific in assertions** - Test exact values, not just existence
7. **Include improvement suggestions** - Help make the TagHelper better over time

Remember: You are generating tests that a senior developer would write, not basic smoke tests. Quality and comprehensiveness are paramount.

---

## Begin Test Generation

Analyze the provided TagHelper and generate the complete test suite following all rules and patterns specified above.