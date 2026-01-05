---
description: "API design patterns including REST, GraphQL, versioning, pagination, and authentication"
---

# API Design Patterns

## RESTful Design

```go
// GET /api/v1/users
r.GET("/users", handlers.GetUsers)

// GET /api/v1/users/:id
r.GET("/users/:id", handlers.GetUserByID)

// POST /api/v1/users
r.POST("/users", handlers.CreateUser)

// PUT /api/v1/users/:id
r.PUT("/users/:id", handlers.UpdateUser)

// DELETE /api/v1/users/:id
r.DELETE("/users/:id", handlers.DeleteUser)
```

## Pagination

```go
type Pagination struct {
    Page  int `form:"page"`
    Limit int `form:"limit"`
}

func Paginate(page, limit int) (int, int) {
    if page < 1 {
        page = 1
    }
    if limit < 1 || limit > 100 {
        limit = 10
    }
    offset := (page - 1) * limit
    return offset, limit
}
```

## Key Principles

1. Use nouns for resources
2. Use appropriate HTTP methods
3. Implement pagination for lists
4. Use consistent response formats
5. Version your APIs
