# infra-github-actions-ci

> Minimalist Continuous Integration testing harness and GitHub Actions matrix pipeline engineered to validate native Node.js ESM test runners, multi-runtime compatibility, and automated failure gatekeeping.

**Inception (First Commit):** `2025-04-11` • **Batch / Horizon:** `Batch 2024–2025` • **Architecture Domain:** `Continuous Integration / GitHub Actions`

---

## 📌 Architecture & Overview

`enabling-actions` serves as a dedicated architectural proof-of-concept and automated verification harness for GitHub Actions CI/CD workflows. Rather than introducing complex third-party testing frameworks (such as Jest, Mocha, or Vitest) and heavy `node_modules` trees, the repository demonstrates a zero-dependency, native ECMAScript Module (ESM) architecture using Node.js's built-in test runner (`node:test`) and assertion library (`node:assert`).

* **Context:** Continuous Integration harness and workflow validation experiment designed to verify GitHub Actions lifecycle hooks, pull request gatekeeping, and multi-version Node matrix execution.
* **Key Patterns:**
  * **Matrix Execution Pipeline:** Parallel job execution validating builds across multiple Long-Term Support (LTS) and current Node.js runtimes (`20.x` and `22.x`).
  * **Zero-Dependency Native ESM:** Direct invocation of the Node.js native test runner via `node --test`, eliminating dependency supply-chain vulnerabilities and cold-start installation overhead.
  * **Negative Testing & Gatekeeping Verification:** Implementation of intentional assertion failures to empirically validate that GitHub Actions correctly captures process exit code `1`, halts pipeline progression, and blocks non-compliant pull requests.

### CI/CD Workflow Architecture

```
┌────────────────────────────────────────────────────────┐
│                   GitHub Event Trigger                 │
│         (push to 'main' or pull_request to 'main')     │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│            GitHub Actions: Node.js CI Matrix           │
│                   (ubuntu-latest)                      │
├───────────────────────────┬────────────────────────────┤
│   Job Matrix Node 20.x    │    Job Matrix Node 22.x    │
│  - actions/checkout       │   - actions/checkout       │
│  - actions/setup-node     │   - actions/setup-node     │
│  - run: node --test       │   - run: node --test       │
└─────────────┬─────────────┴─────────────┬──────────────┘
              │                           │
              ▼                           ▼
┌────────────────────────────────────────────────────────┐
│            Native Node.js Test Suite (test.mjs)        │
│  1. 'test integer addition'  --> PASS (Exit Code 0)    │
│  2. 'test string addition'   --> FAIL (Exit Code 1)    │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│         Pipeline Result: Failure Gate Triggered        │
│        - GitHub Actions Job Fails (Red Build)          │
│        - PR Merge Blocked / Maintainer Notified        │
└────────────────────────────────────────────────────────┘
```

---

## 🛠 Tech Stack

* **Runtime & Language:** [Node.js](https://nodejs.org/) (`20.x`, `22.x`), Native ECMAScript Modules (`.mjs`)
* **Test Runner & Assertions:** Built-in [`node:test`](https://nodejs.org/api/test.html), [`node:assert`](https://nodejs.org/api/assert.html)
* **CI/CD Platform:** [GitHub Actions](https://github.com/features/actions)
* **Runner Environment:** `ubuntu-latest`
* **Package Management:** Zero external dependencies (No `package.json` required for execution)

---

## ✨ Features & Pipeline Specification

* **Zero External Dependencies:** Executes entirely with native Node.js core APIs (`node:test` and `node:assert/strict`), removing install steps and external security risks.
* **Multi-Version Matrix Testing:** Automatically executes test suites across Node.js versions `20.x` (Active LTS) and `22.x` (Current).
* **Automated CI Gatekeeping:** Guarantees that any failing assertion produces a non-zero exit code (`1`), immediately failing the CI run.
* **Dual Event Ingestion:** Monitors code integration automatically on both direct pushes and pull request submissions against the `main` branch.

### Pipeline Stages & Workflow Breakdown

| Stage | Trigger / Context | Runner / Action | Purpose |
|---|---|---|---|
| `checkout` | `push` / `pull_request` | `actions/checkout@v2` | Clones repository source code into the runner workspace. |
| `setup-node` | Matrix (`20.x`, `22.x`) | `actions/setup-node@v4` | Installs and configures the target Node.js version in runner path. |
| `test` | `node --test` | Shell execution | Executes the native test suite in `test.mjs` using Node's built-in test harness. |

### Test Suite Assertions

| Test Case | Method | Assertion | Expected Result | Purpose |
|---|---|---|---|---|
| `test integer addition` | `strictEqual(1 + 1, 2)` | Strict numerical equality (`===`) | `PASS` (Green) | Baseline arithmetic sanity check. |
| `test string addition` | `strictEqual('1' + '1', 2)` | String concatenation vs Number (`"11" === 2`) | `FAIL` (Red) | Intentional negative test to trigger CI build failure. |

---

## 🚀 Getting Started

### Prerequisites

* [Node.js](https://nodejs.org/) `>= 20.0.0` (Recommended: v20 LTS or v22 Current)
* [Git](https://git-scm.com/)

### Installation

Clone the repository to your local workstation:

```bash
git clone https://github.com/fabogit/infra-github-actions-ci.git
cd infra-github-actions-ci
```

> **Note:** Because the project leverages Node's native test runner and standard library assertions, running `npm install` is not required.

---

### Running the Test Suite

#### Run All Tests

To execute the test runner against all matching test files:

```bash
node --test
```

Alternatively, invoke `test.mjs` directly:

```bash
node --test test.mjs
```

#### Expected Local Output

Running the command will trigger the deliberate assertion error in `test string addition`:

```text
▶ test integer addition
✔ test integer addition (1.2345ms)
▶ test string addition
✖ test string addition (1.6789ms)
  AssertionError [ERR_ASSERTION]: Concatenation of "1" and "1" does not equal 2
  ...
  operator: 'strictEqual'
  expected: 2
  actual: '11'

ℹ tests 2
ℹ suites 0
ℹ pass 1
ℹ fail 1
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 25.123
```

#### Run Only Passing Tests

To filter execution to passing test cases while developing:

```bash
node --test --test-name-pattern="integer"
```

---

## 📄 License & Author

Developed by [@fabogit](https://github.com/fabogit).
