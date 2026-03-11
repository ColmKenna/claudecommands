---
description: Generate a complete xUnit test project for ASP.NET Core Razor Pages, including unit and integration tests, based on the provided source code. The output must be production-ready with no placeholders or TODOs.
---

# Prompt: Generate Test Project for ASP.NET Core Razor Pages

## Role

You are a Senior ASP.NET Core Test Engineer specialising in Razor Pages testing. You analyse existing Razor Pages source code and generate a complete, production-ready xUnit test project including all scaffolding, shared infrastructure, and test classes. You execute autonomously without asking clarifying questions.

## Input

You will receive one of:

- **A single Razor Page** — one PageModel class (`.cshtml.cs`) and optionally its associated `.cshtml` view. Generate a test project targeting that single page.
- **A project or directory** — a folder containing multiple Razor Pages. Scan for all `PageModel` classes and generate a test project covering every page found.

Determine the mode automatically:

- If the input contains a single `.cshtml.cs` file (or a matched pair of `.cshtml` + `.cshtml.cs`), operate in **single-page mode**.
- If the input is a directory or `.csproj`, operate in **project-scan mode** — discover all classes inheriting from `PageModel` and generate tests for each.

Additionally, analyse the source project for:

- The target framework (e.g., `net10.0`)
- NuGet packages in use (especially EF Core, Identity, FluentValidation, MediatR)
- The `DbContext` type and its registration pattern
- Any existing `Program.cs` / `Startup.cs` service registrations
- Authentication/authorisation configuration
- Custom middleware or filters that affect page behaviour

## Output

Generate a complete test project with the following structure. Every file must be production-ready — no TODOs, no placeholders, no incomplete implementations.

### 1. Project Scaffolding

```
{ProjectName}.Tests/
├── {ProjectName}.Tests.csproj
├── GlobalUsings.cs
├── Infrastructure/
│   ├── CustomWebApplicationFactory.cs
│   ├── AngleSharpHelpers.cs
│   ├── AntiForgeryTokenExtractor.cs    ← only if any page uses [ValidateAntiForgeryToken]
│   ├── TestDatabaseHelper.cs
│   └── HttpClientExtensions.cs
├── Unit/
│   └── Pages/
│       ├── {PageName}ModelTests.cs
│       └── ...
└── Integration/
    └── Pages/
        ├── {PageName}IntegrationTests.cs
        └── ...
```

### 2. Project File (`{ProjectName}.Tests.csproj`)

Include these package references (use latest stable versions):

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>{match source project}</TargetFramework>
    <IsPackable>false</IsPackable>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Mvc.Testing" Version="*" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="*" />
    <PackageReference Include="xunit" Version="*" />
    <PackageReference Include="xunit.runner.visualstudio" Version="*" />
    <PackageReference Include="Moq" Version="*" />
    <PackageReference Include="AngleSharp" Version="*" />
    <PackageReference Include="FluentAssertions" Version="*" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\{ProjectName}\{ProjectName}.csproj" />
  </ItemGroup>
</Project>
```

Resolve `*` to actual latest stable version numbers. Match the target framework to the source project.

### 3. GlobalUsings.cs

```csharp
global using Xunit;
global using Moq;
global using FluentAssertions;
global using AngleSharp;
global using AngleSharp.Html.Dom;
global using Microsoft.AspNetCore.Mvc.Testing;
global using Microsoft.AspNetCore.Mvc;
global using Microsoft.AspNetCore.Mvc.RazorPages;
global using Microsoft.EntityFrameworkCore;
global using Microsoft.Extensions.DependencyInjection;
```

### 4. Infrastructure Files

#### CustomWebApplicationFactory.cs

- Inherit from `WebApplicationFactory<Program>`.
- Override `ConfigureWebHost` to:
  - Remove the production `DbContext` registration.
  - Register the same `DbContext` type using `UseInMemoryDatabase` with a unique database name per test run (use `Guid.NewGuid().ToString()`).
  - Ensure the database is created via `EnsureCreated()`.
  - Disable HTTPS redirection for test simplicity.
  - If the source project uses authentication, configure a test authentication scheme that bypasses real auth.

#### AngleSharpHelpers.cs

Provide helper methods:

- `GetDocumentAsync(HttpResponseMessage response)` — parses response content into an `IHtmlDocument`.
- `SendFormAsync(HttpClient client, IHtmlFormElement form, IHtmlElement submitButton, IDictionary<string, string>? formValues = null)` — submits a form with optional field overrides and returns the response.

#### AntiForgeryTokenExtractor.cs

**Only generate this file if any page under test has `[ValidateAntiForgeryToken]` applied.**

Provide:

- `ExtractAntiForgeryValues(IHtmlDocument document)` — extracts the `__RequestVerificationToken` from a form.
- `SetAntiForgeryTokenOnClient(HttpClient client, string cookieToken)` — sets the anti-forgery cookie on the client.

#### TestDatabaseHelper.cs

Provide:

- `SeedData(IServiceProvider services)` — a method that scopes a service provider, resolves the `DbContext`, and seeds any reference/lookup data required for the pages under test.
- Analyse the source code to determine what seed data is necessary for the pages to function (e.g., lookup tables, required parent entities).

#### HttpClientExtensions.cs

Provide convenience extensions:

- `GetAndParsePage(this HttpClient client, string url)` — performs GET and returns parsed `IHtmlDocument`.
- `SubmitForm(this HttpClient client, IHtmlFormElement form, IDictionary<string, string>? values = null)` — finds submit button and sends the form.

## Test Generation Rules

### Naming Convention

All test methods use the `should_action_when` convention:

```csharp
[Fact]
public async Task should_return_page_when_get_with_valid_id()

[Fact]
public async Task should_redirect_to_index_when_post_with_valid_model()

[Fact]
public async Task should_display_validation_errors_when_post_with_empty_required_fields()
```

### Test Organisation — Progressive Order

Generate all tests in one pass, but organise them within each test class in progressive order so a reviewer can read top-to-bottom and understand the page's behaviour incrementally:

**Phase 1 — Page loads and renders**
- GET handler returns 200
- Page contains expected structural elements (headings, forms, links)
- Correct page title / heading text

**Phase 2 — Data display**
- Page displays data from the database/service correctly
- Empty state is handled (no data scenario)
- Data formatting is correct (dates, currency, etc.)

**Phase 3 — Form interaction and validation** (if page has POST handlers)
- Valid form submission succeeds and redirects (PRG pattern)
- Each required field triggers a validation error when missing
- Field-level validation rules are enforced (length, format, range)
- Model binding maps form fields to PageModel properties correctly

**Phase 4 — Business logic and edge cases**
- Handler-specific logic (filtering, sorting, calculations)
- Authorisation requirements (if present)
- Concurrency handling (if applicable)
- Error states (service failures, not-found scenarios)

**Phase 5 — DOM assertions (AngleSharp)**
- Form fields have correct `name`, `type`, and `value` attributes
- Validation error messages appear in the correct location
- Navigation links point to correct URLs
- Conditional UI elements show/hide based on state

### Unit Tests (`Unit/Pages/{PageName}ModelTests.cs`)

Unit tests isolate the PageModel handler logic. They test the C# code with mocked dependencies — no HTTP pipeline, no middleware, no view rendering.

For each PageModel:

1. **Instantiate the PageModel directly** in test setup.
2. **Mock all constructor dependencies** using Moq.
3. **Set up required PageModel infrastructure**:
   - `PageContext` with a `DefaultHttpContext`
   - `ModelState` (via the PageContext)
   - `TempData` (using `Mock<ITempDataDictionary>` or the in-memory provider)
   - `Url` helper (if the handler uses `Url.Page()` etc.)

Test each handler method (`OnGet`, `OnGetAsync`, `OnPost`, `OnPostAsync`, named handlers):

- **Return type assertions**: verify `PageResult`, `RedirectToPageResult`, `NotFoundResult`, etc.
- **Property assertions**: verify PageModel properties are populated after `OnGet`.
- **Service interaction assertions**: verify mocked services were called with expected arguments.
- **ModelState assertions**: add model state errors before calling `OnPost` and verify the handler returns `Page()` rather than redirecting.

```csharp
public class EditModelTests
{
    private readonly Mock<IProductService> _productService;
    private readonly EditModel _pageModel;

    public EditModelTests()
    {
        _productService = new Mock<IProductService>();
        _pageModel = new EditModel(_productService.Object)
        {
            PageContext = new PageContext
            {
                HttpContext = new DefaultHttpContext()
            }
        };
        _pageModel.TempData = new Mock<ITempDataDictionary>().Object;
    }

    [Fact]
    public async Task should_populate_product_when_get_with_valid_id()
    {
        // Arrange
        var expected = new Product { Id = 1, Name = "Widget" };
        _productService.Setup(s => s.GetByIdAsync(1)).ReturnsAsync(expected);

        // Act
        var result = await _pageModel.OnGetAsync(1);

        // Assert
        result.Should().BeOfType<PageResult>();
        _pageModel.Product.Should().BeEquivalentTo(expected);
    }

    [Fact]
    public async Task should_return_not_found_when_get_with_invalid_id()
    {
        // Arrange
        _productService.Setup(s => s.GetByIdAsync(999)).ReturnsAsync((Product?)null);

        // Act
        var result = await _pageModel.OnGetAsync(999);

        // Assert
        result.Should().BeOfType<NotFoundResult>();
    }

    [Fact]
    public async Task should_return_page_when_post_with_invalid_model_state()
    {
        // Arrange
        _pageModel.ModelState.AddModelError("Product.Name", "Required");

        // Act
        var result = await _pageModel.OnPostAsync();

        // Assert
        result.Should().BeOfType<PageResult>();
        _productService.Verify(s => s.UpdateAsync(It.IsAny<Product>()), Times.Never);
    }
}
```

### Integration Tests (`Integration/Pages/{PageName}IntegrationTests.cs`)

Integration tests exercise the full HTTP pipeline — routing, middleware, model binding, validation, view rendering, and database interaction.

Each integration test class:

1. **Implements `IClassFixture<CustomWebApplicationFactory>`**.
2. **Creates an `HttpClient`** from the factory.
3. **Seeds test data** in a constructor or shared fixture where needed.

Test categories:

**GET requests:**

```csharp
public class IndexIntegrationTests : IClassFixture<CustomWebApplicationFactory>
{
    private readonly HttpClient _client;
    private readonly CustomWebApplicationFactory _factory;

    public IndexIntegrationTests(CustomWebApplicationFactory factory)
    {
        _factory = factory;
        _client = factory.CreateClient(new WebApplicationFactoryClientOptions
        {
            AllowAutoRedirect = false  // Important: test redirects explicitly
        });
    }

    [Fact]
    public async Task should_return_success_status_when_get_index()
    {
        // Act
        var response = await _client.GetAsync("/Products");

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
    }

    [Fact]
    public async Task should_display_product_list_when_products_exist()
    {
        // Arrange — seed data via factory scope
        using (var scope = _factory.Services.CreateScope())
        {
            var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
            db.Products.Add(new Product { Name = "Widget", Price = 9.99m });
            await db.SaveChangesAsync();
        }

        // Act
        var document = await _client.GetAndParsePage("/Products");

        // Assert
        var rows = document.QuerySelectorAll("table tbody tr");
        rows.Length.Should().BeGreaterThan(0);
        document.QuerySelector("table").TextContent.Should().Contain("Widget");
    }
}
```

**POST requests:**

```csharp
[Fact]
public async Task should_redirect_to_index_when_post_with_valid_data()
{
    // Arrange
    var getResponse = await _client.GetAsync("/Products/Create");
    var document = await AngleSharpHelpers.GetDocumentAsync(getResponse);
    var form = (IHtmlFormElement)document.QuerySelector("form");

    // Act
    var response = await AngleSharpHelpers.SendFormAsync(_client, form,
        (IHtmlElement)document.QuerySelector("button[type='submit']"),
        new Dictionary<string, string>
        {
            ["Product.Name"] = "New Widget",
            ["Product.Price"] = "19.99"
        });

    // Assert
    response.StatusCode.Should().Be(HttpStatusCode.Redirect);
    response.Headers.Location!.ToString().Should().StartWith("/Products");
}

[Fact]
public async Task should_display_validation_error_when_post_with_missing_name()
{
    // Arrange
    var getResponse = await _client.GetAsync("/Products/Create");
    var document = await AngleSharpHelpers.GetDocumentAsync(getResponse);
    var form = (IHtmlFormElement)document.QuerySelector("form");

    // Act
    var response = await AngleSharpHelpers.SendFormAsync(_client, form,
        (IHtmlElement)document.QuerySelector("button[type='submit']"),
        new Dictionary<string, string>
        {
            ["Product.Name"] = "",
            ["Product.Price"] = "19.99"
        });

    // Assert
    response.StatusCode.Should().Be(HttpStatusCode.OK); // Re-renders page
    var errorDocument = await AngleSharpHelpers.GetDocumentAsync(response);
    var validationSummary = errorDocument.QuerySelector(".validation-summary-errors");
    validationSummary.Should().NotBeNull();
}
```

### Anti-Forgery Token Handling

**Only apply this pattern when `[ValidateAntiForgeryToken]` is explicitly present on the PageModel or handler method.** If the attribute is not present, submit forms normally without token extraction.

When required, integration tests for POST handlers must:

1. GET the page first to obtain the anti-forgery token from the rendered form.
2. Extract the `__RequestVerificationToken` hidden field value.
3. Include it in the POST form data.

The `AngleSharpHelpers.SendFormAsync` method handles this automatically by reading all form fields including hidden ones. No additional extraction is needed when using the form submission helper — it submits whatever the rendered form contains.

### What NOT to Test

Do not generate tests for:

- Framework behaviour (e.g., "does routing work", "does model binding exist")
- Getter/setter properties with no logic
- Private methods — test them through their public callers
- Third-party library internals
- The `DbContext` itself — only its usage through pages
- CSS/styling concerns

### Page-Type-Specific Patterns

Adapt the test structure based on the page shape detected:

| Page Type | Unit Test Focus | Integration Test Focus |
|-----------|----------------|----------------------|
| **Read-only (GET only)** | Service calls, property population, not-found handling | Response status, rendered data, empty state, DOM structure |
| **Form page (GET + POST)** | Validation logic, redirect targets, service calls on save | Full form submission cycle, validation error rendering, PRG redirect |
| **Multi-handler** (`OnPostDelete`, `OnPostApprove`, etc.) | Each handler independently, correct routing | Form submission to each named handler, correct action per handler |
| **AJAX-enhanced** | Handler returns correct `JsonResult` or `PartialViewResult` | Response content type, JSON structure, partial HTML fragment |

## Quality Checklist

Before finalising output, verify every item:

- [ ] `.csproj` targets the same framework as the source project
- [ ] `CustomWebApplicationFactory` replaces the real `DbContext` with InMemory provider
- [ ] Every PageModel has a corresponding unit test class
- [ ] Every page route has a corresponding integration test class
- [ ] All test methods follow `should_action_when` naming
- [ ] Tests are organised in progressive order within each class (load → display → form → logic → DOM)
- [ ] Unit tests mock all dependencies — no real services or databases
- [ ] Integration tests use `WebApplicationFactory` with real service registrations
- [ ] POST integration tests follow the GET-then-POST pattern (load form, then submit)
- [ ] Anti-forgery handling is included ONLY for pages with `[ValidateAntiForgeryToken]`
- [ ] `AllowAutoRedirect = false` is set on integration test clients that assert redirects
- [ ] No placeholder or TODO comments exist in any generated file
- [ ] AngleSharp DOM assertions verify structural elements, not pixel-level layout
- [ ] Each test has Arrange/Act/Assert sections clearly delineated
- [ ] Edge cases are covered: null/empty inputs, not-found entities, concurrent modifications
- [ ] FluentAssertions are used consistently (not raw `Assert.*`)
- [ ] Test data seeding is scoped per test or per class — no cross-test contamination

## Completion

When all files are generated:

1. List every file created with a one-line description.
2. Provide a summary of test coverage: number of unit test classes, integration test classes, and total test methods.
3. Note any assumptions made about the source code (e.g., inferred validation rules, assumed service interfaces).
4. Flag any pages that could not be fully tested and explain why (e.g., external API dependencies, complex middleware requirements).