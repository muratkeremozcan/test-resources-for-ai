# Murat Ozcan Testing Philosophy & Patterns for TEA Agent Enhancement

## Purpose

This document captures the comprehensive testing philosophy, patterns, and implementation details extracted from Murat Ozcan's books, blog posts, and sample repositories. It serves as the knowledge base for enhancing the Test Architect (TEA) agent to generate tests, configurations, and architectures in Murat's distinctive style.

## Reference Resources

### Books

- **CCTDD: Cypress Component Test Driven Design** - [https://github.com/muratkeremozcan/cctdd](https://github.com/muratkeremozcan/cctdd)
- **UI Testing Best Practices** - [https://github.com/NoriSte/ui-testing-best-practices](https://github.com/NoriSte/ui-testing-best-practices)

### Blog Posts

All blog posts available at: [https://dev.to/muratkeremozcan](https://dev.to/muratkeremozcan)

Key posts include:
- Functional Programming Test Patterns with Cypress
- Page Objects vs. Functional Helpers
- Building the test architecture, increasing adoption, improving the developer experience
- Effective Test Strategies for Front-end Applications using LaunchDarkly Feature Flags
- CI CD strategies for UI apps and deployed services
- Cypress Component Testing vs React Testing Library - the complete comparison
- API e2e testing event driven systems
- Cypress and Pact contract testing patterns
- Testing Email-Based Authentication Systems
- The 32+ ways of selective testing with Cypress

### Sample Repositories

- **Cypress vs Playwright Examples** - [https://github.com/muratkeremozcan/cy-vs-pw-murats-version](https://github.com/muratkeremozcan/cy-vs-pw-murats-version)
- **Playwright Book Examples** - [https://github.com/muratkeremozcan/pw-book](https://github.com/muratkeremozcan/pw-book)
- **Tour of Heroes (React/Vite/Cypress/TS)** - [https://github.com/muratkeremozcan/tour-of-heroes-react-vite-cypress-ts](https://github.com/muratkeremozcan/tour-of-heroes-react-vite-cypress-ts)
- **Pact.js Consumer Example** - [https://github.com/muratkeremozcan/pact-js-example-consumer](https://github.com/muratkeremozcan/pact-js-example-consumer)
- **Pact.js React Consumer** - [https://github.com/muratkeremozcan/pact-js-example-react-consumer](https://github.com/muratkeremozcan/pact-js-example-react-consumer)
- **Pact.js Provider Example** - [https://github.com/muratkeremozcan/pact-js-example-provider](https://github.com/muratkeremozcan/pact-js-example-provider)

## Core Philosophy

### The Murat Testing Manifesto
1. **"Functional helpers over Page Objects"** - Composition wins over inheritance
2. **"Test at the lowest level possible"** - Component > Integration > E2E (1:3 to 1:5 ratio)
3. **"Network boundary is the test boundary"** - Mock at the edge, not service level
4. **"Visual debugging changes everything"** - See the component while testing
5. **"No flaky tests, ever"** - Deterministic or delete (0 tolerance)
6. **"Setup via API, assert via UI"** - Fast setup, user-centric assertions
7. **"One test = one concern"** - Focused and clear
8. **"Explicit over implicit"** - Assertions in tests, not hidden in helpers
9. **"Data factories over fixtures"** - Dynamic > Static
10. **"Shift left, test often"** - Early and continuous

## Testing Patterns

### 1. Test Structure Pattern
```javascript
// ALWAYS this structure:
describe('Feature/Component', () => {
  context('specific scenario', () => {  // Group related tests
    beforeEach(() => {
      // Setup that's VISIBLE in the test
      // Network mocks BEFORE navigation
      // Data setup via API, not UI
    })
    
    it('should <action> when <condition>', () => {
      // Arrange - Act - Assert clearly separated
      // Assertions explicit in test, not helpers
    })
  })
})
```

### 2. Fixture & Helper Architecture

#### Composable Fixture System (SEON Production Pattern)

```typescript
// playwright/support/merged-fixtures.ts - The Murat Way
import { test as base, mergeTests } from '@playwright/test'
import { test as apiRequestFixture } from './fixtures/api-request-fixture'
import { test as networkFixture } from './fixtures/network-fixture'
import { test as authFixture } from './fixtures/auth-fixture'
import { test as logFixture } from './fixtures/log-fixture'

// Merge all fixtures for comprehensive capabilities
export const test = mergeTests(
  base,
  apiRequestFixture,
  networkFixture,
  authFixture,
  logFixture
)
```

#### Pure Function → Fixture Pattern

```typescript
// Step 1: Pure function (always first!)
export async function apiRequest({ request, method, url }) {
  // Core implementation - testable independently
}

// Step 2: Fixture wrapper
export const apiRequestFixture = base.extend({
  apiRequest: async ({ request }, use) => {
    await use((params) => apiRequest({ request, ...params }))
  }
})

// Step 3: Export for subpath imports
// package.json exports: "./api-request", "./api-request/fixtures"
```

#### Helper Function Rules

- **3+ uses** → Create fixture with subpath export
- **2-3 uses** → Create utility module
- **1 use** → Keep inline
- **Complex logic** → Factory function pattern

### 3. Network Interception Strategy

#### The Network-First Pattern
```javascript
// ALWAYS intercept before action
const networkCall = interceptNetworkCall({ url: '**/api/data' })
await page.goto('/page')  // THEN navigate
const response = await networkCall  // THEN await

// Cypress equivalent
cy.intercept('GET', '**/api/data').as('getData')
cy.visit('/page')
cy.wait('@getData')
```

### 4. Selector Strategy (Non-Negotiable)
```javascript
// Priority order - ALWAYS
1. data-cy="element"        // Cypress
2. data-testid="element"    // Playwright/RTL
3. ARIA attributes          // Future-proof
4. Text content             // User-centric

// Never use:
- CSS classes (unless no other option)
- IDs (unless absolutely necessary)
- Complex XPath
- Index-based selectors
```

### 5. Waiting Strategies
```javascript
// ✅ Deterministic waiting
await page.waitForResponse('**/api/data')
cy.wait('@getUsers')

// ✅ Event-based waiting
await page.waitForLoadState('networkidle')
cy.get('[data-cy="spinner"]').should('not.exist')

// ❌ NEVER use hard waits
await page.waitForTimeout(3000) // NEVER
cy.wait(3000) // NEVER
```

### 6. Test Data Management

#### Factory Pattern (Always)
```typescript
export const createUser = (overrides: Partial<User> = {}): User => ({
  id: faker.string.uuid(),
  email: faker.internet.email(),
  name: faker.person.fullName(),
  role: 'user',
  ...overrides
})

// Usage in tests:
const adminUser = createUser({ role: 'admin' })
```

#### API-First Setup
```javascript
beforeEach(() => {
  // ✅ Setup via API
  cy.task('db:seed', { users: [createUser()] })
  
  // ❌ NOT via UI
  // cy.visit('/signup')
  // cy.fill('#email', 'test@example.com')
})
```

### 7. Assertion Patterns

#### Flexible for Shared State
```typescript
// When state might be shared:
expect(await items.count()).toBeGreaterThanOrEqual(3)
// NOT: expect(await items.count()).toBe(3)
```

#### Explicit in Tests
```javascript
// ✅ Assertions in test
cy.get('@apiCall').should((xhr) => {
  expect(xhr.request.body).to.deep.equal(expectedPayload)
  expect(xhr.response.statusCode).to.equal(200)
})

// ❌ NOT hidden in helpers
// validateApiCall('@apiCall', expectedPayload, 200)
```

## Configuration Templates

### Playwright Configuration (The Murat Way - SEON Production Pattern)

```javascript
// playwright.config.ts - Environment-based config loading
import { config as dotenvConfig } from 'dotenv'
import path from 'path'

dotenvConfig({
  path: path.resolve(__dirname, '../../.env')
})

const envConfigMap = {
  local: require('./playwright/config/local.config').default,
  staging: require('./playwright/config/staging.config').default,
  production: require('./playwright/config/production.config').default
}

const environment = process.env.TEST_ENV || 'local'

if (!Object.keys(envConfigMap).includes(environment)) {
  console.error(`No configuration found for environment: ${environment}`)
  process.exit(1)
}

export default envConfigMap[environment as keyof typeof envConfigMap]
```

#### Base Configuration Pattern
```javascript
// playwright/config/base.config.ts
export default defineConfig({
  testDir: path.resolve(__dirname, '../tests'),
  outputDir: path.resolve(__dirname, '../../test-results'),
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html', { outputFolder: 'playwright-report', open: 'never' }],
    ['junit', { outputFile: 'results.xml' }],
    ['list']
  ],
  use: {
    actionTimeout: 15000,
    navigationTimeout: 30000,
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  },
  globalSetup: path.resolve(__dirname, '../support/global-setup.ts'),
  timeout: 60000,
  expect: { timeout: 10000 }
})
```

### Cypress Configuration (The Murat Way)
```javascript
import { defineConfig } from 'cypress'

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    viewportWidth: 1920,
    viewportHeight: 1080,
    video: false,
    screenshotOnRunFailure: true,
    defaultCommandTimeout: 10000,
    requestTimeout: 10000,
    responseTimeout: 10000,
    retries: {
      runMode: 2,
      openMode: 0
    },
    env: {
      API_URL: 'http://localhost:3001/api',
      coverage: false
    },
    setupNodeEvents(on, config) {
      on('task', {
        'db:seed': seedDatabase,
        'db:reset': resetDatabase,
        log(message) {
          console.log(message)
          return null
        }
      })
      return config
    }
  },
  component: {
    devServer: {
      framework: 'react',
      bundler: 'vite'
    },
    specPattern: 'src/**/*.cy.{js,jsx,ts,tsx}',
    supportFile: 'cypress/support/component.tsx'
  }
})
```

## CI/CD Patterns

### GitHub Actions Workflow Template
```yaml
name: E2E Tests
on:
  pull_request:
  push:
    branches: [main]

jobs:
  install:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: '.nvmrc'
          cache: 'npm'
      - run: npm ci --prefer-offline
      - uses: actions/cache@v4
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

  test-changed:
    needs: install
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Run changed tests first
        run: |
          CHANGED_SPECS=$(git diff --name-only HEAD~1 | grep -E '\.(cy|spec)\.(ts|js)x?$' || true)
          if [ ! -z "$CHANGED_SPECS" ]; then
            npm run test -- --spec "$CHANGED_SPECS"
          fi

  test-e2e:
    needs: install
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        containers: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4
      - uses: cypress-io/github-action@v6
        with:
          start: npm run dev
          wait-on: 'http://localhost:3000'
          wait-on-timeout: 120
          browser: chrome
          record: true
          parallel: true
          group: 'E2E'
        env:
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
```

### Burn-in Testing Script
```bash
#!/bin/bash
# scripts/burn-in-changed.sh
CHANGED_SPECS=$(git diff --name-only HEAD~1 | grep -E '\.(cy|spec)\.(ts|js)x?$')
if [ ! -z "$CHANGED_SPECS" ]; then
  for i in {1..10}; do
    echo "Burn-in run $i of 10"
    npm run test -- --spec "$CHANGED_SPECS"
    if [ $? -ne 0 ]; then
      echo "Burn-in failed on run $i"
      exit 1
    fi
  done
  echo "Burn-in passed - 10 successful runs"
fi
```

## Feature Flag Testing

### Feature Flag Enum Pattern
```javascript
// src/utils/flags.js
export const FLAGS = Object.freeze({
  NEW_FEATURE: 'new-feature',
  EXPERIMENT: 'experiment-flag',
  DARK_MODE: 'dark-mode'
})
```

### Stubbing Feature Flags
```javascript
// Stub in most tests
cy.stubFeatureFlags({
  [FLAGS.NEW_FEATURE]: true,
  [FLAGS.DARK_MODE]: false
})

// Dedicated FF tests with cleanup
describe('Feature Flag Tests', () => {
  const userId = generateUserId()
  
  afterEach(() => {
    removeUserTarget(FLAGS.NEW_FEATURE, userId)
  })
  
  it('should test flag variation', () => {
    setFlagVariation(FLAGS.NEW_FEATURE, userId, 1)
    // test the feature
  })
})
```

## Component Testing (CCTDD)

### Component Test Driven Development Flow
```typescript
// 1. Start with failing test
it('should render button', () => {
  cy.mount(<Button />)
  cy.get('button').should('exist')
})

// 2. Make it pass with minimal code
const Button = () => <button>Click</button>

// 3. Refactor and add behavior
it('should handle click', () => {
  const onClick = cy.stub().as('click')
  cy.mount(<Button onClick={onClick} />)
  cy.get('button').click()
  cy.get('@click').should('have.been.called')
})
```

### Component Test Utilities
```typescript
// test-utils.tsx
const AllTheProviders: FC<{children: React.ReactNode}> = ({children}) => {
  return (
    <QueryClientProvider client={new QueryClient()}>
      <ErrorBoundary fallback={<ErrorComp />}>
        <Suspense fallback={<PageSpinner />}>
          <BrowserRouter>{children}</BrowserRouter>
        </Suspense>
      </ErrorBoundary>
    </QueryClientProvider>
  )
}

// Custom mount command
Cypress.Commands.add('wrappedMount', (component) => {
  return cy.mount(<AllTheProviders>{component}</AllTheProviders>)
})
```

## Directory Structure

```
project/
├── .nvmrc                    # ALWAYS specify Node version
├── .npmrc                    # Registry configuration
├── playwright.config.ts      # OR cypress.config.ts
├── tests/ (or cypress/)
│   ├── fixtures/            # Test data
│   ├── support/            
│   │   ├── fixtures.ts      # Fixture composition
│   │   ├── fixture-helpers/ # Pure functions
│   │   ├── commands.ts      # Custom commands
│   │   └── utils/          # Shared utilities
│   ├── unit/               # Unit tests
│   ├── integration/        # API/Integration
│   ├── e2e/               # User journeys
│   └── feature-flags/     # FF-specific tests
├── src/
│   ├── components/
│   │   └── **/*.cy.tsx    # Component tests near source
│   └── utils/
│       └── flags.js       # Feature flag enums
└── .github/
    └── workflows/         # CI/CD pipelines
```

## Performance Optimization Strategies

### The 20x Speed Improvement Pattern
1. **Parallel Execution** - Run tests in parallel with proper isolation
2. **Smart Test Selection** - 32+ ways to select relevant tests
3. **API Setup** - Use API for data setup instead of UI navigation
4. **Network Recording** - HAR file recording and playback
5. **Changed File Detection** - Run only affected tests first
6. **Burn-in Testing** - Stress test changed components

### Test Selection Strategies
```bash
# By tag/grep
npm run test -- --grep "@smoke"
npm run test -- --grep "@critical"

# By file pattern
npm run test -- --spec "**/*checkout*"
npm run test -- --spec "cypress/e2e/user/*"

# Changed files only
npm run test:changed

# By test level
npm run test:unit
npm run test:integration
npm run test:e2e
```

## Contract Testing (Pact)

### Consumer-Driven Contract Pattern
```javascript
const interaction = {
  state: 'user with id 1 exists',
  uponReceiving: 'a request for user 1',
  withRequest: {
    method: 'GET',
    path: '/users/1',
    headers: {
      Accept: 'application/json'
    }
  },
  willRespondWith: {
    status: 200,
    headers: {
      'Content-Type': 'application/json'
    },
    body: like({
      id: 1,
      name: string('John Doe'),
      email: email('john@example.com')
    })
  }
}
```

## Definition of Done & Quality Checklist

### Core Requirements
Every test must pass these criteria:

- [ ] **No Flaky Tests** - 0 tolerance. Ensure reliability through proper async handling, explicit waits, and atomic test design
- [ ] **No Hard Waits/Sleeps** - Use dynamic waiting strategies (polling, event-based triggers)
- [ ] **Stateless & Parallelizable** - Tests run independently; use cron jobs or semaphores only if unavoidable
- [ ] **No Order Dependency** - Every it/describe/context block works in isolation (supports .only execution)
- [ ] **Self-Cleaning Tests** - Test sets up its own data and automatically deletes/deactivates entities created
- [ ] **Tests Live Near Source Code** - Co-locate test files with code they validate (e.g., *.spec.js alongside components)
- [ ] **< 200 lines** - Keep test files focused; split/chunk large tests logically
- [ ] **< 1.5 minutes** - Individual test execution time; optimize slow setups with shared fixtures
- [ ] **Explicit Assertions** - Keep them in tests; avoid abstraction into helpers
- [ ] **Proper Selectors** - data-cy/data-testid priority
- [ ] **No Conditionals** - Tests should refrain from if/else or try/catch to control flow

### Test Coverage Strategy

- **Shifted Left**: Start with local environments, validate across all stages (local → dev → stage → prod)
- **Happy Path**: Core user journeys are prioritized
- **Edge Cases**: Critical error/validation scenarios covered
- **Feature Flags**: Test both enabled and disabled states where applicable
- **API Coverage**: Both happy path and negative/error cases
- **Idempotency**: Test duplicate requests where applicable

### CI/CD Integration

- [ ] **CI Execution Evidence** - Integrate into pipelines with clear logs/artifacts
- [ ] **Visibility** - Generate test reports (JUnit XML, HTML) for failures and trends
- [ ] **Response Logs** - Only printed in case of failure
- [ ] **Auth Tests** - Validate token expiration and renewal

### API Testing Specific

- [ ] **Factory Data** - No hardcoded data, use factories and per-test setup
- [ ] **Parallel Safe** - No global state shared
- [ ] **Data Cleanup** - Tests clean up their data
- [ ] **Error Testing** - Always test error scenarios

## Error Handling Patterns

### Expected Error Testing
```javascript
// Cypress
describe('Error Handling', () => {
  it('should handle API errors gracefully', () => {
    Cypress.on('uncaught:exception', (err) => {
      if (err.message.includes('NetworkError')) return false
    })
    
    cy.intercept('GET', '/api/data', { statusCode: 500 })
    cy.visit('/dashboard')
    cy.get('[data-cy="error-message"]').should('be.visible')
  })
})

// Playwright
test('should handle API errors gracefully', async ({ page }) => {
  page.on('pageerror', (error) => {
    if (!error.message.includes('NetworkError')) throw error
  })
  
  await page.route('/api/data', route => 
    route.fulfill({ status: 500 })
  )
  await page.goto('/dashboard')
  await expect(page.getByTestId('error-message')).toBeVisible()
})
```

## Naming Conventions

### File Naming
```
ComponentName.cy.tsx        # Cypress component test
ComponentName.test.tsx      # Jest/RTL test
component-name.spec.ts      # Playwright test
component-name-KEY.spec.ts  # Key pattern demonstration
```

### Test Naming
```javascript
describe('Feature/Component Name', () => {
  context('when condition exists', () => {
    it('should perform expected action', () => {})
    it('should handle error case', () => {})
  })
  
  context('when condition does not exist', () => {
    it('should show default state', () => {})
  })
})
```

### Data Attributes
```html
<!-- Always kebab-case -->
<button data-cy="submit-button">Submit</button>
<input data-testid="user-email" />
```

## Risk-Based Testing Philosophy

### Test Level Decision Matrix
- **P0 (Critical)**: Payment, authentication, data integrity
- **P1 (Core)**: Primary user journeys, business logic
- **P2 (Secondary)**: Supporting features, edge cases
- **P3 (Nice-to-have)**: Cosmetic, non-functional improvements

### Coverage Strategy
```
Unit Tests: 70% - Pure functions, algorithms, utilities
Integration: 20% - API contracts, service boundaries
E2E: 10% - Critical user paths only
```

## Testing Tools Preferences

### Framework Choices
- **Component Testing**: Cypress Component Testing > RTL
- **E2E Testing**: Playwright = Cypress (context-dependent)
- **API Testing**: Playwright API testing > REST clients
- **Contract Testing**: Pact.js
- **Visual Testing**: Percy, Chromatic, or Playwright screenshots

### Supporting Tools
- **Test Data**: Faker.js for dynamic data
- **Network**: MSW for service workers, cy.intercept for Cypress
- **Assertions**: Cypress chains, Playwright expects
- **Reporting**: HTML reports, JUnit for CI

## Key Principles Summary

1. **Shift Left** - Test early in development cycle
2. **Test Pyramid** - More unit/component, fewer E2E
3. **Deterministic** - No randomness, no flakiness
4. **Independent** - Tests don't affect each other
5. **Fast Feedback** - Quick execution, clear failures
6. **User-Centric** - Test from user perspective
7. **Maintainable** - Easy to understand and modify
8. **Documented** - Clear intent and requirements
9. **Automated** - CI/CD integration
10. **Measured** - Track metrics and improve

## TEA Agent Enhancement Commands

The Test Architect agent should implement these streamlined commands that incorporate all the Murat testing philosophy:

```
*automate   - Create comprehensive test automation following all patterns
                  (Generates tests with proper fixtures, helpers, factories, 
                   selectors, network handling, data management - all in Murat style)

*ci         - Setup complete CI/CD pipeline with GitHub Actions
                  (Includes parallel execution, burn-in for changed files,
                   environment-based configs, proper reporting)

*framework  - Initial test framework setup (one-time)
                  (Creates folder structure, base configs, fixture architecture,
                   auth setup, global setup, all utilities)
```

These three commands should internally apply ALL the patterns and principles:
- Functional helpers over Page Objects
- Pure function → Fixture pattern
- Network-first interception
- Factory-based test data
- Proper selector strategy (data-cy/data-testid)
- No hard waits, deterministic waiting
- Self-cleaning tests
- Component-first approach
- Risk-based test design
- Environment-based configuration
- Modular fixture composition

The commands execute comprehensively without requiring users to know the underlying patterns - they just get production-quality test architecture in the Murat style.

## Reference Links for Agent

When the TEA agent needs specific examples or deeper context, it should reference:

### Primary Sources
- CCTDD Book: Component testing patterns and philosophy
- Blog posts in test-resources-for-ai/blog/: Specific patterns and strategies
- Sample repos: Working implementations

### Pattern References
- Functional Helpers: Page-Objects-vs-Functional-Helpers.mkd
- Feature Flags: Effective-Test-Strategies-Feature-Flags.mkd
- CI/CD: CI-CD-strategies.mkd
- Component Testing: Cypress-Component-Testing-vs-RTL.mkd

### Implementation Examples
- **cy-vs-pw-murats-version**: Framework comparison and parallel implementations
- **tour-of-heroes**: Full application with component and E2E tests
- **pact-js-example repos**: Contract testing patterns
- **pw-book**: Comprehensive Playwright patterns and examples

## TODO: TEA Agent Implementation Plan

### Phase 1: Core Integration
- [ ] Integrate this testing philosophy document into the TEA agent at `src/modules/bmm/agents/tea.md`
- [ ] Update existing TEA commands to reference this document for patterns
- [ ] Ensure `*tea-automate`, `*tea-ci`, and `*tea-framework` commands are implemented

### BMAD Method Integration Structure

The BMAD method follows an `agent -> task -> template` architecture. For the three new TEA commands:

#### Task File Locations

**Note**: BMAD is transitioning to a 3-file system per task (md template, yaml process, md checklist).
For now, we create XML-based task definitions in markdown files.

```
src/modules/bmm/testarch/
├── automate.md       # Test automation task (XML format in .md)
├── ci.md            # CI/CD pipeline task (XML format in .md)  
└── framework.md     # Framework setup task (XML format in .md)
```

Templates are embedded within the task XML definitions. Focus on good LLM prompts for user interaction.

### Phase 2: Command Implementation Details

#### *automate Command  
When executed, should:
1. **Check for Epic/Story Context FIRST**:
   - If story/epic exists, collaborate with dev agent to understand:
     - What was implemented (check story tasks/subtasks)
     - Source files modified (from File List in story)
     - API specs if backend work
     - UI components if frontend work
   - If no story/epic, ask user what they're automating:
     - UI feature? → analyze components
     - API endpoint? → check API specs
     - Full stack? → gather both
2. Analyze the current codebase structure (React, Vue, Node.js, etc.)
3. Auto-detect existing test framework (Playwright, Cypress, Jest, Vitest)
   - If no framework detected, prompt user for preference
4. Generate tests following:
   - Functional helper pattern (not Page Objects)
   - Proper fixture architecture with mergeTests
   - Factory-based test data generation
   - Network-first interception setup
   - Proper selector strategy (data-cy/data-testid)
   - Self-cleaning test patterns
   - API setup for data, UI for assertions

#### *ci Command
When executed, should:
1. Detect repository type (GitHub, GitLab, etc.)
2. Generate workflow files with:
   - Environment-based configuration
   - Parallel execution matrix
   - Burn-in testing for changed files
   - Proper caching strategies
   - Test result reporting
   - Artifact storage

#### *framework Command
When executed, should:
1. Create complete folder structure
2. Setup configuration files:
   - Environment-based configs (local, staging, production)
   - Global setup for auth
   - Fixture architecture
3. Install necessary dependencies
4. Create utility functions and helpers
5. Setup example tests demonstrating patterns

### Task Definition Structure

#### *tea-automate Task
```xml
<task id="bmad/bmm/testarch/tea-automate" name="Test Automation Implementation">
  <flow>
    1. Analyze Codebase Structure
    2. Setup Test Architecture 
    3. Implement Core Test Fixtures
    4. Generate Test Scenarios
    5. Implement Test Patterns
    6. Validate Test Quality
  </flow>
  <output>
    - Complete test suite following functional helper pattern
    - Fixture architecture with mergeTests composition
    - Factory-based test data system
    - Self-cleaning, parallelizable tests
    - test-automation-plan.yaml with implementation details
  </output>
</task>
```

#### *tea-ci Task
```xml
<task id="bmad/bmm/testarch/tea-ci" name="CI/CD Pipeline Setup">
  <flow>
    1. Detect Repository Type
    2. Analyze Test Framework
    3. Generate Workflow Files
    4. Setup Parallel Execution
    5. Configure Burn-in Testing
    6. Setup Reporting
  </flow>
  <output>
    - .github/workflows/e2e.yml (or equivalent)
    - Parallel execution matrix
    - Burn-in scripts for changed files
    - Test reporting configuration
    - ci-pipeline.yaml with pipeline details
  </output>
</task>
```

#### *tea-framework Task  
```xml
<task id="bmad/bmm/testarch/tea-framework" name="Test Framework Setup">
  <flow>
    1. Detect Application Framework
    2. Create Directory Structure
    3. Setup Base Configuration
    4. Implement Fixture Architecture
    5. Create Helper Functions
    6. Generate Example Tests
  </flow>
  <output>
    - Complete test directory structure
    - Environment-based configurations
    - Fixture composition setup
    - Helper utilities and factories
    - test-framework-config.yaml with setup details
  </output>
</task>
```

### Template Definitions

#### test-automation-plan.yaml Template
```yaml
template:
  id: test-automation-plan-v1
  name: Test Automation Plan
  output:
    format: yaml
    filename: tea.teaLocation/automation/{{epic_num}}.{{story_num}}-test-plan.yml

schema: 1
story: "{{epic_num}}.{{story_num}}"
framework: "{{test_framework}}"  # playwright|cypress|jest|vitest
application_type: "{{app_type}}"  # react|vue|angular|node
generated: "{{iso_timestamp}}"

test_architecture:
  pattern: "functional_helpers"  # ALWAYS
  fixture_composition: true
  factory_data: true
  network_interception: true
  selector_strategy: "data-testid"  # or data-cy

test_distribution:
  unit: {{unit_count}}
  integration: {{integration_count}}
  e2e: {{e2e_count}}
  
files_generated:
  fixtures: []
  helpers: []
  tests: []
  
coverage_summary:
  acceptance_criteria_covered: []
  risk_mitigations_addressed: []
```

#### ci-pipeline.yaml Template
```yaml
template:
  id: ci-pipeline-v1
  name: CI/CD Pipeline Configuration
  output:
    format: yaml
    filename: tea.teaLocation/ci/pipeline-config.yml

schema: 1
repository_type: "{{repo_type}}"  # github|gitlab|bitbucket
test_framework: "{{test_framework}}"
generated: "{{iso_timestamp}}"

pipeline_features:
  parallel_execution: true
  matrix_strategy:
    containers: {{container_count}}
  burn_in_testing: true
  changed_file_detection: true
  environment_configs: ["local", "staging", "production"]
  
workflow_files:
  - path: ".github/workflows/e2e.yml"
    triggers: ["pull_request", "push"]
  - path: ".github/workflows/burn-in.yml"  
    triggers: ["pull_request"]
    
optimizations:
  cache_strategy: "npm"
  artifact_retention: "30 days"
  test_sharding: true
```

#### test-framework-config.yaml Template
```yaml
template:
  id: test-framework-config-v1
  name: Test Framework Configuration
  output:
    format: yaml
    filename: tea.teaLocation/framework/setup-config.yml

schema: 1
framework: "{{test_framework}}"
application: "{{app_framework}}"
generated: "{{iso_timestamp}}"

directory_structure:
  tests_root: "{{test_dir}}"  # tests/ or cypress/
  fixtures: "{{test_dir}}/fixtures"
  support: "{{test_dir}}/support"
  helpers: "{{test_dir}}/support/helpers"
  
configuration_files:
  - name: "{{framework}}.config.ts"
    environments: ["local", "staging", "production"]
  - name: ".nvmrc"
    node_version: "{{node_version}}"
  - name: ".npmrc"
    registry: "{{npm_registry}}"
    
fixture_architecture:
  base_fixtures:
    - apiRequest
    - network
    - auth
    - log
  composition_method: "mergeTests"
  
utilities_created:
  factories: []
  helpers: []
  commands: []
```

### Phase 3: Advanced Patterns Integration
- [ ] Auth session management patterns from sample repos
- [ ] Contract testing patterns from pact-js repos
- [ ] Component testing patterns from tour-of-heroes
- [ ] Framework comparison patterns from cy-vs-pw repo
- [ ] Advanced intercept patterns with stateful mocking

### Phase 4: Documentation & Examples
- [ ] Create example implementations for each pattern
- [ ] Document migration path from Page Objects to functional helpers
- [ ] Provide framework comparison guide (when to use Playwright vs Cypress)
- [ ] Create troubleshooting guide for common issues

### Integration Notes

**Current TEA Agent Location**: `src/modules/bmm/agents/tea.md`

**Key Integration Points**:
1. The TEA agent should reference this document as its knowledge base
2. Existing commands (*risk, *design, *trace, *nfr, *review, *gate) remain unchanged
3. New commands (*tea-automate, *tea-ci, *tea-framework) are additive

**BMAD Method Task Execution Flow**:
1. User invokes command (e.g., `*automate`)
2. TEA agent triggers corresponding task in `src/modules/bmm/testarch/`
3. Task follows XML-defined flow steps
4. Task generates output using embedded template definitions
5. Results saved to `tea.teaLocation` directory structure

**Task XML Structure Requirements**:
- Must include `<llm critical="true">` instructions
- Define explicit `<flow>` with numbered steps
- Include `<halt-conditions>` for error handling
- Specify `<output>` format and location
- Add `<validation>` criteria

**Template YAML Requirements**:
- Follow BMAD template schema version 1
- Include `template` metadata block
- Define output format and filename pattern
- Use mustache variables ({{variable}})
- Provide examples section for clarity

**Critical Patterns to Enforce**:
- ALWAYS check for story/epic context first (collaborate with dev agent)
- NEVER generate Page Object Model code
- ALWAYS use functional helpers
- ALWAYS setup network interception before navigation
- NEVER use hard waits (wait(3000))
- ALWAYS use factory functions for test data
- ALWAYS prefer API for setup, UI for assertions

### Key Patterns from Open Source Repos

For Playwright testing patterns, use these repositories:
- **Cypress vs Playwright Examples**: [https://github.com/muratkeremozcan/cy-vs-pw-murats-version](https://github.com/muratkeremozcan/cy-vs-pw-murats-version) - Side-by-side implementations showing how to write the same tests in both frameworks
- **Playwright Book Examples**: [https://github.com/muratkeremozcan/pw-book](https://github.com/muratkeremozcan/pw-book) - Comprehensive Playwright patterns and examples

For Contract Testing and Component Testing patterns, use these repositories:
- **Pact.js Consumer Example**: [https://github.com/muratkeremozcan/pact-js-example-consumer](https://github.com/muratkeremozcan/pact-js-example-consumer)
- **Pact.js React Consumer**: [https://github.com/muratkeremozcan/pact-js-example-react-consumer](https://github.com/muratkeremozcan/pact-js-example-react-consumer) - Also includes **Vite component testing patterns**
- **Pact.js Provider Example**: [https://github.com/muratkeremozcan/pact-js-example-provider](https://github.com/muratkeremozcan/pact-js-example-provider)

These repos demonstrate:
- Contract testing with Pact.js
- **Vite component testing setup and patterns** (React Consumer repo)
- Modular fixture patterns
- Pure function → Fixture wrapper pattern
- Factory-based test data generation
- Network interception patterns
- Proper test organization and naming

### Success Criteria

The TEA agent enhancement is successful when:
1. Users can run `*tea-automate` and get tests written in Murat's style
2. Tests follow all patterns without users knowing the patterns exist
3. CI/CD setup works out of the box with optimizations
4. Framework setup creates production-ready test architecture
5. Generated code looks like Murat wrote it himself

## Conclusion

This document represents the complete testing philosophy and implementation patterns of Murat Ozcan. The TEA agent should use these patterns to generate tests, configurations, and architectures that match this style exactly. The goal is to serve as a force multiplier, allowing teams to implement testing strategies as if Murat himself were writing them.

When in doubt, the agent should:
1. Prefer simplicity over complexity
2. Choose composition over inheritance
3. Favor explicit over implicit
4. Prioritize maintainability over cleverness
5. Focus on user value over technical elegance

This is not just a testing strategy - it's a philosophy of building reliable, maintainable, and valuable software through disciplined testing practices.

## Document Status

**Last Updated**: 2025-01-15
**Context**: This document was created by analyzing Murat Ozcan's books, blog posts, and sample repositories to enhance the BMAD Method's Test Architect (TEA) agent.
**Continuity Note**: If continuing work in a new chat session, start by reviewing:
1. The TODO section above for pending implementation tasks
2. BMAD Method Integration Structure section for task/template definitions
3. Check integration status with `src/modules/bmm/agents/tea.md`
4. Verify all task files exist in `src/modules/bmm/testarch/`
5. Commands are *automate, *ci, *framework (no tea- prefix)

**Implementation Readiness**: The document now contains complete task and template structures for the three new TEA commands, ready for implementation in the BMAD method architecture.

## Quick Reference for Implementation

### Answers to Common Questions
1. **TEA Agent Location**: Expand existing TEA agent at `src/modules/bmm/agents/tea.md` (maintain backward compatibility)
2. **Sample Code Access**: Use GitHub URLs directly (not flattened text exports)
3. **Templates**: Create XML task definitions in `src/modules/bmm/testarch/` (BMAD transitioning to 3-file system)
4. **Quality Gate Integration**: New commands (*automate, *ci, *framework) are additive to existing TEA commands
5. **Framework Detection**: Auto-detect; prompt user if unclear