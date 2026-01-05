---
description: "Add PostgreSQL integration with GORM to your Gin API"
argument-hint: ""
---

# Setup PostgreSQL

This command adds PostgreSQL integration with GORM ORM to your Gin API project.

## Usage

```
/golang-api:setup-postgresql
```

## What It Adds

### 1. Update go.mod
```go
require (
    gorm.io/gorm v1.25.5
    gorm.io/driver/postgres v1.5.4
)
```

### 2. Create Database Setup (api/database/postgresql.go)
```go
package database

import (
    "fmt"
    "log"
    "time"

    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "gorm.io/gorm/logger"
)

var DB *gorm.DB

// ConnectPostgreSQL establishes connection to PostgreSQL
func ConnectPostgreSQL(dsn string) error {
    config := &gorm.Config{
        Logger: logger.Default.LogMode(logger.Info),
        NowFunc: func() time.Time {
            return time.Now().UTC()
        },
    }

    var err error
    DB, err = gorm.Open(postgres.Open(dsn), config)
    if err != nil {
        return fmt.Errorf("failed to connect to database: %w", err)
    }

    sqlDB, err := DB.DB()
    if err != nil {
        return fmt.Errorf("failed to get database instance: %w", err)
    }

    // Connection pool settings
    sqlDB.SetMaxIdleConns(10)
    sqlDB.SetMaxOpenConns(100)
    sqlDB.SetConnMaxLifetime(time.Hour)

    log.Println("Connected to PostgreSQL!")
    return nil
}

// GetDB returns database instance
func GetDB() *gorm.DB {
    return DB
}

// DisconnectPostgreSQL closes PostgreSQL connection
func DisconnectPostgreSQL() {
    if DB != nil {
        sqlDB, err := DB.DB()
        if err != nil {
            log.Printf("Error getting database instance: %v", err)
            return
        }
        err = sqlDB.Close()
        if err != nil {
            log.Printf("Error disconnecting from PostgreSQL: %v", err)
        } else {
            log.Println("Disconnected from PostgreSQL")
        }
    }
}

// MigrateDB runs auto migrations
func MigrateDB(models ...interface{}) error {
    return DB.AutoMigrate(models...)
}
```

### 3. Update configs/config.go
```go
package configs

import (
    "os"

    "github.com/spf13/viper"
)

type Config struct {
    Port         string
    Environment  string
    PostgreSQLURI string
    DBName       string
    JWTSecret    string
}

var AppConfig Config

func LoadConfig() {
    viper.SetConfigFile(".env")
    viper.SetConfigType("env")

    // Set defaults
    viper.SetDefault("PORT", "8080")
    viper.SetDefault("GIN_MODE", "debug")
    viper.SetDefault("POSTGRES_URI", "host=localhost user=postgres password=postgres dbname=mydb port=5432 sslmode=disable")
    viper.SetDefault("DB_NAME", "mydb")
    viper.SetDefault("JWT_SECRET", "your-secret-key")

    viper.AutomaticEnv()

    if err := viper.ReadInConfig(); err != nil {
        viper.BindEnv("PORT", "PORT")
        viper.BindEnv("POSTGRES_URI", "POSTGRES_URI")
    }

    AppConfig = Config{
        Port:         viper.GetString("PORT"),
        Environment:  viper.GetString("GIN_MODE"),
        PostgreSQLURI: viper.GetString("POSTGRES_URI"),
        DBName:       viper.GetString("DB_NAME"),
        JWTSecret:    viper.GetString("JWT_SECRET"),
    }
}
```

### 4. Update cmd/server/main.go
```go
package main

import (
    "log"
    "os"

    "github.com/gin-gonic/gin"
    "github.com/your-org/my-api/api/database"
    "github.com/your-org/my-api/api/routes"
    "github.com/your-org/my-api/configs"
)

func main() {
    // Load configuration
    configs.LoadConfig()

    // Set Gin mode
    gin.SetMode(configs.AppConfig.Environment)

    // Connect to PostgreSQL
    if err := database.ConnectPostgreSQL(configs.AppConfig.PostgreSQLURI); err != nil {
        log.Fatalf("Failed to connect to PostgreSQL: %v", err)
    }
    defer database.DisconnectPostgreSQL()

    // Setup Gin router
    r := gin.Default()

    // Setup routes
    routes.SetupRoutes(r)

    // Start server
    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    log.Printf("Server starting on port %s", port)
    if err := r.Run(":" + port); err != nil {
        log.Fatalf("Failed to start server: %v", err)
    }
}
```

### 5. Update .env.example
```bash
PORT=8080
GIN_MODE=debug
POSTGRES_URI=host=localhost user=postgres password=postgres dbname=mydb port=5432 sslmode=disable TimeZone=UTC
DB_NAME=mydb
JWT_SECRET=your-secret-key
LOG_LEVEL=debug
```

### 6. Update docker-compose.yml
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - POSTGRES_URI=host=postgres user=postgres password=postgres dbname=mydb port=5432 sslmode=disable TimeZone=UTC
      - DB_NAME=mydb
    depends_on:
      - postgres
    networks:
      - app-network

  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_INITDB_ARGS=--encoding=UTF-8
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  postgres_data:
    driver: local
```

## Features

✅ **GORM ORM** for database modeling
✅ **Connection management** with pooling
✅ **Environment-based configuration**
✅ **Model relationships** support (hasMany, belongsTo, etc.)
✅ **Validation** at the model level
✅ **Migrations** via AutoMigrate
✅ **Soft deletes** with DeletedAt field
✅ **Docker integration** with PostgreSQL container
✅ **UUID primary keys** support

## Setup Instructions

### 1. Install Dependencies
```bash
go get gorm.io/gorm
go get gorm.io/driver/postgres
```

### 2. Configure Environment
```bash
# Copy .env.example to .env
cp .env.example .env

# Edit .env with your PostgreSQL configuration
POSTGRES_URI=host=localhost user=postgres password=postgres dbname=mydb port=5432 sslmode=disable TimeZone=UTC
```

### 3. Start PostgreSQL

**Option A: Local Installation**
```bash
# Install PostgreSQL
sudo apt-get install postgresql postgresql-contrib

# Start PostgreSQL
sudo systemctl start postgresql

# Create database
sudo -u postgres psql
CREATE DATABASE mydb;
\q
```

**Option B: Docker**
```bash
docker-compose up postgres
```

**Option C: Cloud Database (Neon, Supabase, AWS RDS)**
```bash
# Get your connection string from your cloud provider
POSTGRES_URI=host=your-host user=your-user password=your-password dbname=your-db port=5432 sslmode=require TimeZone=UTC
```

## Usage Examples

### Creating a Model
```go
package models

import (
    "time"

    "gorm.io/gorm"
    "github.com/google/uuid"
)

type User struct {
    ID        uuid.UUID      `json:"id" gorm:"type:uuid;primary_key;default:gen_random_uuid()"`
    Name      string         `json:"name" gorm:"not null" validate:"required"`
    Email     string         `json:"email" gorm:"uniqueIndex;not null" validate:"required,email"`
    Password  string         `json:"-" gorm:"not null" validate:"required,min=6"`
    Role      string         `json:"role" gorm:"default:user" validate:"oneof=user admin"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
    DeletedAt gorm.DeletedAt `json:"-" gorm:"index"`
}
```

### Using in Service
```go
package services

import (
    "github.com/your-org/my-api/api/database"
    "github.com/your-org/my-api/api/models"
    "github.com/google/uuid"
)

type UserService struct {
    db *gorm.DB
}

func NewUserService() *UserService {
    return &UserService{
        db: database.GetDB(),
    }
}

func (s *UserService) CreateUser(user *models.User) (*models.User, error) {
    result := s.db.Create(user)
    if result.Error != nil {
        return nil, result.Error
    }
    return user, nil
}

func (s *UserService) GetUserByID(id uuid.UUID) (*models.User, error) {
    var user models.User
    result := s.db.First(&user, "id = ?", id)
    if result.Error != nil {
        return nil, result.Error
    }
    return &user, nil
}
```

### Model Relationships
```go
type User struct {
    ID        uuid.UUID      `json:"id" gorm:"type:uuid;primary_key"`
    Name      string         `json:"name"`
    Orders    []Order        `json:"orders" gorm:"foreignKey:UserID"`
}

type Order struct {
    ID     uuid.UUID `json:"id" gorm:"type:uuid;primary_key"`
    UserID uuid.UUID `json:"user_id" gorm:"type:uuid;not null"`
    Items  []OrderItem `json:"items" gorm:"foreignKey:OrderID"`
}

type OrderItem struct {
    ID      uuid.UUID `json:"id" gorm:"type:uuid;primary_key"`
    OrderID uuid.UUID `json:"order_id" gorm:"type:uuid;not null"`
    Name    string    `json:"name"`
    Price   float64   `json:"price"`
}

// Querying with relationships
var user models.User
database.GetDB().Preload("Orders.Items").First(&user, "id = ?", userID)
```

### Advanced Queries
```go
// Pagination
var users []models.User
offset := (page - 1) * limit
result := s.db.Offset(offset).Limit(limit).Find(&users)

// Filtering
result := s.db.Where("role = ?", "admin").Find(&users)

// Sorting
result := s.db.Order("created_at DESC").Find(&users)

// Searching
result := s.db.Where("name LIKE ?", "%"+search+"%").Find(&users)

// Transactions
err := s.db.Transaction(func(tx *gorm.DB) error {
    if err := tx.Create(&user).Error; err != nil {
        return err
    }
    if err := tx.Create(&order).Error; err != nil {
        return err
    }
    return nil
})
```

## Testing

```go
// tests/integration/user.test.go
package integration

import (
    "testing"

    "github.com/your-org/my-api/api/database"
    "github.com/your-org/my-api/api/models"
    "github.com/your-org/my-api/api/services"
)

func TestUserService(t *testing.T) {
    // Setup test database connection
    database.ConnectPostgreSQL("host=localhost user=postgres password=postgres dbname=testdb port=5432 sslmode=disable")
    defer database.DisconnectPostgreSQL()

    // Run migrations
    database.MigrateDB(&models.User{})

    userService := services.NewUserService()

    t.Run("CreateUser", func(t *testing.T) {
        user := &models.User{
            Name:     "Test User",
            Email:    "test@example.com",
            Password: "password123",
            Role:     "user",
        }

        created, err := userService.CreateUser(user)
        if err != nil {
            t.Fatalf("Failed to create user: %v", err)
        }

        if created.Name != user.Name {
            t.Errorf("Expected name %s, got %s", user.Name, created.Name)
        }
    })
}
```

## Next Steps

1. Run `/golang-api:add-endpoint <resource>` to create models with PostgreSQL
2. Set up migrations for version control
3. Define model relationships
4. Add indexes for performance
5. Implement data seeding for development
6. Use transactions for complex operations

## Troubleshooting

**Connection Error**
```
failed to connect to database: connection refused
```
- Make sure PostgreSQL is running
- Check your POSTGRES_URI in .env
- Verify database exists

**Authentication Error**
```
password authentication failed
```
- Verify POSTGRES_URI includes correct username and password
- Check pg_hba.conf if using local PostgreSQL

**Migration Errors**
```
column already exists
```
- Drop and recreate database for testing
- Use proper migration tools (golang-migrate)
