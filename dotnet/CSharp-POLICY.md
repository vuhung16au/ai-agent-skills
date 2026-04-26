# CSharp-POLICY.md

## Purpose

Define standards for writing clean, maintainable, and modern C# code. This policy governs agent behavior when generating or modifying C# in any .NET context including APIs, services, libraries, and application code.

---

## Scope / When to Use

Apply this policy to all C# code generation, including:
- .NET 6+ application code.
- Class libraries and NuGet packages.
- ASP.NET Core controllers, services, and middleware.
- Console applications and background workers.

---

## Core Principles

- **Clarity over cleverness.** Write code that the next developer can understand without asking questions.
- **Null safety is explicit.** Enable nullable reference types and handle nulls at boundaries.
- **Async all the way down.** Async patterns are the default for I/O-bound work.
- **Dependency injection is the standard composition model.**
- **Fail fast, fail clearly.** Validate early and surface descriptive errors.

---

## Required Behavior

### Language and SDK Version
- Target .NET 8 LTS or the version specified by the project. Do not silently downgrade.
- Enable nullable reference types: `<Nullable>enable</Nullable>` in the `.csproj`.
- Enable implicit usings: `<ImplicitUsings>enable</ImplicitUsings>`.
- Use file-scoped namespaces.

### Naming Conventions
- `PascalCase` for types, methods, properties, and public fields.
- `camelCase` for local variables and method parameters.
- `_camelCase` for private instance fields.
- `IPascalCase` for interfaces.
- `TPascalCase` for generic type parameters.
- `UPPER_SNAKE_CASE` is not used in C# — use `PascalCase` for constants too.

```csharp
public class UserService
{
    private readonly IUserRepository _repository;

    public UserService(IUserRepository repository)
    {
        _repository = repository;
    }

    public async Task<User?> GetByIdAsync(Guid id, CancellationToken cancellationToken = default)
    {
        return await _repository.FindAsync(id, cancellationToken);
    }
}
```

### Nullability
- Annotate all method signatures with nullable return types (`T?`) where null is a valid return.
- Use null-conditional (`?.`) and null-coalescing (`??`, `??=`) operators.
- Use `ArgumentNullException.ThrowIfNull()` for guard clauses on public method parameters.
- Do not suppress nullable warnings with `!` (null-forgiving operator) without a comment justifying it.

### Async Patterns
- Suffix all async methods with `Async`.
- Return `Task` or `Task<T>`, not `void`, from async methods (except event handlers).
- Accept `CancellationToken` as the last parameter in all async public methods.
- Do not use `.Result` or `.Wait()` to synchronously block on a `Task` — use `await`.
- Use `ConfigureAwait(false)` in library code (not in ASP.NET Core application code where context is needed).

### Dependency Injection
- Register services in `Program.cs` or dedicated extension methods on `IServiceCollection`.
- Use constructor injection as the default — avoid `ServiceLocator` or static access to `IServiceProvider`.
- Use `IOptions<T>` for configuration binding — do not inject `IConfiguration` directly into services.
- Choose service lifetime deliberately: `Singleton` only for stateless services, `Scoped` for per-request, `Transient` for short-lived.

### Error Handling
- Define domain-specific exception types for expected failure modes.
- Use `try/catch` only at boundaries where you can meaningfully handle or transform an exception.
- Do not catch `Exception` broadly unless at the top-level global handler.
- Log exception context (operation name, relevant IDs) before rethrowing or wrapping.

### Records and Value Types
- Use `record` for immutable data transfer objects and value objects.
- Use `readonly struct` for small, frequently allocated value types.
- Prefer `record` over classes for DTOs — it provides equality semantics by default.

### Collections and LINQ
- Use LINQ for readable query expressions — avoid verbose loops when LINQ is clearer.
- Do not use LINQ on `IEnumerable<T>` when an `IQueryable<T>` provider could push the query to the database.
- Use `IReadOnlyList<T>` or `IReadOnlyCollection<T>` for return types when mutation is not intended.

### Testing
- Write unit tests with xUnit.
- Use Moq or NSubstitute for mocking.
- Test public behavior, not implementation details.
- Arrange-Act-Assert is the required test structure.

---

## Things to Avoid

- Nullable warnings suppressed with `!` without explanation.
- Synchronous blocking on async code (`.Result`, `.Wait()`).
- Public mutable fields — use properties.
- `static` methods and classes for stateful or I/O-bound logic — use DI instead.
- `dynamic` type unless interfacing with a dynamic external system.
- God classes with more than one responsibility.
- Catching exceptions and returning `null` silently.

---

## Output Expectations

Generated C# code must:
- Target .NET 8 or the specified version.
- Enable nullable reference types.
- Follow naming conventions consistently.
- Use async/await correctly with `CancellationToken`.
- Use constructor injection for dependencies.
- Not contain hardcoded connection strings, secrets, or environment-specific values.

---

## Optional Checklist

- [ ] `<Nullable>enable</Nullable>` in `.csproj`.
- [ ] All async methods suffixed with `Async` and return `Task`/`Task<T>`.
- [ ] `CancellationToken` accepted in all public async methods.
- [ ] No `.Result` or `.Wait()` on `Task`.
- [ ] Dependencies injected via constructor.
- [ ] DTOs defined as `record` types.
- [ ] Guard clauses use `ArgumentNullException.ThrowIfNull()`.
- [ ] No public mutable fields.
- [ ] Unit tests exist and use Arrange-Act-Assert structure.
