---
description: "Generate Swagger/OpenAPI documentation for your Gin API"
argument-hint: ""
---

# Generate Swagger Documentation

This command adds complete Swagger/OpenAPI documentation to your Gin API.

## Usage

```
/golang-api:generate-swagger
```

## What It Adds

### 1. Update go.mod
```go
require (
    github.com/swaggo/gin-swagger v1.6.0
    github.com/swaggo/files v1.0.1
    github.com/swaggo/swag v1.16.2
)
```

### 2. Add to main.go
```go
// @title           Example API
// @version         1.0
// @description     This is a sample server Petstore server.
// @termsOfService  http://swagger.io/terms/

// @contact.name   API Support
// @contact.url    http://www.swagger.io/support
// @contact.email  support@swagger.io

// @license.name  Apache 2.0
// @license.url   http://www.apache.org/licenses/LICENSE-2.0.html

// @host      localhost:8080
// @BasePath  /api/v1
```

### 3. Add Swagger Route
```go
import (
    swaggerFiles "github.com/swaggo/files"
    ginSwagger "github.com/swaggo/gin-swagger"
)

r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
```

## Generating Documentation

```bash
swag init -g cmd/server/main.go -o docs
```

## Examples

```go
// CreateUser godoc
// @Summary Create a new user
// @Description Create a new user with the input payload
// @Tags users
// @Accept json
// @Produce json
// @Param user body models.User true "User object"
// @Success 201 {object} utils.Response
// @Failure 400 {object} utils.Response
// @Router /users [post]
func CreateUser(c *gin.Context) {
    // implementation
}
```

## Access Documentation

```
http://localhost:8080/swagger/index.html
```

## Features

✅ OpenAPI 3.0 specification
✅ Interactive UI with Swagger
✅ Auto-generated from code comments
✅ Type-safe documentation
