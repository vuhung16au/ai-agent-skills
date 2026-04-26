# DOTNET-BLAZOR-POLICY.md

## Purpose

Define standards for building component-driven, performant, and maintainable Blazor applications. This policy governs agent behavior when generating Blazor Server or Blazor WebAssembly components and application code.

---

## Scope / When to Use

Apply this policy when generating or modifying:
- Blazor Server or Blazor WebAssembly components (`.razor` files).
- Blazor component code-behind (`.razor.cs` files) and inline `@code` blocks.
- Service registrations and DI configuration for Blazor apps.
- State management and component communication patterns.

---

## Core Principles

- **Components are the unit of composition.** Build small, focused, reusable components.
- **Separate markup from logic.** Use code-behind (`.razor.cs`) for non-trivial logic.
- **Async is the default for I/O.** Never block the UI thread.
- **State flows down, events flow up.** Use `[Parameter]` and `EventCallback` for parent-child communication.
- **Performance is a design constraint, not an afterthought.**

---

## Required Behavior

### Component Design
- Each `.razor` file defines one component.
- Keep components focused on a single responsibility; extract child components when markup grows beyond ~100 lines.
- Use code-behind files (`.razor.cs`) for components with non-trivial logic; use inline `@code` blocks only for simple components.
- Name components with `PascalCase` identifiers.

### Parameters and Communication
- Declare component parameters with `[Parameter]` on public properties.
- Use `[Parameter, EditorRequired]` for parameters that are always required.
- Use `EventCallback<T>` (not `Action<T>` or `Func<T>`) for child-to-parent callbacks.
- Use `CascadingParameter` only for values that are genuinely ambient (e.g., authentication state, theme) — not for passing data to direct children.

```razor
@code {
    [Parameter, EditorRequired]
    public string Title { get; set; } = default!;

    [Parameter]
    public EventCallback<string> OnSave { get; set; }

    private async Task HandleSave()
    {
        await OnSave.InvokeAsync(_inputValue);
    }
}
```

### Async and Lifecycle
- Override `OnInitializedAsync` for async initialization — not `OnInitialized`.
- Await all async operations; do not fire-and-forget inside lifecycle methods.
- Call `StateHasChanged()` only when state has changed outside the normal Blazor event cycle (e.g., from a background thread or event subscription).
- Dispose event subscriptions and external resources by implementing `IDisposable` or `IAsyncDisposable`.

```razor
@implements IAsyncDisposable

@code {
    private IDisposable? _subscription;

    protected override async Task OnInitializedAsync()
    {
        _subscription = await SomeService.SubscribeAsync(OnUpdate);
    }

    public async ValueTask DisposeAsync()
    {
        _subscription?.Dispose();
    }
}
```

### Rendering Optimization
- Add `@key` directives to items in rendered lists to enable efficient diffing.
- Use `ShouldRender()` override to prevent unnecessary re-renders for purely display components.
- Avoid creating large object allocations in `OnParametersSetAsync` for components that receive frequently-changing parameters.
- In Blazor WebAssembly, use lazy loading for large component assemblies.

### Forms and Validation
- Use `<EditForm>` with `DataAnnotationsValidator` and `ValidationSummary` for form validation.
- Bind form inputs to a dedicated model class, not directly to domain entities.
- Use `OnValidSubmit` to process form data — not `OnSubmit` without validation.

```razor
<EditForm Model="@_input" OnValidSubmit="HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />
    <InputText @bind-Value="_input.Name" />
    <button type="submit">Save</button>
</EditForm>
```

### State Management
- Use scoped services for state shared within a user session (Blazor Server) or app lifetime (Blazor WASM).
- Notify components of state changes using events or `INotifyPropertyChanged` rather than polling.
- For complex cross-component state, use the Flux/Redux pattern (e.g., Fluxor) or a simple observable state container.
- Do not share state through static fields.

### Dependency Injection
- Register services using the appropriate lifetime in `Program.cs`.
- Inject services via `[Inject]` in components or constructor injection in code-behind classes.
- Do not access `IServiceProvider` directly inside components.

### JavaScript Interop (JSInterop)
- Keep JSInterop calls minimal and isolated in a dedicated service.
- Always handle `JSDisconnectedException` and `TaskCanceledException` from JSInterop calls in Blazor Server.
- Clean up JavaScript resources using `IJSObjectReference` disposal patterns.

---

## Things to Avoid

- Placing complex business logic directly in `.razor` markup.
- Using `Action<T>` or `Func<T>` for component callbacks — use `EventCallback<T>`.
- Calling `StateHasChanged()` from within lifecycle methods unnecessarily.
- Blocking on async code with `.Result` or `.Wait()`.
- Subscribing to external events without implementing `IDisposable`.
- Static mutable fields for shared state.
- Overly deep component trees with excessive cascading parameters.

---

## Output Expectations

Generated Blazor code must:
- Separate markup and logic into `.razor` and `.razor.cs` files for non-trivial components.
- Use `[Parameter, EditorRequired]` for required parameters.
- Use `EventCallback<T>` for all callbacks.
- Implement `IDisposable` or `IAsyncDisposable` for components with subscriptions.
- Use `<EditForm>` with validation for all user input forms.
- Not contain blocking async calls.

---

## Optional Checklist

- [ ] Components have a single responsibility.
- [ ] Code-behind used for non-trivial logic.
- [ ] `[Parameter, EditorRequired]` on required parameters.
- [ ] `EventCallback<T>` used for child-to-parent callbacks.
- [ ] `OnInitializedAsync` used for async initialization.
- [ ] `IDisposable`/`IAsyncDisposable` implemented where needed.
- [ ] `@key` on list items.
- [ ] `<EditForm>` with `DataAnnotationsValidator` for forms.
- [ ] No `.Result` or `.Wait()` on `Task`.
- [ ] State shared via scoped services, not static fields.
