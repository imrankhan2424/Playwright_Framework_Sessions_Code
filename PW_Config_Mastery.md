# ⚙️ Playwright Configuration & Node Foundation

> **Purpose:** Deep dive into the configuration files that power the Playwright framework.  
> **Key Files:** `playwright.config.ts`, `package.json`, `package-lock.json`, `tsconfig.json`, `.gitignore`

> [!NOTE]
> Interviewers often ask about these files to see if you understand *how* the framework is put together, not just how to write test code. Understanding these files proves you can set up a framework from scratch.

---

## 1️⃣ `playwright.config.ts` — The Control Center

This is the global configuration file for Playwright. It controls how tests execute, what browsers run, how retries are handled, and what reports are generated.

### 🧠 The Magic of `process.env.CI` (Crucial for Interviews)

If you look at the config, you will see `process.env.CI` used everywhere. 

**What is it?**  
`process.env` is Node.js's way of reading environment variables. When tests run in GitHub Actions (or Jenkins/GitLab), the CI server automatically injects an environment variable named `CI` and sets it to `"true"`. When you run tests locally on your laptop, this variable does not exist.

We use **Ternary Operators** (`condition ? true : false`) to make the framework behave one way locally, and a different way in the pipeline, *without changing any code*.

### 🚀 Execution Settings

```typescript
export default defineConfig({
  testDir: './tests',         
  fullyParallel: true,        
  forbidOnly: !!process.env.CI, 
  retries: process.env.CI ? 2 : 0, 
  workers: process.env.CI ? 1 : undefined, 
});
```

| Property | Value Explained | Why we use it |
|----------|-----------------|---------------|
| **`testDir`** | `'./tests'` | Tells Playwright exactly which folder holds the `.spec.ts` files. |
| **`fullyParallel`** | `true` | Runs all files at once. |
| **`forbidOnly`** | `!!process.env.CI` | **Locally (false):** You can use `test.only()` to debug one specific test. <br>**In CI (true):** If you accidentally commit `test.only`, the pipeline immediately FAILS instead of skipping the rest of the tests. |
| **`retries`** | `process.env.CI ? 2 : 0` | **Locally (0):** Fail immediately so you can fix your code. <br>**In CI (2):** Retry failing tests twice to handle network blips or rendering flakiness. |
| **`workers`** | `process.env.CI ? 1 : undefined` | **Locally (undefined):** Uses 50% of your laptop's CPU cores for max speed. <br>**In CI (1):** Runs sequentially on the CI runner to prevent tests from fighting for limited RAM/CPU, which causes flakiness. |

---

### 🎥 `use: {}` Block — Browser Context Settings

These settings apply to the browser context injected into every test.

```typescript
  use: {
    baseURL: process.env.BASE_URL, 
    headless: process.env.CI ? true : false, 
    actionTimeout: 10000,          
    navigationTimeout: 15000,      
    
    // Artifact capture
    screenshot: 'only-on-failure', 
    video: 'retain-on-failure',
    trace: 'on-first-retry',
  },
```

> [!TIP]
> **Artifact Capture Strategy:** 
> - **Screenshot:** `'only-on-failure'` ensures we only capture images when things go wrong, saving disk space.
> - **Video:** `'retain-on-failure'` deletes videos for passing tests but keeps them for debugging failed tests.
> - **Trace:** `'on-first-retry'` captures the rich trace viewer ONLY if a test fails and retries. This prevents massive zip files for passing runs while still catching flakiness.
> - **Headless:** `process.env.CI ? true : false` means tests run invisibly in the pipeline (mandatory for CI runners), but pop open a visible browser when you run them on your laptop so you can watch them.

---

### 📊 Reporters & 🌐 Browser Projects

```typescript
  reporter: [
    ['list'],                               
    ['html', { outputFolder: 'reports/html-report', open: 'never' }], 
    ['allure-playwright', { outputFolder: 'allure-results' }],        
  ],
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
```

- **`list`**: Prints pass/fail to the terminal in real-time.
- **`html`**: Playwright's built-in static HTML. Easy to upload as a GitHub Artifact.
- **`allure-playwright`**: Connects to Allure CLI for rich historical management reporting.
- **`projects`**: Playwright tests against **browser engines** (Chromium), not branded browsers (like Google Chrome). This makes execution significantly faster.

---

## 2️⃣ The Node.js Foundation

### 📦 `package.json`

This file is the blueprint of the project. It holds metadata, dependencies, and most importantly, **executable scripts**.

```json
{
  "name": "playwright-automation-framework",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "test": "npx playwright test",
    "test:headed": "npx playwright test --headed",
    "test:chrome": "npx playwright test --project=chromium",
    "test:smoke": "npx playwright test --grep @smoke",
    "test:api": "npx playwright test tests/api/",
    "report": "npx allure generate allure-results --clean -o allure-report && npx allure open"
  },
  "devDependencies": {
    "@playwright/test": "^1.60.0",
    "@types/node": "^25.9.1",
    "allure-commandline": "^2.42.0",
    "allure-playwright": "^3.9.0"
  },
  "dependencies": {
    "dotenv": "^17.4.2",
    "csv-parse": "^6.2.1",
    "xlsx": "^0.18.5",
    "ajv": "^8.20.0"
  }
}
```

> [!WARNING]
> **`"type": "module"`** is crucial. It tells Node.js to use modern ES6 Modules (`import/export`) instead of legacy CommonJS (`require()`). Without this, TypeScript imports will break.

---

### 🛠️ What do the Scripts actually do?

Scripts are shortcuts. Instead of typing long commands, developers just type `npm run <script-name>`. 

*Note: `npx` means "execute this node package".*

| Script | What it actually executes | Use Case |
|--------|---------------------------|----------|
| **`test`** | `npx playwright test` | Runs all tests in the `tests/` folder in headless mode. |
| **`test:headed`** | `... test --headed` | Overrides config. Opens a visible browser so you can watch tests run. |
| **`test:chrome`** | `... --project=chromium` | Runs tests ONLY on the Chromium engine (ignores Firefox/WebKit if they exist). |
| **`test:smoke`** | `... --grep @smoke` | Runs only tests that have the word `@smoke` in their title (e.g. `test('login @smoke', ...)`). |
| **`test:api`** | `... tests/api/` | Runs only the specs located inside the `tests/api/` folder. |
| **`report`** | `npx allure generate...` | Converts the raw JSON data in `allure-results` into a beautiful HTML dashboard, and then opens it in your browser. |

---

### 🧩 Explaining the Dependencies (Interview Goldmine)

If an interviewer asks, *"Why did you choose these specific packages?"*, here is exactly what each one does:

#### **`dependencies`** (Libraries our framework code actively uses to run)

| Package | Why we need it in this framework |
|---------|----------------------------------|
| **`dotenv`** | **The most important dependency for local runs.** Node.js cannot read `.env` files natively. The `dotenv` library reads the `config/.env.qa` file, parses the key-value pairs (`BASE_URL=xyz`), and **injects them directly into Node's `process.env` memory space**. Once dotenv runs, any test can call `process.env.BASE_URL` and get the value. *(Note: In CI/CD, GitHub Secrets inject directly into `process.env`, so `dotenv` isn't actually used in the pipeline, only on local laptops!)* |
| **`csv-parse`** | Used in `CsvHelper.ts` for Data-Driven Testing. We use this instead of basic JavaScript `.split(',')` because `csv-parse` safely handles headers, skips empty lines, and correctly parses values that have commas *inside* quotes. |
| **`xlsx`** | Used in `ExcelHelper.ts`. Allows us to read multi-sheet `.xlsx` files provided by business analysts. It has built-in functions like `sheet_to_json` which convert Excel rows directly into JavaScript arrays of objects. |
| **`ajv`** | "Another JSON Validator". Used heavily in our API tests for **Contract Testing**. We define a JSON Schema (types, required fields), and AJV validates the API response against it. If the backend changes a field from a `number` to a `string`, AJV catches it immediately. |

#### **`devDependencies`** (Libraries only needed for testing/compiling, not application logic)

| Package | Why we need it in this framework |
|---------|----------------------------------|
| **`@playwright/test`** | The core test runner, assertion library, and browser automation engine. |
| **`@types/node`** | Because we are using TypeScript, this provides the "type definitions" for Node.js built-in modules. Without this, TypeScript will throw an error if you try to type `process.env` or `fs.readFileSync()` because it doesn't know what they are. |
| **`allure-playwright`**| The adapter that intercepts Playwright test results and converts them into raw Allure XML/JSON data during the test run. |
| **`allure-commandline`**| The CLI tool that takes the raw XML/JSON data and renders it into the beautiful, interactive HTML report you can open in a browser. |

---

### 🔒 `package-lock.json`

If `package.json` is the blueprint, `package-lock.json` is the exact photograph of the house after it was built.

| File | What it does | Example Version |
|------|--------------|-----------------|
| **`package.json`** | Requests a version range | `"xlsx": "^0.18.5"` (Means "give me 0.18.5 or newer minor/patch") |
| **`package-lock.json`**| Locks the **exact** installed version | `"xlsx": "0.18.9"` (The exact version currently in node_modules) |

> [!TIP]
> **Why do we commit package-lock.json to Git?**  
> So every developer and the CI/CD pipeline installs the **exact same versions**. It prevents the "it works on my machine" problem caused by unexpected minor package updates.

---

## 3️⃣ TypeScript Configuration

### 🧩 `tsconfig.json`

This file tells the TypeScript compiler how strict to be and what JavaScript version to compile down to.

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "es2020",
    "lib": ["ES2020", "DOM"],
    "types": ["node"],
    "strict": true,
    "esModuleInterop": true,
    "resolveJsonModule": true
  },
  "include": ["**/*.ts"],
  "exclude": ["node_modules"]
}
```

**Key Settings Explained:**
- **`target: "ES2020"`**: Compiles TS down to modern JavaScript (ES2020), which supports `async/await` natively.
- **`strict: true`**: Enforces strict type checking. It catches bugs during coding (like forgetting to handle `null` values) rather than failing at runtime.
- **`types: ["node"]`**: Tells TS to recognize Node.js globals like `process.env` (critical for our `process.env.BASE_URL` logic).
- **`resolveJsonModule: true`**: Allows us to `import data from './data.json'` directly in our tests.

---

## 4️⃣ Git Configuration

### 🛑 `.gitignore`

This file tells Git exactly which folders and files **should never be committed to the repository**. 

```text
# Dependencies
node_modules/

# Playwright Test Artifacts
test-results/
playwright-report/
reports/
trace/
blob-report/

# Allure Reports
allure-results/
allure-report/

# Environment Variables (Security Risk!)
.env
.env.dev
.env.qa
.env.stage
```

| Ignored Item | Why we ignore it |
|--------------|------------------|
| **`node_modules/`** | **Size & Replicability.** This folder contains hundreds of megabytes of third-party code. We don't commit it. Instead, we commit `package.json` and `package-lock.json`, and the CI server runs `npm ci` to reconstruct it perfectly on the fly. |
| **Reports & Traces** | **Clutter & History Bloat.** We ignore `playwright-report/`, `allure-results/`, screenshots, and videos. These change every single test run. If we committed them, the Git history size would balloon massively. In CI/CD, we upload these as **GitHub Artifacts** attached to the specific pipeline run, not into the code branch. |
| **`.env.*` files** | **Security.** These files contain real passwords, OAuth client secrets, and API tokens. If committed to a public or even private repository, it is a massive security breach. They must stay local on your machine. In CI/CD, we use secure GitHub Secrets instead. |

---

## 🎯 Interview Q&A — Configuration Mastery

### Q1: How do you configure Playwright to behave differently in CI vs Locally?
> We use `process.env.CI` inside `playwright.config.ts`. GitHub Actions automatically sets `CI=true`. By using a ternary operator (like `retries: process.env.CI ? 2 : 0`), we tell Playwright to retry flaky tests in the pipeline, but fail immediately on our local machines. We do the same thing for headless mode, workers, and forbidding `test.only`.

### Q2: How does `dotenv` actually work in your framework?
> Locally, Node.js doesn't understand `.env` files. We use `dotenv.config({ path: 'config/.env.qa' })`. The `dotenv` library reads that file, parses the key-value strings, and injects them directly into Node's global `process.env` memory object. Once it does that, any test file can safely read `process.env.USERNAME`. In CI, we don't actually need `dotenv` because GitHub Actions injects Secrets directly into `process.env` natively!

### Q3: Why did you use `csv-parse` instead of just splitting strings?
> While you can use standard JavaScript `string.split(',')` to read a CSV, it breaks easily. If a product description in our test data contains a comma *inside* quotes (e.g., `"Laptop, 15 inch"`), a simple split will break the data into two columns. `csv-parse` handles quotes correctly, skips empty lines, and automatically maps headers to JSON object keys.

### Q4: Why do you have `@types/node` installed?
> We write our framework in TypeScript. When we use Node.js native globals like `process.env` or `fs.readFileSync()`, TypeScript doesn't natively know what those are and will throw linting errors. `@types/node` provides the type definitions for the entire Node.js runtime, giving us auto-complete and type safety for server-side code.

### Q5: Explain the difference between `npm install` and `npm ci`?
> `npm install` reads `package.json` and might upgrade packages dynamically based on semantic versioning ranges (`^` or `~`), updating the lock file. `npm ci` (Clean Install) is built for CI/CD pipelines — it bypasses `package.json`, reads ONLY `package-lock.json`, wipes the `node_modules` folder entirely, and installs the exact deterministic versions that the developer tested locally.

### Q6: Why do we ignore `.env` files and `node_modules` in `.gitignore`?
> We ignore `.env` files because they contain sensitive secrets like passwords and API tokens — committing them is a major security risk. We ignore `node_modules` because it's massive and unnecessary; we commit `package-lock.json` instead, and the CI server reconstructs the folder by running `npm ci`. We also ignore all report folders (`allure-results/`, `reports/`) because they are generated dynamically every run and would bloat the Git history.
