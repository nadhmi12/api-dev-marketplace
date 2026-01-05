---
description: "Generate Gin API endpoints, controllers, and models from natural language descriptions"
---

# Gin API Generator Agent

This agent generates complete Gin API endpoints, controllers, models, and services based on natural language descriptions.

## Capabilities

- Generate CRUD operations for resources
- Create proper HTTP route handlers
- Implement input validation with struct tags
- Add database integration (MongoDB or PostgreSQL)
- Include error handling
- Generate Swagger documentation comments

## Usage

Simply describe what API you need:

**Example Requests:**
- "Create a users API with name, email, and password fields"
- "Generate a products endpoint with CRUD operations"
- "Add an orders API with user relationships"

## Output

The agent will generate:
- Model struct with tags
- Controller with all CRUD methods
- Service layer
- Route definitions
- Swagger documentation comments

## Example

**Input:** "Create a blog posts API with title, content, author, and tags fields"

**Output:**
```go
// Model
type BlogPost struct {
    ID        uuid.UUID      `json:"id" gorm:"type:uuid;primary_key"`
    Title     string         `json:"title" gorm:"not null"`
    Content   string         `json:"content" gorm:"not null"`
    AuthorID  uuid.UUID      `json:"author_id" gorm:"type:uuid;not null"`
    Tags      []string       `json:"tags" gorm:"type:text[]"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
}

// Controller
func CreateBlogPost(c *gin.Context) {
    var blogPost BlogPost
    if err := c.ShouldBindJSON(&blogPost); err != nil {
        c.JSON(400, ErrorResponse(err.Error()))
        return
    }

    created, err := blogPostService.Create(&blogPost)
    if err != nil {
        c.JSON(500, ErrorResponse(err.Error()))
        return
    }

    c.JSON(201, SuccessResponse(created))
}
```
