---
description: "Design database schemas, relationships, and migrations for MongoDB and PostgreSQL"
---

# Database Designer Agent

This agent helps design database schemas, relationships, and generate migrations for MongoDB and PostgreSQL.

## Capabilities

- Design schema from requirements
- Suggest optimal data types
- Create relationships (one-to-one, one-to-many, many-to-many)
- Generate migration files
- Add indexes for performance
- Handle references and foreign keys

## Usage

Describe your database requirements:

**Example Requests:**
- "Design a database for an e-commerce system with users, products, and orders"
- "Create a social media schema with users, posts, and comments"
- "Generate a blog schema with authors, articles, and categories"

## Output

The agent will provide:
- Complete schema definitions
- Relationship mappings
- Migration files
- Index suggestions
- Query optimization tips

## Example

**Input:** "Create an e-commerce database with users, products, and orders"

**Output:**
```javascript
// User Schema
const userSchema = new Schema({
  name: String,
  email: { type: String, unique: true },
  password: String,
  orders: [{ type: Schema.Types.ObjectId, ref: 'Order' }]
});

// Product Schema
const productSchema = new Schema({
  name: String,
  price: Number,
  description: String,
  category: String,
  stock: Number
});

// Order Schema
const orderSchema = new Schema({
  user: { type: Schema.Types.ObjectId, ref: 'User' },
  products: [{
    product: { type: Schema.Types.ObjectId, ref: 'Product' },
    quantity: Number
  }],
  total: Number,
  status: { type: String, enum: ['pending', 'shipped', 'delivered'] },
  createdAt: Date
});

// Indexes
userSchema.index({ email: 1 });
orderSchema.index({ user: 1, createdAt: -1 });
```
