# COSC2759 Assignment 1
## YeetCode Inc Notes App - CI Pipeline
- Full Name/Names: Le Bao Nguyen, Quoc Chien Phung
- Student ID/IDs: s4071164, s3954261

## 1. How to start the CI pipeline 

The pipeline starts by either push or pull request events.
```
"on":
    push:
        branches:
        - "**"
    pull_request:
        branches:
        - "**"
```
### 1.1 Push
- Push events: If a push event occurs to any branch, the pipeline starts.
```
    push:
        branches:
        - "**"
```
### 1.2 Pull Requests
- Pull request events: If a pull request is made to any branch, the pipeline starts.
```
    pull_request:
        branches:
        - main
```
## 2. Commands to run the pipeline

### 2.1 Pipeline environment setup
Step 1: Checkout code
- Code:
```
  - name: Checkout code
    uses: actions/checkout@v4
```
- Purpose: Clones the latest version of the repository to the Github-hosted runner 
- Screenshot: 
  <div style="text-align: center;">
      <img src="/img/checkout-code-result.png" style="width: 1000px; height: auto"/>
  </div>

Step 2: Setup Node.js 
- Code:
```
  - name: Set up Node.js
    uses: actions/setup-node@v4
    with:
      node-version: "24"
```
- Purpose: Setup the Node.js environment in the runner using version 24 to run the application and tests.
- Screenshot: 
  <div style="text-align: center;">
      <img src="/img/nodejs-code-result.png" style="width: 1000px; height: auto"/>
  </div>

Step 3: Install dependencies
- Code
```
  - name: Install dependencies
    run: npm ci --prefix src
```
- Purpose: Install all the required dependencies and libraries for the project to run the pipeline.
- Screenshot:
 <div style="text-align: center;">
      <img src="/img/install-dep-result.png" style="width: 1000px; height: auto"/>
 </div>

NOTE: Repeat each step and it's code for each test method (Take lint and Unit Test's code as example):
```
lint:
    name: Static Code Analysis / Lint
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code         // Steps here are repeated.
        uses: actions/checkout@v4   
      - name: Set up Node.js        // Steps here are repeated.
        uses: actions/setup-node@v4 
        with:
          node-version: "24"
      - name: Install dependencies  // Steps here are repeated.
        run: npm ci --prefix src
      - name: Run lint
        run: npm run test:lint --prefix src

  unit-test:
    name: Unit Testing
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code         // Steps here are repeated.
        uses: actions/checkout@v4
      - name: Set up Node.js        // Steps here are repeated.
        uses: actions/setup-node@v4
        with:
          node-version: "24"
      - name: Install dependencies  // Steps here are repeated.
        run: npm ci --prefix src
      - name: Run Unit Tests
        run: npm run test:unit --prefix src
```
### 2.2 Lint
- Code:
```
      - name: Run lint
        run: npm run test:lint --prefix src
```
- Purpose: This stage runs ESLint to check for potential syntax errors or stylistic inconsistencies.
- Expected Output: Linting should run successfully if there are no issues. Otherwise, errors will be reported and the pipeline fails.
- Screenshot:
  <div style="text-align: center;">
      <img src="/img/lint-result.png" style="width: 1000px; height: auto"/>
  </div>

### 2.3 Unit Test
- Code: 
```
      - name: Run Unit Tests
        run: npm run test:unit --prefix src
```
- Purpose: This stage runs Jest to perform unit testing on various functions of the code, ensuring they run as expected.
- Expected output: Unit testing if successful, will show which tests pass or fail
- Screenshot: 
  <div style="text-align: center;">
      <img src="/img/unit-test-result.png" style="width: 1000px; height: auto"/>
  </div>
### 2.4 Code Coverage
- Code:
```
      - name: Run Code Coverage
        run: npm run test:unit -- --coverage
```
- Purpose: This stage runs Jest to perform code coverage after lint and unit testing. This is to ensure the majority of code in the code base is ran properly and tested thoroughly.
- Expected output: Code coverage information should be displayed with no errors.
- Screenshot:
  <div style="text-align: center;">
      <img src="/img/code-coverage-result.png" style="width: 1000px; height: auto;"/>
  </div>

### 2.5 Integration Test
NOTE: Setup MongoDB first before running tests using the following code:
```
      - name: Use MongoDB in Github Actions
        uses: MongoCamp/mongodb-github-action@1.2.0
```
- Screenshot: 
  <div style="text-align: center;">
      <img src="/img/mongo-db-setup-result.png" style="width: 1000px; height: auto;"/>
  </div>

- Code:
```
      - name: Use MongoDB in Github Actions
        uses: MongoCamp/mongodb-github-action@1.2.0
      - name: Install dependencies
        run: npm ci --prefix src
      - name: Run Integration Tests
        run: npm run test:integration --prefix src
```
- Purpose: This stage uses Jest for integration testing by running different components of the Notes application as a whole, ensuring there are no communication or data transfer problems between these components.
- Expected Output: Integration testing if successful, should show passing results, meaning the application is running as intended.
- Screenshot:
  <div style="text-align: center;">
      <img src="/img/integration-test-result.png" style="width: 1000px; height: auto;"/>
  </div>

### 2.6 End-to-End Test
NOTE: Setup MongoDB and Playwright before running tests using the following code:
- Code for setting up MongoDB:
```
      - name: Use MongoDB in Github Actions
        uses: MongoCamp/mongodb-github-action@1.2.0
```
  <div style="text-align: center;">
      <img src="/img/mongo-db-setup-result.png" style="width: 1000px; height: auto;"/>
  </div>

- Code for setting up Playwright (with dependencies):
```
      - name: Install Playwright Browsers
        run: npx playwright install --with-deps
```
  <div style="text-align: center;">
      <img src="/img/playwright-setup-result-1.png" style="width: 1000px; height: auto;"/>
  </div>
  <div style="text-align: center;">
      <img src="/img/playwright-setup-result-2.png" style="width: 1000px; height: auto;"/>
  </div>

- Code:
```
  e2e-tests:
    name: End-to-End Testing
    runs-on: ubuntu-latest
    needs: [code-coverage, integration-test]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "24"
      - name: Use MongoDB in Github Actions
        uses: MongoCamp/mongodb-github-action@1.2.0
      - name: Install dependencies
        run: npm ci
        working-directory: src
      - name: Install Playwright Browsers
        run: npx playwright install --with-deps
      - name: Run the application
        run: npm run start &
        working-directory: src
      - name: Wait for app
        run: npx wait-on http://localhost:3000
        working-directory: src
      - name: Run E2E tests
        run: npx playwright test
        working-directory: src
      - name: Upload E2E test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: src/playwright-report
      - name: Upload E2E test failure logs
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: e2e-tests-failure-logs
          path: src/e2e-tests-failed.log
```
- Purpose: This stage uses Playwright for End-to-End testing by automatically perform inputs (hovering over elements, clicking on buttons,...) that a real user would perform. This ensures that the application flows as expected in a specific scenario.
- Expected Output: If run successfully, the results should show if the tests pass or fail for end to end tests and reports errors if fail.
- Screenshot:
  <div style="text-align: center;">
      <img src="/img/e2e-test-result.png" style="width: 1000px; height: auto;"/>
  </div>

### 2.7 Generate Artifact
NOTE: Ensure that artifact is generated the main branch by the following code:
```
if: github.ref == 'refs/heads/main'
```

- Code:
```
build:
    name: Generating Deployable Artifact
    runs-on: ubuntu-latest
    needs: [code-coverage, e2e-tests]
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "24"
      - name: Install dependencies
        run: npm ci --prefix src
      - name: Build Project
        run: npm run build --prefix src
      - name: Upload Build Artifact
        uses: actions/upload-artifact@v4
        with:
          name: build-artifact
          path: src/dist
```
- Purpose: This stage builds the application and uploads the build artifact when changes are pushed to the main branch. This ensures all tests have been successfully ran and shows that thorough testing of the application has been done before deployment.
- Expected outcome: Artifacts are generated ONLY on the main branch, if built successfully.
- Screenshot:

NOTE: Images below is a commit (from a pull request) of the main branch
  <div style="text-align: center;">
      <img src="/img/main-generate-artifact-result.png" style="width: 1000px; height: auto;"/>
  </div>
    <div style="text-align: center;">
      <img src="/img/generate-artifact-result.png" style="width: 1000px; height: auto;"/>
  </div>


## 3. Pipeline results
### 3.1 Successful runs 
- All stages are ran successfully. The testing of the application has passed and the build process has completed without issues.

- Screenshot of commit No.111 on the main branch:
  <div style="text-align: center;">
      <img src="/img/main-generate-artifact-result.png" style="width: 1000px; height: auto;"/>
  </div>

- Screenshot of commit No.120 on the feature/documentation branch:
  <div style="text-align: center;">
      <img src="/img/doc-feature-result.png" style="width: 1000px; height: auto;"/>
  </div>

### 3.2 Failed runs
- Fails could occur at a stage, testing is halted and a detailed error message is shown:

- Screenshot of commit No.53 of the feature/e2e-test branch, failed because Playwright is not properly installed:
  <div style="text-align: center;">
      <img src="/img/e2e-test-fail-result-1.png" style="width: 1000px; height: auto;"/>
  </div>
  <div style="text-align: center;">
      <img src="/img/e2e-test-fail-result-2.png" style="width: 1000px; height: auto;"/>
  </div>

  ### 3.3 Generated build artifact:
  - Build artifact is only generated on the main branch:
  - Screenshot of a downloaded artifact's content (Artifact from commit No.111 on the main branch):
  <div style="text-align: center;">
      <img src="/img/artifact-content.png" style="width: 1000px; height: auto;"/>
  </div>