---
description: "Security best practices including authentication, authorization, input validation, and protection against common vulnerabilities"
---

# Security Considerations

## JWT Authentication

```go
import "github.com/golang-jwt/jwt/v5"

func GenerateToken(userID string) (string, error) {
    claims := jwt.MapClaims{
        "user_id": userID,
        "exp":     time.Now().Add(time.Hour * 24).Unix(),
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString([]byte(secret))
}
```

## Input Validation

```go
import "github.com/go-playground/validator/v10"

type User struct {
    Name  string `validate:"required,min=2,max=50"`
    Email string `validate:"required,email"`
}

func ValidateStruct(u User) error {
    validate := validator.New()
    return validate.Struct(u)
}
```

## Password Hashing

```go
import "golang.org/x/crypto/bcrypt"

func HashPassword(password string) (string, error) {
    bytes, err := bcrypt.GenerateFromPassword([]byte(password), 14)
    return string(bytes), err
}
```

## Security Checklist

1. ✅ Use HTTPS in production
2. ✅ Validate all inputs
3. ✅ Hash passwords with bcrypt
4. ✅ Use JWT for authentication
5. ✅ Implement rate limiting
6. ✅ Set security headers
7. ✅ Use prepared statements
8. ✅ Keep dependencies updated
