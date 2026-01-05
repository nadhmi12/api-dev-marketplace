---
description: "Add MongoDB integration with official driver to your Gin API"
argument-hint: ""
---

# Setup MongoDB

This command adds MongoDB integration with the official Go driver to your Gin API project.

## Usage

```
/golang-api:setup-mongodb
```

## What It Adds

### 1. Update go.mod
```go
require (
    go.mongodb.org/mongo-driver v1.12.1
)
```

### 2. Create Database Setup (api/database/mongodb.go)
```go
package database

import (
    "context"
    "log"
    "time"

    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
)

var Client *mongo.Client
var Database *mongo.Database

// ConnectMongoDB establishes connection to MongoDB
func ConnectMongoDB(uri string) error {
    clientOptions := options.Client().
        ApplyURI(uri).
        SetMaxPoolSize(100).
        SetMinPoolSize(10).
        SetServerSelectionTimeout(10 * time.Second)

    var err error
    Client, err = mongo.Connect(context.Background(), clientOptions)
    if err != nil {
        return err
    }

    // Ping to verify connection
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    err = Client.Ping(ctx, nil)
    if err != nil {
        return err
    }

    log.Println("Connected to MongoDB!")
    return nil
}

// GetDatabase returns the database instance
func GetDatabase(dbName string) *mongo.Database {
    Database = Client.Database(dbName)
    return Database
}

// GetCollection returns a collection
func GetCollection(collectionName string) *mongo.Collection {
    return Database.Collection(collectionName)
}

// DisconnectMongoDB closes the MongoDB connection
func DisconnectMongoDB() {
    if Client != nil {
        ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
        defer cancel()

        err := Client.Disconnect(ctx)
        if err != nil {
            log.Printf("Error disconnecting from MongoDB: %v", err)
        } else {
            log.Println("Disconnected from MongoDB")
        }
    }
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
    Port        string
    Environment string
    MongoDBURI string
    DBName      string
    JWTSecret   string
}

var AppConfig Config

func LoadConfig() {
    viper.SetConfigFile(".env")
    viper.SetConfigType("env")

    // Set defaults
    viper.SetDefault("PORT", "8080")
    viper.SetDefault("GIN_MODE", "debug")
    viper.SetDefault("MONGODB_URI", "mongodb://localhost:27017/mydb")
    viper.SetDefault("DB_NAME", "mydb")
    viper.SetDefault("JWT_SECRET", "your-secret-key")

    viper.AutomaticEnv()

    if err := viper.ReadInConfig(); err != nil {
        // Use environment variables if .env doesn't exist
        viper.BindEnv("PORT", "PORT")
        viper.BindEnv("MONGODB_URI", "MONGODB_URI")
    }

    AppConfig = Config{
        Port:        viper.GetString("PORT"),
        Environment: viper.GetString("GIN_MODE"),
        MongoDBURI: viper.GetString("MONGODB_URI"),
        DBName:      viper.GetString("DB_NAME"),
        JWTSecret:   viper.GetString("JWT_SECRET"),
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

    // Connect to MongoDB
    if err := database.ConnectMongoDB(configs.AppConfig.MongoDBURI); err != nil {
        log.Fatalf("Failed to connect to MongoDB: %v", err)
    }
    defer database.DisconnectMongoDB()

    // Get database
    database.GetDatabase(configs.AppConfig.DBName)

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
MONGODB_URI=mongodb://localhost:27017/mydb
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
      - MONGODB_URI=mongodb://mongodb:27017/mydb
      - DB_NAME=mydb
    depends_on:
      - mongodb
    networks:
      - app-network

  mongodb:
    image: mongo:7.0
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    environment:
      - MONGO_INITDB_DATABASE=mydb
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  mongodb_data:
    driver: local
```

## Features

✅ **MongoDB Go Driver** for database operations
✅ **Connection management** with pooling
✅ **Environment-based configuration**
✅ **Docker integration** with MongoDB container
✅ **Graceful shutdown** handling
✅ **Context management** for operations
✅ **Connection pooling** for performance

## Setup Instructions

### 1. Install Dependencies
```bash
go get go.mongodb.org/mongo-driver/mongo
```

### 2. Configure Environment
```bash
# Copy .env.example to .env
cp .env.example .env

# Edit .env and add your MongoDB URI
MONGODB_URI=mongodb://localhost:27017/mydb
DB_NAME=mydb
```

### 3. Start MongoDB

**Option A: Local Installation**
```bash
# Install MongoDB (Ubuntu/Debian)
sudo apt-get install mongodb

# Start MongoDB
sudo systemctl start mongodb
```

**Option B: Docker**
```bash
docker-compose up mongodb
```

**Option C: MongoDB Atlas (Cloud)**
```bash
# Get your connection string from MongoDB Atlas
MONGODB_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/mydb
```

## Usage Examples

### Creating a Model
```go
package models

import (
    "time"

    "go.mongodb.org/mongo-driver/bson/primitive"
)

type User struct {
    ID        primitive.ObjectID `json:"id,omitempty" bson:"_id,omitempty"`
    Name      string           `json:"name" bson:"name" validate:"required"`
    Email     string           `json:"email" bson:"email" validate:"required,email"`
    Password  string           `json:"-" bson:"password" validate:"required,min=6"`
    Role      string           `json:"role" bson:"role" validate:"oneof=user admin"`
    CreatedAt time.Time        `json:"created_at" bson:"created_at"`
    UpdatedAt time.Time        `json:"updated_at" bson:"updated_at"`
}
```

### Using in Service
```go
package services

import (
    "context"
    "time"

    "github.com/your-org/my-api/api/database"
    "github.com/your-org/my-api/api/models"
    "go.mongodb.org/mongo-driver/bson"
    "go.mongodb.org/mongo-driver/bson/primitive"
    "go.mongodb.org/mongo-driver/mongo/options"
)

type UserService struct {
    collection *mongo.Collection
}

func NewUserService() *UserService {
    return &UserService{
        collection: database.GetCollection("users"),
    }
}

func (s *UserService) CreateUser(user *models.User) (*models.User, error) {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    user.ID = primitive.NewObjectID()
    user.CreatedAt = time.Now()
    user.UpdatedAt = time.Now()

    result, err := s.collection.InsertOne(ctx, user)
    if err != nil {
        return nil, err
    }

    user.ID = result.InsertedID.(primitive.ObjectID)
    return user, nil
}

func (s *UserService) GetUserByID(id string) (*models.User, error) {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    objectID, err := primitive.ObjectIDFromHex(id)
    if err != nil {
        return nil, err
    }

    var user models.User
    err = s.collection.FindOne(ctx, bson.M{"_id": objectID}).Decode(&user)
    if err != nil {
        return nil, err
    }

    return &user, nil
}
```

### Advanced Operations

```go
// Update with options
filter := bson.M{"_id": objectID}
update := bson.M{"$set": bson.M{"name": "New Name"}}
opts := options.Update().SetUpsert(true)

result, err := s.collection.UpdateOne(ctx, filter, update, opts)

// Aggregation pipeline
pipeline := []bson.M{
    {"$match": bson.M{"status": "active"}},
    {"$group": bson.M{
        "_id":   "$category",
        "count": bson.M{"$sum": 1},
    }},
}

cursor, err := s.collection.Aggregate(ctx, pipeline)

// Index creation
indexModel := mongo.IndexModel{
    Keys: bson.M{"email": 1},
    Options: options.Index().SetUnique(true),
}

_, err = s.collection.Indexes().CreateOne(ctx, indexModel)
```

## Testing

```go
// tests/integration/user.test.go
package integration

import (
    "testing"
    "time"

    "github.com/your-org/my-api/api/database"
    "github.com/your-org/my-api/api/models"
    "github.com/your-org/my-api/api/services"
)

func TestUserService(t *testing.T) {
    // Setup test database connection
    database.ConnectMongoDB("mongodb://localhost:27017/testdb")
    database.GetDatabase("testdb")
    defer database.DisconnectMongoDB()

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

1. Run `/golang-api:add-endpoint <resource>` to create models with MongoDB
2. Add indexes for frequently queried fields
3. Implement relationships between models (referencing or embedding)
4. Add validation and sanitization
5. Set up database seeding for development
6. Implement transactions for complex operations

## Troubleshooting

**Connection Error**
```
Failed to connect to MongoDB: connection refused
```
- Make sure MongoDB is running
- Check your MONGODB_URI in .env
- Verify MongoDB port (default: 27017)

**Authentication Error**
```
Authentication failed
```
- Verify username and password in connection string
- Check if IP whitelist includes your address (Atlas)

**Timeout Errors**
```
context deadline exceeded
```
- Increase connection timeout in ConnectMongoDB
- Check network connectivity
- Verify MongoDB server is responsive
