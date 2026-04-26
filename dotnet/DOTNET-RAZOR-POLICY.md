# DOTNET-RAZOR-POLICY.md

## Purpose

Define standards for building clean, maintainable, and accessible server-rendered UIs using ASP.NET Core Razor Pages. This policy governs agent behavior when generating Razor Pages, partials, view components, and associated code-behind files.

---

## Scope / When to Use

Apply this policy when generating or modifying:
- Razor Pages (`.cshtml` and `.cshtml.cs` files).
- Partial views and view components.
- Tag helpers and HTML helpers in Razor syntax.
- ASP.NET Core MVC views when used alongside Razor Pages.

---

## Core Principles

- **Separation of concerns.** Page logic belongs in the PageModel (code-behind), not in `.cshtml`.
- **Thin pages, rich services.** PageModels delegate business logic to injected services.
- **Validation at the model level.** Use data annotations and FluentValidation — not manual if/else chains.
- **Accessibility is required.** Generate semantic, keyboard-navigable HTML.
- **Reuse via partials and components.** Do not copy-paste Razor markup.

---

## Required Behavior

### Page Structure
- Each Razor Page consists of a `.cshtml` file and a `.cshtml.cs` PageModel class.
- The PageModel must inherit from `PageModel`.
- Keep `.cshtml` files focused on layout and rendering; delegate all logic to the PageModel or services.
- Use `OnGetAsync` and `OnPostAsync` (or HTTP verb-specific handlers) consistently.

```csharp
public class EditUserModel : PageModel
{
    private readonly IUserService _userService;

    [BindProperty]
    public EditUserInputModel Input { get; set; } = default!;

    public EditUserModel(IUserService userService)
    {
        _userService = userService;
    }

    public async Task<IActionResult> OnGetAsync(Guid id)
    {
        var user = await _userService.GetByIdAsync(id);
        if (user is null) return NotFound();
        Input = new EditUserInputModel { Name = user.Name, Email = user.Email };
        return Page();
    }

    public async Task<IActionResult> OnPostAsync(Guid id)
    {
        if (!ModelState.IsValid) return Page();
        await _userService.UpdateAsync(id, Input);
        return RedirectToPage("./Index");
    }
}
```

### Binding and Validation
- Use `[BindProperty]` for form-bound properties.
- Define a dedicated input model class for each form — do not bind to domain entities directly.
- Use data annotations (`[Required]`, `[MaxLength]`, `[EmailAddress]`) on input models.
- Always check `ModelState.IsValid` before processing a POST.
- Display validation errors using Tag Helpers: `<span asp-validation-for="Input.Field"></span>`.
- Include `<div asp-validation-summary="ModelOnly"></div>` on forms with model-level errors.

### Razor Syntax
- Use Tag Helpers (`asp-for`, `asp-action`, `asp-route-*`) instead of HTML Helpers (`@Html.TextBoxFor`) in new code.
- Use `@model` directive at the top of every `.cshtml` file that binds to a model.
- Use partial views (`<partial name="..." />`) for reusable markup blocks.
- Use View Components for reusable UI elements that require server-side logic.

### Layouts and Sections
- Define shared layout in `_Layout.cshtml`.
- Use `@RenderSection("Scripts", required: false)` for page-specific scripts injected at the bottom of the layout.
- Add page-specific CSS and scripts in the appropriate section, not inline in the body.

### Accessibility
- Use semantic HTML elements (`<form>`, `<fieldset>`, `<legend>`, `<label>`) for all forms.
- Associate every `<input>` with a `<label>` using `asp-for` (which generates `for`/`id` attributes automatically).
- Ensure error messages are associated with inputs via `aria-describedby` where Tag Helpers do not do this automatically.
- All interactive elements must be keyboard-operable.

### Routing
- Use `@page` directive with explicit route templates for pages that require route parameters.
- Use named routes for redirects — do not hardcode URL strings.
- Use `RedirectToPage` for POST-Redirect-GET patterns; always redirect after a successful POST.

### Anti-Forgery
- All POST forms must include `@Html.AntiForgeryToken()` or use the Tag Helper equivalent (`<form asp-antiforgery="true">`).
- Do not disable anti-forgery validation on POST endpoints that modify state.

---

## Things to Avoid

- Business logic or database access directly in `.cshtml` files.
- Binding `[BindProperty]` to domain or database entity types.
- Using `@Html.Raw()` with user-supplied input (XSS risk).
- Copy-pasting Razor markup across pages — extract to partials or view components.
- Hardcoded URLs in Razor files — use `asp-page`, `asp-action`, or `Url.Page()`.
- Missing `ModelState.IsValid` check before processing POST data.
- POST handlers without redirect on success (breaks the PRG pattern).

---

## Output Expectations

Generated Razor Pages code must:
- Separate markup (`.cshtml`) from logic (`.cshtml.cs` PageModel).
- Use `[BindProperty]` with a dedicated input model for forms.
- Validate with data annotations and check `ModelState.IsValid`.
- Use Tag Helpers for form elements and validation messages.
- Include anti-forgery tokens on all POST forms.
- Follow POST-Redirect-GET on successful form submissions.
- Use semantic HTML for forms and interactive elements.

---

## Optional Checklist

- [ ] PageModel inherits from `PageModel`.
- [ ] Business logic delegated to injected services.
- [ ] Dedicated input model for each form (not domain entity).
- [ ] `[BindProperty]` used on form-bound properties.
- [ ] `ModelState.IsValid` checked in all POST handlers.
- [ ] Tag Helpers used for form elements.
- [ ] Validation messages wired up with `asp-validation-for`.
- [ ] Anti-forgery tokens included on all forms.
- [ ] Successful POST redirects (PRG pattern).
- [ ] No `@Html.Raw()` with user input.
