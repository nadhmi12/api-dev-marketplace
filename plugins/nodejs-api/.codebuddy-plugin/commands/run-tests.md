---
description: "Run tests with different options and generate reports"
argument-hint: "[test-type] [options]"
---

# Run Tests

This command provides various options to run tests with different configurations and generate reports.

## Usage

```bash
/nodejs-api:run-tests                    # Run all tests
/nodejs-api:run-tests unit               # Run only unit tests
/nodejs-api:run-tests integration        # Run only integration tests
/nodejs-api:run-tests coverage           # Run tests with coverage report
/nodejs-api:run-tests watch              # Run tests in watch mode
/nodejs-api:run-tests verbose            # Run tests with verbose output
```

## Test Types

### All Tests
```bash
npm test
```
Runs all tests (unit and integration) with coverage report.

### Unit Tests Only
```bash
npm run test:unit
```
Tests individual functions and classes in isolation:
- Models
- Controllers
- Services
- Utilities

### Integration Tests Only
```bash
npm run test:integration
```
Tests API endpoints and database interactions:
- HTTP requests
- Database operations
- Authentication/authorization
- Error handling

### Watch Mode
```bash
npm run test:watch
```
Automatically re-runs tests when files change. Ideal for development.

### Coverage Report
```bash
npm run test:coverage
```
Generates a detailed coverage report showing:
- Lines covered
- Branches covered
- Functions covered
- Statements covered

## Test Coverage Report

### View Coverage in Terminal
```bash
npm test
```
Coverage summary is displayed in the terminal.

### View Detailed HTML Report
```bash
# After running tests
open coverage/lcov-report/index.html
# or
xdg-open coverage/lcov-report/index.html
```

### Coverage Thresholds

Set minimum coverage requirements in `jest.config.js`:

```javascript
module.exports = {
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/controllers/': {
      branches: 90,
      functions: 90
    }
  }
};
```

## Filtering Tests

### Run Specific Test File
```bash
npx jest tests/unit/models/user.test.js
```

### Run Tests Matching Pattern
```bash
# Run all user-related tests
npx jest --testNamePattern="User"

# Run tests containing "create"
npx jest --testNamePattern="create"
```

### Run Tests in Specific Directory
```bash
npx jest tests/unit/controllers
```

### Run Tests by File Pattern
```bash
npx jest "**/*.test.js"
```

## Advanced Options

### Debug Mode
```bash
node --inspect-brk node_modules/.bin/jest --runInBand
```

### Verbose Output
```bash
npm run test:verbose
```

### Update Snapshots
```bash
npx jest --updateSnapshot
```

### List All Tests
```bash
npx jest --listTests
```

## Test Reports

### JSON Report
```bash
npx jest --json --outputFile=test-results.json
```

### JUnit Report (for CI/CD)
```bash
npx jest --ci --reporters=default --reporters=jest-junit
```

### Tap Report
```bash
npx jest --coverage --coverageReporters=text-lcov > coverage.lcov
```

## Running Tests Before Commit

### Using Git Hooks (Husky)
```bash
# Install Husky
npm install husky --save-dev

# Setup Husky
npx husky install
npx husky add .husky/pre-commit "npm test"
```

### Using lint-staged
```bash
# Install lint-staged
npm install lint-staged --save-dev

# Add to package.json
{
  "lint-staged": {
    "*.js": [
      "npm test -- --findRelatedTests"
    ]
  }
}
```

## CI/CD Integration

### GitHub Actions
```yaml
name: Run Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      mongodb:
        image: mongo:7.0
        ports:
          - 27017:27017

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
        env:
          TEST_MONGODB_URI: mongodb://localhost:27017/testdb

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

### GitLab CI
```yaml
stages:
  - test

test:
  stage: test
  image: node:20
  services:
    - mongo:7.0

  variables:
    TEST_MONGODB_URI: mongodb://mongo:27017/testdb

  script:
    - npm ci
    - npm test

  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
```

## Performance Testing

### Measure Test Execution Time
```bash
npx jest --bail --maxWorkers=50%
```

### Run Tests Sequentially
```bash
npx jest --runInBand
```

## Troubleshooting

### Tests Fail Randomly (Flaky Tests)

1. **Add retry logic**:
```javascript
jest.retryTimes(3);
```

2. **Increase timeout**:
```javascript
it('should complete', async () => {
  // test code
}, 10000); // 10 second timeout
```

3. **Fix timing issues**:
- Use proper await/async
- Mock time-dependent functions
- Ensure cleanup between tests

### Memory Issues

```javascript
// In jest.config.js
module.exports = {
  maxWorkers: 1, // Run tests sequentially
  maxConcurrency: 1
};
```

### Database Connection Issues

```javascript
// In tests/setup.js
beforeAll(async () => {
  await mongoose.connect(MONGODB_URI, {
    serverSelectionTimeoutMS: 5000
  });
});
```

## Best Practices

1. **Run tests before every commit**
2. **Keep test execution time under 5 minutes**
3. **Aim for 80%+ code coverage**
4. **Write descriptive test names**
5. **Test edge cases and error conditions**
6. **Use watch mode during development**
7. **Fix failing tests immediately**
8. **Keep tests independent of each other**
9. **Mock external dependencies**
10. **Run tests in CI/CD pipeline**

## Next Steps

1. Set up pre-commit hooks to run tests
2. Configure test coverage reporting in CI/CD
3. Add performance benchmarks
4. Set up test data fixtures
5. Implement test containers for isolation
