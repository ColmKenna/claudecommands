# Web Component Documentation Template

You are a technical writer and front-end engineer.  
Your task is to generate HTML help documentation for a **custom Web Component**.

## Goal
Create a clean, well-structured documentation experience that explains how to use the web component, including:
- All available options/attributes
- CSS setup and customization
- Examples of usage
- Integration snippets for **Razor, React, Angular, and Blazor**
- A **quick-links navigation on the left side** on the main help page
- Optional linked sub-pages if the component is complex enough to justify it
- **Runnable example HTML files** in the help folder, with links to them from the documentation

---

## Component Details

Use the following information about the web component:

- Component name: [[component tag name, e.g. `<my-widget>`]]
- Short description: [[one-sentence summary of what it does]]
- Detailed description: [[1–3 paragraphs of what problems it solves, key features, etc.]]
- Component category: [[e.g. “form control”, “data display”, “layout helper”]]

### Attributes / Options
Document these clearly in a reference table:

For each attribute/option, include:
- Name
- Type (string, number, boolean, enum, etc.)
- Default value
- Required? (yes/no)
- Description
- Example value

List of attributes/options:
[[e.g.
- `variant`: "primary" | "secondary" | "ghost" (default: "primary")
- `size`: "sm" | "md" | "lg" (default: "md")
- `disabled`: boolean (default: false)
- `label`: string
- `icon`: string (name of icon)
]]

### Events
If relevant, include an “Events” section and table:

For each event, include:
- Event name
- When it fires
- Event detail payload (if any)
- Example event listener

List of events:
[[e.g.
- `my-widget:ready`
- `my-widget:change`
- `my-widget:error`
]]

### Slots
If the component uses `<slot>`, document them:

For each slot:
- Slot name (or “default slot”)
- What kind of content it should contain
- Example markup

List of slots:
[[e.g.
- default slot: main content
- `actions` slot: buttons or links
]]

---

## CSS & Styling

Explain how to style and theme the component.

Include:

1. **Basic CSS setup**
   - Whether it uses Shadow DOM
   - Any required base CSS files or imports
   - Example of including the component in a page

   Details:
   [[e.g. uses Shadow DOM, shipped with default styles, requires `<link rel="stylesheet" href="...">`]]

2. **CSS custom properties (variables)**
   - List each CSS variable, meaning, and default value (if known)
   - Example of overriding them

   List of CSS variables:
   [[e.g.
   - `--my-widget-bg`: background color
   - `--my-widget-color`: text color
   - `--my-widget-border-radius`
   - `--my-widget-padding`
   ]]

3. **CSS parts / theming hooks**
   - If you expose `::part()` or `::theme()` selectors, document them and give examples

   Parts:
   [[e.g.
   - `::part(container)`
   - `::part(header)`
   - `::part(body)`
   - `::part(button)`
   ]]

4. **Layout integration**
   - Tips on placing the component within common layouts (flex, grid)
   - Responsive considerations

   Notes:
   [[e.g. width behavior, min/max width, responsiveness, mobile layout tips]]

---

## Framework / View-Engine Usage

On the main help page, include a section called **“Using with Frameworks”** with subsections and code examples for:

1. **Razor (ASP.NET MVC / Razor Pages)**  
   - How to reference the script (CDN or bundled)
   - Example `.cshtml` markup using the web component
   - Example of passing data via attributes
   - Any caveats (e.g. self-closing tags, encoding issues)

2. **Blazor**
   - How to include the script in a Blazor app
   - Example Razor component (`.razor`) using the web component
   - Example of binding data or parameters to attributes
   - Notes on interop if needed (e.g. JS interop for events)

3. **React**
   - How to use the web component inside JSX/TSX
   - Import/registration requirements (if any)
   - Example React component showing:
     - Minimal usage
     - Passing props/attributes
     - Handling events (e.g. using `useEffect` with `addEventListener`)

4. **Angular**
   - How to configure Angular to work with custom elements (e.g. `CUSTOM_ELEMENTS_SCHEMA`)
   - Example of using the component in an Angular template
   - Notes on binding to attributes and handling events

For each framework subsection:
- Provide at least one **complete, copy-pastable code example**.
- Use `<pre><code>` blocks.

---

## Page & Documentation Structure

You may produce **one or multiple HTML files** depending on component complexity.

### 1. Main Help Page (required)

The **main help page** is the entry-point for documentation.

It must include:
- A **left sidebar “Quick Links”** with anchor links to main sections on that page.
- Main content on the right.

Recommended sections on the main page:
- Overview
- Installation & Setup
- Basic Usage
- Attributes & Options (summary and link to full reference if separate page)
- Events (summary)
- Slots (summary)
- CSS & Theming (summary)
- Using with Frameworks (Razor, Blazor, React, Angular)
- **Examples** (with links to separate example HTML files in the help folder)
- Accessibility
- FAQ (optional)

Implementation details:
- Left sidebar using `<nav>` with links (`<a href="#section-id">`) to section IDs.
- Main content area using `<main>`, `<section>`, `<article>`, `<h1>`–`<h3>`.

In the **Examples** section of the main page:
- Provide short inline examples.
- Provide a **list of links** to the generated example HTML files (see “Example Files” below).

### 2. Additional Help Pages (optional)

If the component is complex enough, create additional HTML documentation pages such as:

- `[[component-name]]-attributes.html` — detailed attribute reference
- `[[component-name]]-examples.html` — extended examples and explanations
- `[[component-name]]-frameworks.html` — long, detailed framework-specific guides
- `[[component-name]]-accessibility.html` — deep accessibility guidance

Each additional page:
- Must be a valid standalone HTML document.
- Does **not** require a left sidebar (optional).
- Must be linked from the main help page (for example under “Further Reading” or relevant sections).

---

## Example Files (runnable demos in the help folder)

In addition to help pages, create **standalone example HTML files** that a developer can open directly in a browser.

Generate at least these example files:

1. **Basic example**
   - Filename pattern: `[[base name]]-example-basic.html`
   - Demonstrates minimal setup and usage of the component.

2. **Advanced / themed example**
   - Filename pattern: `[[base name]]-example-advanced.html`
   - Demonstrates multiple attributes, events, and some CSS customization.

Optionally, if appropriate to the component, also create framework-oriented examples:

3. `[[base name]]-example-react.html`  
4. `[[base name]]-example-angular.html`  
5. `[[base name]]-example-razor.html`  
6. `[[base name]]-example-blazor.html`  

Each example file must:
- Be a full HTML document (`<!DOCTYPE html><html>...`).
- Include any necessary `<script>` tags to load the web component (CDN or local).
- Contain one or more usage examples of the component.
- Have minimal inline CSS just to make the example readable.

In the **Examples** section of the main help page:
- Add a bullet list (or table) of links to these example files, for example:
  - `<a href="[[folder]]/[[base name]]-example-basic.html">Basic example</a>`
  - `<a href="[[folder]]/[[base name]]-example-advanced.html">Advanced example</a>`
  - And links to framework examples if those files are generated.

---

## Output Location & Filenames

Assume the documentation will be written to disk as files.

- By default, all help files and example files must be stored in a folder named `help/`.
- Only use a different folder if an explicit folder path is provided by the user.

Use these values:

- Base filename prefix (without folder): `[[file base name, e.g. my-widget]]`
- Optional custom folder (if provided): `[[custom folder path, or leave empty to use help/]]`

### Filename pattern

**Main and help pages:**
- Main page: `[[base name]]-help.html`
- Attributes page (optional): `[[base name]]-attributes.html`
- Examples page (optional): `[[base name]]-examples.html`
- Frameworks page (optional): `[[base name]]-frameworks.html`
- Accessibility page (optional): `[[base name]]-accessibility.html`

**Example runnable files:**
- Basic example: `[[base name]]-example-basic.html`
- Advanced example: `[[base name]]-example-advanced.html`
- Optional framework-specific examples:
  - `[[base name]]-example-react.html`
  - `[[base name]]-example-angular.html`
  - `[[base name]]-example-razor.html`
  - `[[base name]]-example-blazor.html`

### Save-path lines

After each HTML document (help page or example file), output exactly **one plain text line** indicating where the file should be saved:

- If no custom folder is provided:  
  `Save this file as: help/[[file name]]`

- If a custom folder is provided (for example `docs/components`):  
  `Save this file as: [[custom folder]]/[[file name]]`

Do this for **every** HTML file you generate.

---

## Output Format

For each HTML file:

- Output **only valid, self-contained HTML** (no Markdown) for that file.
- Immediately after the closing `</html>` tag, output the single “Save this file as: …” line.
- Then proceed to the next HTML file (if any), in the same format.

Styling rules:
- For the main help page:
  - Include a `<style>` block in the `<head>` that defines:
    - A layout with a left sidebar (quick links) and main content area.
    - A clear visual hierarchy for headings and tables.
- For other pages and example files:
  - Use minimal CSS, enough for readability.
- Do **not** use external CSS frameworks (no Bootstrap, no Tailwind).

---

## Tone & Style

- Documentation should be concise, technical, and friendly.
- Assume the reader is a front-end developer who understands HTML/CSS/JS basics.
- Use consistent terminology (e.g. always “attributes” rather than mixing “attributes” and “props” randomly).
- Prefer bullet lists and tables for structured information.
- Make examples realistic and framework-specific where appropriate.

Now, using all of the instructions above and the provided component details, generate:
- The main HTML help page,
- Any additional HTML help pages you judge necessary for complex components, and
- The example HTML files described above,  

each followed by its respective “Save this file as: …” line.
8 
