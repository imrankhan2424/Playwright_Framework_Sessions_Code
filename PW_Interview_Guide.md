# 🎭 Playwright Framework — Interview Reference Guide

> **Stack:** Playwright · TypeScript · Node.js · GitHub Actions  
> **Pattern:** Page Object Model (POM) · Single Responsibility Principle (SRP)  
> **Application-agnostic** — works for e-commerce, ERP, insurance portal, SaaS, banking app. Only page class names and locators change per project.

---

## 📖 What is POM? — Quick Reference for Yourself

> [!NOTE]
> Read this first. This is the foundation of the entire framework.

**POM = Page Object Model** — a design pattern for organizing test automation code.

The idea is simple:

> **Every screen/page in the application gets its own TypeScript class.**

That class has two things:
- **Locators** (`private`) — how to find elements on that page (button IDs, input selectors, text roles). **No test file ever sees these directly.**
- **Page Actions** (`public`) — what you can DO on that page (`doLogin()`, `searchProduct()`, `addTenant()`). **These are the only things tests call.**

**Why POM?**
- A locator changes → fix it in **one class**, all tests using that page are automatically fixed
- A test breaks → you know exactly which page class and action to look at
- Adding a new test → no copy-pasting locators — just call the page action methods

**BasePage** is an extension of POM — a parent class that holds locators and actions **shared across ALL pages** (logo, navigation bar, footer, search box). All page classes extend `BasePage` so these shared elements don't get duplicated.

```
Without POM:                           With POM:
─────────────────────────────────      ─────────────────────────────────
test('login', async ({page}) => {      test('login', async ({loginPage}) => {
  await page.fill('#email', 'u@c.com') await loginPage.doLogin('u@c.com','pw')
  await page.fill('#pwd', 'pw')        expect(await homePage.isLoggedIn())
  await page.click('button[Login]')      .toBeTruthy();
  // locator right inside the test!    // zero locators in test file ✅
});                                    });
```

**SRP (Single Responsibility Principle)** — builds on POM:
- Page class → ONLY locators + actions
- Test file → ONLY test steps + assertions
- Fixture file → ONLY dependency injection
- Utils → ONLY data reading
- Config → ONLY global settings

---

## 🌐 PART 1 — Web / UI Framework

---

## 🎤 "Explain Your Web Automation Framework"

> [!TIP]
> Use **Full Answer** for technical rounds. Use **Elevator Pitch** for quick intro.

---

### ✅ Full Answer — Web Framework (~2 min)

*"I have built a UI test automation framework using **Playwright with TypeScript**. It's designed to work with any web application — e-commerce, property management ERP, insurance portal, or any multi-page app. The framework is built on the **Page Object Model** combined with the **Single Responsibility Principle**.*

*Here's how it's structured:*

---

*First, the **Page Layer** (`src/pages/`). For every screen in the application, there is a dedicated TypeScript class. For example, in a property ERP you'd have `LoginPage.ts`, `DashboardPage.ts`, `TenantPage.ts`, `LeasePage.ts`. Each page class has **private locators** — element selectors that only that class knows about — and **public action methods** like `doLogin()`, `searchTenant()`, `submitLease()`. Test files call only the actions, never the locators directly.*

*On top of these, we have a `BasePage.ts` parent class that holds elements common to ALL pages — the header logo, search bar, navigation, footer links. Every page class extends BasePage using TypeScript inheritance. So if the footer selector changes, I fix it once in BasePage and all pages are fixed.*

---

*Second, the **Fixtures Layer** (`src/fixtures/pagefixtures.ts`). Instead of writing `const loginPage = new LoginPage(page)` in every test file, we define all page objects once in the fixtures file using Playwright's `baseTest.extend()`. Any test that needs a login page just declares `{ loginPage }` as a parameter — Playwright creates it, injects it, and cleans it up. This is Dependency Injection — test files have zero setup boilerplate.*

---

*Third, the **Test Layer** (`tests/`). Spec files import from our custom fixtures. They follow the AAA pattern — Arrange in `beforeEach`, Act with page actions, Assert with `expect`. We use `expect.soft()` for soft assertions so all checks run even if one fails.*

---

*Fourth, **Data-Driven Testing** (`src/data/`). We support three data formats: CSV using `csv-parse`, Excel using `xlsx`, and JSON using Node.js native `fs`. We have static helper classes — `CsvHelper`, `ExcelHelper`, `JsonHelper`. Tests loop through data either sequentially (all rows in one test) or dynamically generate one test per row for parallel execution.*

---

*Fifth, the **Config** (`playwright.config.ts`). It reads an `ENV` shell variable (defaulting to qa) and uses dotenv to load the right environment file. Three reporters run simultaneously — console list, Playwright HTML report, and Allure. Retry and worker settings auto-adjust based on whether `CI=true`.*

---

*Sixth, **Environment Management**. We have three env files — dev, qa, stage — for local runs only. In CI/CD, the same variable keys (`BASE_URL`, `USERNAME`, `PASSWORD`) are stored as GitHub Secrets scoped per GitHub Environment. Tests always read from `process.env` — they never know if the values came from a .env file or GitHub Secrets.*

---

*Finally, **CI/CD with GitHub Actions**. The workflow triggers on push, pull request, and nightly cron. A matrix strategy runs QA and Stage as two parallel jobs. Each job targets its GitHub Environment, gets the right secrets injected, installs the browser, runs tests in headless mode, and then uploads the HTML report and Allure results as artifacts. After every run, a Slack message goes to the QA channel and an email goes to the manager — whether tests pass or fail.*"

---

### ⚡ Web Framework — Elevator Pitch (30 sec)

> *"I built a Playwright TypeScript UI framework using Page Object Model — each screen has a class with private locators and public actions, with a BasePage parent for shared elements. Playwright fixtures handle dependency injection so test files are clean. We support data-driven testing via CSV, Excel, and JSON helpers. Config picks the environment dynamically via an ENV variable, and CI runs through GitHub Actions with GitHub Secrets per environment and Slack/email notifications after every run."*

---

### 🗂️ Web Framework — One-Liners

| Component | What to say |
|-----------|-------------|
| **POM** | Each screen → one class. Private locators. Public actions. Tests call actions, never locators. |
| **BasePage** | Parent class with shared UI elements. All pages extend it via TypeScript inheritance. |
| **pagefixtures.ts** | Playwright's DI — page objects injected as test params. No `new Page()` in test files. |
| **Test Data** | CSV via `csv-parse`, Excel via `xlsx`, JSON via native `fs`. Static helper classes. |
| **Config** | ENV variable → dotenv loads right file → baseURL and creds in `process.env`. |
| **Environments** | Same keys in dev/qa/stage — values differ. Local = .env files. CI = GitHub Secrets. |
| **CI/CD** | Push/PR/cron → matrix jobs (QA+Stage parallel) → secrets auto-scoped → Slack + Email. |
| **Reporting** | Console + Playwright HTML + Allure. Screenshots/video/trace auto-captured on failure. |

---

## 📁 Web Framework — Repository Structure

```
your-app-playwright-framework/
│
├── 📄 package.json                   # Project config, npm scripts, dependencies
├── 📄 playwright.config.ts           # Global config — browsers, reporters, retries
├── 📄 tsconfig.json                  # TypeScript compiler settings
├── 📄 .gitignore                     # Excludes node_modules, reports, .env files
│
├── 📁 config/                        # ⚠️  LOCAL ONLY — temp placeholders
│   ├── .env.dev                      # Dev environment variables
│   ├── .env.qa                       # QA environment variables
│   └── .env.stage                    # Stage environment variables
│   (In CI/CD → replaced by GitHub Secrets with same keys, different values)
│
├── 📁 src/
│   ├── 📁 pages/                     # ← Page Object Model classes
│   │   ├── BasePage.ts               # Parent — shared locators + generic actions
│   │   ├── LoginPage.ts              # Login screen
│   │   ├── HomePage.ts               # Dashboard/Home
│   │   ├── SearchResultsPage.ts      # Search results
│   │   └── ProductInfoPage.ts        # Product detail page
│   │
│   ├── 📁 fixtures/
│   │   └── pagefixtures.ts           # ← Registers all UI page objects (DI)
│   │
│   ├── 📁 utils/                     # ← Data reading helpers
│   │   ├── CsvHelper.ts
│   │   ├── ExcelHelper.ts
│   │   └── JsonHelper.ts
│   │
│   └── 📁 data/                      # ← Test data files
│       ├── loginData.csv
│       ├── logindata.json
│       ├── product.csv
│       └── AppTestData.xlsx
│
├── 📁 tests/                         # ← Web UI test specs
│   ├── ✅ loginpagefix.spec.ts        # Login — fixtures + DDT (CSV/Excel/JSON)
│   ├── ✅ homepagefix.spec.ts         # Homepage — fixtures
│   ├── ✅ search.spec.ts              # Search + navigation — CSV DDT
│   ├── ✅ productpage.spec.ts         # Product info validation
│   ├── ⚠️  loginpage.spec.ts          # Trainer example only — raw POM, not used
│   └── ⚠️  homepage.spec.ts           # Trainer example only — raw POM, not used
│
└── 📁 .github/workflows/
    └── playwright.yml                 # CI/CD pipeline
```

---

## 🏗️ Web Framework — Architecture (SRP Layers)

```
┌──────────────────────────────────────────────────────────────────┐
│        SRP — Single Responsibility Principle                     │
│        Each layer has exactly ONE job                            │
├────────────────────┬───────────────────┬─────────────────────────┤
│  PAGE LAYER        │  TEST LAYER        │  FIXTURES               │
│  src/pages/        │  tests/*.spec.ts   │  src/fixtures/          │
│                    │                    │  pagefixtures.ts        │
│  BasePage.ts       │  NO locators       │                         │
│   └ LoginPage.ts   │  AAA pattern       │  DI: injects page       │
│   └ HomePage.ts    │  beforeEach setup  │  objects into tests     │
│   └ SearchPage.ts  │  expect.soft()     │  automatically          │
│                    │                    │                         │
│  private: locators │  only page actions │  No new Page() in       │
│  public: actions   │  + assertions      │  any test file          │
├────────────────────┼───────────────────┼─────────────────────────┤
│  TEST DATA         │  CONFIG            │  ENVIRONMENT            │
│  src/data/         │  playwright.config │  config/.env.*          │
│                    │                    │                         │
│  loginData.csv     │  ENV → dotenv      │  dev / qa / stage       │
│  logindata.json    │  3 reporters       │  same keys per env      │
│  AppTestData.xlsx  │  retries CI=2      │  local = .env file      │
│                    │  workers CI=1      │  CI = GitHub Secrets    │
│  CsvHelper.ts      │  screenshot/video  │                         │
│  ExcelHelper.ts    │  headless in CI    │  process.env always     │
│  JsonHelper.ts     │                    │  reads same way         │
└────────────────────┴───────────────────┴─────────────────────────┘
                          ↓ CI/CD ↓
┌──────────────────────────────────────────────────────────────────┐
│  GITHUB ACTIONS  .github/workflows/playwright.yml                │
│  push / PR / nightly cron / manual trigger                       │
│  matrix: QA job ──── Stage job (parallel)                        │
│  GitHub Environments → secrets auto-scoped per env               │
│  → HTML Report + Allure artifacts                                │
│  → Slack notification + Manager email (pass AND fail)            │
└──────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Page Layer — Code Deep Dive

### `BasePage.ts`

```typescript
export class BasePage {
    protected readonly page: Page;
    protected readonly logo: Locator;       // protected = child classes can use
    protected readonly searchBox: Locator;  // test files CANNOT access these
    protected readonly footerLinks: Locator;

    constructor(page: Page) {
        this.page = page;
        this.logo        = page.getByAltText('company-logo');
        this.searchBox   = page.getByPlaceholder('Search');
        this.footerLinks = page.locator('footer a');
    }

    async getPageTitle(): Promise<string>    { return await this.page.title(); }
    async isLogoVisible(): Promise<boolean>  { return await this.logo.isVisible(); }
    async getFootersCount(): Promise<number> { return await this.footerLinks.count(); }
    async takeScreenshot(name: string) {
        await this.page.screenshot({ fullPage: true, path: `reports/screenshot/${name}.png` });
    }
}
```

### `LoginPage.ts` — Child Page

```typescript
export class LoginPage extends BasePage {
    private readonly emailId: Locator;      // private = ONLY LoginPage sees this
    private readonly password: Locator;
    private readonly loginBtn: Locator;
    private readonly errorMessage: Locator;

    constructor(page: Page) {
        super(page);   // loads BasePage — gets logo, searchBox, footerLinks
        this.emailId      = page.getByRole('textbox', { name: 'E-Mail Address' });
        this.password     = page.getByRole('textbox', { name: 'Password' });
        this.loginBtn     = page.getByRole('button',  { name: 'Login' });
        this.errorMessage = page.locator('.alert.alert-danger');
    }

    async goToLoginPage()                     { await this.page.goto('/account/login'); }
    async doLogin(user: string, pass: string) {
        await this.emailId.fill(user);
        await this.password.fill(pass);
        await this.loginBtn.click();
    }
    async isErrorDisplayed()  { return await this.errorMessage.isVisible(); }
    async isForgotLinkVisible(){ return await this.forgotPwdLink.isVisible(); }
}
```

### All Pages Summary

| Class | Parent | Private Locators | Public Actions |
|-------|--------|-----------------|----------------|
| `BasePage` | — | logo, searchBox, footerLinks | getPageTitle, isLogoVisible, takeScreenshot |
| `LoginPage` | BasePage | emailId, password, loginBtn, errorMsg | goToLoginPage, doLogin, isErrorDisplayed |
| `HomePage` | BasePage | logoutLink, headers | isLogoutLinkExist, getHeaders, doSearch |
| `SearchResultsPage` | BasePage | searchResults | getResultsCount, selectProduct |
| `ProductInfoPage` | BasePage | header, images, metaData, pricing | getProductInfo → returns `Map<string, string>` |

---

## 2️⃣ Fixtures — `pagefixtures.ts`

```typescript
import { test as baseTest } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { HomePage } from '../pages/HomePage';
// ... other pages

type pageFixtures = {
    basePage: BasePage;
    loginPage: LoginPage;
    homePage: HomePage;
    searchResultsPage: SearchResultsPage;
    productInfoPage: ProductInfoPage;
    testData: Record<string, string>[];   // ← CSV data as a fixture too
};

export let test = baseTest.extend<pageFixtures>({
    loginPage: async ({ page }, use) => {
        await use(new LoginPage(page));    // create → inject → auto cleanup
    },
    homePage: async ({ page }, use) => {
        await use(new HomePage(page));
    },
    testData: async ({ }, use) => {
        await use(CsvHelper.readCsv('src/data/loginData.csv'));
    },
    // ... all other pages
});

export { expect } from '@playwright/test';
```

**How tests use it:**
```typescript
import { test, expect } from '../src/fixtures/pagefixtures'; // ← YOUR fixtures

test.beforeEach(async ({ loginPage }) => {   // ← injected, no new LoginPage(page)
    await loginPage.goToLoginPage();
});

test('login test', async ({ loginPage, homePage }) => {   // ← multiple injected
    await loginPage.doLogin(process.env.USERNAME!, process.env.PASSWORD!);
    expect.soft(await homePage.isLogoutLinkExist()).toBeTruthy();
    expect.soft(await homePage.getPageTitle()).toBe('My Account');
});
```

---

## 3️⃣ Test Data — DDT

````carousel
### CSV — `CsvHelper.ts` (uses `csv-parse`)

```typescript
export class CsvHelper {
    static readCsv(filePath: string): Record<string, string>[] {
        return parse(fs.readFileSync(filePath, "utf-8"), {
            columns: true,           // row 1 = headers
            skip_empty_lines: true,
            trim: true,
        });
    }
}
```

```csv
username,password
wrong@company.com,wrongpass
,emptyuser
validuser@company.com,
```

<!-- slide -->
### Excel — `ExcelHelper.ts` (uses `xlsx`)

```typescript
export class ExcelHelper {
    static readExcel(filePath: string, sheetName?: string): Record<string, string>[] {
        const workbook = XLSX.readFile(filePath);
        const sheet = workbook.Sheets[sheetName || workbook.SheetNames[0]];
        return XLSX.utils.sheet_to_json(sheet, { defval: "" }); // empty cell = "" not undefined
    }
}

// Multi-sheet usage:
ExcelHelper.readExcel('src/data/AppTestData.xlsx', 'login');
ExcelHelper.readExcel('src/data/AppTestData.xlsx', 'products');
```

<!-- slide -->
### JSON — `JsonHelper.ts` (native `fs`)

```typescript
export class JsonHelper {
    static readJson(filePath: string): Record<string, string>[] {
        return JSON.parse(fs.readFileSync(filePath, "utf-8"));
    }
}
```

```json
[
    { "username": "wrong@company.com", "password": "wrong@123" },
    { "username": "invalid@company.com", "password": " " }
]
```

<!-- slide -->
### DDT Patterns

**Pattern 1: Sequential (1 test in report)**
```typescript
test('invalid login — all rows', async ({ loginPage, testData }) => {
    for (let row of testData) {
        await loginPage.doLogin(row.username, row.password);
        expect(await loginPage.isErrorDisplayed()).toBeTruthy();
    }
});
```

**Pattern 2: Parallel (1 test per row in report)**
```typescript
const data = ExcelHelper.readExcel('src/data/AppTestData.xlsx', 'login');
for (let row of data) {
    test(`invalid login — ${row.username}`, async ({ loginPage }) => {
        await loginPage.doLogin(row.username, row.password);
        expect(await loginPage.isErrorDisplayed()).toBeTruthy();
    });
}
```

````

---

## 4️⃣ `playwright.config.ts`

```typescript
const ENV = process.env.ENV || "qa";
dotenv.config({ path: `config/.env.${ENV}` }); // local only

export default defineConfig({
    fullyParallel: true,
    forbidOnly: !!process.env.CI,       // CI fails if test.only left in code
    retries: process.env.CI ? 2 : 0,    // 2 retries in CI, 0 locally
    workers: process.env.CI ? 1 : undefined,

    reporter: [
        ["list"],                                                       // console
        ["html", { outputFolder: "reports/html-report", open: "never" }], // HTML
        ["allure-playwright", { outputFolder: "allure-results" }],     // Allure
    ],

    use: {
        baseURL: process.env.BASE_URL,
        screenshot: 'only-on-failure',
        video:      'retain-on-failure',
        trace:      'on-first-retry',
    },

    projects: [
        { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
        // { name: 'firefox',  ... }   ← enable anytime
        // { name: 'webkit',   ... }   ← enable anytime
    ],
});
```

---

## 5️⃣ Environment Management

> [!CAUTION]
> `.env` files in `config/` = **LOCAL ONLY**. They are in `.gitignore`. Never used in CI/CD.

```
LOCAL:  ENV=qa → dotenv loads config/.env.qa → process.env populated
CI/CD:  GitHub Environment "qa" → GitHub Secrets injected into process.env
        Tests behave identically — same process.env.BASE_URL reads
```

| Secret Key | QA | Stage | Dev |
|-----------|----|----|-----|
| `BASE_URL` | `https://qa.app.com` | `https://stage.app.com` | `https://dev.app.com` |
| `USERNAME` | `qa@test.com` | `stage@test.com` | `dev@test.com` |
| `PASSWORD` | `qa-pass` | `stage-pass` | `dev-pass` |
| `SLACK_WEBHOOK_URL` | `hooks.slack.com/qa` | `hooks.slack.com/stage` | — |
| `MAIL_TO` | `qa-lead@company.com` | `manager@company.com` | — |

---

## 6️⃣ GitHub Actions CI/CD

```yaml
name: Playwright UI Tests

on:
  push:          { branches: [main] }
  pull_request:  { branches: [main] }
  schedule:      [ cron: '0 4 * * 1-5' ]  # 9:30 AM IST Mon-Fri
  workflow_dispatch:                        # manual button

jobs:
  ui-tests:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        environment: [qa, stage]
    environment: ${{ matrix.environment }}  # ← scopes GitHub Secrets to correct env

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npx playwright install --with-deps chromium
      - run: npx playwright test --project=chromium
        env:
          CI: true
          BASE_URL:  ${{ secrets.BASE_URL }}   # same key, QA env gets QA value
          USERNAME:  ${{ secrets.USERNAME }}   # stage env gets stage value
          PASSWORD:  ${{ secrets.PASSWORD }}   # auto-scoped by "environment:" above
          API_TOKEN: ${{ secrets.API_TOKEN }}

      - name: Upload HTML Report
        uses: actions/upload-artifact@v4
        if: always()
        with: { name: report-${{ matrix.environment }}, path: reports/html-report/ }

      - name: Slack Notification
        uses: slackapi/slack-github-action@v1.26.0
        if: always()
        with:
          payload: '{"text":"[${{ matrix.environment }}] Playwright — ${{ job.status }}\nRun: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"}'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK

      - name: Email Manager
        uses: dawidd6/action-send-mail@v3
        if: always()
        with:
          server_address: smtp.gmail.com
          server_port: 465
          username: ${{ secrets.MAIL_USERNAME }}
          password: ${{ secrets.MAIL_PASSWORD }}
          subject: "[${{ matrix.environment }}] Test Results — ${{ job.status }}"
          to: ${{ secrets.MAIL_TO }}
          from: "QA Bot <qa-bot@company.com>"
          body: |
            Environment : ${{ matrix.environment }}
            Status      : ${{ job.status }}
            Branch      : ${{ github.ref_name }}
            Report link : ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

---
---

## 🔌 PART 2 — API Framework Extension

---

> [!IMPORTANT]
> This is a **separate section** from the Web framework. Explain it AFTER the web story.  
> Key message: *"My framework also supports API testing — it has a dedicated fixture, a separate helper class, and its own test folder. Everything is cleanly isolated from the UI tests."*

---

## 🎤 "Does Your Framework Support API Testing?"

### ✅ Full Answer — API Extension (~1.5 min)

*"Yes — our framework also supports REST API testing, and it's cleanly separated from the UI tests.*

*On the API side, we have a dedicated fixture file — `apifixtures.ts` — which is completely separate from `pagefixtures.ts`. This API fixture extends Playwright's base test and injects an `ApiHelper` instance. The `ApiHelper` class wraps Playwright's built-in `APIRequestContext` — it provides clean `get()`, `post()`, `put()`, and `delete()` methods that all return a standardized `{ status, body }` object. So API tests never deal with raw response handling.*

*The API test specs live in their own folder — `tests/api/`. They import from `apifixtures.ts`, not from `pagefixtures.ts`. There's no browser involved — these are pure HTTP tests.*

*Our API test coverage includes:*

*First, **basic CRUD** — GET, POST, PUT, DELETE operations on REST endpoints.*

*Second, **serial E2E flows** — using `test.describe.serial()` so POST creates a user, PUT updates that same user, DELETE removes it. They share state and run in sequence.*

*Third, **independent CRUD tests** — each test creates its own resource, operates on it, and cleans up. Fully self-contained.*

*Fourth, **JSON Schema validation** using the AJV library. We define a JSON schema object with expected field types and required fields, then validate the response body against it. This acts as a contract test — if a backend developer renames a field or changes its type, our schema test catches it immediately.*

*Fifth, **network interception and mocking** using `page.route()`. We can intercept browser-level HTTP requests and return fake responses — useful for testing edge cases without hitting the real backend.*

*Sixth, **OAuth2 token-based auth** — we POST to the token URL with client credentials in `test.beforeEach`, receive a Bearer token, and use it in all API calls in that spec file.*

*The API tests run in the same CI pipeline as the UI tests — same GitHub Actions workflow, same environment secrets, same reporting."*

---

### ⚡ API — Elevator Pitch (20 sec)

> *"Yes, my framework supports API testing separately. We have a dedicated `apifixtures.ts` that injects an `ApiHelper` — a wrapper around Playwright's `APIRequestContext` with clean GET/POST/PUT/DELETE methods. API tests live in `tests/api/`, completely separate from UI tests. We also do JSON Schema validation with AJV, network mocking with `page.route()`, and OAuth2 token flows in `beforeEach`."*

---

### 🗂️ API Framework — One-Liners

| Component | What to say |
|-----------|-------------|
| **apifixtures.ts** | Separate from pagefixtures. Injects ApiHelper using Playwright's `request` fixture. |
| **ApiHelper** | Wraps `APIRequestContext`. Returns `{status, body}` for every call. |
| **tests/api/** | Dedicated folder — completely separate from UI tests. No browser involved. |
| **CRUD tests** | GET/POST/PUT/DELETE against REST endpoints. Asserts on status codes + response body. |
| **describe.serial()** | Forces sequential run when tests share state (e.g. POST userId → PUT → DELETE). |
| **Schema (AJV)** | Validates response structure — types, required fields. Catches API contract breaks. |
| **page.route()** | Intercepts browser HTTP — `continue()` to log, `fulfill()` to return fake data. |
| **OAuth2** | `test.beforeEach` POSTs for token → Bearer token used in all tests in that file. |

---

## 📁 API Framework — Folder Structure

```
src/
│
├── 📁 api/                               # ← API ABSTRACTION LAYER (separate from pages)
│   └── ApiHelper.ts                      # Generic wrapper: GET/POST/PUT/DELETE
│
└── 📁 fixtures/
    ├── pagefixtures.ts                   # UI fixtures — injects page objects
    └── apifixtures.ts                    # API fixtures — injects ApiHelper ← SEPARATE

tests/
│
├── loginpagefix.spec.ts                  # UI tests — uses pagefixtures
├── homepagefix.spec.ts                   # UI tests — uses pagefixtures
├── search.spec.ts                        # UI tests — uses pagefixtures
│
└── 📁 api/                               # ← API TESTS FOLDER (completely separate)
    ├── users.api.spec.ts                 # Basic CRUD — raw request (no helper)
    ├── users.api.sep.spec.ts             # Serial E2E: POST→PUT→DELETE (shared userId)
    ├── users.api.indi.spec.ts            # Independent: each test creates own user
    ├── users.api.schema.spec.ts          # Schema validation with AJV
    ├── intercept.spec.ts                 # Network intercept + mocking
    └── amadeus.oauth2.spec.ts            # OAuth2 client_credentials flow
```

---

## 🔀 Web vs API — How They're Separated

```
UI TESTS                              API TESTS
tests/*.spec.ts                       tests/api/*.spec.ts
───────────────────────               ─────────────────────────
import { test } from                  import { test } from
  '../fixtures/pagefixtures'            '../../fixtures/apifixtures'
         │                                      │
Uses: { loginPage, homePage }         Uses: { apiHelper }
         │                                      │
Drives real Chromium browser          NO browser — pure HTTP
Playwright browser context            Playwright APIRequestContext
         │                                      │
Asserts on: UI elements               Asserts on: status codes + JSON
            page titles                          response body
            visible text
         │                                      │
SLOWER (browser launch)               FASTER (direct HTTP)

Both: same playwright.config.ts → same CI pipeline → same reports
```

---

## 🔧 API Layer — Code Deep Dive

### `apifixtures.ts`

```typescript
import { test as baseTest } from '@playwright/test';
import { ApiHelper } from '../api/ApiHelper';

type ApiFixtures = {
    apiHelper: ApiHelper;
};

export let test = baseTest.extend<ApiFixtures>({
    apiHelper: async ({ request }, use) => {
        // request = Playwright's built-in APIRequestContext
        // API_BASE_URL comes from GitHub Secret (or .env locally)
        await use(new ApiHelper(request, process.env.API_BASE_URL!));
    },
});

export { expect } from '@playwright/test';
```

### `ApiHelper.ts`

```typescript
import { APIRequestContext } from "@playwright/test";

export class ApiHelper {
    constructor(
        private readonly request: APIRequestContext,
        private readonly baseURL: string
    ) {}

    async get(endpoint: string, headers?: Record<string, string>) {
        const res = await this.request.get(`${this.baseURL}${endpoint}`, { headers });
        return { status: res.status(), body: await res.json() };
    }

    async post(endpoint: string, data: object, headers?: Record<string, string>) {
        const res = await this.request.post(`${this.baseURL}${endpoint}`, { data, headers });
        return { status: res.status(), body: await res.json() };
    }

    async put(endpoint: string, data: object, headers?: Record<string, string>) {
        const res = await this.request.put(`${this.baseURL}${endpoint}`, { data, headers });
        return { status: res.status(), body: await res.json() };
    }

    async delete(endpoint: string, headers?: Record<string, string>) {
        const res = await this.request.delete(`${this.baseURL}${endpoint}`, { headers });
        return { status: res.status() };   // DELETE → no body
    }
}
```

---

## 🧪 API Test Patterns

````carousel
### Pattern 1 — Basic CRUD (raw request, no helper)
`users.api.spec.ts`

```typescript
import { test, expect } from '@playwright/test'; // ← direct, no apifixtures

const AUTH = { Authorization: 'Bearer your-token' };

test('GET all users', async ({ request }) => {
    const response = await request.get('https://gorest.co.in/public/v2/users', {
        headers: AUTH
    });
    expect(response.status()).toBe(200);
    const body = await response.json();
    console.log(body);
});

test('POST create user', async ({ request }) => {
    const response = await request.post('https://gorest.co.in/public/v2/users', {
        headers: AUTH,
        data: { name: 'Test User', email: `test_${Date.now()}@co.com`,
                gender: 'male', status: 'active' }
    });
    expect(response.status()).toBe(201);
});
```

<!-- slide -->
### Pattern 2 — Serial E2E CRUD (shared state)
`users.api.sep.spec.ts`

```typescript
import { test, expect } from '../../src/fixtures/apifixtures'; // ← apifixtures

let userId: number; // shared across tests in this describe block

test.describe.serial('E2E CRUD flow', () => {  // ← serial = sequential, in order

    test('POST — create user', async ({ apiHelper }) => {
        const res = await apiHelper.post('/public/v2/users', {
            name: 'API Test', email: `api_${Date.now()}@co.com`,
            gender: 'male', status: 'active'
        }, AUTH_HEADER);
        expect(res.status).toBe(201);
        userId = res.body.id;   // ← captured for next tests
    });

    test('PUT — update user', async ({ apiHelper }) => {
        const res = await apiHelper.put(`/public/v2/users/${userId}`,
            { name: 'Updated Name', status: 'inactive' }, AUTH_HEADER);
        expect(res.status).toBe(200);
        expect(res.body.name).toBe('Updated Name');
    });

    test('DELETE — remove user', async ({ apiHelper }) => {
        const res = await apiHelper.delete(`/public/v2/users/${userId}`, AUTH_HEADER);
        expect(res.status).toBe(204);
    });
});
```

<!-- slide -->
### Pattern 3 — Independent (self-contained AAA)
`users.api.indi.spec.ts`

```typescript
// Each test creates its own user → operates → no shared state
async function createUser(apiHelper: ApiHelper) {
    const res = await apiHelper.post('/public/v2/users', {
        name: 'API Test', email: `test_${Date.now()}@co.com`,
        gender: 'male', status: 'active'
    }, AUTH_HEADER);
    expect(res.status).toBe(201);
    return res.body;
}

test('PUT — update user', async ({ apiHelper }) => {
    const user = await createUser(apiHelper);  // ARRANGE: create fresh user

    const res = await apiHelper.put(           // ACT: update
        `/public/v2/users/${user.id}`,
        { name: 'Updated', status: 'inactive' }, AUTH_HEADER);

    expect(res.status).toBe(200);              // ASSERT
    expect(res.body.name).toBe('Updated');
});
```

<!-- slide -->
### Pattern 4 — Schema Validation (AJV)
`users.api.schema.spec.ts`

```typescript
import Ajv from 'ajv';
const ajv = new Ajv();

const userSchema = {
    type: "object",
    properties: {
        id:     { type: "number" },
        name:   { type: "string" },
        email:  { type: "string" },
        status: { type: "string" }
    },
    required: ["id", "name", "email", "status"]
};

test('GET user — validate schema', async ({ apiHelper }) => {
    const response = await apiHelper.get('/public/v2/users/123', AUTH_HEADER);

    const validate = ajv.compile(userSchema);
    const isValid  = validate(response.body);

    if (!isValid) console.log('Schema errors:', validate.errors);
    expect(isValid).toBeTruthy();
    // ↑ Fails if backend renames id → userId, changes type, or removes required field
});
```

<!-- slide -->
### Pattern 5 — Network Intercept & Mock
`intercept.spec.ts`

```typescript
// LOG all network requests
test('intercept and log all requests', async ({ page }) => {
    await page.route('**/*', async (route) => {
        console.log(route.request().method(), route.request().url());
        await route.continue();   // let request through — just logging
    });
    await page.goto(process.env.BASE_URL!);
});

// MOCK — return fake response without hitting real backend
test('mock product search API', async ({ page }) => {
    await page.route('**/api/products/search*', (route) => {
        route.fulfill({
            status: 200,
            contentType: 'application/json',
            body: JSON.stringify([
                { name: 'Mock Product A', price: '$100' },
                { name: 'Mock Product B', price: '$200' }
            ])
        });
    });
    await page.goto(`${process.env.BASE_URL}/search?q=product`);
    // UI now renders mock data — test edge cases without real backend
});
```

<!-- slide -->
### Pattern 6 — OAuth2 Token Flow
`amadeus.oauth2.spec.ts`

```typescript
let accessToken: string;

test.beforeEach('generate OAuth2 Bearer token', async ({ request }) => {
    const response = await request.post('https://api.company.com/oauth2/token', {
        form: {
            grant_type:    process.env.GRANT_TYPE,         // client_credentials
            client_id:     process.env.OAUTH_CLIENT_ID,
            client_secret: process.env.OAUTH_CLIENT_SECRET
        }
    });
    expect(response.status()).toBe(200);
    accessToken = (await response.json()).access_token;
    // Now every test in this file uses: Authorization: `Bearer ${accessToken}`
});

test('GET location data', async ({ request }) => {
    const res = await request.get('https://api.company.com/v1/locations', {
        headers: { Authorization: `Bearer ${accessToken}` },
        params: { subType: 'CITY', keyword: 'MUC', countryCode: 'DE' }
    });
    expect(res.status()).toBe(200);
});
```

````

---

## 🎯 Combined Interview Q&A

### Q1: What is POM and why do you use it?
> Each web page/screen has a TypeScript class with `private` locators and `public` action methods. Tests call actions only — never locators directly. If a locator changes, fix it in one class. All tests using that page are automatically fixed. Promotes reusability, maintainability, and separation of concerns.

### Q2: What is BasePage and why?
> Parent class with UI elements shared across ALL pages — logo, nav bar, search box, footer. All page classes extend BasePage (`extends BasePage`, `super(page)` in constructor). Avoids duplication (DRY). One place to update shared locators.

### Q3: What is a Playwright fixture?
> Playwright's built-in dependency injection. Define page objects once in `pagefixtures.ts` using `baseTest.extend()`. Tests declare them as parameters — Playwright creates, injects, and cleans them up automatically. Zero `new LoginPage(page)` boilerplate in test files.

### Q4: How are Web and API tests separated?
> UI tests use `pagefixtures.ts` → page objects → drive real Chromium browser → assert on UI. API tests use `apifixtures.ts` → ApiHelper → pure HTTP via `APIRequestContext` → no browser → assert on `{status, body}`. Different fixture file, different test folder (`tests/` vs `tests/api/`), different import. Same playwright.config.ts, same CI pipeline, same reports.

### Q5: What is `apifixtures.ts`?
> A separate fixture file (not pagefixtures). It extends Playwright's base test and injects an `ApiHelper` instance using the built-in `request` fixture (Playwright's `APIRequestContext`). API tests import from `apifixtures.ts` — they get `{ apiHelper }` injected automatically.

### Q6: What is schema validation and why is it important?
> AJV (Another JSON Validator) validates the response structure — field names, data types, required fields — not just values. If a backend developer renames `id` to `userId`, changes a field from `number` to `string`, or removes a required field, the schema test fails immediately. Acts as a contract test between producer and consumer.

### Q7: What is `test.describe.serial()`?
> By default Playwright runs tests in parallel. `serial()` forces sequential execution in order within that block. Used for E2E CRUD flows where POST creates a user and captures `userId`, PUT updates that user, DELETE removes it — they depend on each other's output.

### Q8: How do you handle environments?
> Three .env files locally (dev, qa, stage) — temp placeholders only. In CI, same variable keys stored as GitHub Secrets in GitHub Environments. GitHub auto-scopes the right secrets per job based on `environment: ${{ matrix.environment }}`. Tests always read from `process.env` — code never changes.

### Q9: How do CI/CD notifications work?
> Two steps in the workflow: `slackapi/slack-github-action` posts to a Slack channel via webhook URL (stored as GitHub Secret). `dawidd6/action-send-mail` sends email via SMTP (credentials as GitHub Secret). Both use `if: always()` — fire on pass AND fail. Message includes environment, status, branch, run link.

### Q10: What is `expect.soft()`?
> Soft assertion — test continues even if assertion fails. All soft failures collected and reported at the end. Regular `expect()` stops immediately on first failure. Use `expect.soft()` when you want to see ALL failures on a page in one test run.

### Q11: What external libraries does the framework use?
> `dotenv` — local env loading. `csv-parse` — CSV parsing with header support. `xlsx` — Excel multi-sheet reading. `ajv` — JSON Schema contract validation. `allure-playwright` + `allure-commandline` — rich reporting. `@types/node` — TypeScript definitions for Node.js built-ins.

### Q12: What design principles does the framework follow?
> **SRP** — each file has one job. **POM** — pages encapsulate UI interactions. **DRY** — BasePage for shared elements, fixtures for shared setup. **AAA** — Arrange/Act/Assert structure. **DI** — fixtures inject dependencies. **Open/Closed** — add new page = add new class, existing code untouched.

---

## 📊 Quick Run Reference

```bash
npm install && npx playwright install chromium  # setup

ENV=qa    npx playwright test                   # UI tests on QA
ENV=stage npx playwright test                   # UI tests on Stage

npx playwright test tests/api/                  # API tests only
npx playwright test tests/loginpagefix.spec.ts  # specific UI file
npx playwright test --grep @smoke               # by tag
npx playwright test --headed                    # visible browser
npx playwright test --debug                     # inspector mode

npm run allure:report                           # generate + open Allure
```

---

*Web framework and API framework live in the same repo — clean separation via different fixtures, different test folders, different imports.*
