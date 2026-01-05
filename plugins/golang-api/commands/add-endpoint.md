---
description: "Generate CRUD endpoints for a resource with controller, model, and routes"
argument-hint: "<resource-name>"
---

# Add CRUD Endpoints

This command generates complete CRUD (Create, Read, Update, Delete) endpoints for a resource using Gin framework.

## Usage

```
/golang-api:add-endpoint users
/golang-api:add-endpoint products
/golang-api:add-endpoint orders
```

## What It Creates

### 1. Model (api/models/resource.go)
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

func (u *User) TableName() string {
    return "users"
}
```

### 2. Controller (api/controllers/resourceController.go)
```go
package controllers

import (
    "net/http"

    "github.com/gin-gonic/gin"
    "github.com/your-org/my-api/api/models"
    "github.com/your-org/my-api/api/services"
    "github.com/your-org/my-api/utils"
)

var userService services.UserService

// CreateUser godoc
// @Summary Create a new user
// @Description Create a new user with the provided data
// @Tags users
// @Accept  json
// @Produce  json
// @Param user body models.User true "User object"
// @Success 201 {object} utils.Response
// @Failure 400 {object} utils.Response
// @Router /users [post]
func CreateUser(c *gin.Context) {
    var user models.User
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(http.StatusBadRequest, utils.ErrorResponse(err.Error()))
        return
    }

    createdUser, err := userService.CreateUser(&user)
    if err != nil {
        c.JSON(http.StatusInternalServerError, utils.ErrorResponse(err.Error()))
        return
    }

    c.JSON(http.StatusCreated, utils.SuccessResponse(createdUser))
}

// GetUsers godoc
// @Summary Get all users
// @Description Get list of all users with pagination
// @Tags users
// @Accept  json
// @Produce  json
// @Param page query int false "Page number" default(1)
// @Param limit query int false "Items per page" default(10)
// @Success 200 {object} utils.PaginatedResponse
// @Router /users [get]
func GetUsers(c *gin.Context) {
    page, _ := c.GetQuery("page")
    limit, _ := c.GetQuery("limit")

    users, err := userService.GetUsers(page, limit)
    if err != nil {
        c.JSON(http.StatusInternalServerError, utils.ErrorResponse(err.Error()))
        return
    }

    c.JSON(http.StatusOK, utils.SuccessResponse(users))
}

// GetUserByID godoc
// @Summary Get user by ID
// @Description Get a single user by ID
// @Tags users
// @Accept  json
// @Produce  json
// @Param id path string true "User ID"
// @Success 200 {object} utils.Response
// @Failure 404 {object} utils.Response
// @Router /users/{id} [get]
func GetUserByID(c *gin.Context) {
    id := c.Param("id")

    user, err := userService.GetUserByID(id)
    if err != nil {
        c.JSON(http.StatusNotFound, utils.ErrorResponse("User not found"))
        return
    }

    c.JSON(http.StatusOK, utils.SuccessResponse(user))
}

// UpdateUser godoc
// @Summary Update user
// @Description Update user information
// @Tags users
// @Accept  json
// @Produce  json
// @Param id path string true "User ID"
// @Param user body models.User true "User object"
// @Success 200 {object} utils.Response
// @Failure 404 {object} utils.Response
// @Router /users/{id} [put]
func UpdateUser(c *gin.Context) {
    id := c.Param("id")
    var user models.User

    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(http.StatusBadRequest, utils.ErrorResponse(err.Error()))
        return
    }

    updatedUser, err := userService.UpdateUser(id, &user)
    if err != nil {
        c.JSON(http.StatusNotFound, utils.ErrorResponse(err.Error()))
        return
    }

    c.JSON(http.StatusOK, utils.SuccessResponse(updatedUser))
}

// DeleteUser godoc
// @Summary Delete user
// @Description Delete a user by ID
// @Tags users
// @Accept  json
// @Produce  json
// @Param id path string true "User ID"
// @Success 200 {object} utils.Response
// @Failure 404 {object} utils.Response
// @Router /users/{id} [delete]
func DeleteUser(c *gin.Context) {
    id := c.Param("id")

    err := userService.DeleteUser(id)
    if err != nil {
        c.JSON(http.StatusNotFound, utils.ErrorResponse(err.Error()))
        return
    }

    c.JSON(http.StatusOK, utils.SuccessResponse(nil))
}
```

### 3. Service (api/services/resourceService.go)
```go
package services

import (
    "errors"

    "github.com/your-org/my-api/api/models"
    "github.com/your-org/my-api/api/database"
    "go.mongodb.org/mongo-driver/bson"
    "go.mongodb.org/mongo-driver/bson/primitive"
)

type UserService struct {
    collection *database.Collection
}

func NewUserService() *UserService {
    return &UserService{
        collection: database.GetCollection("users"),
    }
}

func (s *UserService) CreateUser(user *models.User) (*models.User, error) {
    user.ID = primitive.NewObjectID()
    user.CreatedAt = time.Now()
    user.UpdatedAt = time.Now()

    result, err := s.collection.InsertOne(user)
    if err != nil {
        return nil, err
    }

    user.ID = result.InsertedID.(primitive.ObjectID)
    return user, nil
}

func (s *UserService) GetUsers(page, limit string) ([]*models.User, error) {
    var users []*models.User

    // Parse pagination
    skip := 0
    limitInt := 10
    if page != "" && limit != "" {
        skip = (parseInt(page) - 1) * parseInt(limit)
        limitInt = parseInt(limit)
    }

    cursor, err := s.collection.Find(bson.M{},
        options.Find().SetSkip(int64(skip)).SetLimit(int64(limitInt)))
    if err != nil {
        return nil, err
    }
    defer cursor.Close(context.Background())

    for cursor.Next(context.Background()) {
        var user models.User
        if err := cursor.Decode(&user); err != nil {
            return nil, err
        }
        users = append(users, &user)
    }

    return users, nil
}

func (s *UserService) GetUserByID(id string) (*models.User, error) {
    objectID, err := primitive.ObjectIDFromHex(id)
    if err != nil {
        return nil, err
    }

    var user models.User
    err = s.collection.FindOne(bson.M{"_id": objectID}).Decode(&user)
    if err != nil {
        return nil, errors.New("user not found")
    }

    return &user, nil
}

func (s *UserService) UpdateUser(id string, user *models.User) (*models.User, error) {
    objectID, err := primitive.ObjectIDFromHex(id)
    if err != nil {
        return nil, err
    }

    user.UpdatedAt = time.Now()
    update := bson.M{"$set": user}

    _, err = s.collection.UpdateOne(
        bson.M{"_id": objectID},
        update,
    )
    if err != nil {
        return nil, err
    }

    return s.GetUserByID(id)
}

func (s *UserService) DeleteUser(id string) error {
    objectID, err := primitive.ObjectIDFromHex(id)
    if err != nil {
        return err
    }

    _, err = s.collection.DeleteOne(bson.M{"_id": objectID})
    return err
}
```

### 4. Update Routes (api/routes/routes.go)
```go
package routes

import (
    "github.com/gin-gonic/gin"
    "github.com/your-org/my-api/api/controllers"
    "github.com/your-org/my-api/api/middleware"
)

func SetupRoutes(r *gin.Engine) {
    // Health check
    r.GET("/health", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "status": "ok",
            "time":   time.Now(),
        })
    })

    // API v1 routes
    v1 := r.Group("/api/v1")
    {
        // User routes
        users := v1.Group("/users")
        {
            users.Use(middleware.AuthMiddleware())
            users.GET("", controllers.GetUsers)
            users.POST("", controllers.CreateUser)
            users.GET("/:id", controllers.GetUserByID)
            users.PUT("/:id", controllers.UpdateUser)
            users.DELETE("/:id", controllers.DeleteUser)
        }
    }
}
```

## Features

✅ **CRUD Operations**: GET, POST, PUT, DELETE
✅ **MongoDB Integration** (if MongoDB is set up)
✅ **Proper Error Handling** with status codes
✅ **RESTful API Design** following best practices
✅ **Swagger Documentation** for all endpoints
✅ **Validation** using struct tags
✅ **Service Layer** for business logic
✅ **Context Management** for database operations

## API Endpoints Created

- `GET /api/v1/users` - Get all users with pagination
- `GET /api/v1/users/:id` - Get single user by ID
- `POST /api/v1/users` - Create new user
- `PUT /api/v1/users/:id` - Update user
- `DELETE /api/v1/users/:id` - Delete user

## Examples

```bash
# Create a users resource
/golang-api:add-endpoint users

# This creates:
# - api/models/user.go
# - api/controllers/userController.go
# - api/services/userService.go
# - Updates api/routes/routes.go
```

## Customization

After generation, you can:
1. Add custom fields to the model
2. Add validation rules using struct tags
3. Implement business logic in the service
4. Add middleware for authentication/authorization
5. Add query parameters for filtering, pagination, sorting

## Authentication

To protect endpoints, add authentication middleware:

```go
users.Use(middleware.AuthMiddleware())
```

## Next Steps

1. Run `/golang-api:generate-swagger` to add API documentation
2. Add validation using struct tags
3. Implement business logic in the service
4. Write tests for your endpoints
5. Add authorization middleware for role-based access
