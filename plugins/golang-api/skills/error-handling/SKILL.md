---
description: "Error handling strategies including custom errors, middleware, and logging"
---

# Error Handling

## Custom Error Types

```go
type AppError struct {
    Code    int
    Message string
    Err     error
}

func (e *AppError) Error() string {
    return e.Message
}

// Usage
return &AppError{
    Code:    404,
    Message: "User not found",
    Err:     err,
}
```

## Error Middleware

```go
func ErrorMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next()

        if len(c.Errors) > 0 {
            err := c.Errors.Last().Err

            if appErr, ok := err.(*AppError); ok {
                c.JSON(appErr.Code, gin.H{
                    "success": false,
                    "error": appErr.Message,
                })
                return
            }

            c.JSON(500, gin.H{
                "success": false,
                "error": "Internal server error",
            })
        }
    }
}
```

## Key Principles

1. Define custom error types
2. Wrap errors with context
3. Handle errors at appropriate levels
4. Log errors appropriately
5. Return meaningful error messages
