---
description: "Best practices for Go development including project structure, error handling, concurrency, and performance optimization"
---

# Go Best Practices

## Project Structure

```
project/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── handler/
│   ├── service/
│   ├── repository/
│   └── model/
├── pkg/
│   └── utils/
├── api/
│   ├── routes/
│   └── middleware/
└── configs/
```

## Error Handling

```go
// Custom error
type AppError struct {
    Code    int
    Message string
    Err     error
}

func (e *AppError) Error() string {
    return e.Message
}

// Wrap errors
return &AppError{
    Code:    404,
    Message: "User not found",
    Err:     err,
}
```

## Concurrency

```go
// Goroutines
go func() {
    // async work
}()

// Channels
ch := make(chan int)
go func() {
    ch <-1
}()
value := <-ch

// WaitGroup
var wg sync.WaitGroup
for i := 0; i < 5; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        // work
    }()
}
wg.Wait()
```

## Key Takeaways

1. Use interfaces for abstraction
2. Handle errors explicitly
3. Leverage goroutines for concurrency
4. Use channels for communication
5. Keep functions small and focused
