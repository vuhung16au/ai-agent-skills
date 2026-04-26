# SPRINGBOOT-BACKEND-API-POLICY.md

## Purpose

Define standards for building clean, testable, and production-ready REST APIs with Spring Boot. This policy governs agent behavior when generating Spring Boot backend API code, including controllers, services, repositories, DTOs, and configuration.

---

## Scope / When to Use

Apply this policy when generating or modifying:
- Spring Boot 3.x REST APIs.
- Spring MVC controllers and service layers.
- Spring Data JPA repositories.
- DTO and validation classes.
- Exception handling and global error responses.
- Configuration and observability setup.

---

## Core Principles

- **Layered architecture is non-negotiable.** Controller → Service → Repository. No layer skipping.
- **DTOs at the boundary.** Entities never leave the service layer.
- **Validation at the entry point.** Validate request DTOs before business logic executes.
- **Consistent error responses.** All errors return a structured, documented response body.
- **Testability is a design constraint.** Every layer must be independently testable.

---

## Required Behavior

### Project Structure
- Organize code by feature package: `com.example.app.<feature>.controller`, `...service`, `...repository`, `...dto`, `...entity`.
- Alternatively, use a layered package structure for small APIs: `controller/`, `service/`, `repository/`, `dto/`, `entity/`.
- Keep `main` class minimal — delegate configuration to `@Configuration` classes.

### Controllers
- Annotate controllers with `@RestController` and `@RequestMapping("/api/v1/<resource>")`.
- Use specific HTTP method annotations: `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`.
- Controllers receive and return DTOs only — never JPA entities.
- Validate request bodies with `@Valid` and method parameters with `@Validated` at the controller level.
- Return `ResponseEntity<T>` with explicit HTTP status codes.

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUser(@PathVariable UUID id) {
        return ResponseEntity.ok(userService.getById(id));
    }

    @PostMapping
    public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
        UserResponse created = userService.create(request);
        URI location = URI.create("/api/v1/users/" + created.id());
        return ResponseEntity.created(location).body(created);
    }
}
```

### Service Layer
- Annotate service classes with `@Service`.
- Services contain all business logic and orchestrate repository calls.
- Services accept and return DTOs — map to/from entities internally.
- Use `@Transactional` on service methods that perform multiple persistence operations.
- Throw domain-specific exceptions for expected error conditions.

### Repository Layer
- Extend `JpaRepository<Entity, ID>` or `CrudRepository<Entity, ID>`.
- Define custom queries with `@Query` using JPQL (preferred) or native SQL when necessary.
- Do not write SQL in service or controller layers.
- Use Spring Data projections or DTOs in repository queries to avoid loading full entities when only a subset of fields is needed.

### DTOs and Records
- Define request and response DTOs as Java `record` types (Java 17+).
- Use Bean Validation annotations (`@NotNull`, `@NotBlank`, `@Size`, `@Email`, `@Pattern`) on request DTOs.
- Create a separate response DTO for each resource — do not reuse request DTOs as responses.

```java
public record CreateUserRequest(
    @NotBlank @Size(max = 100) String name,
    @NotBlank @Email String email
) {}

public record UserResponse(UUID id, String name, String email, Instant createdAt) {}
```

### Exception Handling
- Define a `@RestControllerAdvice` class as the global exception handler.
- Map domain exceptions to HTTP status codes in the advice class.
- Return a consistent error response body: `{ "status": 400, "error": "Bad Request", "message": "...", "timestamp": "..." }`.
- Log exceptions at `WARN` level for client errors (4xx) and `ERROR` level for server errors (5xx).
- Do not expose stack traces or internal implementation details in error responses.

### Validation
- Use `@Valid` on `@RequestBody` parameters and `@Validated` on controllers for path/query parameter validation.
- Handle `MethodArgumentNotValidException` in the global exception handler and return field-level validation errors.
- Do not implement validation logic manually in controllers or services when Bean Validation covers the case.

### Configuration
- Externalize all configuration to `application.yml` (preferred over `.properties`).
- Use typed configuration properties with `@ConfigurationProperties(prefix = "...")`.
- Do not hardcode URLs, credentials, or environment-specific values in source code.
- Use Spring profiles (`application-dev.yml`, `application-prod.yml`) for environment-specific overrides.

### Observability
- Use SLF4J with Logback for all logging — never `System.out.println`.
- Log at appropriate levels: `DEBUG` for diagnostic details, `INFO` for significant events, `WARN` for handled errors, `ERROR` for unexpected failures.
- Include correlation IDs (request IDs) in log output using MDC.
- Expose Spring Actuator endpoints (`/actuator/health`, `/actuator/info`) in production.
- Integrate with Micrometer for metrics if the project uses a metrics backend.

### Testing
- Unit test services with JUnit 5 and Mockito.
- Integration test controllers with `@WebMvcTest` and MockMvc.
- Integration test the full stack with `@SpringBootTest` for critical flows.
- Test happy paths, validation failures, and not-found/error cases.

---

## Things to Avoid

- Returning JPA entities directly from controllers.
- Calling repositories directly from controllers (skipping the service layer).
- Business logic in controllers.
- `System.out.println` for logging.
- Hardcoded database credentials or URLs.
- Missing `@Valid` on request body parameters.
- Catching all exceptions in the service layer and returning null.
- Generic 500 responses for expected client errors.

---

## Output Expectations

Generated Spring Boot API code must:
- Follow controller → service → repository layering.
- Use DTOs (Java records) at the API boundary.
- Include Bean Validation annotations on request DTOs.
- Include a `@RestControllerAdvice` for exception handling.
- Use `@Transactional` on service methods with multiple persistence operations.
- Return `ResponseEntity<T>` with appropriate status codes.
- Not contain hardcoded credentials or environment-specific values.

---

## Optional Checklist

- [ ] Controller → Service → Repository layering maintained.
- [ ] Request and response DTOs defined as Java `record` types.
- [ ] Bean Validation annotations on all request DTOs.
- [ ] `@Valid` used on `@RequestBody` parameters.
- [ ] `@RestControllerAdvice` defined for global error handling.
- [ ] Consistent error response body format.
- [ ] `@Transactional` on service methods with multiple writes.
- [ ] SLF4J used for all logging.
- [ ] Configuration via `@ConfigurationProperties`.
- [ ] Unit tests for service layer with Mockito.
