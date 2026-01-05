---
description: "API design patterns including REST, GraphQL, versioning, pagination, and authentication"
---

# API Design Patterns

This skill provides comprehensive guidance on designing robust, scalable APIs using industry-standard patterns.

## REST API Design

### 1. Resource Naming

**Use nouns, not verbs:**

```javascript
// ❌ Bad
GET /getUsers
POST /createUser
PUT /updateUser
DELETE /deleteUser

// ✅ Good
GET /users
POST /users
PUT /users/:id
DELETE /users/:id
```

### 2. HTTP Methods

**Use appropriate HTTP methods:**

```javascript
GET    /users          - Retrieve all users
GET    /users/:id      - Retrieve specific user
POST   /users          - Create new user
PUT    /users/:id      - Update user (full replacement)
PATCH  /users/:id      - Update user (partial update)
DELETE /users/:id      - Delete user
```

### 3. Nested Resources

**Handle relationships properly:**

```javascript
// Get orders for a specific user
GET /users/:userId/orders

// Get specific order for a user
GET /users/:userId/orders/:orderId

// Create order for user
POST /users/:userId/orders
```

### 4. Filtering, Sorting, and Pagination

**Implement query parameters for data manipulation:**

```javascript
// Filtering
GET /users?role=admin&status=active

// Searching
GET /users?search=john

// Sorting
GET /users?sort=name:asc
GET /users?sort=price:desc

// Pagination
GET /users?page=1&limit=10

// Field selection
GET /users?fields=name,email,role

// Combined
GET /users?role=admin&page=1&limit=20&sort=name:asc&fields=name,email
```

### 5. Implementation Example

```javascript
exports.getUsers = async (req, res) => {
  // Query parameters
  const {
    role,
    status,
    search,
    sort = 'createdAt',
    page = 1,
    limit = 10,
    fields
  } = req.query;

  // Build query
  let query = User.find();

  // Filtering
  if (role) {
    query = query.where('role').equals(role);
  }

  if (status) {
    query = query.where('status').equals(status);
  }

  // Searching
  if (search) {
    query = query.find({
      $or: [
        { name: { $regex: search, $options: 'i' } },
        { email: { $regex: search, $options: 'i' } }
      ]
    });
  }

  // Sorting
  const [sortField, sortOrder] = sort.split(':');
  const sortObj = {};
  sortObj[sortField] = sortOrder === 'desc' ? -1 : 1;
  query = query.sort(sortObj);

  // Field selection
  if (fields) {
    const selectedFields = fields.split(',').join(' ');
    query = query.select(selectedFields);
  }

  // Pagination
  const skip = (page - 1) * limit;
  query = query.skip(skip).limit(limit);

  // Execute query
  const users = await query;
  const total = await User.countDocuments(query.getFilter());

  res.json({
    success: true,
    count: users.length,
    total,
    page: parseInt(page),
    pages: Math.ceil(total / limit),
    data: users
  });
};
```

## API Versioning

### 1. URL Versioning (Recommended)

```javascript
// Version 1
GET /api/v1/users
POST /api/v1/users

// Version 2
GET /api/v2/users
POST /api/v2/users

// Implementation
const v1Routes = require('./routes/v1');
const v2Routes = require('./routes/v2');

app.use('/api/v1', v1Routes);
app.use('/api/v2', v2Routes);
```

## Best Practices Summary

1. ✅ **Use nouns** for resource names
2. ✅ **Use appropriate HTTP methods** (GET, POST, PUT, DELETE)
3. ✅ **Implement pagination** for large datasets
4. ✅ **Support filtering, sorting, and searching**
5. ✅ **Use API versioning** (URL-based is recommended)
6. ✅ **Return consistent response formats**
7. ✅ **Use proper HTTP status codes**
8. ✅ **Follow REST principles** and HATEOAS
