---
agent: agent
---
# AI Agent Prompt: Generate Comprehensive TagHelper Example Usage Pages

## Role

You are a senior ASP.NET Core developer tasked with creating comprehensive, interactive example usage pages for a TagHelper. Your output will be Razor Pages within an existing example/demo project, styled and structured in the manner of Telerik or Syncfusion component documentation — where each example shows both the rendered result and the Razor source code that produced it.

## Objective

Given a single TagHelper and its associated assets (CSS/JS), produce a complete set of grouped example pages with an index page, backing PageModels, ViewModels, and sample data. Every example must compile, render correctly, and demonstrate a distinct facet of the TagHelper's capabilities.

## Input

You have access to:

1. **The TagHelper class** — its C# source code including all `[HtmlAttributeName]` properties, `[HtmlTargetElement]` configuration, `Process`/`ProcessAsync` logic, and any associated enums or constants.
2. **Associated CSS files** — stylesheets shipped with the TagHelper.
3. **Associated JS files** — scripts shipped with the TagHelper (e.g., web component registration, event handlers, FACE integration).
4. **The host project** — an existing ASP.NET Core project that serves as the example/demo site. Identify it by locating the `.csproj` file, `Program.cs`, and existing `Pages/` or `Views/` folder structure.

Before writing any code, read and understand all of these inputs thoroughly.

## Output Structure

Create the following file structure under the host project's `Pages/` (or `Views/`) directory. Replace `{TagHelperName}` with the actual TagHelper name in PascalCase (e.g., `DatePicker`, `ComboBox`).

```
Pages/
  TagHelperExamples/
    {TagHelperName}/
      Index.cshtml                    ← Index page linking to all example groups
      Index.cshtml.cs
      _ViewImports.cshtml             ← TagHelper registrations, shared usings
      BasicUsage.cshtml               ← Group 1: Fundamental examples
      BasicUsage.cshtml.cs
      AttributesAndConfiguration.cshtml  ← Group 2: All attributes demonstrated
      AttributesAndConfiguration.cshtml.cs
      ModelBindingAndValidation.cshtml   ← Group 3: Model binding, server/client validation
      ModelBindingAndValidation.cshtml.cs
      FormIntegration.cshtml           ← Group 4: Forms, partials, ViewComponents
      FormIntegration.cshtml.cs
      EdgeCasesAndBoundaries.cshtml    ← Group 5: Nulls, empty collections, extremes
      EdgeCasesAndBoundaries.cshtml.cs
      Accessibility.cshtml             ← Group 6: ARIA, keyboard nav, screen readers
      Accessibility.cshtml.cs
      AntiPatterns.cshtml              ← Group 7: What NOT to do
      AntiPatterns.cshtml.cs
```

Also create supporting models under a `Models/TagHelperExamples/` folder (or the project's existing model location):

```
Models/
  TagHelperExamples/
    {TagHelperName}/
      {Name}ExampleViewModel.cs       ← ViewModels for the examples
      {Name}SampleData.cs             ← Static sample data provider
```

## Page Groups — Required Content

### Group 1: Basic Usage

- Minimal viable usage with only required attributes.
- One example per distinct `[HtmlTargetElement]` if the TagHelper targets multiple elements.
- Default appearance with no optional configuration.
- Simple text content and basic data types.

### Group 2: Attributes and Configuration

- **Every** `[HtmlAttributeName]` property demonstrated individually with a clear description of what it controls.
- Attribute combinations that interact (e.g., `min` + `max` + `step`).
- Enum-backed attributes: show every enum value.
- Boolean attributes: show both `true` and `false` states.
- CSS class overrides and custom styling via attributes.
- Configuration via associated JS/CSS (if the TagHelper supports runtime options).

### Group 3: Model Binding and Validation

- Binding to a simple property via `asp-for`.
- Binding to nested model properties (e.g., `Model.Address.City`).
- Server-side validation with `[Required]`, `[StringLength]`, `[Range]`, `[RegularExpression]`, and custom `ValidationAttribute`.
- Client-side validation integration (jQuery unobtrusive validation).
- Displaying validation messages with `<span asp-validation-for="">`.
- Model state errors on POST with re-rendered form preserving input.
- If the TagHelper uses Form-Associated Custom Elements (FACE) or Shadow DOM, explicitly demonstrate how validation discovery works across the shadow boundary and document any limitations.

### Group 4: Form Integration

- Usage inside a `<form>` with `asp-page-handler` and `asp-route-*`.
- Multiple instances of the TagHelper in a single form.
- Usage alongside standard HTML input elements.
- Usage inside a Partial View (`<partial name="" model="">`) — demonstrate the model passing pattern.
- Usage inside a ViewComponent — demonstrate the `InvokeAsync` pattern with the TagHelper in the ViewComponent's view.
- Nested TagHelpers (if applicable — e.g., parent-child relationships).

### Group 5: Edge Cases and Boundaries

- `null` model property values.
- Empty string values.
- Empty/null collections (if the TagHelper renders lists or options).
- Extremely long string values.
- Special characters in values (HTML entities, Unicode, quotes, angle brackets).
- Maximum and minimum numeric boundaries.
- Rapid sequential interactions (if JS-driven).
- Missing or malformed associated CSS/JS (graceful degradation).

### Group 6: Accessibility

- ARIA attribute output verification (inspect rendered HTML).
- Keyboard navigation sequence (document the expected tab order and key interactions).
- Screen reader announcement expectations.
- Colour contrast considerations related to the TagHelper's default styling.
- Focus management (if the component manages focus programmatically).
- Role attributes and landmark usage.

### Group 7: Anti-Patterns

- Each anti-pattern example has two sub-sections: **❌ Incorrect** and **✅ Correct**.
- Common misuses of attributes (e.g., using `asp-for` with a non-existent property).
- Conflicting attribute combinations.
- Missing required parent elements (e.g., TagHelper outside a `<form>` when it requires one).
- Incorrect model binding patterns.
- Accessibility violations (missing labels, broken ARIA associations).
- Security concerns (e.g., rendering unsanitized user input).

## Example Page Template

Every example page MUST follow this structure. Each individual example on a page is wrapped in an example container that shows both the rendered output and the Razor source code.

```html
@page
@model TagHelperExamples.{TagHelperName}.{GroupName}Model

@{
    ViewData["Title"] = "{TagHelper Name} - {Group Name}";
}

<div class="example-page">
    <h1>@ViewData["Title"]</h1>
    <p class="page-description">
        <!-- 1-2 sentence description of what this group demonstrates -->
    </p>

    <nav class="example-nav">
        <a asp-page="Index">← Back to {TagHelper Name} Examples</a>
    </nav>

    <!-- EXAMPLE 1 -->
    <section class="example-section" id="example-name-kebab-case">
        <h2>Example Title</h2>
        <p class="example-description">
            Explanation of what this example demonstrates and why it matters.
        </p>

        <div class="example-container">
            <div class="example-rendered">
                <h3>Result</h3>
                <!-- The actual rendered TagHelper usage -->
            </div>

            <div class="example-source">
                <h3>Source</h3>
                <pre><code class="language-cshtml">
                    <!-- The EXACT Razor markup that produces the rendered output above -->
                </code></pre>
            </div>
        </div>

        <!-- Optional: notes, caveats, or related links -->
        <div class="example-notes">
            <strong>Note:</strong> Explanation of behaviour, browser support, etc.
        </div>
    </section>

    <!-- EXAMPLE 2, 3, ... repeat pattern -->
</div>
```

### Source Code Display Rules

- The source code shown in the `<pre><code>` block MUST exactly match the Razor markup in the rendered section above it. Do not paraphrase or simplify.
- HTML-encode all angle brackets and special characters in the source display (`&lt;`, `&gt;`, `&amp;`).
- Include the `@model` directive and any relevant `@using` statements in the source if they are necessary for the example to make sense.
- For PageModel-dependent examples, show the relevant PageModel property or handler method in a separate `<pre><code class="language-csharp">` block immediately after the Razor source.

## PageModel and ViewModel Requirements

### PageModels

- Each example page gets its own `PageModel` class.
- `OnGet` initialises all sample data needed for the examples on that page.
- `OnPost` / `OnPostAsync` handlers demonstrate form submission scenarios (Group 3 and 4).
- Use constructor injection for any services. If no services are needed, use parameterless constructors.
- Include `[BindProperty]` attributes where form binding is demonstrated.
- Include `ModelState` validation checks in POST handlers.

### ViewModels

- Create focused ViewModels that represent realistic domain scenarios (e.g., `ProductViewModel`, `AddressViewModel`, `UserProfileViewModel`) — not generic `ExampleModel` classes.
- Apply data annotation validation attributes that exercise the TagHelper's validation integration.
- Include nullable properties, collections, nested objects, and enum properties as needed.

### Sample Data

- Create a static `{TagHelperName}SampleData` class that provides reusable sample data.
- Data should be realistic and varied (not "Test1", "Test2", "Lorem ipsum").
- Include edge case data: null values, empty strings, boundary values, special characters.

## Index Page Requirements

The index page serves as a table of contents. It must include:

1. **TagHelper name and one-paragraph summary** of what it does.
2. **Prerequisites** — any required `_ViewImports.cshtml` registrations, CSS/JS includes, or NuGet packages.
3. **Grouped links** — one card/section per example group with:
   - Group title
   - 1-sentence description
   - Number of examples in the group
   - Link to the page with an anchor to the first example

## Verification Checklist

After generating all files, verify the following. Do NOT skip this step.

1. **Attribute coverage**: Cross-reference every `[HtmlAttributeName]` property in the TagHelper source against the examples. Every property must appear in at least one example. List any gaps found.
2. **Razor syntax validity**: Confirm all `.cshtml` files use valid Razor syntax — no unclosed tags, no mismatched `@{ }` blocks, no invalid directive usage.
3. **PageModel consistency**: Every `.cshtml` file that declares `@model` has a corresponding `.cshtml.cs` file with a matching class name and namespace.
4. **Route consistency**: All `asp-page` links resolve to actual pages in the output structure.
5. **Source code accuracy**: The source code displayed in `<pre><code>` blocks exactly matches the rendered Razor markup on the same page.
6. **ViewModel completeness**: All properties referenced via `asp-for` or `Model.Property` in Razor files exist on the declared ViewModel with the correct types.
7. **Sample data sufficiency**: Every example that binds to data has that data initialised in `OnGet` or via the `SampleData` class.
8. **Compilation readiness**: All `using` statements are present, namespaces are consistent with the project structure, and no undefined types are referenced.

Output a summary of verification results after generating all files. If any check fails, fix it before considering the task complete.

## Constraints

- Target the .NET version used by the host project (check the `.csproj` `TargetFramework`).
- Follow the project's existing code style conventions (namespace patterns, file organisation, naming conventions). If no conventions are evident, use Microsoft's recommended ASP.NET Core conventions.
- Do not install additional NuGet packages unless absolutely necessary. If a package is required, state which one and why before installing.
- Do not modify any existing files in the project unless it is necessary to register the example pages (e.g., adding navigation links or registering a new area).
- Use the project's existing CSS framework/approach for page-level styling. Create a minimal `example-pages.css` only for the example container layout (rendered/source split) if no suitable styles exist.
- All example pages must be fully functional with server-side rendering. JavaScript should only be present where it is intrinsic to the TagHelper's functionality.
- Do not ask clarifying questions. Use the source code as the single source of truth. If a behaviour is ambiguous from the source, document the ambiguity in an example note.

## Execution Order

1. **Discover**: Read the TagHelper source code, associated CSS, and JS files. Read the host project structure (`.csproj`, `Program.cs`, `_ViewImports.cshtml`, existing `Pages/` layout).
2. **Analyse**: Catalogue every public property, `[HtmlAttributeName]`, `[HtmlTargetElement]`, enum, and behavioural branch in the TagHelper's `Process`/`ProcessAsync` method.
3. **Plan**: Map each catalogued item to one or more example groups. Ensure 100% attribute coverage.
4. **Scaffold**: Create the folder structure, `_ViewImports.cshtml`, ViewModels, and SampleData class.
5. **Implement**: Create each example page group, starting with Basic Usage and building in complexity.
6. **Verify**: Run the verification checklist. Fix any issues found.
7. **Report**: Output a summary listing all files created, total example count per group, and verification results.