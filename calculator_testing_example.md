# Testing a Calculator Web App: Unit, Integration, and System Tests

This document walks through all three testing levels for a simple calculator web app using the **React + Express (Node.js)** stack. The same concepts apply to Laravel and Flask — equivalent examples are shown where the approach differs significantly.

## App Description

The calculator app has:

- A **React frontend** with number/operator buttons and a display.
- An **Express REST API** backend with a single endpoint: `POST /calculate`.
- A **pure calculation module** (`calculate.js`) containing the business logic.

```
/calculator-app
  package.json              ← all dependencies and npm scripts
  jest.config.js            ← Jest configuration
  vite.config.js            ← Vite (frontend bundler) configuration
  playwright.config.js      ← Playwright configuration
  /frontend
    index.html              ← Vite HTML entry point
    /src
      main.jsx              ← React entry point
      App.jsx               ← root React component
      Calculator.jsx        ← calculator UI component
      calculate.js          ← pure business logic (no framework dependency)
  /backend
    app.js                  ← Express app (routes + middleware)
    server.js               ← starts the HTTP server
  /tests
    /unit
      calculate.test.js
    /integration
      api.test.js
    /system
      calculator.spec.js    ← Playwright
```

---

## Project Setup

> **Prerequisites:** [Node.js 18+](https://nodejs.org) (which includes npm) and Git must be installed. Confirm with `node -v` and `npm -v` in your terminal.

### Step 1 — Create the project folder and initialise npm

```bash
mkdir calculator-app
cd calculator-app
npm init -y
```

This creates a `package.json` with default values. You will replace its contents in Step 4.

---

### Step 2 — Create the full folder structure

Run these commands from inside `calculator-app/`:

```bash
# Application source
mkdir -p frontend/src
mkdir -p backend

# Test folders
mkdir -p tests/unit
mkdir -p tests/integration
mkdir -p tests/system
```

Your project should now look like this (empty folders):

```
calculator-app/
  frontend/
    src/
  backend/
  tests/
    unit/
    integration/
    system/
```

---

### Step 3 — Install dependencies

Install everything from the project root in one command:

```bash
npm install express
npm install --save-dev jest supertest @playwright/test vite @vitejs/plugin-react react react-dom concurrently
```

| Package                | Purpose                                              |
| ---------------------- | ---------------------------------------------------- |
| `express`              | Backend web framework                                |
| `jest`                 | Unit and integration test runner                     |
| `supertest`            | Makes HTTP requests against Express inside Jest      |
| `@playwright/test`     | Browser-based system test runner                     |
| `vite`                 | Frontend dev server and bundler                      |
| `@vitejs/plugin-react` | JSX support for Vite                                 |
| `react`, `react-dom`   | Frontend UI library                                  |
| `concurrently`         | Runs backend and frontend dev servers simultaneously |

After installing, download the Playwright browser binaries (Chromium, Firefox, WebKit):

```bash
npx playwright install
```

---

### Step 4 — Configure `package.json`

Replace the entire contents of `package.json` with:

```json
{
  "name": "calculator-app",
  "version": "1.0.0",
  "scripts": {
    "dev:backend": "node backend/server.js",
    "dev:frontend": "vite",
    "dev": "concurrently \"npm run dev:backend\" \"npm run dev:frontend\"",
    "test:unit": "jest tests/unit",
    "test:integration": "jest tests/integration",
    "test:system": "playwright test",
    "test": "npm run test:unit && npm run test:integration && npm run test:system"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "@playwright/test": "^1.43.0",
    "@vitejs/plugin-react": "^4.2.1",
    "concurrently": "^8.2.2",
    "jest": "^29.7.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "supertest": "^6.3.4",
    "vite": "^5.2.0"
  }
}
```

---

### Step 5 — Configure Jest

Create `jest.config.js` at the project root:

```js
// jest.config.js
module.exports = {
  // Only pick up unit and integration tests — Playwright handles system tests
  testMatch: [
    "**/tests/unit/**/*.test.js",
    "**/tests/integration/**/*.test.js",
  ],
  testEnvironment: "node",
};
```

---

### Step 6 — Configure Vite

**What is Vite?**

Vite is the tool that serves and builds your React frontend. It does two jobs:

1. **Dev server** — When you run `npm run dev:frontend`, Vite starts a local server (port 3000) that serves your React app. Whenever you save a file, the browser updates instantly without a full page reload. This is called Hot Module Replacement (HMR).
2. **Bundler** — When you eventually build for production (`npx vite build`), Vite compiles all your JSX/JS files into plain HTML, CSS, and JS that any browser can run — no React or JSX syntax remains in the output.

Without Vite (or an equivalent tool like Create React App / webpack), the browser cannot understand JSX syntax like `<Calculator />` directly.

**What is the proxy for?**

During development you have two servers running on different ports:

- Port 3000 → Vite (React frontend)
- Port 3001 → Express (backend API)

When your React code calls `fetch('/calculate', ...)`, browsers apply the **Same-Origin Policy**: a page loaded from port 3000 is not normally allowed to make requests to port 3001 — this is a security feature called CORS (Cross-Origin Resource Sharing). Without extra configuration, you would see a CORS error in the browser console and your API calls would fail.

The Vite proxy solves this: any request to `/calculate` that arrives at the Vite dev server on port 3000 is silently forwarded to `http://localhost:3001`. From the browser's perspective, every request goes to port 3000, so CORS is never triggered.

```
Browser (port 3000)
  │
  ├── GET /  → Vite serves React app  ✓
  │
  └── POST /calculate
        │
        └── Vite proxy → Express on port 3001  ✓ (browser never sees the redirect)
```

Create `vite.config.js` at the project root:

```js
// vite.config.js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  // Tell Vite to look for index.html inside the frontend/ folder
  root: "frontend",
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      // Any request starting with /calculate is forwarded to Express
      "/calculate": "http://localhost:3001",
    },
  },
});
```

---

### Step 7 — Configure Playwright

Create `playwright.config.js` at the project root:

```js
// playwright.config.js
const { defineConfig } = require("@playwright/test");

module.exports = defineConfig({
  testDir: "./tests/system",
  use: {
    // System tests navigate to this base URL
    baseURL: "http://localhost:3000",
  },
  // Playwright starts both servers automatically before running system tests
  webServer: [
    {
      command: "node backend/server.js",
      port: 3001,
      reuseExistingServer: !process.env.CI,
    },
    {
      command: "npx vite",
      port: 3000,
      reuseExistingServer: !process.env.CI,
    },
  ],
});
```

---

### Step 8 — Create stub files for TDD

In TDD, you do not write implementation code before tests demand it. However, Node.js needs the files to at least _exist_ (even if empty) for `require()` to resolve without crashing the test runner on import. Create the following stubs now — they will be filled in during the RED→GREEN phases below.

> **Only create the files listed here.** Do not write any logic yet.

---

#### `frontend/src/calculate.js` — Empty stub

```js
// frontend/src/calculate.js
// TODO: implementation written in Part 1 GREEN phase
function calculate(a, b, operation) {
  throw new Error("Not implemented");
}

module.exports = { calculate };
```

This stub exports the function so the test file's `require()` resolves, but every call throws — which is exactly what we want: the tests will fail for the right reason (wrong behaviour, not a missing module).

---

#### `backend/server.js` — Entry point stub

```js
// backend/server.js
// TODO: app.js is created in Part 2 GREEN phase
const app = require("./app");

const PORT = process.env.PORT || 3001;

app.listen(PORT, () => {
  console.log(`Backend running on http://localhost:${PORT}`);
});
```

> **Do not create `backend/app.js` yet.** It will be created during the Part 2 GREEN phase. `server.js` is only listed here so you know where it lives.

---

The frontend files (`index.html`, `main.jsx`, `App.jsx`, `Calculator.jsx`) are **not created yet** either. They are built during the Part 3 GREEN phase, driven by what the system tests require.

---

### Step 9 — Verify the test runners are configured correctly

Before writing any tests, confirm Jest and Playwright can discover test files:

```bash
# Should print "No tests found" (no test files exist yet) without errors
npx jest --listTests

# Should print the Playwright version without errors
npx playwright --version
```

If either command errors, double-check that `jest.config.js` and `playwright.config.js` were saved at the project root.

---

## What Is Being Tested at Each Phase

Before writing any code, it helps to know exactly what each testing level targets — so you test the right things at the right level.

### Unit Tests — `calculate.js` (pure function)

**Target:** The `calculate(a, b, operation)` function in isolation. No HTTP, no Express, no browser.

| What is tested                               | Example                                     |
| -------------------------------------------- | ------------------------------------------- |
| Each operation produces the correct output   | `calculate(2, 3, 'add')` → `5`              |
| Floating-point arithmetic handled correctly  | `calculate(0.1, 0.2, 'add')` ≈ `0.3`        |
| Division by zero throws the right error      | throws `Error('Division by zero')`          |
| Unknown operation throws a descriptive error | throws `Error('Unknown operation: modulo')` |
| Non-numeric inputs rejected with `TypeError` | throws `TypeError` for `'five'`             |

> **Does NOT test:** whether the route calls this function, whether HTTP status codes are correct, or whether the UI shows anything.

---

### Integration Tests — `POST /calculate` route (Express + `calculate.js`)

**Target:** The Express route handler, body parsing, number coercion, and `calculate.js` all working together through real HTTP requests.

| What is tested                                 | Example                                      |
| ---------------------------------------------- | -------------------------------------------- |
| Valid request returns `200` and correct JSON   | `{ result: 15 }` for `10 + 5`                |
| String operands coerced to numbers correctly   | `'8'` and `'2'` → `{ result: 16 }`           |
| Missing fields return `400` with error message | omitting `operation` → `400`                 |
| Division by zero returns `422` not `500`       | `b: 0` → `422 { error: 'Division by zero' }` |
| Unknown operation handled gracefully           | `'power'` → `422`                            |

> **Does NOT test:** whether the browser sends the right request, or whether the result is displayed correctly in the UI.

---

### System Tests — Full app in a real browser (Playwright)

**Target:** Complete user journeys driven through a real Chromium browser. One test per user story.

| What is tested                             | Example                                                                   |
| ------------------------------------------ | ------------------------------------------------------------------------- |
| Full user journey for each operation       | Fill inputs → click Calculate → `20` appears in display                   |
| Error message shown in the correct element | `error-display` contains "Division by zero"; `result-display` stays empty |
| UI reacts correctly to user interactions   | Changing an input clears the previous result                              |
| Frontend is wired to the backend correctly | A wrong API URL or missing `fetch` call fails here                        |

> **Does NOT test:** internal logic of `calculate()` directly, or specific HTTP status codes. Only what the user can see.

---

## TDD Walkthrough

The rest of this document follows the actual TDD process. You will write code **only in response to failing tests** — never ahead of them. The three parts below map directly to the three testing levels, each following the same loop:

```
RED    → Write a failing test. Run it. Confirm it fails.
GREEN  → Write the minimum code to make it pass. Run it. Confirm it passes.
REFACTOR → Clean up the code. Run tests again. Confirm nothing broke.
```

> At no point do you write production code before a test demands it.

---

## Part 1: Unit Testing with TDD

**Target:** `frontend/src/calculate.js` — the pure business logic function.
**Tool:** Jest
**What unit tests cover:** isolated function behaviour — correct outputs, edge cases, error throwing. No HTTP, no browser, no database.

---

### 🔴 RED — Write the failing unit tests first

The `calculate.js` stub from Step 8 already exists, so the `require()` will resolve — but every call throws `"Not implemented"`. Create the test file now. The tests describe the behaviour you _want_ the function to have.

Create `tests/unit/calculate.test.js`:

```js
// tests/unit/calculate.test.js
const { calculate } = require("../../frontend/src/calculate");

describe("calculate()", () => {
  describe("addition", () => {
    test("adds two positive integers", () => {
      expect(calculate(2, 3, "add")).toBe(5);
    });

    test("adds a negative and a positive number", () => {
      expect(calculate(-4, 6, "add")).toBe(2);
    });

    test("adds two floating-point numbers", () => {
      expect(calculate(0.1, 0.2, "add")).toBeCloseTo(0.3);
    });
  });

  describe("subtraction", () => {
    test("subtracts second operand from first", () => {
      expect(calculate(10, 4, "subtract")).toBe(6);
    });

    test("result can be negative", () => {
      expect(calculate(3, 7, "subtract")).toBe(-4);
    });
  });

  describe("multiplication", () => {
    test("multiplies two positive integers", () => {
      expect(calculate(4, 5, "multiply")).toBe(20);
    });

    test("multiplying by zero returns zero", () => {
      expect(calculate(99, 0, "multiply")).toBe(0);
    });
  });

  describe("division", () => {
    test("divides first operand by second", () => {
      expect(calculate(10, 2, "divide")).toBe(5);
    });

    test("returns a decimal result", () => {
      expect(calculate(7, 2, "divide")).toBe(3.5);
    });
  });

  describe("error handling", () => {
    test("throws an error when dividing by zero", () => {
      expect(() => calculate(5, 0, "divide")).toThrow("Division by zero");
    });

    test("throws an error for an unknown operation", () => {
      expect(() => calculate(1, 2, "modulo")).toThrow(
        "Unknown operation: modulo",
      );
    });

    test("throws a TypeError when operands are not numbers", () => {
      expect(() => calculate("five", 2, "add")).toThrow(TypeError);
    });
  });
});
```

Now run the tests:

```bash
npx jest tests/unit
```

**Expected output — tests must FAIL:**

```
FAIL tests/unit/calculate.test.js
  calculate()
    addition
      ✕ adds two positive integers (5ms)
      ✕ adds a negative and a positive number (1ms)
      ✕ adds two floating-point numbers (1ms)
    subtraction
      ✕ subtracts second operand from first (1ms)
      ✕ result can be negative (1ms)
    multiplication
      ✕ multiplies two positive integers (1ms)
      ✕ multiplying by zero returns zero (1ms)
    division
      ✕ divides first operand by second (1ms)
      ✕ returns a decimal result (1ms)
    error handling
      ✕ throws an error when dividing by zero (1ms)
      ✕ throws an error for an unknown operation (1ms)
      ✕ throws a TypeError when operands are not numbers (1ms)

  ● calculate() › addition › adds two positive integers

    Error: Not implemented

      at calculate (frontend/src/calculate.js:3:9)
      at Object.<anonymous> (tests/unit/calculate.test.js:6:14)

Tests: 12 failed, 12 total
```

This is correct. The stub exists but throws `"Not implemented"` — the tests fail for the right reason: the function exists but has no real implementation yet. **Do not write any implementation code until you see failures like these.**

---

### 🟢 GREEN — Write the minimum code to make the tests pass

Now that the tests exist and are failing, create `frontend/src/calculate.js` with just enough code to satisfy every test:

```js
// frontend/src/calculate.js
function calculate(a, b, operation) {
  if (typeof a !== "number" || typeof b !== "number") {
    throw new TypeError("Operands must be numbers");
  }
  switch (operation) {
    case "add":
      return a + b;
    case "subtract":
      return a - b;
    case "multiply":
      return a * b;
    case "divide":
      if (b === 0) throw new Error("Division by zero");
      return a / b;
    default:
      throw new Error(`Unknown operation: ${operation}`);
  }
}

module.exports = { calculate };
```

Run the tests again:

```bash
npx jest tests/unit
```

**Expected output — all tests must PASS:**

```
PASS tests/unit/calculate.test.js
  calculate()
    addition
      ✓ adds two positive integers (2ms)
      ✓ adds a negative and a positive number (1ms)
      ✓ adds two floating-point numbers (1ms)
    subtraction
      ✓ subtracts second operand from first
      ✓ result can be negative
    multiplication
      ✓ multiplies two positive integers
      ✓ multiplying by zero returns zero
    division
      ✓ divides first operand by second
      ✓ returns a decimal result
    error handling
      ✓ throws an error when dividing by zero
      ✓ throws an error for an unknown operation
      ✓ throws a TypeError when operands are not numbers

Tests: 12 passed, 12 total
```

Do not add anything beyond what the tests require. Notice there is no `console.log`, no default export, no extra validation — only what the tests demand.

---

### 🔵 REFACTOR — Improve the code without changing behaviour

The Green implementation is already reasonably clean, but there is one thing worth tightening: the `switch` statement has inconsistent spacing and the `divide` case embeds error logic inline. Extract it to make the intent clearer:

```js
// frontend/src/calculate.js  (refactored)
function calculate(a, b, operation) {
  if (typeof a !== "number" || typeof b !== "number") {
    throw new TypeError("Operands must be numbers");
  }

  const operations = {
    add: (x, y) => x + y,
    subtract: (x, y) => x - y,
    multiply: (x, y) => x * y,
    divide: (x, y) => {
      if (y === 0) throw new Error("Division by zero");
      return x / y;
    },
  };

  if (!(operation in operations)) {
    throw new Error(`Unknown operation: ${operation}`);
  }

  return operations[operation](a, b);
}

module.exports = { calculate };
```

**What changed:** the `switch` was replaced with an object-keyed dispatch table. Each operation is now a named entry — adding a new operation in the future (e.g., `modulo`) is a one-line change instead of a new `case` block.

Run the tests one more time to confirm nothing broke:

```bash
npx jest tests/unit
```

All 12 tests must still pass. If any fail, undo the refactor and try again — the tests are the safety net.

---

## Part 2: Integration Testing with TDD

**Target:** `POST /calculate` — the Express route wired to `calculate.js`.
**Tool:** Jest + Supertest
**What integration tests cover:** the route handler, request body parsing, number coercion, and the business logic all working together through a real HTTP request. Still no browser.

> `calculate.js` must exist and all unit tests must be passing before starting this part.

---

### 🔴 RED — Write the failing integration tests first

Create `tests/integration/api.test.js`. `app.js` does not exist yet, so these tests will fail immediately:

```js
// tests/integration/api.test.js
const request = require("supertest");
const app = require("../../backend/app");

describe("POST /calculate", () => {
  // Valid requests
  test("returns 200 and the correct result for addition", async () => {
    const res = await request(app)
      .post("/calculate")
      .send({ a: 10, b: 5, operation: "add" });

    expect(res.status).toBe(200);
    expect(res.body).toEqual({ result: 15 });
  });

  test("returns 200 and the correct result for division", async () => {
    const res = await request(app)
      .post("/calculate")
      .send({ a: 9, b: 3, operation: "divide" });

    expect(res.status).toBe(200);
    expect(res.body).toEqual({ result: 3 });
  });

  test("returns 200 when operands are sent as strings (coerced to numbers)", async () => {
    const res = await request(app)
      .post("/calculate")
      .send({ a: "8", b: "2", operation: "multiply" });

    expect(res.status).toBe(200);
    expect(res.body.result).toBe(16);
  });

  // Validation errors
  test("returns 400 when the operation field is missing", async () => {
    const res = await request(app).post("/calculate").send({ a: 5, b: 3 });

    expect(res.status).toBe(400);
    expect(res.body).toHaveProperty("error");
  });

  test("returns 400 when an operand is missing", async () => {
    const res = await request(app)
      .post("/calculate")
      .send({ a: 5, operation: "add" });

    expect(res.status).toBe(400);
    expect(res.body).toHaveProperty("error");
  });

  // Business rule errors
  test("returns 422 on division by zero", async () => {
    const res = await request(app)
      .post("/calculate")
      .send({ a: 7, b: 0, operation: "divide" });

    expect(res.status).toBe(422);
    expect(res.body.error).toMatch(/division by zero/i);
  });

  test("returns 422 for an unknown operation", async () => {
    const res = await request(app)
      .post("/calculate")
      .send({ a: 5, b: 3, operation: "power" });

    expect(res.status).toBe(422);
    expect(res.body).toHaveProperty("error");
  });
});
```

Run the tests:

```bash
npx jest tests/integration
```

**Expected output — tests must FAIL:**

```
FAIL tests/integration/api.test.js
  ● Test suite failed to run
    Cannot find module '../../backend/app'
```

Good. The route does not exist yet. This is the Red phase.

---

### 🟢 GREEN — Create the Express app and route

Create `backend/app.js`. (`backend/server.js` was already created as a stub in Step 8 — no changes needed there.)

**Why two files?** Supertest needs to import the Express `app` object without actually binding it to a port. Keeping the server startup in a separate `server.js` makes this clean.

Create `backend/app.js`:

```js
// backend/app.js
const express = require("express");
const { calculate } = require("../frontend/src/calculate");

const app = express();
app.use(express.json());

app.post("/calculate", (req, res) => {
  const { a, b, operation } = req.body;

  if (a === undefined || b === undefined || !operation) {
    return res
      .status(400)
      .json({ error: "Missing required fields: a, b, operation" });
  }

  try {
    const result = calculate(Number(a), Number(b), operation);
    res.json({ result });
  } catch (err) {
    res.status(422).json({ error: err.message });
  }
});

module.exports = app;
```

Run the integration tests:

```bash
npx jest tests/integration
```

**Expected output — all tests must PASS:**

```
PASS tests/integration/api.test.js
  POST /calculate
    ✓ returns 200 and the correct result for addition (18ms)
    ✓ returns 200 and the correct result for division (4ms)
    ✓ returns 200 when operands are sent as strings (coerced to numbers) (3ms)
    ✓ returns 400 when the operation field is missing (3ms)
    ✓ returns 400 when an operand is missing (2ms)
    ✓ returns 422 on division by zero (2ms)
    ✓ returns 422 for an unknown operation (2ms)

Tests: 7 passed, 7 total
```

Also confirm that unit tests still pass:

```bash
npx jest tests/unit
```

---

### 🔵 REFACTOR — Clean up the route handler

The route handler currently does three separate concerns in one function: presence validation, type coercion, and error-to-status mapping. Extract the validation into a small helper so the route handler reads at a higher level of abstraction:

```js
// backend/app.js  (refactored)
const express = require("express");
const { calculate } = require("../frontend/src/calculate");

const app = express();
app.use(express.json());

function missingFields(body) {
  const { a, b, operation } = body;
  return a === undefined || b === undefined || !operation;
}

app.post("/calculate", (req, res) => {
  if (missingFields(req.body)) {
    return res
      .status(400)
      .json({ error: "Missing required fields: a, b, operation" });
  }

  const { a, b, operation } = req.body;

  try {
    const result = calculate(Number(a), Number(b), operation);
    res.json({ result });
  } catch (err) {
    res.status(422).json({ error: err.message });
  }
});

module.exports = app;
```

**What changed:** the validation condition is now named (`missingFields`), making the route handler read as a sequence of clear decisions rather than one dense block.

Run both test suites to confirm nothing broke:

```bash
npx jest tests/unit && npx jest tests/integration
```

All tests must pass before moving to Part 3.

---

## Part 3: System Testing with TDD

**Target:** The full application — React frontend + Express backend — running in a real browser.
**Tool:** Playwright
**What system tests cover:** complete user journeys as described by user stories. The browser clicks buttons, fills inputs, and reads what appears on screen.

> Both `calculate.js` and `backend/app.js` must exist and all previous tests must be passing before starting this part.

**User stories being tested:**

1. As a user, I want to add two numbers so that I can see the sum displayed.
2. As a user, I want to divide two numbers so that I can see the quotient.
3. As a user, I want to see an error message when I divide by zero so that I understand why there is no result.

---

### 🔴 RED — Write the failing system tests first

Create `tests/system/calculator.spec.js`. The frontend React component (`Calculator.jsx`) does not exist yet, so these tests will fail:

```js
// tests/system/calculator.spec.js
const { test, expect } = require("@playwright/test");

// User story 1: Add two numbers and see the sum
test("user can add two numbers and see the result", async ({ page }) => {
  await page.goto("/");

  await page.getByTestId("operand-a").fill("12");
  await page.getByTestId("operand-b").fill("8");
  await page.getByTestId("operation").selectOption("add");
  await page.getByTestId("calculate-btn").click();

  await expect(page.getByTestId("result-display")).toHaveText("20");
});

// User story 2: Divide two numbers and see the quotient
test("user can divide two numbers and see the quotient", async ({ page }) => {
  await page.goto("/");

  await page.getByTestId("operand-a").fill("15");
  await page.getByTestId("operand-b").fill("3");
  await page.getByTestId("operation").selectOption("divide");
  await page.getByTestId("calculate-btn").click();

  await expect(page.getByTestId("result-display")).toHaveText("5");
});

// User story 3: See a meaningful error when dividing by zero
test("user sees an error message when dividing by zero", async ({ page }) => {
  await page.goto("/");

  await page.getByTestId("operand-a").fill("9");
  await page.getByTestId("operand-b").fill("0");
  await page.getByTestId("operation").selectOption("divide");
  await page.getByTestId("calculate-btn").click();

  await expect(page.getByTestId("error-display")).toBeVisible();
  await expect(page.getByTestId("error-display")).toContainText(
    "Division by zero",
  );
  await expect(page.getByTestId("result-display")).toBeEmpty();
});

// Additional: result clears when inputs change
test("previous result is cleared when inputs are changed", async ({ page }) => {
  await page.goto("/");

  await page.getByTestId("operand-a").fill("5");
  await page.getByTestId("operand-b").fill("5");
  await page.getByTestId("operation").selectOption("add");
  await page.getByTestId("calculate-btn").click();
  await expect(page.getByTestId("result-display")).toHaveText("10");

  // Changing an input should clear the previous result
  await page.getByTestId("operand-a").fill("3");
  await expect(page.getByTestId("result-display")).toBeEmpty();
});
```

Run the system tests:

```bash
npx playwright test
```

**Expected output — tests must FAIL:**

```
Error: page.getByTestId: Test Id locator "operand-a" resolved to 0 elements.
  (or the page fails to load entirely because the frontend files don't exist)
```

This is the Red phase. The UI elements the tests need (`data-testid="operand-a"`, etc.) do not exist yet.

---

### 🟢 GREEN — Build the minimum frontend to make the tests pass

Create the four frontend files. The tests tell you exactly what the UI needs: inputs with specific `data-testid` values, a select, a button, and two display elements. Write the minimum React code to provide those.

Create `frontend/index.html`:

```html
<!-- frontend/index.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Calculator</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

Create `frontend/src/main.jsx`:

```jsx
// frontend/src/main.jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
```

Create `frontend/src/App.jsx`:

```jsx
// frontend/src/App.jsx
import Calculator from "./Calculator";

export default function App() {
  return <Calculator />;
}
```

Create `frontend/src/Calculator.jsx`. The test file is your specification: it tells you every `data-testid` that must exist, what clicking the button should do, and what clearing an input should trigger:

```jsx
// frontend/src/Calculator.jsx
import { useState } from "react";

export default function Calculator() {
  const [a, setA] = useState("");
  const [b, setB] = useState("");
  const [operation, setOperation] = useState("add");
  const [result, setResult] = useState("");
  const [error, setError] = useState("");

  function handleAChange(e) {
    setA(e.target.value);
    setResult(""); // test 4 requires this: changing an input clears the result
    setError("");
  }

  function handleBChange(e) {
    setB(e.target.value);
    setResult("");
    setError("");
  }

  async function handleCalculate() {
    setResult("");
    setError("");

    try {
      const res = await fetch("/calculate", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ a, b, operation }),
      });
      const data = await res.json();

      if (!res.ok) {
        setError(data.error || "An error occurred");
      } else {
        setResult(String(data.result));
      }
    } catch {
      setError("Failed to reach the server");
    }
  }

  return (
    <div>
      <h1>Calculator</h1>

      <input
        data-testid="operand-a"
        type="number"
        value={a}
        onChange={handleAChange}
        placeholder="First number"
      />

      <select
        data-testid="operation"
        value={operation}
        onChange={(e) => setOperation(e.target.value)}
      >
        <option value="add">+</option>
        <option value="subtract">−</option>
        <option value="multiply">×</option>
        <option value="divide">÷</option>
      </select>

      <input
        data-testid="operand-b"
        type="number"
        value={b}
        onChange={handleBChange}
        placeholder="Second number"
      />

      <button data-testid="calculate-btn" onClick={handleCalculate}>
        Calculate
      </button>

      {/* Two separate elements — tests assert them independently */}
      <div data-testid="result-display">{result}</div>
      <div data-testid="error-display">{error}</div>
    </div>
  );
}
```

Run the system tests:

```bash
npx playwright test
```

**Expected output — all tests must PASS:**

```
Running 4 tests using 1 worker

  ✓ user can add two numbers and see the result (843ms)
  ✓ user can divide two numbers and see the quotient (711ms)
  ✓ user sees an error message when dividing by zero (698ms)
  ✓ previous result is cleared when inputs are changed (1.2s)

4 passed (4.2s)
```

> Playwright's `webServer` config (set up in Step 7) automatically starts both the Express backend and Vite frontend before running these tests, so you do not need to manually start the servers.

---

### 🔵 REFACTOR — Clean up the component

The Green implementation is functional but the component is doing too much: state management, event handling, and the API call are all mixed together. Extract the API call into its own async function so the component is easier to read and future changes are more localised:

```jsx
// frontend/src/Calculator.jsx  (refactored)
import { useState } from "react";

async function postCalculate(a, b, operation) {
  const res = await fetch("/calculate", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ a, b, operation }),
  });
  const data = await res.json();
  if (!res.ok) throw new Error(data.error || "An error occurred");
  return data.result;
}

export default function Calculator() {
  const [a, setA] = useState("");
  const [b, setB] = useState("");
  const [operation, setOperation] = useState("add");
  const [result, setResult] = useState("");
  const [error, setError] = useState("");

  function clearOutput() {
    setResult("");
    setError("");
  }

  async function handleCalculate() {
    clearOutput();
    try {
      const value = await postCalculate(a, b, operation);
      setResult(String(value));
    } catch (err) {
      setError(err.message);
    }
  }

  return (
    <div>
      <h1>Calculator</h1>

      <input
        data-testid="operand-a"
        type="number"
        value={a}
        onChange={(e) => {
          setA(e.target.value);
          clearOutput();
        }}
        placeholder="First number"
      />

      <select
        data-testid="operation"
        value={operation}
        onChange={(e) => setOperation(e.target.value)}
      >
        <option value="add">+</option>
        <option value="subtract">−</option>
        <option value="multiply">×</option>
        <option value="divide">÷</option>
      </select>

      <input
        data-testid="operand-b"
        type="number"
        value={b}
        onChange={(e) => {
          setB(e.target.value);
          clearOutput();
        }}
        placeholder="Second number"
      />

      <button data-testid="calculate-btn" onClick={handleCalculate}>
        Calculate
      </button>

      <div data-testid="result-display">{result}</div>
      <div data-testid="error-display">{error}</div>
    </div>
  );
}
```

**What changed:**

- `postCalculate` is now a standalone async function — it can be moved to its own file or mocked in tests without touching the component.
- `clearOutput` is extracted as a named helper, removing the duplication between the two `onChange` handlers and `handleCalculate`.

Run the full test suite one final time:

```bash
npx jest tests/unit && npx jest tests/integration && npx playwright test
```

**All tests must pass before this counts as done.**

---

## Summary: What Each Level Catches

| Scenario                                           | Unit | Integration       | System                 |
| -------------------------------------------------- | ---- | ----------------- | ---------------------- |
| `calculate(5, 0, 'divide')` throws the right error | ✅   | ✅ (via HTTP 422) | ✅ (error shown in UI) |
| Route returns `400` for a missing field            | ❌   | ✅                | ❌ (UI prevents this)  |
| Wrong HTTP status code returned by route           | ❌   | ✅                | ❌                     |
| Button click doesn't call the API                  | ❌   | ❌                | ✅                     |
| Result displays in the wrong HTML element          | ❌   | ❌                | ✅                     |
| `Number('abc')` coercion bug in the route          | ❌   | ✅                | ✅                     |
| `calculate()` returns wrong value for subtraction  | ✅   | ✅                | ✅                     |

**Key insight:** Each level catches a different class of bug. A fully passing unit test suite is no guarantee the app works end-to-end.

---

## Running All Tests Together

```bash
# Unit + integration tests (Jest)
npx jest

# System tests (Playwright — starts servers automatically)
npx playwright test

# All three in sequence (matches the test npm script in package.json)
npm test
```
