# PLAYWRIGHT-TESTING-POLICY.md

## Purpose

Define standards for writing reliable, maintainable, and CI-ready end-to-end tests with Playwright. This policy governs agent behavior when generating test files, fixtures, selectors, and test configuration.

---

## Scope / When to Use

Apply this policy when generating or modifying:
- Playwright test files (`*.spec.ts`, `*.test.ts`).
- Page Object Models and fixtures.
- Playwright configuration (`playwright.config.ts`).
- CI pipeline steps that run Playwright tests.

---

## Core Principles

- **Tests must be deterministic.** A test that passes and fails non-deterministically provides negative value.
- **Isolation by default.** Each test starts with a clean, independent state.
- **Selectors reflect user intent.** Use what the user sees and interacts with, not implementation details.
- **Realistic scope.** E2E tests verify user-visible behavior end-to-end; do not test unit-level concerns with Playwright.
- **Debuggability is a first-class requirement.** Tests must be diagnosable when they fail.

---

## Required Behavior

### Selectors
- Use role-based locators as the first choice: `getByRole()`, `getByLabel()`, `getByPlaceholder()`, `getByText()`.
- Use `getByTestId()` as a fallback when semantic selectors are insufficient.
- Never use CSS class selectors or XPath unless absolutely no other option exists — they couple tests to implementation.
- Never use element index selectors (e.g., `nth(2)`) without a specific comment explaining the intent.
- Add `data-testid` attributes to components that need to be targeted but lack a stable accessible name.

```ts
// Preferred
await page.getByRole('button', { name: 'Submit' }).click();
await page.getByLabel('Email address').fill('user@example.com');

// Acceptable fallback
await page.getByTestId('submit-btn').click();

// Avoid
await page.locator('.btn-primary').click();
await page.locator('#form > div:nth-child(3) > input').fill('value');
```

### Test Isolation
- Each test must be self-contained — set up its own preconditions and clean up afterward.
- Do not share mutable state between tests. Use `test.beforeEach` for per-test setup.
- Use Playwright's `storageState` for authentication: log in once, save the storage state, and reuse it — do not log in inside each test.
- Reset database state before each test using API calls or fixtures, not by manually navigating through the UI.

### Fixtures
- Define custom fixtures in a `fixtures.ts` file and extend `test` from Playwright's `base`.
- Use fixtures for: authenticated page context, pre-created test data, and reusable component interactions.
- Keep fixtures focused — one fixture per concern.

```ts
import { test as base } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

type Fixtures = { loginPage: LoginPage };

export const test = base.extend<Fixtures>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
  },
});
```

### Page Object Model (POM)
- Create a Page Object class for each major page or significant UI component.
- Page Object methods represent user actions: `fillLoginForm()`, `submitOrder()`, `navigateToSettings()`.
- Assertions do not belong in Page Object classes — keep them in the test file.
- Page Objects should not contain `expect()` calls.

```ts
export class LoginPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.page.getByLabel('Email').fill(email);
    await this.page.getByLabel('Password').fill(password);
    await this.page.getByRole('button', { name: 'Sign in' }).click();
  }
}
```

### Assertions
- Use Playwright's `expect` with web-first assertions: `toBeVisible()`, `toHaveText()`, `toBeEnabled()`.
- Web-first assertions automatically wait for the condition — do not add manual `waitForSelector` before them.
- Be specific: assert what the user sees, not internal state.
- Do not assert on CSS properties unless visual regression testing.

```ts
await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
await expect(page.getByRole('alert')).toHaveText('File uploaded successfully');
```

### Waits and Timing
- Never use `page.waitForTimeout()` (a fixed delay) — always wait for a specific condition.
- Use `waitForURL()`, `waitForResponse()`, or Playwright's web-first assertion auto-waiting instead.
- If a test requires a specific network response to complete, intercept and wait for it explicitly.

```ts
// Bad
await page.waitForTimeout(2000);

// Good
await page.waitForURL('**/dashboard');
await expect(page.getByText('Welcome')).toBeVisible();
```

### Test Scope
- E2E tests must cover complete user flows: start from a realistic entry point, perform meaningful actions, assert on visible outcomes.
- Do not use Playwright to test individual React/Vue components in isolation — use unit or component tests for that.
- One test should cover one user scenario. Do not assert on more than one independent behavior per test.

### Configuration
- Set `baseURL` in `playwright.config.ts`.
- Configure `trace: 'on-first-retry'` to capture traces on failure.
- Configure `screenshot: 'only-on-failure'` and `video: 'retain-on-failure'` for CI diagnostics.
- Run tests in parallel across workers; ensure tests do not share state that prevents parallelism.
- Define at minimum a desktop browser project; add mobile viewports for responsive tests.

```ts
export default defineConfig({
  testDir: './e2e',
  use: {
    baseURL: process.env.BASE_URL ?? 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'mobile-safari', use: { ...devices['iPhone 14'] } },
  ],
});
```

### CI Integration
- Run `playwright install --with-deps` in CI before the test step.
- Use Playwright's HTML reporter and publish the report as a CI artifact.
- Use `--shard` for large test suites to parallelize across CI workers.
- Fail the CI build if any test fails — do not allow flaky tests to be skipped silently.

---

## Things to Avoid

- CSS class or implementation-detail selectors.
- `page.waitForTimeout()` for any timing control.
- Sharing mutable browser state or cookies across tests.
- Logging in through the UI in every test — use storage state.
- Assertions inside Page Object classes.
- Testing the same behavior in multiple tests unnecessarily.
- Skipping tests with `test.skip()` permanently without a tracking issue.

---

## Output Expectations

Generated Playwright tests must:
- Use role-based or accessible locators.
- Be fully isolated (no shared mutable state).
- Use `test.beforeEach` for per-test setup, not global mutable variables.
- Never use `waitForTimeout`.
- Use `expect` with web-first assertions.
- Include `playwright.config.ts` with `baseURL`, trace, screenshot, and video settings.
- Be runnable in CI with `playwright install --with-deps && npx playwright test`.

---

## Optional Checklist

- [ ] All selectors use `getByRole`, `getByLabel`, `getByText`, `getByPlaceholder`, or `getByTestId`.
- [ ] No CSS class or XPath selectors.
- [ ] No `waitForTimeout` calls.
- [ ] Tests isolated with `test.beforeEach` setup.
- [ ] Authentication uses stored storage state.
- [ ] Page Objects defined for major pages.
- [ ] Assertions use Playwright web-first assertions.
- [ ] `playwright.config.ts` includes trace, screenshot, and video on failure.
- [ ] Tests parallelizable (no shared mutable state).
- [ ] CI step includes `playwright install --with-deps`.
