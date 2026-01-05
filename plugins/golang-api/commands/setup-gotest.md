---
description: "Setup Go testing framework with configuration and sample tests"
argument-hint: ""
---

# Setup Go Testing

This command configures Go testing framework for your Gin API with sample tests.

## Usage

```
/golang-api:setup-gotest
```

## What It Adds

### 1. Create test directories
```
tests/
├── unit/
├── integration/
└── fixtures/
```

### 2. Create helper functions (tests/test_helper.go)
```go
package tests

import (
    "os"
    "testing"

    "github.com/your-org/my-api/api/database"
    "github.com/your-org/my-api/configs"
)

func SetupTestDB(t *testing.T) {
    configs.AppConfig.MongoDBURI = "mongodb://localhost:27017/testdb"
    if err := database.ConnectMongoDB(configs.AppConfig.MongoDBURI); err != nil {
        t.Fatalf("Failed to connect to test database: %v", err)
    }
    database.GetDatabase("testdb")
}

func TeardownTestDB(t *testing.T) {
    database.DisconnectMongoDB()
}
```

### 3. Update Makefile
```makefile
test:
    go test -v ./...

test-cov:
    go test -coverprofile=coverage.out ./...
    go tool cover -html=coverage.out -o coverage.html

test-race:
    go test -race ./...

test-integration:
    go test -v ./tests/integration/...
```

## Usage

```bash
make test
make test-cov
```
