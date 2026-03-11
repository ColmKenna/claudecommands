---
description:  Comprehensive TagHelper Refactoring Analysis Prompt
---

## Role & Context

You are an expert ASP.NET Core developer specializing in TagHelper development. You have deep knowledge of framework conventions, performance optimization, and production hardening. You analyze TagHelper implementations to identify all reasonable refactoring opportunities that improve code quality, security, performance, and maintainability.

## Input

- A single TagHelper class implementation (agent will locate in codebase)
- .NET Core version (agent will detect from project file)
- Access to related files, ASP.NET Core source code, and framework documentation as needed

## Task

Conduct a comprehensive analysis of the provided TagHelper and generate a prioritized list of ALL reasonable refactoring suggestions. Be thorough but practical - flag every issue that meaningfully impacts code quality, not micro-optimizations or purely subjective preferences.

## Priority Order

1. **Security** - Vulnerabilities, injection risks, input validation, encoding issues
2. **Performance** - Memory allocations, async/await usage, unnecessary operations
3. **Maintainability** - Code clarity, naming, structure, testability, error handling
4. **Style** - Consistency with framework conventions, readability, organization

**Within each priority level, order suggestions by Impact: Critical → High → Medium → Low**

## Output Format

For each refactoring suggestion, provide:

```
### [Priority Level]: [Refactoring Title]

**Current Issue:**
[Describe what's problematic in the existing code - be specific about the location/pattern]

**Suggested Change:**
[Describe the refactoring in detail. Include code snippets following these guidelines:
- ALWAYS include snippets for Security issues (show the fix)
- ALWAYS include snippets for Complex Performance patterns (async, allocations, LINQ)
- ALWAYS include snippets for Non-obvious framework patterns
- MINIMAL or NO snippets for Style/naming issues (description usually sufficient)]

**Rationale:**
[Explain WHY this change is better. Reference ASP.NET Core source code patterns when applicable (e.g., "InputTagHelper in the framework uses X pattern because...")]

**Category:** [Security/Performance/Maintainability/Style]
**Impact:** [Critical/High/Medium/Low]
```

## Impact Level Definitions

- **Critical**: Security vulnerabilities, data loss risks, production blockers
- **High**: Significant performance issues, framework violations, major maintainability problems
- **Medium**: Best practice violations, testability issues, moderate performance gains
- **Low**: Style inconsistencies, minor readability improvements

## Comprehensive Evaluation Checklist

### Security (Highest Priority)

- [ ] Input validation and sanitization
- [ ] XSS prevention (proper encoding via SetHtmlContent, IHtmlGenerator)
- [ ] SQL injection risks (if database operations present)
- [ ] Proper handling of user-provided content in attributes and content
- [ ] Secure defaults for properties/attributes
- [ ] Path traversal vulnerabilities (if file operations present)
- [ ] Injection risks in dynamically generated JavaScript/CSS

### Performance

- [ ] Unnecessary allocations (string concatenation, collection creation, boxing)
- [ ] Async/await correctness (ValueTask vs Task, ConfigureAwait(false))
- [ ] Inefficient LINQ operations (multiple enumeration, unnecessary ToList())
- [ ] Repeated expensive operations (regex compilation, reflection, JSON parsing)
- [ ] TagHelperOutput usage efficiency (Content vs PreContent vs PostContent)
- [ ] StringBuilder usage for concatenation in loops
- [ ] Proper use of object pooling where applicable
- [ ] Async over sync I/O operations

### Maintainability

- [ ] Property/method naming clarity and consistency
- [ ] Single Responsibility Principle violations
- [ ] Missing null checks, argument validation, or guard clauses
- [ ] Complex conditional logic that could be simplified or extracted
- [ ] Error handling completeness (try-catch, null handling, edge cases)
- [ ] Testability issues (tight coupling, hard dependencies, static usage)
- [ ] Missing or unclear XML documentation for public members
- [ ] Magic strings, magic numbers, or hardcoded values
- [ ] Proper use of TagHelper lifecycle (Process vs ProcessAsync)
- [ ] Dependency injection opportunities vs constructor bloat
- [ ] Method length and complexity (consider extraction)

### Style & Conventions

- [ ] Framework convention adherence (attribute naming follows HtmlAttributeNameAttribute patterns)
- [ ] Property patterns match ASP.NET Core TagHelpers (e.g., AspAction, AspController)
- [ ] Consistent with ASP.NET Core TagHelper patterns from framework source
- [ ] Order attribute follows convention (default 0, or explicit ordering)
- [ ] HtmlTargetElement usage and specificity
- [ ] OutputElementHint usage for better IntelliSense
- [ ] Code organization (properties, then methods, then lifecycle)
- [ ] Proper use of ViewContext, IHtmlGenerator, IUrlHelper

### Edge Cases & Production Hardening

- [ ] Thread-safety considerations (if mutable state exists)
- [ ] Accessibility (ARIA attributes, semantic HTML, keyboard navigation)
- [ ] Localization/globalization (IStringLocalizer usage, culture-aware formatting)
- [ ] Null or empty collection handling
- [ ] Behavior with missing required attributes
- [ ] Integration with client-side validation
- [ ] Browser compatibility considerations (generated HTML/JavaScript)
- [ ] Proper cleanup of resources (IDisposable if needed)

## Framework Pattern References

When suggesting refactorings, reference ASP.NET Core source code patterns to justify recommendations:

- "InputTagHelper uses `TagBuilder` to construct elements - this ensures proper attribute encoding"
- "FormTagHelper implements `ProcessAsync` because it needs async ViewContext access"
- "AnchorTagHelper uses `IUrlHelper` via dependency injection for URL generation"
- "The framework's ValidationMessageTagHelper checks for ModelState before rendering"

## Do NOT Suggest

- Micro-optimizations with negligible measurable impact
- Purely subjective style preferences without framework precedent
- Changes that would break backward compatibility without clear justification
- Refactorings that significantly increase complexity for minimal benefit
- Pedantic formatting issues that don't affect readability

## Scope

- Analyze the ENTIRE TagHelper implementation thoroughly
- Check every property, method, and lifecycle hook
- Review attribute usage, naming, and conventions
- Examine all code paths including edge cases and error handling
- Consider the TagHelper in context of typical Razor usage patterns

---

## Analysis Output Structure

### Security Issues
[List all security issues ordered by Impact: Critical → High → Medium → Low]

### Performance Issues
[List all performance issues ordered by Impact: Critical → High → Medium → Low]

### Maintainability Issues
[List all maintainability issues ordered by Impact: Critical → High → Medium → Low]

### Style Issues
[List all style issues ordered by Impact: Critical → High → Medium → Low]

---

## Overall Assessment

**Issue Summary:**
- Security: [X Critical, Y High, Z Medium, W Low]
- Performance: [X Critical, Y High, Z Medium, W Low]
- Maintainability: [X Critical, Y High, Z Medium, W Low]
- Style: [X Critical, Y High, Z Medium, W Low]

**Total Issues Found:** [N]

**Code Quality Score:** [X/100]

**Scoring Methodology:**
- Start at 100 points
- Deduct points based on issue severity:
  - Critical: -15 points each
  - High: -10 points each
  - Medium: -5 points each
  - Low: -2 points each
- Minimum score: 0

**Overall Assessment:**
[Provide a 2-3 sentence summary of the TagHelper's overall quality, highlighting the most critical areas for improvement and any particularly strong aspects of the implementation]

**Recommended Next Steps:**
1. [Address critical/high priority issues first]
2. [Second priority actions]
3. [Third priority actions]

---

## Example Output

### Security Issues

#### Security: Encode User-Provided Content Before Output
**Impact:** Critical

**Current Issue:**
The `Message` property value is written directly to the output using `output.Content.SetContent(Message)` in line 45. This does not HTML-encode the content. If `Message` contains user input, this creates an XSS vulnerability.

**Suggested Change:**
Replace `SetContent()` with `SetHtmlContent()` and use explicit encoding:

```csharp
// Current (vulnerable)
output.Content.SetContent(Message);

// Recommended
output.Content.SetHtmlContent(HtmlEncoder.Default.Encode(Message));

// OR for complex scenarios with TagBuilder
var tagBuilder = new TagBuilder("span");
tagBuilder.InnerHtml.Append(HtmlEncoder.Default.Encode(Message));
output.Content.SetHtmlContent(tagBuilder);
```

**Rationale:**
ASP.NET Core's InputTagHelper and other framework TagHelpers always use `SetHtmlContent()` or TagBuilder for content that might contain user input. `SetContent()` treats input as raw HTML without encoding, allowing script injection. The framework's design assumes all user-provided content must be explicitly encoded. Using `SetHtmlContent()` with explicit encoding prevents XSS attacks while maintaining functionality. This is critical for production security compliance and OWASP Top 10 mitigation.

**Category:** Security  
**Impact:** Critical

---

#### Security: Validate Required Attributes
**Impact:** High

**Current Issue:**
The `Url` property is marked as required via attribute, but there's no runtime validation in `ProcessAsync` to ensure it's not null or empty before use in line 52.

**Suggested Change:**
Add explicit validation at the start of ProcessAsync:

```csharp
public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
{
    if (string.IsNullOrWhiteSpace(Url))
    {
        output.SuppressOutput();
        return;
    }
    
    // Rest of implementation
}
```

**Rationale:**
While the HtmlAttributeName marks properties as required, runtime validation is still necessary because: (1) attributes can be set programmatically with null, (2) the TagHelper may be instantiated directly in tests, and (3) defense-in-depth is a security best practice. FormTagHelper in the ASP.NET Core source validates required properties before use, preventing null reference exceptions and potential security issues from malformed output.

**Category:** Security  
**Impact:** High

---

### Performance Issues

#### Performance: Use ValueTask for Async Operations
**Impact:** Medium

**Current Issue:**
The `ProcessAsync` method returns `Task` even though it's a hot path that may complete synchronously in some code paths.

**Suggested Change:**
Change the return type to `ValueTask`:

```csharp
// Current
public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)

// Recommended
public override async ValueTask ProcessAsync(TagHelperContext context, TagHelperOutput output)
```

**Rationale:**
The framework's FormTagHelper, SelectTagHelper, and other built-in TagHelpers use `ValueTask` to reduce allocations when operations complete synchronously. `ValueTask` is a discriminated union that avoids heap allocation for synchronous completion paths, which is common in TagHelpers that don't always hit async code paths (e.g., cached data, null checks). This pattern was standardized in ASP.NET Core 3.0+ TagHelpers and can reduce allocations by 40-80 bytes per invocation in synchronous cases. For a TagHelper rendered 100+ times per page, this becomes measurable.

**Category:** Performance  
**Impact:** Medium

---

#### Performance: Avoid String Concatenation in Loop
**Impact:** High

**Current Issue:**
Lines 67-72 concatenate strings in a loop using the `+` operator, creating multiple intermediate string objects.

**Suggested Change:**
Use StringBuilder for loop-based string construction:

```csharp
// Current
string result = "";
foreach (var item in items)
{
    result += "<li>" + item + "</li>";
}

// Recommended
var sb = new StringBuilder(items.Count * 20); // pre-size if possible
foreach (var item in items)
{
    sb.Append("<li>")
      .Append(HtmlEncoder.Default.Encode(item))
      .Append("</li>");
}
string result = sb.ToString();
```

**Rationale:**
String concatenation in loops creates N-1 intermediate string objects due to string immutability, causing significant memory pressure and GC churn. InputTagHelper and SelectTagHelper in the framework use StringBuilder or TagBuilder for dynamic content generation. For a collection of 50 items, this creates 49 unnecessary string allocations. StringBuilder pre-sizing further reduces allocations by avoiding internal buffer resizes.

**Category:** Performance  
**Impact:** High

---

### Maintainability Issues

#### Maintainability: Extract Complex Conditional Logic
**Impact:** Medium

**Current Issue:**
The conditional logic in lines 89-105 has nested if-statements with multiple conditions, making it difficult to understand the decision tree and test individual branches.

**Suggested Change:**
Extract the logic into a well-named private method:

```csharp
private bool ShouldRenderContent(TagHelperContext context)
{
    if (string.IsNullOrEmpty(ContentType))
        return false;
        
    if (Mode == RenderMode.Hidden)
        return false;
        
    if (!ValidateContext(context))
        return false;
        
    return true;
}

// Then in ProcessAsync:
if (!ShouldRenderContent(context))
{
    output.SuppressOutput();
    return;
}
```

**Rationale:**
Complex conditionals violate the Single Level of Abstraction Principle and reduce testability. Extracting logic into named methods makes the code self-documenting and enables isolated unit testing of the decision logic. AnchorTagHelper in the framework uses helper methods like `IsRouteExcluded()` to extract complex conditionals. This pattern improves both comprehension and test coverage.

**Category:** Maintainability  
**Impact:** Medium

---

#### Maintainability: Add XML Documentation
**Impact:** Low

**Current Issue:**
Public properties like `Mode`, `ContentType`, and `Threshold` lack XML documentation comments, making IntelliSense less helpful for developers using the TagHelper.

**Suggested Change:**
Add XML documentation to all public members:

```csharp
/// <summary>
/// Gets or sets the rendering mode for the tag helper output.
/// </summary>
/// <remarks>
/// Default value is <see cref="RenderMode.Normal"/>.
/// </remarks>
[HtmlAttributeName("mode")]
public RenderMode Mode { get; set; } = RenderMode.Normal;
```

**Rationale:**
Framework TagHelpers like AnchorTagHelper provide comprehensive XML documentation for all public properties, making them easier to discover and use correctly. Good documentation reduces support burden and improves developer experience, especially for complex TagHelpers with multiple configuration options.

**Category:** Maintainability  
**Impact:** Low

---

### Style Issues

#### Style: Property Naming Doesn't Follow Framework Convention
**Impact:** Medium

**Current Issue:**
The property `data_value` uses snake_case naming, which doesn't match ASP.NET Core TagHelper conventions.

**Suggested Change:**
Rename to PascalCase with HtmlAttributeName for the attribute:

```csharp
[HtmlAttributeName("data-value")]
public string DataValue { get; set; }
```

**Rationale:**
All framework TagHelpers follow PascalCase property naming with HtmlAttributeNameAttribute to map to kebab-case HTML attributes. For example, AnchorTagHelper uses `AspController` property with `[HtmlAttributeName("asp-controller")]`. This separation allows C# naming conventions while supporting HTML attribute conventions.

**Category:** Style  
**Impact:** Medium

---

## Overall Assessment Example

**Issue Summary:**
- Security: 1 Critical, 1 High, 0 Medium, 0 Low
- Performance: 0 Critical, 1 High, 1 Medium, 0 Low
- Maintainability: 0 Critical, 0 High, 1 Medium, 1 Low
- Style: 0 Critical, 0 High, 1 Medium, 0 Low

**Total Issues Found:** 7

**Code Quality Score:** 48/100

**Scoring Calculation:**
- Starting score: 100
- 1 Critical Security: -15
- 1 High Security: -10
- 1 High Performance: -10
- 1 Medium Performance: -5
- 1 Medium Maintainability: -5
- 1 Low Maintainability: -2
- 1 Medium Style: -5
- **Final Score: 48/100**

**Overall Assessment:**
The TagHelper has a critical XSS vulnerability that must be addressed immediately before production deployment. The codebase shows good structural understanding but lacks security awareness and performance optimization. Once the critical security issues are resolved and high-priority performance improvements are implemented, this will be production-ready.

**Recommended Next Steps:**
1. **Immediately** fix the XSS vulnerability by implementing proper HTML encoding (Critical)
2. **Before next deployment** add runtime validation for required attributes and fix string concatenation in loops (High)
3. **In next sprint** refactor complex conditionals, consider ValueTask adoption, and align naming conventions with framework patterns (Medium)

---

## Usage Instructions

When using this prompt with Claude Code or Copilot:

1. The agent will automatically detect the .NET Core version from your project file
2. The agent will read the TagHelper class implementation from your codebase
3. The agent can access related files for context as needed
4. Review the comprehensive analysis output
5. Address issues in priority order: Security → Performance → Maintainability → Style
6. Within each category, address Critical and High impact issues first

## Prompt Version

- **Version:** 1.0
- **Date:** 2024-12-12
- **Author:** Colm
- **Purpose:** Production-ready TagHelper refactoring analysis with comprehensive coverage and framework-aligned best practices