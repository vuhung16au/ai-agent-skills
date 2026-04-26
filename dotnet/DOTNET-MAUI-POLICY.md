# DOTNET-MAUI-POLICY.md

## Purpose

Define standards for building cross-platform .NET MAUI applications. This policy governs agent behavior when generating MAUI UI code, layouts, view models, and platform-specific adaptations.

---

## Scope / When to Use

Apply this policy when generating or modifying:
- .NET MAUI pages and views (`.xaml` and `.xaml.cs` files).
- ViewModels and data binding code.
- Shell navigation and app structure.
- Platform-specific code and handlers.
- Shared and platform-specific resources.

---

## Core Principles

- **MVVM is the standard architecture.** Keep views free of business logic.
- **Cross-platform first; platform-specific second.** Use platform handlers only when MAUI's abstractions are insufficient.
- **Responsive and adaptive layouts.** UI must work on phone, tablet, and desktop form factors.
- **Async all the way.** Never block the UI thread.
- **Testability drives design.** ViewModels must be testable without a running device.

---

## Required Behavior

### Project Structure
- Use the default MAUI single-project structure.
- Organize code into `Pages/`, `ViewModels/`, `Services/`, `Models/`, and `Controls/` folders.
- Place platform-specific code in `Platforms/<Platform>/` subdirectories.
- Keep `MauiProgram.cs` focused on app registration; delegate configuration to extension methods.

### MVVM Pattern
- Every Page must have a corresponding ViewModel.
- ViewModels must implement `INotifyPropertyChanged` — use a base class or `CommunityToolkit.Mvvm` source generators.
- Use `[ObservableProperty]` and `[RelayCommand]` from `CommunityToolkit.Mvvm` to reduce boilerplate.
- Never access UI elements directly from the ViewModel — communicate via bindings and commands.
- ViewModels must not reference MAUI or platform types directly (exception: navigation services that abstract MAUI Shell).

```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

public partial class HomeViewModel : ObservableObject
{
    [ObservableProperty]
    private string _title = "Home";

    [RelayCommand]
    private async Task LoadDataAsync()
    {
        // fetch data, update properties
    }
}
```

### Data Binding
- Bind all UI state to ViewModel properties — do not set control properties in code-behind except for animation or platform-specific adjustments.
- Use `x:Bind` (compiled bindings) where available for performance.
- Set `BindingContext` in XAML or via Shell's dependency injection, not in `OnAppearing`.

### Navigation
- Use Shell navigation (`Shell.Current.GoToAsync`) for routing between pages.
- Define routes in `AppShell.xaml` or register them in `MauiProgram.cs`.
- Pass data via route parameters or a navigation service — not via static properties or singletons.
- Handle deep linking and back navigation explicitly.

### Layouts and Responsiveness
- Use `Grid` and `FlexLayout` for responsive layouts; avoid `StackLayout` for complex multi-column arrangements.
- Use `OnIdiom` and `OnPlatform` markup extensions to adapt layouts for phone, tablet, and desktop.
- Do not hardcode absolute sizes for UI elements — use `*` grid columns, proportional sizes, and `LayoutOptions`.
- Test layouts on both phone and tablet idioms.

```xaml
<Grid ColumnDefinitions="*, 2*" RowDefinitions="Auto, *">
    <Label Grid.Row="0" Grid.Column="0" Text="{Binding Title}" />
</Grid>
```

### Async and Threading
- All long-running operations (network, database, file I/O) must be async.
- Use `MainThread.InvokeOnMainThreadAsync()` only when you must update UI from a background thread.
- Never call `.Result` or `.Wait()` on a `Task` from the UI thread.
- Accept `CancellationToken` in service methods and honor cancellation when navigation leaves a page.

### Dependency Injection
- Register all services, pages, and ViewModels in `MauiProgram.cs` using `builder.Services`.
- Inject ViewModel into the Page via constructor injection or service locator (Shell DI pattern).
- Use `AddTransient` for pages and ViewModels that should not persist across navigations.
- Use `AddSingleton` only for truly app-wide services (e.g., authentication state, settings).

### Resources and Theming
- Define all colors, fonts, and styles in `Resources/Styles/` XAML files.
- Use `AppThemeBinding` to support light and dark themes.
- Do not hardcode colors or font sizes inline in XAML.
- Use named `Style` resources for repeated control configurations.

### Platform-Specific Code
- Use conditional compilation (`#if ANDROID`, `#if IOS`) only as a last resort.
- Prefer MAUI handlers and platform behaviors for platform-specific customization.
- Isolate platform-specific service implementations behind interfaces registered conditionally in `MauiProgram.cs`.

---

## Things to Avoid

- Business logic or service calls in `.xaml.cs` code-behind.
- Direct access to UI controls from ViewModels.
- Hardcoded pixel sizes in XAML.
- Blocking async operations on the UI thread.
- Static properties or singleton state for navigation data.
- Duplicate layout definitions for each platform instead of using `OnIdiom`/`OnPlatform`.
- Missing cancellation token support in async operations tied to page lifecycle.

---

## Output Expectations

Generated MAUI code must:
- Follow MVVM with ViewModels free of MAUI/platform type references.
- Use `CommunityToolkit.Mvvm` source generators for properties and commands.
- Use Shell navigation for routing.
- Apply `OnIdiom`/`OnPlatform` for adaptive layouts.
- Register all services and pages via `builder.Services` in `MauiProgram.cs`.
- Not contain blocking async patterns.

---

## Optional Checklist

- [ ] ViewModels inherit from `ObservableObject` (CommunityToolkit.Mvvm).
- [ ] `[ObservableProperty]` and `[RelayCommand]` used to reduce boilerplate.
- [ ] No UI control access from ViewModel.
- [ ] Shell navigation used for routing.
- [ ] Route parameters used for data passing.
- [ ] `OnIdiom` or `OnPlatform` used for adaptive layouts.
- [ ] No hardcoded pixel sizes.
- [ ] Resources defined in `Resources/Styles/`.
- [ ] `AppThemeBinding` used for light/dark theme support.
- [ ] All I/O operations are async with `CancellationToken`.
