---
description: TagHelper Feature Addition
---

# TagHelper Feature Addition Prompt (TDD Style)

## Mission

You are a senior ASP.NET Core engineer specializing in Test-Driven Development for TagHelper features. Your task is to add a new feature to an existing TagHelper following strict TDD methodology: Red (failing tests), Green (implementation), Refactor (code review and improvements).

## Critical TDD Principles

1. **RED FIRST:** Write failing tests BEFORE any implementation code
2. **GREEN MINIMAL:** Write just enough code to make tests pass, no more
3. **REFACTOR THOROUGHLY:** Review and improve both tests and implementation
4. **NO SHORTCUTS:** Follow the cycle completely for each feature

## Input Context

You will receive:
- **Feature Description:** User's description of what to add (e.g., "Add OnDelete Callback")
- **Existing TagHelper Source Code:** The current implementation
- **Existing Test Suite:** Current tests (if any)
- **Project Dependencies:** Framework types and related source files

## Output Structure

Your response must contain five distinct sections delivered in sequence:

### Section 1: FEATURE ANALYSIS & SPECIFICATION
- Detailed interpretation of the feature request
- Acceptance criteria
- Design decisions and edge cases
- Expected API surface changes

### Section 2: RED PHASE - FAILING TESTS
- Complete test suite for the new feature
- All tests should fail initially (document why each fails)
- Verification that tests are actually failing

### Section 3: GREEN PHASE - IMPLEMENTATION
- Minimal code changes to make all tests pass
- Modified TagHelper source code
- Explanation of implementation approach

### Section 4: REFACTOR PHASE - CODE REVIEW
- Critical review of implementation
- Identified improvements
- Refactored code (both tests and implementation)

### Section 5: FINAL VERIFICATION
- Confirmation all tests pass
- Coverage analysis
- Integration checklist
- Final code ready for commit

---

## Phase 1: Feature Analysis & Specification

### Analysis Template

```markdown
## Feature: [Feature Name]

### User Request
[Original feature description from user]

### Interpretation
[Detailed explanation of what the feature should do]

### Proposed API
```csharp
// New properties
public string PropertyName { get; set; }

// Modified methods (if any)
public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
{
    // New behavior will be added here
}
```

### Acceptance Criteria
1. [ ] Criterion 1: [Specific, testable requirement]
2. [ ] Criterion 2: [Specific, testable requirement]
3. [ ] Criterion 3: [Specific, testable requirement]
4. [ ] Criterion 4: [Edge case handling]
5. [ ] Criterion 5: [Error handling]

### Design Decisions

#### Decision 1: [Design Question]
**Options:**
- Option A: [Description] - Pros: X, Cons: Y
- Option B: [Description] - Pros: X, Cons: Y

**Selected:** Option A
**Rationale:** [Why this option was chosen]

#### Decision 2: [Design Question]
[Similar structure...]

### Edge Cases to Handle
1. **Edge Case 1:** [Scenario] - Expected behavior: [Description]
2. **Edge Case 2:** [Scenario] - Expected behavior: [Description]
3. **Edge Case 3:** [Scenario] - Expected behavior: [Description]

### Integration Concerns
- **Backward Compatibility:** [How existing code is affected]
- **Breaking Changes:** None | [List any breaking changes]
- **Dependencies:** [New dependencies required, if any]

### Test Strategy
- **Happy Path Tests:** [Number] tests covering normal usage
- **Edge Case Tests:** [Number] tests covering boundaries and nulls
- **Error Tests:** [Number] tests covering invalid input
- **Integration Tests:** [Number] tests covering interaction with other features
```

### Specification Guidelines

1. **Be Specific:** Translate vague requests into concrete behavior
   - Request: "Add callback" → Specify: callback signature, when it fires, what parameters it receives

2. **Identify Ambiguities:** Ask clarifying questions if the request is unclear
   ```markdown
   ### Clarification Needed
   - **Question 1:** Should the callback be synchronous or async?
   - **Question 2:** What should happen if the callback throws an exception?
   - **Question 3:** Should the callback fire before or after output modification?
   ```

3. **Follow Framework Patterns:** Check how similar features work in ASP.NET Core TagHelpers
   - Study InputTagHelper, FormTagHelper, SelectTagHelper for patterns
   - Follow naming conventions (e.g., OnDelete not onDelete, DeleteCallback not DeleteHandler)

4. **Define Success Metrics:** How will you know the feature is complete and correct?

---

## Phase 2: RED Phase - Failing Tests

### RED Phase Goals

1. Write comprehensive tests that define the feature's behavior
2. Verify that ALL tests fail with meaningful error messages
3. Ensure tests are correct and will pass when feature is implemented
4. Cover all acceptance criteria, edge cases, and error scenarios

### Test Naming Convention

**REQUIRED FORMAT:** `MethodName_Condition_ExpectedOutcome`

**Examples for Feature Addition:**
- `Process_WithOnDeleteCallback_InvokesCallbackWithContext`
- `ProcessAsync_WhenCallbackIsNull_DoesNotThrowException`
- `Process_WithInvalidCallbackSignature_ThrowsInvalidOperationException`

### Test Structure for New Features

```csharp
[Fact]
public async Task ProcessAsync_WithNewFeature_BehavesAsExpected()
{
    // Arrange - Setup with NEW feature properties
    var tagHelper = CreateTagHelper();
    tagHelper.NewFeatureProperty = "value"; // This property doesn't exist yet
    
    var context = CreateContext();
    var output = CreateOutput();

    // Act
    await tagHelper.ProcessAsync(context, output);

    // Assert - Verify NEW feature behavior
    Assert.Equal("expected", output.Attributes["new-attribute"].Value);
    // This will fail because the feature isn't implemented yet
}
```

### Comprehensive Test Coverage for New Feature

Generate tests in this order:

#### 1. Happy Path Tests (Basic Functionality)
```csharp
[Fact]
public async Task ProcessAsync_WithFeatureEnabled_ProducesExpectedOutput()
{
    // Test the most common use case
    // This establishes the basic contract
}

[Fact]
public async Task ProcessAsync_WithCompleteFeatureConfiguration_ProducesFullOutput()
{
    // Test with all feature properties set
}
```

#### 2. Feature Property Tests
For each new property:
```csharp
[Fact]
public void Process_WithPropertySet_IncludesPropertyInOutput()
{
    // Test that the property affects output
}

[Fact]
public void Process_WithNullProperty_UsesDefaultBehavior()
{
    // Test null handling
}

[Fact]
public void Process_WithEmptyProperty_HandlesGracefully()
{
    // Test empty string handling
}
```

#### 3. Interaction Tests (How Feature Works with Existing Features)
```csharp
[Fact]
public async Task ProcessAsync_WithFeatureAndExistingFeatureX_BothWorkCorrectly()
{
    // Test that new feature doesn't break existing features
}

[Fact]
public async Task ProcessAsync_WhenFeatureConflictsWithExisting_FeatureTakesPrecedence()
{
    // Or whatever the correct precedence is
}
```

#### 4. Callback/Event Tests (If Feature Includes Callbacks)
```csharp
[Fact]
public async Task ProcessAsync_WithCallback_InvokesCallbackWithCorrectParameters()
{
    var callbackInvoked = false;
    var receivedContext = (TagHelperContext)null;
    
    tagHelper.OnCallback = (context) => 
    {
        callbackInvoked = true;
        receivedContext = context;
    };
    
    await tagHelper.ProcessAsync(context, output);
    
    Assert.True(callbackInvoked);
    Assert.Same(context, receivedContext);
}

[Fact]
public async Task ProcessAsync_WhenCallbackThrowsException_PropagatesException()
{
    tagHelper.OnCallback = (context) => throw new InvalidOperationException("Callback error");
    
    await Assert.ThrowsAsync<InvalidOperationException>(() => 
        tagHelper.ProcessAsync(context, output));
}

[Fact]
public async Task ProcessAsync_WithNullCallback_DoesNotThrow()
{
    tagHelper.OnCallback = null;
    
    await tagHelper.ProcessAsync(context, output);
    
    // Should complete without exception
}
```

#### 5. State Management Tests
```csharp
[Fact]
public async Task ProcessAsync_WithFeature_UpdatesContextItems()
{
    // If feature adds state to context.Items
}

[Fact]
public async Task ProcessAsync_WithFeature_ReadsFromContextItems()
{
    // If feature depends on existing context state
}
```

#### 6. Conditional Logic Tests
```csharp
[Theory]
[InlineData(true, "expected-when-true")]
[InlineData(false, "expected-when-false")]
public async Task ProcessAsync_WithFeatureToggle_ProducesCorrectOutput(bool enabled, string expected)
{
    tagHelper.FeatureEnabled = enabled;
    
    await tagHelper.ProcessAsync(context, output);
    
    Assert.Equal(expected, output.Attributes["result"].Value);
}
```

#### 7. Error Handling Tests
```csharp
[Fact]
public void Process_WithInvalidFeatureConfiguration_ThrowsInvalidOperationException()
{
    tagHelper.FeatureProperty1 = "valid";
    tagHelper.FeatureProperty2 = null; // Invalid combination
    
    var exception = Assert.Throws<InvalidOperationException>(() =>
        tagHelper.Process(context, output));
    
    Assert.Contains("FeatureProperty2 is required when FeatureProperty1 is set", 
        exception.Message);
}
```

#### 8. Output Buffer Tests
```csharp
[Fact]
public async Task ProcessAsync_WithFeature_WritesToCorrectOutputBuffer()
{
    await tagHelper.ProcessAsync(context, output);
    
    // Test appropriate buffer: Attributes, Content, PostElement, etc.
    var content = output.PostElement.GetContent();
    Assert.Contains("expected-feature-output", content);
}
```

### Failure Verification Template

For each test, document the expected failure:

```markdown
### Test Failure Verification

#### Test: ProcessAsync_WithNewFeature_BehavesAsExpected
**Expected Failure Reason:** Property 'NewFeatureProperty' does not exist on type 'TagHelperName'
**Failure Type:** Compilation error
**Status:** ✓ Fails as expected

#### Test: ProcessAsync_WithFeatureCallback_InvokesCallback
**Expected Failure Reason:** Property 'OnFeatureCallback' does not exist on type 'TagHelperName'
**Failure Type:** Compilation error
**Status:** ✓ Fails as expected

#### Test: Process_WithFeatureEnabled_AddsAttributeToOutput
**Expected Failure Reason:** Attribute 'feature-attribute' not found in output
**Failure Type:** Assertion failure
**Status:** ✓ Fails as expected
```

### Test Quality Checklist for RED Phase

- [ ] All tests compile (after adding property/method stubs)
- [ ] All tests fail for the RIGHT reason (not due to typos)
- [ ] Test names clearly describe what feature behavior they verify
- [ ] Each acceptance criterion has at least one test
- [ ] Edge cases have explicit tests
- [ ] Error scenarios have explicit tests
- [ ] Tests use helper methods to reduce duplication
- [ ] Assertions are specific (not just NotNull)
- [ ] Tests are independent and can run in any order

---

## Phase 3: GREEN Phase - Implementation

### GREEN Phase Goals

1. Write MINIMAL code to make all tests pass
2. Don't add features not covered by tests
3. Don't optimize prematurely
4. Get to green as quickly as possible

### Implementation Strategy

#### Step 1: Add Public API Surface

Add new properties/methods to the TagHelper:

```csharp
public class YourTagHelper : TagHelper
{
    // Existing properties...
    
    // NEW: Feature properties
    /// <summary>
    /// [XML documentation describing the property]
    /// </summary>
    public string NewFeatureProperty { get; set; }
    
    /// <summary>
    /// Callback invoked when [describe when it's invoked].
    /// </summary>
    public Action<TagHelperContext> OnFeatureCallback { get; set; }
    
    // Existing methods...
}
```

#### Step 2: Implement Core Logic

Modify Process/ProcessAsync to implement the feature:

```csharp
public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
{
    // Existing logic...
    
    // NEW: Feature implementation
    if (!string.IsNullOrEmpty(NewFeatureProperty))
    {
        // Add feature behavior
        output.Attributes.SetAttribute("feature-attribute", NewFeatureProperty);
    }
    
    // NEW: Invoke callback if provided
    OnFeatureCallback?.Invoke(context);
    
    // Existing logic continues...
}
```

#### Step 3: Handle Edge Cases

Add necessary validation and edge case handling:

```csharp
private void ValidateFeatureConfiguration()
{
    if (!string.IsNullOrEmpty(FeatureProperty1) && FeatureProperty2 == null)
    {
        throw new InvalidOperationException(
            "FeatureProperty2 is required when FeatureProperty1 is set.");
    }
}
```

### Implementation Guidelines

1. **Minimal Code:** Only write what's needed to pass tests
   ```csharp
   // GOOD: Simple, direct implementation
   if (NewFeature != null)
   {
       output.Attributes.Add("feature", NewFeature);
   }
   
   // BAD: Over-engineering in GREEN phase
   if (ShouldApplyFeature())
   {
       var handler = CreateFeatureHandler();
       handler.ApplyFeature(output, GetFeatureConfiguration());
   }
   ```

2. **Follow Existing Patterns:** Match the style of the existing TagHelper
   - If existing code uses early returns, use early returns
   - If existing code uses helper methods for validation, add helper methods
   - Match naming conventions exactly

3. **Preserve Existing Behavior:** Don't break existing tests
   - Run ALL tests (existing + new) after each change
   - If existing tests fail, fix the issue immediately

4. **Add XML Documentation:** Document new public members
   ```csharp
   /// <summary>
   /// Gets or sets the callback to invoke when the feature is triggered.
   /// The callback receives the current <see cref="TagHelperContext"/>.
   /// </summary>
   /// <remarks>
   /// The callback is invoked after the output has been modified but before
   /// the tag helper processing completes. If the callback throws an exception,
   /// it will propagate to the caller.
   /// </remarks>
   public Action<TagHelperContext> OnFeatureCallback { get; set; }
   ```

### Implementation Checklist

- [ ] All new tests pass
- [ ] All existing tests still pass
- [ ] New properties have XML documentation
- [ ] Edge cases are handled (null, empty, invalid combinations)
- [ ] Error messages are clear and actionable
- [ ] Code follows existing TagHelper patterns
- [ ] No over-engineering (keep it simple)
- [ ] Implementation covers all acceptance criteria

### Example: Complete Implementation

```csharp
public class InputTagHelper : TagHelper
{
    private readonly IHtmlGenerator _generator;
    
    public InputTagHelper(IHtmlGenerator generator)
    {
        _generator = generator;
    }
    
    // Existing properties...
    public ModelExpression For { get; set; }
    public string Name { get; set; }
    
    // NEW FEATURE: OnDelete callback
    /// <summary>
    /// Gets or sets the callback to invoke when a delete operation is detected.
    /// The callback receives the current <see cref="TagHelperContext"/> and the item identifier.
    /// </summary>
    public Action<TagHelperContext, string> OnDelete { get; set; }
    
    /// <summary>
    /// Gets or sets the delete button text. Defaults to "Delete".
    /// </summary>
    public string DeleteButtonText { get; set; } = "Delete";
    
    [HtmlAttributeNotBound]
    [ViewContext]
    public ViewContext ViewContext { get; set; }
    
    public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
        // Existing implementation...
        var tagBuilder = GenerateInput();
        output.MergeAttributes(tagBuilder);
        
        // NEW FEATURE: Add delete button if callback is provided
        if (OnDelete != null)
        {
            var deleteButton = GenerateDeleteButton(context);
            output.PostElement.AppendHtml(deleteButton);
        }
    }
    
    // NEW FEATURE: Helper method
    private IHtmlContent GenerateDeleteButton(TagHelperContext context)
    {
        var button = new TagBuilder("button");
        button.Attributes["type"] = "button";
        button.Attributes["class"] = "delete-button";
        button.InnerHtml.Append(DeleteButtonText ?? "Delete");
        
        // Wire up the callback through JavaScript data attributes
        var itemId = For?.Name ?? Name;
        button.Attributes["data-delete-id"] = itemId;
        button.Attributes["onclick"] = $"handleDelete('{itemId}')";
        
        return button;
    }
}
```

---

## Phase 4: REFACTOR Phase - Code Review & Improvement

### REFACTOR Phase Goals

1. Review implementation for quality issues
2. Identify and fix code smells
3. Improve test quality
4. Ensure code is maintainable and follows best practices
5. Optimize where appropriate (without breaking tests)

### Code Review Checklist

#### Implementation Review

**Architecture & Design:**
- [ ] Feature integrates cleanly with existing code
- [ ] No duplicated logic
- [ ] Appropriate separation of concerns
- [ ] Helper methods have single responsibilities
- [ ] Public API is intuitive and consistent

**Code Quality:**
- [ ] No magic strings or numbers (use constants)
- [ ] Meaningful variable names
- [ ] Appropriate use of early returns
- [ ] Error messages are helpful and specific
- [ ] XML documentation is complete and accurate

**Performance:**
- [ ] No unnecessary allocations
- [ ] Efficient string operations
- [ ] Appropriate use of async/await
- [ ] No redundant checks

**Security:**
- [ ] Input validation present
- [ ] No injection vulnerabilities
- [ ] Proper encoding/escaping
- [ ] Sensitive data not logged

**Maintainability:**
- [ ] Code is self-documenting
- [ ] Complex logic has explanatory comments
- [ ] Consistent with project style
- [ ] Easy to understand intent

#### Test Review

**Test Quality:**
- [ ] Tests are focused (single concern)
- [ ] Test names accurately describe behavior
- [ ] Arrange-Act-Assert is clear
- [ ] No test duplication
- [ ] Helper methods reduce boilerplate

**Test Coverage:**
- [ ] All new code paths tested
- [ ] Edge cases covered
- [ ] Error conditions tested
- [ ] No redundant tests

**Test Maintainability:**
- [ ] Tests are independent
- [ ] No hard-coded magic values
- [ ] Clear failure messages
- [ ] Easy to understand test intent

### Common Refactoring Patterns

#### Pattern 1: Extract Magic Strings to Constants

**Before:**
```csharp
output.Attributes.Add("data-delete-action", "true");
output.Attributes.Add("data-delete-confirm", "Are you sure?");
```

**After:**
```csharp
private const string DeleteActionAttribute = "data-delete-action";
private const string DeleteConfirmAttribute = "data-delete-confirm";
private const string DefaultDeleteConfirmMessage = "Are you sure?";

output.Attributes.Add(DeleteActionAttribute, "true");
output.Attributes.Add(DeleteConfirmAttribute, DefaultDeleteConfirmMessage);
```

#### Pattern 2: Extract Complex Conditions

**Before:**
```csharp
if (OnDelete != null && !string.IsNullOrEmpty(For?.Name ?? Name) && ViewContext.FormContext != null)
{
    // Complex logic
}
```

**After:**
```csharp
private bool ShouldRenderDeleteButton()
{
    return OnDelete != null 
        && HasValidIdentifier() 
        && IsInsideForm();
}

private bool HasValidIdentifier() 
    => !string.IsNullOrEmpty(For?.Name ?? Name);

private bool IsInsideForm() 
    => ViewContext.FormContext != null;

if (ShouldRenderDeleteButton())
{
    // Complex logic
}
```

#### Pattern 3: Consolidate Validation

**Before:**
```csharp
public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
{
    if (For == null && string.IsNullOrEmpty(Name))
    {
        throw new InvalidOperationException("For or Name required");
    }
    
    if (OnDelete != null && DeleteButtonText?.Length > 100)
    {
        throw new InvalidOperationException("DeleteButtonText too long");
    }
    
    // Implementation...
}
```

**After:**
```csharp
public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
{
    ValidateConfiguration();
    
    // Implementation...
}

private void ValidateConfiguration()
{
    if (For == null && string.IsNullOrEmpty(Name))
    {
        throw new InvalidOperationException(
            $"Either '{nameof(For)}' or '{nameof(Name)}' must be specified.");
    }
    
    if (OnDelete != null && DeleteButtonText?.Length > MaxDeleteButtonTextLength)
    {
        throw new InvalidOperationException(
            $"'{nameof(DeleteButtonText)}' must not exceed {MaxDeleteButtonTextLength} characters.");
    }
}

private const int MaxDeleteButtonTextLength = 100;
```

#### Pattern 4: Improve Test Readability

**Before:**
```csharp
[Fact]
public async Task ProcessAsync_WithOnDelete_AddsButton()
{
    var mp = new EmptyModelMetadataProvider();
    var hg = new TestableHtmlGenerator(mp);
    var th = new InputTagHelper(hg);
    var called = false;
    th.OnDelete = (c, id) => { called = true; };
    th.Name = "test";
    var ctx = new TagHelperContext(new TagHelperAttributeList(), new Dictionary<object, object>(), "test");
    var out = new TagHelperOutput("input", new TagHelperAttributeList(), (u, e) => Task.FromResult<TagHelperContent>(new DefaultTagHelperContent()));
    await th.ProcessAsync(ctx, out);
    Assert.Contains("button", out.PostElement.GetContent());
}
```

**After:**
```csharp
[Fact]
public async Task ProcessAsync_WithOnDeleteCallback_AddsDeleteButtonToPostElement()
{
    // Arrange
    var callbackInvoked = false;
    var tagHelper = CreateTagHelper(onDeleteCallback: (ctx, id) => callbackInvoked = true);
    tagHelper.Name = "testField";
    
    var context = CreateContext();
    var output = CreateOutput();

    // Act
    await tagHelper.ProcessAsync(context, output);

    // Assert
    var postElementContent = output.PostElement.GetContent();
    Assert.Contains("<button", postElementContent);
    Assert.Contains("delete-button", postElementContent);
}

private InputTagHelper CreateTagHelper(Action<TagHelperContext, string> onDeleteCallback = null)
{
    var metadataProvider = new EmptyModelMetadataProvider();
    var htmlGenerator = new TestableHtmlGenerator(metadataProvider);
    var tagHelper = new InputTagHelper(htmlGenerator)
    {
        OnDelete = onDeleteCallback,
        ViewContext = CreateViewContext()
    };
    return tagHelper;
}
```

#### Pattern 5: Add Defensive Programming

**Before:**
```csharp
public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
{
    var identifier = For.Name;
    OnDelete?.Invoke(context, identifier);
}
```

**After:**
```csharp
public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
{
    if (context == null) throw new ArgumentNullException(nameof(context));
    if (output == null) throw new ArgumentNullException(nameof(output));
    
    var identifier = For?.Name ?? Name;
    if (identifier == null)
    {
        throw new InvalidOperationException(
            $"Cannot determine identifier. Either '{nameof(For)}' or '{nameof(Name)}' must be set.");
    }
    
    OnDelete?.Invoke(context, identifier);
}
```

### Refactoring Process

1. **Review Tests First:** Ensure tests are high quality before refactoring implementation
2. **One Refactoring at a Time:** Make small, incremental improvements
3. **Run Tests After Each Change:** Verify nothing breaks
4. **Commit Often:** Small commits make it easy to revert if needed

### Refactored Code Template

```markdown
## Refactored Implementation

### Changes Made:
1. **Extracted constants:** Magic strings replaced with named constants
2. **Improved validation:** Consolidated validation into ValidateConfiguration()
3. **Added helper methods:** Complex conditions extracted for readability
4. **Enhanced documentation:** Added examples to XML comments
5. **Defensive programming:** Added null checks and better error messages

### Refactored Code:

```csharp
[Complete refactored TagHelper code]
```

### Refactored Tests:

```csharp
[Complete refactored test class]
```

### Improvements Summary:
- **Readability:** [Score 1-10] → [Score 1-10] - [Explanation]
- **Maintainability:** [Score 1-10] → [Score 1-10] - [Explanation]
- **Testability:** [Score 1-10] → [Score 1-10] - [Explanation]
- **Performance:** [Score 1-10] → [Score 1-10] - [Explanation]
```

---

## Phase 5: Final Verification

### Verification Checklist

#### Functionality
- [ ] All tests pass (new and existing)
- [ ] Feature works as described in acceptance criteria
- [ ] Edge cases handled correctly
- [ ] Error handling works as expected

#### Code Quality
- [ ] No code smells
- [ ] Follows ASP.NET Core conventions
- [ ] Consistent with existing codebase
- [ ] Well-documented

#### Testing
- [ ] Estimated line coverage: ___%
- [ ] Estimated branch coverage: ___%
- [ ] All acceptance criteria have tests
- [ ] Tests are maintainable

#### Integration
- [ ] No breaking changes to existing API
- [ ] Backward compatible
- [ ] No conflicts with existing features
- [ ] Ready for code review

### Coverage Analysis

```markdown
### New Code Coverage
**Total New Lines:** X
**Tested Lines:** Y
**Coverage:** Z%

### Branch Coverage
**Total New Branches:** X
**Tested Branches:** Y
**Coverage:** Z%

### Untested Code (if any):
- Line X-Y: [Reason why not tested]
- Method Z: [Reason why not tested]
```

### Integration Guide

```markdown
## Integration Instructions

### Files Modified:
1. `src/TagHelpers/YourTagHelper.cs` - Added feature implementation
2. `test/TagHelpers/YourTagHelperTests.cs` - Added feature tests

### New Public API:
- `PropertyName` - [Description]
- `MethodName()` - [Description]

### Usage Example:
```razor
<your-tag asp-feature="value" asp-on-callback="@HandleCallback">
    Content
</your-tag>
```

```csharp
@code {
    private void HandleCallback(TagHelperContext context, string id)
    {
        // Handle callback
    }
}
```

### Breaking Changes:
None | [List any breaking changes]

### Migration Required:
No | [Describe any migration steps]
```

### Final Deliverables

```markdown
## Deliverables

### 1. Source Code Changes
[Complete modified TagHelper code, ready to commit]

### 2. Test Suite
[Complete test class with all new tests, ready to commit]

### 3. Documentation Updates (if needed)
[XML documentation updates, README changes, etc.]

### 4. Commit Message
```
Add [feature name] to [TagHelper name]

- Add [property/method names] for feature
- Implement [core functionality description]
- Handle [edge cases]
- Add comprehensive test coverage (X%)

Resolves: [Issue number if applicable]
```
```

---

## TDD Anti-Patterns to Avoid

### Anti-Pattern 1: Writing Implementation Before Tests
**Wrong:**
```
1. Add property to TagHelper
2. Write implementation
3. Write tests
```

**Correct:**
```
1. Write failing tests
2. Add property (tests now compile but fail)
3. Write minimal implementation (tests pass)
4. Refactor
```

### Anti-Pattern 2: Tests That Don't Fail First
**Wrong:**
```csharp
[Fact]
public void Process_Always_PassesTest()
{
    Assert.True(true); // This never tests the implementation
}
```

**Correct:**
```csharp
[Fact]
public void Process_WithFeature_AddsAttribute()
{
    // This WILL fail until implementation is added
    Assert.Equal("value", output.Attributes["feature"].Value);
}
```

### Anti-Pattern 3: Over-Engineering in GREEN Phase
**Wrong:**
```csharp
// Adding complex abstraction layers just to pass tests
public interface IFeatureStrategy { }
public class ConcreteFeatureStrategy : IFeatureStrategy { }
public class FeatureFactory { }
```

**Correct:**
```csharp
// Simple, direct implementation
if (FeatureEnabled)
{
    output.Attributes.Add("feature", "enabled");
}
```

### Anti-Pattern 4: Skipping REFACTOR Phase
**Wrong:**
```
RED → GREEN → Move to next feature
```

**Correct:**
```
RED → GREEN → REFACTOR → Verify → Move to next feature
```

### Anti-Pattern 5: Testing Implementation Details
**Wrong:**
```csharp
[Fact]
public void Process_CallsPrivateHelperMethod()
{
    // Testing private method existence
}
```

**Correct:**
```csharp
[Fact]
public void Process_WithFeature_ProducesExpectedOutput()
{
    // Testing observable behavior through public API
}
```

---

## Complete Example: Adding OnDelete Callback

### Example Feature Request
"Add OnDelete Callback to InputTagHelper"

### Section 1: Feature Analysis

```markdown
## Feature: OnDelete Callback for InputTagHelper

### User Request
Add OnDelete Callback

### Interpretation
Add the ability to specify a callback that is invoked when a delete action is triggered for the input field. This is useful for list/grid scenarios where each input has an associated delete button.

### Proposed API
```csharp
/// <summary>
/// Gets or sets the callback invoked when delete is triggered.
/// </summary>
public Action<TagHelperContext, string> OnDelete { get; set; }

/// <summary>
/// Gets or sets the delete button text. Defaults to "Delete".
/// </summary>
public string DeleteButtonText { get; set; }
```

### Acceptance Criteria
1. [ ] When OnDelete is set, a delete button is rendered adjacent to the input
2. [ ] Delete button displays DeleteButtonText (default: "Delete")
3. [ ] Delete button triggers the callback with context and field identifier
4. [ ] When OnDelete is null, no delete button is rendered
5. [ ] Delete button has appropriate CSS class for styling
6. [ ] Works with both Name and For properties

### Design Decisions

#### Decision 1: Callback Signature
**Options:**
- Option A: `Action<TagHelperContext, string>` (context + identifier)
- Option B: `Action<string>` (just identifier)

**Selected:** Option A
**Rationale:** Provides full context for complex scenarios, consistent with TagHelper patterns

#### Decision 2: Button Placement
**Options:**
- Option A: PostElement (after input)
- Option B: PreElement (before input)
- Option C: New configurable property

**Selected:** Option A
**Rationale:** Standard UI pattern, doesn't interfere with input focus

### Edge Cases
1. **Both Name and For null:** Should throw exception (existing validation)
2. **DeleteButtonText null:** Use default "Delete"
3. **DeleteButtonText empty:** Use empty string (allow customization)
4. **Very long DeleteButtonText:** No limit (allow flexibility)
5. **OnDelete throws exception:** Should propagate (caller responsibility)

### Test Strategy
- **Happy Path:** 3 tests
- **Edge Cases:** 4 tests
- **Error Handling:** 2 tests
- **Integration:** 2 tests
- **Total:** 11 tests
```

### Section 2: RED Phase

```csharp
using Microsoft.AspNetCore.Razor.TagHelpers;
using Microsoft.AspNetCore.Mvc.ViewFeatures;
using Microsoft.AspNetCore.Mvc.Rendering;
using Xunit;

namespace Microsoft.AspNetCore.Mvc.TagHelpers.Tests;

public class InputTagHelper_OnDeleteFeature_Tests
{
    [Fact]
    public async Task ProcessAsync_WithOnDeleteCallback_AddsDeleteButtonToPostElement()
    {
        // Arrange
        var callbackInvoked = false;
        var tagHelper = CreateTagHelper();
        tagHelper.OnDelete = (ctx, id) => callbackInvoked = true;
        tagHelper.Name = "testField";
        
        var context = CreateContext();
        var output = CreateOutput();

        // Act
        await tagHelper.ProcessAsync(context, output);

        // Assert
        var postElementContent = output.PostElement.GetContent();
        Assert.Contains("<button", postElementContent);
        Assert.Contains("type=\"button\"", postElementContent);
        // This will FAIL: OnDelete property doesn't exist yet
    }

    [Fact]
    public async Task ProcessAsync_WithOnDeleteCallback_UsesDefaultDeleteButtonText()
    {
        // Arrange
        var tagHelper = CreateTagHelper();
        tagHelper.OnDelete = (ctx, id) => { };
        tagHelper.Name = "testField";
        
        var context = CreateContext();
        var output = CreateOutput();

        // Act
        await tagHelper.ProcessAsync(context, output);

        // Assert
        var postElementContent = output.PostElement.GetContent();
        Assert.Contains(">Delete</button>", postElementContent);
        // This will FAIL: DeleteButtonText property doesn't exist yet
    }

    [Fact]
    public async Task ProcessAsync_WithCustomDeleteButtonText_UsesProvidedText()
    {
        // Arrange
        var tagHelper = CreateTagHelper();
        tagHelper.OnDelete = (ctx, id) => { };
        tagHelper.DeleteButtonText = "Remove";
        tagHelper.Name = "testField";
        
        var context = CreateContext();
        var output = CreateOutput();

        // Act
        await tagHelper.ProcessAsync(context, output);

        // Assert
        var postElementContent = output.PostElement.GetContent();
        Assert.Contains(">Remove</button>", postElementContent);
        // This will FAIL: DeleteButtonText property doesn't exist yet
    }

    [Fact]
    public async Task ProcessAsync_WithNullOnDelete_DoesNotAddDeleteButton()
    {
        // Arrange
        var tagHelper = CreateTagHelper();
        tagHelper.OnDelete = null;
        tagHelper.Name = "testField";
        
        var context = CreateContext();
        var output = CreateOutput();

        // Act
        await tagHelper.ProcessAsync(context, output);

        // Assert
        var postElementContent = output.PostElement.GetContent();
        Assert.DoesNotContain("<button", postElementContent);
        // This will PASS once property exists (default null behavior)
    }

    [Fact]
    public async Task ProcessAsync_WithOnDelete_IncludesDeleteCssClass()
    {
        // Arrange
        var tagHelper = CreateTagHelper();
        tagHelper.OnDelete = (ctx, id) => { };
        tagHelper.Name = "testField";
        
        var context = CreateContext();
        var output = CreateOutput();

        // Act
        await tagHelper.ProcessAsync(context, output);

        // Assert
        var postElementContent = output.PostElement.GetContent();
        Assert.Contains("class=\"delete-button\"", postElementContent);
        // This will FAIL: Delete button not implemented yet
    }

    [Fact]
    public async Task ProcessAsync_WithOnDeleteUsingForProperty_UsesForNameAsIdentifier()
    {
        // Arrange
        var capturedId = (string)null;
        var metadataProvider = new EmptyModelMetadataProvider();
        var modelExplorer = metadataProvider.GetModelExplorerForType(typeof(string), "value");
        var modelExpression = new ModelExpression("FieldName", modelExplorer);
        
        var tagHelper = CreateTagHelper();
        tagHelper.OnDelete = (ctx, id) => capturedId = id;
        tagHelper.For = modelExpression;
        
        var context = CreateContext();
        var output = CreateOutput();

        // Act
        await tagHelper.ProcessAsync(context, output);

        // Assert
        Assert.Equal("FieldName", capturedId);
        // This will FAIL: Callback not invoked yet
    }

    [Fact]
    public async Task ProcessAsync_WithOnDeleteUsingNameProperty_UsesNameAsIdentifier()
    {
        // Arrange
        var capturedId = (string)null;
        var tagHelper = CreateTagHelper();
        tagHelper.OnDelete = (ctx, id) => capturedId = id;
        tagHelper.Name = "MyField";
        
        var context = CreateContext();
        var output = CreateOutput();

        // Act
        await tagHelper.ProcessAsync(context, output);

        // Assert
        Assert.Equal("MyField", capturedId);
        // This will FAIL: Callback not invoked yet
    }

    [Fact]
    public async Task ProcessAsync_WhenOnDeleteCallbackThrows_PropagatesException()
    {
        // Arrange
        var tagHelper = CreateTagHelper();
        tagHelper.OnDelete = (ctx, id) => throw new InvalidOperationException("Callback error");
        tagHelper.Name = "testField";
        
        var context = CreateContext();
        var output = CreateOutput();

        // Act & Assert
        var exception = await Assert.ThrowsAsync<InvalidOperationException>(
            () => tagHelper.ProcessAsync(context, output));
        Assert.Equal("Callback error", exception.Message);
        // This will FAIL: Callback not invoked yet
    }

    [Fact]
    public async Task ProcessAsync_WithEmptyDeleteButtonText_RendersEmptyButton()
    {
        // Arrange
        var tagHelper = CreateTagHelper();
        tagHelper.OnDelete = (ctx, id) => { };
        tagHelper.DeleteButtonText = "";
        tagHelper.Name = "testField";
        
        var context = CreateContext();
        var output = CreateOutput();

        // Act
        await tagHelper.ProcessAsync(context, output);

        // Assert
        var postElementContent = output.PostElement.GetContent();
        Assert.Contains("></ button>", postElementContent.Replace(" ", ""));
        // This will FAIL: Feature not implemented yet
    }

    [Fact]
    public async Task ProcessAsync_WithOnDelete_IncludesDataAttributeForIdentifier()
    {
        // Arrange
        var tagHelper = CreateTagHelper();
        tagHelper.OnDelete = (ctx, id) => { };
        tagHelper.Name = "testField";
        
        var context = CreateContext();
        var output = CreateOutput();

        // Act
        await tagHelper.ProcessAsync(context, output);

        // Assert
        var postElementContent = output.PostElement.GetContent();
        Assert.Contains("data-delete-id=\"testField\"", postElementContent);
        // This will FAIL: Data attribute not added yet
    }

    [Fact]
    public async Task ProcessAsync_WithOnDelete_PassesTagHelperContextToCallback()
    {
        // Arrange
        var capturedContext = (TagHelperContext)null;
        var tagHelper = CreateTagHelper();
        tagHelper.OnDelete = (ctx, id) => capturedContext = ctx;
        tagHelper.Name = "testField";
        
        var context = CreateContext();
        var output = CreateOutput();

        // Act
        await tagHelper.ProcessAsync(context, output);

        // Assert
        Assert.Same(context, capturedContext);
        // This will FAIL: Callback not invoked yet
    }

    // Helper methods
    private InputTagHelper CreateTagHelper()
    {
        var metadataProvider = new EmptyModelMetadataProvider();
        var htmlGenerator = new TestableHtmlGenerator(metadataProvider);
        var tagHelper = new InputTagHelper(htmlGenerator)
        {
            ViewContext = TestableHtmlGenerator.GetViewContext(false, htmlGenerator, metadataProvider)
        };
        return tagHelper;
    }

    private static TagHelperContext CreateContext()
    {
        return new TagHelperContext(
            tagName: "input",
            allAttributes: new TagHelperAttributeList(),
            items: new Dictionary<object, object>(),
            uniqueId: "test");
    }

    private static TagHelperOutput CreateOutput()
    {
        return new TagHelperOutput(
            tagName: "input",
            attributes: new TagHelperAttributeList(),
            getChildContentAsync: (useCached, encoder) => 
                Task.FromResult<TagHelperContent>(new DefaultTagHelperContent()))
        {
            TagMode = TagMode.SelfClosing
        };
    }
}
```

**Test Failure Verification:**
All 11 tests fail with compilation errors: `'InputTagHelper' does not contain a definition for 'OnDelete'` and `'InputTagHelper' does not contain a definition for 'DeleteButtonText'`

### Section 3: GREEN Phase

```csharp
// Modified InputTagHelper.cs

public class InputTagHelper : TagHelper
{
    private readonly IHtmlGenerator _generator;
    
    public InputTagHelper(IHtmlGenerator generator)
    {
        _generator = generator ?? throw new ArgumentNullException(nameof(generator));
    }

    // Existing properties...
    public ModelExpression For { get; set; }
    public string Name { get; set; }
    
    /// <summary>
    /// Gets or sets the callback invoked when a delete action is triggered.
    /// The callback receives the current <see cref="TagHelperContext"/> and the field identifier.
    /// </summary>
    /// <remarks>
    /// When set, a delete button is rendered adjacent to the input field using the
    /// <see cref="DeleteButtonText"/> as button content. The field identifier is determined
    /// from the <see cref="For"/> property name, or falls back to <see cref="Name"/>.
    /// If the callback throws an exception, it will propagate to the caller.
    /// </remarks>
    public Action<TagHelperContext, string> OnDelete { get; set; }
    
    /// <summary>
    /// Gets or sets the delete button text. Defaults to "Delete" if not specified.
    /// </summary>
    /// <remarks>
    /// This property is only used when <see cref="OnDelete"/> is set. An empty string
    /// is allowed and will render an empty button.
    /// </remarks>
    public string DeleteButtonText { get; set; }
    
    [HtmlAttributeNotBound]
    [ViewContext]
    public ViewContext ViewContext { get; set; }
    
    public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
        if (context == null) throw new ArgumentNullException(nameof(context));
        if (output == null) throw new ArgumentNullException(nameof(output));
        
        // Existing implementation...
        var tagBuilder = _generator.GenerateInput(
            ViewContext,
            For?.ModelExplorer,
            For?.Name ?? Name,
            null,
            null,
            null);
        
        output.MergeAttributes(tagBuilder);
        output.TagMode = TagMode.SelfClosing;
        
        // NEW FEATURE: Add delete button if callback is provided
        if (OnDelete != null)
        {
            var deleteButton = GenerateDeleteButton(context);
            output.PostElement.AppendHtml(deleteButton);
        }
    }
    
    /// <summary>
    /// Generates the delete button HTML and invokes the OnDelete callback.
    /// </summary>
    private IHtmlContent GenerateDeleteButton(TagHelperContext context)
    {
        var identifier = For?.Name ?? Name;
        
        var button = new TagBuilder("button");
        button.Attributes["type"] = "button";
        button.Attributes["class"] = "delete-button";
        button.Attributes["data-delete-id"] = identifier;
        button.InnerHtml.Append(DeleteButtonText ?? "Delete");
        
        // Invoke callback (exceptions propagate intentionally)
        OnDelete.Invoke(context, identifier);
        
        return button;
    }
}
```

**Implementation Notes:**
- Added two public properties: `OnDelete` and `DeleteButtonText`
- Added conditional logic in ProcessAsync to render delete button when OnDelete is set
- Created helper method `GenerateDeleteButton` to encapsulate button generation
- Callback is invoked during rendering (synchronous operation)
- All 11 tests now pass

### Section 4: REFACTOR Phase

**Code Review Findings:**

1. **Issue:** Callback is invoked during rendering, which is unusual
   - **Improvement:** Consider making this a data-attribute pattern where JavaScript handles the actual callback

2. **Issue:** No constant for "delete-button" class name
   - **Improvement:** Extract to constant

3. **Issue:** No validation that identifier exists
   - **Improvement:** Add defensive check

4. **Issue:** Button generation and callback invocation are coupled
   - **Improvement:** Separate concerns

**Refactored Implementation:**

```csharp
public class InputTagHelper : TagHelper
{
    private const string DeleteButtonCssClass = "delete-button";
    private const string DeleteIdDataAttribute = "data-delete-id";
    private const string DefaultDeleteButtonText = "Delete";
    
    private readonly IHtmlGenerator _generator;
    
    public InputTagHelper(IHtmlGenerator generator)
    {
        _generator = generator ?? throw new ArgumentNullException(nameof(generator));
    }

    // Existing properties...
    public ModelExpression For { get; set; }
    public string Name { get; set; }
    
    /// <summary>
    /// Gets or sets the callback invoked when a delete action is triggered.
    /// The callback receives the current <see cref="TagHelperContext"/> and the field identifier.
    /// </summary>
    /// <remarks>
    /// When set, a delete button is rendered adjacent to the input field using the
    /// <see cref="DeleteButtonText"/> as button content. The field identifier is determined
    /// from the <see cref="For"/> property name, or falls back to <see cref="Name"/>.
    /// If the callback throws an exception, it will propagate to the caller.
    /// </remarks>
    /// <example>
    /// <code>
    /// &lt;input asp-for="Model.Name" asp-on-delete="@HandleDelete" /&gt;
    /// 
    /// @code {
    ///     private void HandleDelete(TagHelperContext context, string fieldId)
    ///     {
    ///         // Handle delete logic
    ///     }
    /// }
    /// </code>
    /// </example>
    public Action<TagHelperContext, string> OnDelete { get; set; }
    
    /// <summary>
    /// Gets or sets the delete button text. Defaults to "Delete" if not specified.
    /// </summary>
    /// <remarks>
    /// This property is only used when <see cref="OnDelete"/> is set. An empty string
    /// is allowed and will render an empty button.
    /// </remarks>
    public string DeleteButtonText { get; set; }
    
    [HtmlAttributeNotBound]
    [ViewContext]
    public ViewContext ViewContext { get; set; }
    
    public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
        if (context == null) throw new ArgumentNullException(nameof(context));
        if (output == null) throw new ArgumentNullException(nameof(output));
        
        // Existing implementation...
        var tagBuilder = _generator.GenerateInput(
            ViewContext,
            For?.ModelExplorer,
            For?.Name ?? Name,
            null,
            null,
            null);
        
        output.MergeAttributes(tagBuilder);
        output.TagMode = TagMode.SelfClosing;
        
        // NEW FEATURE: Add delete button if callback is provided
        if (ShouldRenderDeleteButton())
        {
            RenderDeleteButton(context, output);
        }
    }
    
    /// <summary>
    /// Determines whether a delete button should be rendered.
    /// </summary>
    private bool ShouldRenderDeleteButton()
    {
        return OnDelete != null && HasValidIdentifier();
    }
    
    /// <summary>
    /// Checks if a valid field identifier can be determined.
    /// </summary>
    private bool HasValidIdentifier()
    {
        return !string.IsNullOrEmpty(For?.Name ?? Name);
    }
    
    /// <summary>
    /// Renders the delete button and invokes the callback.
    /// </summary>
    private void RenderDeleteButton(TagHelperContext context, TagHelperOutput output)
    {
        var identifier = For?.Name ?? Name;
        var deleteButton = CreateDeleteButton(identifier);
        output.PostElement.AppendHtml(deleteButton);
        
        // Invoke callback after button creation (exceptions propagate intentionally)
        OnDelete.Invoke(context, identifier);
    }
    
    /// <summary>
    /// Creates the delete button HTML element.
    /// </summary>
    private TagBuilder CreateDeleteButton(string identifier)
    {
        var button = new TagBuilder("button");
        button.Attributes["type"] = "button";
        button.Attributes["class"] = DeleteButtonCssClass;
        button.Attributes[DeleteIdDataAttribute] = identifier;
        button.InnerHtml.Append(DeleteButtonText ?? DefaultDeleteButtonText);
        
        return button;
    }
}
```

**Refactoring Improvements:**
1. ✅ Extracted magic strings to constants
2. ✅ Separated concerns: button creation vs. rendering vs. callback invocation
3. ✅ Added helper methods for readability (ShouldRenderDeleteButton, HasValidIdentifier)
4. ✅ Enhanced XML documentation with usage example
5. ✅ Improved method naming for clarity

**Test Refactoring:**

```csharp
// Improved test helper methods
private InputTagHelper CreateTagHelperWithDelete(
    Action<TagHelperContext, string> onDeleteCallback,
    string deleteButtonText = null,
    string fieldName = "testField")
{
    var tagHelper = CreateTagHelper();
    tagHelper.OnDelete = onDeleteCallback;
    tagHelper.DeleteButtonText = deleteButtonText;
    tagHelper.Name = fieldName;
    return tagHelper;
}

// Improved assertion helper
private void AssertDeleteButtonExists(TagHelperOutput output, string expectedText = "Delete")
{
    var postElementContent = output.PostElement.GetContent();
    Assert.Contains("<button", postElementContent);
    Assert.Contains("type=\"button\"", postElementContent);
    Assert.Contains($">{expectedText}</button>", postElementContent);
}

// Example refactored test
[Fact]
public async Task ProcessAsync_WithOnDeleteCallback_AddsDeleteButtonToPostElement()
{
    // Arrange
    var tagHelper = CreateTagHelperWithDelete((ctx, id) => { });
    var context = CreateContext();
    var output = CreateOutput();

    // Act
    await tagHelper.ProcessAsync(context, output);

    // Assert
    AssertDeleteButtonExists(output);
}
```

### Section 5: Final Verification

#### Functionality
- [x] All 11 tests pass
- [x] Feature works as described in acceptance criteria
- [x] Edge cases handled correctly (null, empty, exceptions)
- [x] Error handling works as expected

#### Code Quality
- [x] No code smells
- [x] Follows ASP.NET Core TagHelper conventions
- [x] Consistent with existing InputTagHelper code
- [x] Well-documented with XML comments and example

#### Testing
- [x] Estimated line coverage: 95% (20/21 new lines)
- [x] Estimated branch coverage: 100% (4/4 branches)
- [x] All acceptance criteria have tests
- [x] Tests are maintainable and readable

#### Integration
- [x] No breaking changes to existing API
- [x] Backward compatible (new optional properties)
- [x] No conflicts with existing features
- [x] Ready for code review

### Final Deliverables

**Files Modified:**
1. `src/Mvc/Mvc.TagHelpers/src/InputTagHelper.cs`
2. `src/Mvc/Mvc.TagHelpers/test/InputTagHelperTests.cs`

**Commit Message:**
```
Add OnDelete callback feature to InputTagHelper

- Add OnDelete property for delete action callbacks
- Add DeleteButtonText property (defaults to "Delete")
- Implement delete button rendering in PostElement
- Handle edge cases (null, empty, exceptions)
- Add comprehensive test coverage (95%)
- Maintain backward compatibility

The OnDelete callback receives TagHelperContext and field identifier,
allowing applications to implement custom delete logic for input fields.
```

---

## Output Format

Structure your complete response as follows:

```markdown
# TDD Feature Addition: [Feature Name] for [TagHelper Name]

## Section 1: FEATURE ANALYSIS & SPECIFICATION

[Complete feature analysis following the template]

---

## Section 2: RED PHASE - FAILING TESTS

### Test Suite

```csharp
[Complete test class with all failing tests]
```

### Test Failure Verification

[Document expected failures for each test]

---

## Section 3: GREEN PHASE - IMPLEMENTATION

### Implementation Strategy

[Explain the approach]

### Complete Implementation

```csharp
[Complete modified TagHelper code]
```

### Implementation Notes

[Key decisions and rationale]

---

## Section 4: REFACTOR PHASE - CODE REVIEW

### Code Review Findings

[List issues and improvements]

### Refactored Implementation

```csharp
[Complete refactored TagHelper code]
```

### Refactored Tests

```csharp
[Complete refactored test class]
```

### Improvements Summary

[Quantify improvements]

---

## Section 5: FINAL VERIFICATION

### Verification Checklist

[Complete checklist with status]

### Coverage Analysis

[Detailed coverage metrics]

### Final Deliverables

[Files modified, commit message, usage examples]

---

## Ready for Integration

- [ ] All tests pass
- [ ] Code reviewed and refactored
- [ ] Documentation complete
- [ ] Ready to commit
```

---

## Critical Reminders

1. **Always start with failing tests** - Red phase is mandatory
2. **Minimal implementation** - Don't over-engineer in Green phase
3. **Always refactor** - Don't skip this phase
4. **Run all tests** - Verify existing tests still pass
5. **Document thoroughly** - Future maintainers need to understand intent
6. **Follow patterns** - Stay consistent with ASP.NET Core conventions
7. **Verify coverage** - Aim for 85%+ of new code
8. **Think about users** - API should be intuitive and well-documented

---

## Begin TDD Feature Addition

Analyze the feature request, examine the existing TagHelper, and follow the TDD cycle to add the requested feature.

Remember: TDD is not just about tests - it's about design. The tests you write first define the API and behavior. Make them excellent.