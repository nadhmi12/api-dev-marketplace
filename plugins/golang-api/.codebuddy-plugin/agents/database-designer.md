---
description: "Design database schemas, relationships, and migrations for MongoDB and PostgreSQL"
---

# Database Designer Agent

This agent helps design database schemas, relationships, and generate migrations for MongoDB and PostgreSQL in Go.

## Capabilities

- Design schema from requirements
- Suggest optimal data types
- Create relationships (one-to-one, one-to-many, many-to-many)
- Generate migration files
- Add indexes for performance
- Handle foreign keys and constraints

## Usage

Describe your database requirements:

**Example Requests:**
- "Design a database for an e-commerce system with users, products, and orders"
- "Create a social media schema with users, posts, and comments"
- "Generate a blog schema with authors, articles, and categories"

## Output

The agent will provide:
- Complete struct definitions with tags
- Relationship mappings
- GORM migration setup
- Index suggestions
- Query optimization tips

## Example

**Input:** "Create an e-commerce database with users, products, and orders"

**Output:**
```go
// User Model
type User struct {
    ID        uuid.UUID      `json:"id" gorm:"type:uuid;primary_key;default:gen_random_uuid()"`
    Name      string         `json:"name" gorm:"not null"`
    Email     string         `json:"email" gorm:"uniqueIndex;not null"`
    Password  string         `json:"-" gorm:"not null"`
    Orders    []Order        `json:"orders" gorm:"foreignKey:UserID"`
}

// Product Model
type Product struct {
    ID          uuid.UUID `json:"id" gorm:"type:uuid;primary_key;default:gen_random_uuid()"`
    Name        string    `json:"name" gorm:"not null"`
    Price       float64   `json:"price" gorm:"not null"`
    Description string    `json:"description"`
    Category    string    `json:"category"`
    Stock       int       `json:"stock" gorm:"default:0"`
    OrderItems  []OrderItem `json:"order_items" gorm:"foreignKey:ProductID"`
}

// Order Model
type Order struct {
    ID        uuid.UUID    `json:"id" gorm:"type:uuid;primary_key;default:gen_random_uuid()"`
    UserID    uuid.UUID    `json:"user_id" gorm:"type:uuid;not null"`
    Items     []OrderItem  `json:"items" gorm:"foreignKey:OrderID"`
    Total     float64      `json:"total" gorm:"not null"`
    Status    string       `json:"status" gorm:"default:'pending'"`
    CreatedAt time.Time    `json:"created_at"`
    UpdatedAt time.Time    `json:"updated_at"`
}

// OrderItem Model
type OrderItem struct {
    ID        uuid.UUID `json:"id" gorm:"type:uuid;primary_key;default:gen_random_uuid()"`
    OrderID   uuid.UUID `json:"order_id" gorm:"type:uuid;not null"`
    ProductID uuid.UUID `json:"product_id" gorm:"type:uuid;not null"`
    Quantity  int       `json:"quantity" gorm:"not null"`
    Price     float64   `json:"price" gorm:"not null"`
}

// Indexes
// In AutoMigrate:
// - User.Email (unique)
// - Order.UserID (index)
// - Order.CreatedAt (index)
// - Product.Category (index)
```
