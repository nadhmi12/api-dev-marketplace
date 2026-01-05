---
description: "Setup CI/CD pipeline with GitHub Actions or GitLab CI"
argument-hint: "<platform>"
---

# Setup CI/CD Pipeline

```bash
/golang-api:setup-ci github
/golang-api:setup-ci gitlab
```

## GitHub Actions (.github/workflows/ci.yml)
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      mongodb:
        image: mongo:7.0
        ports:
          - 27017:27017

      postgres:
        image: postgres:15-alpine
        ports:
          - 5432:5432
        environment:
          - POSTGRES_DB=testdb
          - POSTGRES_USER=postgres
          - POSTGRES_PASSWORD=postgres

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.21'

      - name: Download dependencies
        run: go mod download

      - name: Run tests
        run: go test -v -coverprofile=coverage.out ./...

      - name: Build
        run: go build -v ./...

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage.out
```
