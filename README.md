# SauceLabs Mobile Automation with Maestro

## Project Overview

This project demonstrates a mobile UI automation framework for the SauceLabs React Native demo application using **Maestro**, **Maestro Cloud**, and **GitHub Actions**.

The goal of this project is to show how mobile end-to-end tests can be structured, automated, and executed in a CI/CD pipeline for both local development and cloud-based test execution.

This repository is designed as a practical portfolio project for mobile QA automation, CI pipeline configuration, and cross-platform mobile testing.

---

## Key Highlights

- Built a Maestro-based mobile automation framework
- Automated login flow testing for the SauceLabs React Native demo app
- Integrated Maestro Cloud with GitHub Actions
- Configured CI execution on pull requests and pushes to `main`
- Used GitHub Secrets and Variables for secure test data management
- Structured flows and reusable page components for maintainability
- Prepared the framework to support both Android and iOS simulator builds

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Maestro | Mobile UI automation |
| Maestro Cloud | Cloud-based mobile test execution |
| GitHub Actions | CI/CD workflow automation |
| YAML | Test flow and pipeline configuration |
| SauceLabs My Demo App RN | Sample React Native mobile application |

---

## Repository Structure

```text
.
├── .github/
│   └── workflows/
│       └── maestro.yml
├── .maestro/
│   ├── config.yaml
│   ├── flows/
│   │   └── login_test.yaml
│   └── pages/
│       ├── launch_app.yaml
│       ├── nav_menu.yaml
│       └── login_page.yaml
└── README.md
```

---

## Automation Framework Design

The Maestro workspace is stored in:

```text
.maestro/
```

The framework separates executable test flows from reusable page-level commands.

```text
.maestro/flows/
```

contains executable test scenarios.

```text
.maestro/pages/
```

contains reusable page actions and shared steps.

This structure makes the framework easier to maintain as more tests are added.

---

## Maestro Configuration

The workspace configuration is defined in:

```text
.maestro/config.yaml
```

Current configuration:

```yaml
flows:
  - flows/*.yaml
```

This tells Maestro to run all test flow files directly inside the `flows` directory.

> Note: The `---` YAML document separator should be used inside Maestro flow files, not inside `.maestro/config.yaml`.

---

## Example Test Flow

Example login flow:

```yaml
appId: ${APP_ID}
name: Login Test
---
- launchApp
- tapOn: ${LOGIN_MENU_ITEM}
- tapOn: Username
- inputText: ${VALID_USERNAME}
- tapOn: Password
- inputText: ${VALID_PASSWORD}
- tapOn: Login
```

Environment variables are injected from GitHub Actions, which keeps sensitive values out of the repository.

---

## CI/CD Integration

The project uses GitHub Actions to trigger Maestro Cloud test execution.

The workflow runs on:

- Pushes to `main`
- Pull requests targeting `main` or `master`
- Manual execution using `workflow_dispatch`

Workflow file:

```text
.github/workflows/maestro.yml
```

---

## GitHub Actions Workflow

```yaml
name: Maestro UI Automation

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main, master ]
  workflow_dispatch:

concurrency:
  group: maestro-${{ github.ref }}
  cancel-in-progress: true

jobs:
  maestro-cloud:
    runs-on: ubuntu-latest
    environment: staging
    timeout-minutes: 30

    steps:
      - name: 1. Checkout Code
        uses: actions/checkout@v4

      - name: 2. Download Real SauceLabs RN APK
        run: |
          curl -L --fail --retry 3 \
            https://github.com/saucelabs/my-demo-app-rn/releases/download/v1.3.0/Android-MyDemoAppRN.1.3.0.build-244.apk \
            -o demo-app.apk
          test -f demo-app.apk

      - name: 3. Validate Maestro Workspace
        run: |
          test -f .maestro/config.yaml
          test -d .maestro/flows
          find .maestro -maxdepth 4 -type f -print
          echo "---- config.yaml ----"
          cat .maestro/config.yaml

      - name: 4. Run Maestro Cloud
        uses: mobile-dev-inc/action-maestro-cloud@v2.0.2
        with:
          api-key: ${{ secrets.MAESTRO_API_KEY }}
          project-id: ${{ secrets.MAESTRO_PROJECT_ID }}
          app-file: demo-app.apk
          workspace: .maestro
          env: |
            VALID_USERNAME=${{ secrets.APP_USER }}
            VALID_PASSWORD=${{ secrets.APP_PASS }}
            APP_ID=${{ vars.APP_ID }}
            LOGIN_MENU_ITEM=${{ vars.LOGIN_MENU_ITEM }}
```

---

## Secrets and Variables

The workflow uses GitHub Secrets and Variables to manage configuration securely.

### GitHub Secrets

| Secret | Description |
|---|---|
| `MAESTRO_API_KEY` | Maestro Cloud API key |
| `MAESTRO_PROJECT_ID` | Maestro Cloud project ID |
| `APP_USER` | Valid test username |
| `APP_PASS` | Valid test password |

### GitHub Variables

| Variable | Description |
|---|---|
| `APP_ID` | Android package name or iOS bundle ID |
| `LOGIN_MENU_ITEM` | Login menu text or selector |

---

## Android and iOS Support

The project can be extended to run both Android and iOS tests using a GitHub Actions matrix.

Android uses an APK file:

```text
Android-MyDemoAppRN.1.3.0.build-244.apk
```

iOS simulator uses a zipped simulator app:

```text
iOS-Simulator-MyRNDemoApp.1.3.0-162.zip
```

This allows the same Maestro test strategy to support cross-platform mobile validation.

---

## What This Project Demonstrates

This project demonstrates practical experience with:

- Mobile UI automation
- End-to-end test flow design
- CI/CD pipeline configuration
- Maestro Cloud integration
- Secure secret handling in GitHub Actions
- YAML-based test automation
- Debugging CI workflow failures
- Structuring reusable test components
- Preparing automation for Android and iOS platforms

---

## Challenges Solved

During setup, several real-world CI issues were resolved:

- Corrected Maestro workspace path configuration
- Avoided duplicate workflow runs from branch and pull request triggers
- Improved workflow reliability with validation and retry logic

These are common issues in CI-based mobile automation projects and reflect practical troubleshooting experience.

---

## Running Tests Locally

Install Maestro:

```bash
curl -Ls "https://get.maestro.mobile.dev" | bash
```

Check the installation:

```bash
maestro --version
```

Run a single flow:

```bash
maestro test .maestro/flows/login_test.yaml
```

Run all configured flows:

```bash
maestro test .maestro
```

---

## Maestro Cloud Results

Maestro Cloud provides test execution results in the Maestro Cloud dashboard.

---

## Future Improvements

Planned improvements include:

- Add more end-to-end test flows
- Add Android and iOS matrix execution
- Add negative login test scenarios
- Add checkout flow automation
- Add reusable page command coverage
- Add test tagging for smoke and regression suites
- Add separate staging and production environments

---