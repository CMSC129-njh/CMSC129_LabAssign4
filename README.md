# 🧪 CMSC 129 Laboratory Assignment 4 Guide: Test-Driven Development (TDD)

Author: Nikko Gabriel Hismaña

### NOTE 1: I know some of you skimmed through Labs 1–3. Please don't do that here. The whole point of this lab is the **process**, _not the output_ — and the process is in the details. At least read the Overview and the Requirements carefully.

![NOT_ASKING](images/not_asking.jpg)

### NOTE 2: A database is **not required** for this lab. You can use an in-memory data store (a plain array or object), or use localStorage on the frontend. If you already have a working backend with a database from a previous lab, you're welcome to reuse it — but it's not a grading requirement here. What's being graded is your **testing discipline**, not your database setup.

---

## 📋 Overview

In this lab, you will build a simple single-resource CRUD web application using **Test-Driven Development (TDD)**. The focus is not just writing tests — it is writing tests **first**, letting them fail, then writing only enough code to make them pass, and then cleaning up.

> **Red → Green → Refactor** is the core loop of TDD:
>
> - 🔴 **Red:** Write a test for something that doesn't exist yet. Run it. It must fail.
> - 🟢 **Green:** Write the minimum code to make the test pass. No more.
> - 🔵 **Refactor:** Improve the code structure without changing its behavior. All tests must still pass.

Your TDD process will be verified through your **Git commit history**. Each commit must correspond to a specific phase of the Red-Green-Refactor cycle, committed and pushed **in order**. There is no other way to prove test-first development — commit timestamps and CI logs are the evidence.

---

## 🎯 Learning Objectives

- Understand and apply the Red-Green-Refactor TDD cycle
- Write tests at three levels: unit, integration, and system/E2E
- Distinguish what each testing level is responsible for catching
- Use testing tools appropriate to your tech stack
- Configure a CI pipeline that acts as an impartial witness to your Red phase
- Practice writing clean, testable code driven by test requirements

---

## ✅ Application Requirements

Before you start, create a GitHub repository for your project named `CMSC129-Lab4-LastNameFNInitials` (e.g., `CMSC129-Lab4-HismanaNG`).

---

### 🤏 Minimum Requirements

#### 1. Application Scope

Build a **single-resource CRUD web application**. One entity, full Create/Read/Update/Delete. Examples:

- Task manager (Tasks)
- Book catalog (Books)
- Contact list (Contacts)
- Recipe collection (Recipes)
- Expense tracker (Expenses)
- Note-taking app (Notes)

**Database is not required.** You can store data in-memory (a server-side array/object), in localStorage on the frontend, or in an actual database if you want to reuse your previous lab setup. The choice does not affect your grade — only your tests do.

> Do not build a multi-entity application. The complexity in this lab comes from testing discipline, not feature count.

---

#### 2. Three Levels of Testing

Your application must have tests at all three levels, each following the Red-Green-Refactor cycle:

| Level           | What to test                                                                                    | Minimum count                     |
| --------------- | ----------------------------------------------------------------------------------------------- | --------------------------------- |
| **Unit**        | Isolated business logic (validation, data transformation, pure functions) — no HTTP, no browser | 3 tests                           |
| **Integration** | At least one full request-response cycle (route + handler working together)                     | 2 tests                           |
| **System**      | Complete user journeys through a real browser, one per user story                               | 1 test per user story (minimum 3) |

---

#### 3. Commit Naming Convention

All commits must be prefixed with their TDD phase so your process is verifiable at a glance:

```
[RED]      — a failing test was added; CI must show FAIL for this commit
[GREEN]    — minimum code to pass the failing tests
[REFACTOR] — code improved, no behavior changed
[DOCS]     — README or documentation updated
```

> Commit messages must also be descriptive. `[GREEN] fix` is not acceptable. `[GREEN] implement calculate() for unit tests` is.

---

#### 4. README Requirements

Your `README.md` must include:

1. **App Description** — What your app does (one paragraph).
2. **User Stories** — Exactly **3 user stories** in the format:
   > _As a [user], I want to [action], so that [benefit]._
3. **Tech Stack** — Framework, testing tools, and data storage approach.
4. **Testing Strategy** — A brief explanation of what you are testing at each level (unit, integration, system) and why. Think of this as your test plan before you write any code.
5. **Setup Instructions** — How to clone, install dependencies, and run the project.
6. **Test Results** — Screenshots of passing test output for each level, added as you complete each part.

---

#### Non-Functional Requirements

1. **Code must be in a public GitHub repository** with the required commit history.
2. **No implementation code in `[RED]` commits** — only test files. Stub files that throw `"Not implemented"` are acceptable.
   > NOTE: Stub files refer to placeholder modules or functions that exist solely to allow your tests to run without import errors. They should not contain any real logic; just enough to satisfy the test runner's requirements for file structure and imports.
3. **All tests must pass on the final commit.**

---

### ➕ Expanded Requirements

> NOTE: This is where this lab differs from LAB 1-3, the expanded requirements are not part of a features rubrics (refer to the grading rubric section below), but they are still required to get the perfect score.

To earn expanded requirement points, implement the following **on top of** the minimum:

#### A. Deployment (2 points)

Deploy your application to a publicly accessible URL using a free-tier platform:

| Stack           | Recommended Platform                 |
| --------------- | ------------------------------------ |
| Laravel         | Railway or Render                    |
| Node.js / React | Render or Vercel + Railway (for API) |
| Flask / Python  | Render or Fly.io                     |

Add the live URL to the top of your README.

---

#### B. CI/CD Pipeline (4 points)

Configure **GitHub Actions** (recommended) so that:

1. Tests run automatically on every push to `main`.
2. **The `[RED]` commits show a failing pipeline run on GitHub** — this is the key evidence of your Red phase.
3. **The `[GREEN]` commits show a passing pipeline run** — this confirms minimum implementation.
4. _(For full points)_ Deployment only proceeds if all tests pass.

Add a `## CI/CD Setup` section to your README with:

- The tool used (GitHub Actions recommended)
- What triggers the workflow
- A screenshot of a failing pipeline run (Red phase) and a passing one (Green phase)

> A starter GitHub Actions workflow is available on the course portal. Adjust it for your stack.

NOTE: The CI pipeline is not just an expanded requirement — even for the minimum requirements, you need at least a basic pipeline that runs your tests on push. The difference is that the expanded requirement demands a full deploy-on-pass setup plus documentation.

---

## 🛠️ Tech Stack Options

Choose **one** of the following stacks. All test tooling must match.

| Stack                       | Unit Testing                 | Integration Testing       | System / E2E Testing  |
| --------------------------- | ---------------------------- | ------------------------- | --------------------- |
| Laravel + Blade             | PHPUnit or Pest              | PHPUnit Feature Tests     | Laravel Dusk          |
| React / Next.js (MERN/FERN) | Jest + React Testing Library | Jest + Supertest          | Playwright or Cypress |
| Vue.js                      | Vitest + Vue Test Utils      | Vitest + Supertest        | Playwright or Cypress |
| Flask + Python              | pytest                       | pytest (with test client) | Playwright (Python)   |

> ⚠️ **Laravel + Dusk note:** Laravel Dusk requires a running application server during tests (`php artisan serve`). In CI, you'll need to start this as a background process before running Dusk. This setup is more involved than the JS-stack alternatives — factor that into your time budget if you choose Laravel.

> You are free to use your Lab 1 or Lab 2 stack. You are also free to try a new one — but weigh that against the time you have.

---

## 📁 Suggested Project Structure

This is a guideline, not a requirement. Organize your code logically — what matters is that your test files are clearly separated by level.

```
your-project/
├── src/ (or app/, backend/, etc.)
│   └── ... your application code (initially stubs) ...
├── tests/
│   ├── unit/
│   │   └── *.test.js (or test_*.py, *Test.php, etc.)
│   ├── integration/
│   │   └── *.test.js
│   └── system/
│       └── *.spec.js (or *.spec.ts, test_*.py, etc.)
├── .github/
│   └── workflows/
│       └── ci.yml
├── README.md
└── ... config files ...
```

---

## 📋 Requirements by Part

---

### PART 0: App Planning

**Commit 0 — `[DOCS] Initial README`**

Before writing a single line of code or tests, complete your `README.md` with:

- App description
- 3 user stories
- Tech stack and testing tools
- Testing strategy (what you plan to test at each level)
- Setup instructions (you can fill these in as you go, but the outline should be here)

> This commit establishes your test plan. Everything after this is implementation of what you described here. Push this before anything else.

---

### PART 1: Unit Testing with TDD

Unit tests cover **isolated business logic** — validation rules, data transformation functions, helper utilities. They must not make HTTP requests, open a browser, or touch a database.

**Commit 1 — `[RED] Unit tests for <feature>`**

- Write at least **3 failing unit tests**.
- The tests must target a function or module that **does not exist yet** (or exists only as an empty stub).
- Run the tests locally and confirm they fail.
- Push this commit. **The CI pipeline must show a FAIL for this commit.** This is your Red phase evidence.
- This commit must contain only test files (and stub files if needed for imports to resolve).

**Commit 2 — `[GREEN] Implement <feature>`**

- Write the **minimum code** to make all unit tests pass. No extra features.
- Run the tests locally — all must pass.
- Push. CI must show PASS.

**Commit 3 — `[REFACTOR] Refactor <feature>`**

- Improve code structure: extract a helper, remove duplication, rename for clarity.
- **Describe the specific change in the commit message body.** Example: `Replaced switch statement with dispatch table for operation handling.`
- All unit tests must still pass.

**Commit 4 — `[DOCS] Unit test results`**

- Add a screenshot of your passing unit test terminal output to the README under `## Test Results`.

---

### PART 2: Integration Testing with TDD

Integration tests cover how your components work together — specifically, at least one full **HTTP request-response cycle** (route → handler → data layer).

> Examples: a POST request that creates a record and returns the correct response body + status code; a GET request that returns all records; missing field validation returning a 400.

**Commit 5 — `[RED] Integration tests for <feature>`**

- Write at least **2 failing integration tests**.
- Tests must involve a route/controller **and** your business logic working together through a real HTTP request (no mocking of the logic layer).
  > NOTE: Your in-memory store or localStorage-backed layer counts as a real data layer; the point is don't mock the business logic function itself
- Push. **CI must show FAIL.**

**Commit 6 — `[GREEN] Implement integration layer for <feature>`**

- Write the minimum code to pass the integration tests.
- Also run your unit tests and confirm they still pass. (If they don't, fix it before pushing.)
- Push. CI must show PASS.

**Commit 7 — `[REFACTOR] Refactor integration layer`**

- Refactor as needed. Document the change in the commit message.
- All tests (unit + integration) must pass.

**Commit 8 — `[DOCS] Integration test results`**

- Add a screenshot of passing integration test output to the README.

---

### PART 3: System Testing with TDD

System tests simulate a real user interacting with your application in a real browser. Each test must correspond to one of the **user stories from your README**.

> Use: **Laravel Dusk** (Laravel), **Playwright or Cypress** (JS stacks), **Playwright for Python** (Flask).

**Commit 9 — `[RED] System tests for user stories`**

- Write at least **1 failing system test per user story** (minimum 3 total).
- Each test must reference the user story it covers in a comment or describe block name.
- Push. **CI must show FAIL.** The failure should be because the UI elements don't exist yet — not a configuration error.

**Commit 10 — `[GREEN] Implement UI for system tests`**

- Build only what the tests demand: the UI elements, routes, and wiring that the system tests target.
- Push. CI must show PASS for all three test levels.

**Commit 11 — `[REFACTOR] Final refactor`**

- Clean up any remaining code smells across the whole codebase.
- All tests (unit + integration + system) must pass.

**Commit 12 — `[DOCS] Final test results and reflection`**

- Add a screenshot of the full passing test suite to the README.
- Add a **Reflection** section (minimum 150 words) answering:
  - What did you find most difficult about writing tests before code?
  - Did writing tests first change the way you designed your code? How?

---

## 📤 Submission Requirements

### Deliverables

1. **GitHub Repository**
   - Public repository named `CMSC129-Lab4-LastNameFNInitials`
   - Complete source code with commit history in the required order
   - `README.md` with all required sections and screenshots
   - Passing CI pipeline on the final commit (green checkmark on GitHub)

2. **F2F Demo and Defense**

   Be prepared to answer questions on:
   - What Red-Green-Refactor means and how you applied it
   - What each testing level is responsible for (unit vs. integration vs. system)
   - How you decided what to test at each level
   - Why the CI pipeline is important for TDD
   - How your test suite would catch a specific bug (I will name a bug and ask where it would fail)
   - The refactoring you did and why

Refer to the course Google Sheet for the demo schedule and deadline.

---

## 📊 Rubrics for Grading

| Component                                                                                                                                     | Points |
| --------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| **README** — app description, 3 user stories, testing strategy, tech stack, setup instructions, all screenshots present                       | 3      |
| **Part 1: Unit Tests** — 3+ tests, Red commit has failing CI, Green commit passes CI, Refactor is documented in commit message                | 6      |
| **Part 2: Integration Tests** — 2+ tests, correct scope (HTTP request-response), Red/Green/Refactor cycle followed                            | 6      |
| **Part 3: System Tests** — 3+ tests tied to user stories, Red/Green/Refactor cycle followed, all 3 Red commits have failing CI evidence       | 6      |
| **Commit Discipline** — correct naming convention (`[RED]`/`[GREEN]`/`[REFACTOR]`/`[DOCS]`), correct order, descriptive messages              | 3      |
| **Lab Defense** — able to explain TDD cycle applied, what each testing level catches, refactoring decisions, and where a named bug would fail | 10     |
| **Base Total**                                                                                                                                | **34** |
| **Extended A** — Live deployment with URL in README                                                                                           | 2      |
| **Extended B** — CI/CD: tests run on push, Red commits show fail, Green commits show pass, deploy-on-pass configured                          | 4      |
| **Grand Total**                                                                                                                               | **40** |

> Passing score: **24 / 40**

> As usual, **30% deduction if submitted late.** And yes, I will check the commit timestamps.

> **Academic integrity note:** Rewriting Git history, pushing all commits in one go after the fact, or faking test failures will be treated as academic dishonesty. Commit timestamps, CI run timestamps, and push logs are all verifiable.

---

## 🚀 Getting Started Checklist

### Planning

- [ ] Read this entire guide (seriously)
- [ ] Choose your tech stack and testing tools
- [ ] Choose your app idea and resource
- [ ] Write 3 user stories
- [ ] Write your testing strategy (what you'll test at each level)
- [ ] Create your GitHub repository

### Setup

- [ ] Initialize your project
- [ ] Install testing dependencies
- [ ] Set up your CI workflow (at minimum: run tests on push to `main`)
- [ ] Create initial folder structure
- [ ] Create stub files (empty functions that throw "Not implemented")
- [ ] Verify your test runner finds test files without errors
- [ ] Write and push Commit 0 (README)

### TDD Loop (repeat for each part)

- [ ] Write the failing tests first
- [ ] Run them locally — confirm they fail
- [ ] Push — confirm CI shows FAIL
- [ ] Write minimum code to pass
- [ ] Run tests — confirm they pass
- [ ] Push — confirm CI shows PASS
- [ ] Refactor — document the change
- [ ] Push and update README with screenshot

---

## 🎯 Tips for Success

1. **Read the test output.** "Cannot find module" and "expected 5 but received undefined" are very different failures. Make sure your Red phase is failing for the right reason.
2. **Resist writing code before tests.** It feels slower at first. That's normal. The payoff is that your implementation is exactly as complex as it needs to be.
3. **One test at a time — in focus, not in commits.** During development, think through one test at a time: write it, understand what implementation it demands, then move to the next. But your _commit granularity_ is one `[RED]` per testing level — write all 3 unit tests before pushing your first RED commit, not one test per commit.
4. **Don't over-engineer the Green phase.** The ugliest code that makes the tests pass is fine. That's what Refactor is for.
5. **The refactor step is NOT optional.** "I refactored the code" is not a commit message. Name the specific change: what you extracted, renamed, or removed and why.
6. **Your CI pipeline is your witness.** If your Red commits don't show a failing run on GitHub, there is no evidence that you wrote tests first.
7. **The calculator example on the course portal** (`calculator_testing_example.md`) walks through the full Red-Green-Refactor cycle for all three levels using the React + Express stack. Use it as a reference — but don't copy it for your submission.

---

## 📚 Helpful Resources

### Testing Documentation

- [Jest Documentation](https://jestjs.io/docs/getting-started) — unit + integration (JS)
- [Supertest](https://github.com/ladjs/supertest) — HTTP assertions for Express (JS)
- [Playwright Documentation](https://playwright.dev/docs/intro) — system/E2E (JS or Python)
- [Cypress Documentation](https://docs.cypress.io/) — system/E2E (JS alternative)
- [PHPUnit Documentation](https://docs.phpunit.de/) — unit + integration (Laravel)
- [Laravel Dusk Documentation](https://laravel.com/docs/dusk) — system/E2E (Laravel)
- [pytest Documentation](https://docs.pytest.org/) — unit + integration (Python/Flask)

### TDD References

- [TDD by Example (Kent Beck)](https://www.oreilly.com/library/view/test-driven-development/0321146530/) — the original book; the first few chapters are worth your time
- [TDD Repo](https://github.com/veilair/test-driven-development) — explains red-green-refactor, has a collection of TDD examples in various languages and frameworks (iz good. 👌)

### CI/CD

- [GitHub Actions Documentation](https://docs.github.com/en/actions) — start with "Quickstart"
- Starter workflow for Node.js is on the course portal

---

## 💡 Common Issues & Solutions

### Issue 1: Tests fail with "Cannot find module" even in the Green phase

Your stub file path doesn't match what the test imports. Check that the file actually exists at the exact path the `require()` or `import` statement points to.

### Issue 2: CI passes on the Red commit (bad!)

Your CI workflow is not running the tests, or your test file is not being discovered. Check your test runner config (`jest.config.js`, `pytest.ini`, etc.) and the `run:` command in your workflow YAML.

### Issue 3: System tests pass locally but fail in CI

The CI environment doesn't have the app running. You need to start your servers as part of the CI workflow before running Playwright/Cypress. Playwright's `webServer` config can handle this automatically — see the calculator example.

### Issue 4: "I wrote the code first and now I need to write the tests"

Start over for that feature. Comment out or delete the implementation, write the tests, confirm they fail, then bring the implementation back in the Green commit. The commit history is what's graded — the order matters.

### Issue 5: My refactor broke a test

Good — that's exactly what tests are for. Undo the refactor, understand why it broke, then refactor more carefully. Do not commit broken tests.

### Issue 6: What happens if I commit the wrong thing?

It depends on whether you've pushed the commit or not.

**If you haven't pushed yet:**
Use `git reset HEAD~1` to undo the last commit while keeping your file changes. Fix whatever is wrong, then re-commit.

```bash
git reset HEAD~1       # undo the commit, keep your changes staged/unstaged
# fix your files
git add .
git commit -m "[GREEN] implement <feature>"
```

**If you already pushed:**
Do **not** force-push or rewrite history (`git push --force`). That's a red flag during grading and can obscure your TDD timeline.

Instead, make a new corrective commit:

```bash
# Fix the problem in your files, then:
git add .
git commit -m "[GREEN] fix failing test in task validation"
git push
```

This keeps your history honest --- it shows a mistake was made and corrected, which is normal. What's not acceptable is a history that looks like every commit was perfect on the first try with no evidence of the Red phase.

> **Example scenario:** You committed `[GREEN] implement task creation` but one test is actually still failing. Don't panic [yes, I'm looking at you Jave]. Fix the test failure, add the fix with a new commit `[GREEN] fix off-by-one in task id assignment`, and push. Your original commit message still marks the intent; the follow-up shows the correction. This is fine.

---

## 💬 Support

If you're stuck:

1. Re-read the error message — Jest and Playwright error messages are very descriptive.
2. Check the `calculator_testing_example.md` on the course portal for a working reference.
3. Check the official docs for your testing tool (listed above).
4. Ask classmates — explaining a problem out loud often solves it.
5. Ask the instructor during office hours or via the class channel.
6. Use AI tools (ChatGPT, Claude, GitHub Copilot) for coding assistance — but make sure you understand the code and can explain it during the demo.

Good luck! 🧪
