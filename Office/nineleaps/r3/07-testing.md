# Testing — Round 3 Prep (Uber)

---

## 1. Unit Testing vs Integration Testing

**Unit Test** — tests a single function or component in isolation. Dependencies are mocked.
```
Test: does this function return correct output for given input?
Scope: one function / one component
Speed: very fast
```

**Integration Test** — tests how multiple units work together. Closer to real behavior.
```
Test: does this form submit correctly and show success message?
Scope: multiple components + API calls
Speed: slower than unit tests
```

**E2E Test** — tests the full app from user's perspective in a real browser.
```
Test: can a user log in, search for a movie, and book a ticket?
Scope: entire application
Speed: slowest
```

---

## 2. Unit Testing — Vitest + React Testing Library

**React Testing Library (RTL)** — test components the way users interact with them. No implementation details.

```javascript
// Button.test.jsx
import { render, screen, fireEvent } from "@testing-library/react";
import Button from "./Button";

test("calls onClick when clicked", () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByText("Click me"));

    expect(handleClick).toHaveBeenCalledTimes(1);
});
```

**Testing a component with state:**
```javascript
import { render, screen, fireEvent } from "@testing-library/react";
import Counter from "./Counter";

test("increments count on button click", () => {
    render(<Counter />);
    const button = screen.getByText("+");

    fireEvent.click(button);
    fireEvent.click(button);

    expect(screen.getByText("2")).toBeInTheDocument();
});
```

**Testing async data fetching:**
```javascript
import { render, screen, waitFor } from "@testing-library/react";
import UserList from "./UserList";

// Mock fetch
global.fetch = jest.fn(() =>
    Promise.resolve({
        json: () => Promise.resolve([{ id: 1, name: "Alice" }])
    })
);

test("renders users from API", async () => {
    render(<UserList />);

    expect(screen.getByText("Loading...")).toBeInTheDocument();

    await waitFor(() => {
        expect(screen.getByText("Alice")).toBeInTheDocument();
    });
});
```

**Common RTL queries:**
```javascript
screen.getByText("Submit")         // throws if not found
screen.queryByText("Submit")       // returns null if not found
screen.findByText("Submit")        // async, waits for element
screen.getByRole("button")
screen.getByLabelText("Email")
screen.getByPlaceholderText("Search...")
screen.getByTestId("submit-btn")   // use data-testid attribute
```

---

## 3. Integration Testing

Integration tests test multiple components + their interactions.

```javascript
// LoginFlow.test.jsx — tests login form + API + redirect
import { render, screen, fireEvent, waitFor } from "@testing-library/react";
import { MemoryRouter } from "react-router-dom";
import App from "./App";

// Mock the API module
jest.mock("./api", () => ({
    login: jest.fn(() => Promise.resolve({ token: "abc123", user: { name: "Alice" } }))
}));

test("user can log in and see dashboard", async () => {
    render(<MemoryRouter initialEntries={["/login"]}><App /></MemoryRouter>);

    fireEvent.change(screen.getByLabelText("Email"), {
        target: { value: "alice@example.com" }
    });
    fireEvent.change(screen.getByLabelText("Password"), {
        target: { value: "password123" }
    });
    fireEvent.click(screen.getByRole("button", { name: "Login" }));

    await waitFor(() => {
        expect(screen.getByText("Welcome, Alice")).toBeInTheDocument();
    });
});
```

---

## 4. Playwright (E2E Testing)

Playwright automates a real browser. Tests what users actually experience.

**Setup:**
```bash
npm install -D @playwright/test
npx playwright install
```

**Basic test:**
```javascript
// tests/login.spec.js
import { test, expect } from "@playwright/test";

test("user can log in", async ({ page }) => {
    await page.goto("http://localhost:3000/login");

    await page.fill('[placeholder="Email"]', "alice@example.com");
    await page.fill('[placeholder="Password"]', "password123");
    await page.click('button:has-text("Login")');

    await expect(page).toHaveURL("http://localhost:3000/dashboard");
    await expect(page.locator("h1")).toHaveText("Welcome, Alice");
});
```

**More Playwright features:**
```javascript
// Take screenshot
await page.screenshot({ path: "screenshot.png" });

// Wait for network request
await page.waitForResponse("/api/user");

// Intercept API calls (mock in E2E)
await page.route("/api/user", route =>
    route.fulfill({ json: { name: "Alice" } })
);

// Multiple browsers
test.use({ browserName: "firefox" });

// Mobile viewport
test.use({ viewport: { width: 375, height: 667 } });
```

**Run tests:**
```bash
npx playwright test                    # all tests
npx playwright test login.spec.js      # specific file
npx playwright test --headed           # see browser
npx playwright show-report             # view HTML report
```

---

## 5. Testing Libraries Summary

| Library | Type | Use For |
|---|---|---|
| **Jest** | Test runner + assertions | Unit, integration |
| **Vitest** | Faster Jest alternative | Unit, integration (Vite projects) |
| **React Testing Library** | Component testing | Test React components like users |
| **MSW (Mock Service Worker)** | API mocking | Mock API calls in tests |
| **Playwright** | E2E | Full browser automation |
| **Cypress** | E2E | Alternative to Playwright |

---

## 6. Testing Best Practices

- Test behavior, not implementation details
- Use `getByRole`, `getByLabelText` over `getByTestId` when possible
- One assertion per test (or logically related assertions)
- Mock external dependencies (API calls, timers)
- Don't test library code — test your code

```javascript
// Bad — testing implementation detail
expect(component.state.count).toBe(1);

// Good — testing user-visible behavior
expect(screen.getByText("Count: 1")).toBeInTheDocument();
```
