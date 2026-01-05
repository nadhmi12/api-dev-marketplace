---
description: "Generate Swagger/OpenAPI documentation for your Express API"
argument-hint: ""
---

# Generate Swagger Documentation

This command adds complete Swagger/OpenAPI documentation to your Express.js API.

## Usage

```
/nodejs-api:generate-swagger
```

## What It Adds

### 1. Update package.json
```json
{
  "dependencies": {
    "swagger-ui-express": "^5.0.0",
    "swagger-jsdoc": "^6.2.8"
  },
  "scripts": {
    "swagger": "node scripts/swagger.js"
  }
}
```

### 2. Create Swagger Configuration (src/config/swagger.js)
```javascript
const swaggerJsdoc = require('swagger-jsdoc');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'Express API Documentation',
      version: '1.0.0',
      description: 'A comprehensive Express.js REST API with authentication and CRUD operations',
      contact: {
        name: 'API Support',
        email: 'support@example.com'
      },
      license: {
        name: 'MIT',
        url: 'https://opensource.org/licenses/MIT'
      }
    },
    servers: [
      {
        url: 'http://localhost:3000',
        description: 'Development server'
      },
      {
        url: 'https://api.example.com',
        description: 'Production server'
      }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        }
      },
      schemas: {
        Error: {
          type: 'object',
          properties: {
            success: {
              type: 'boolean',
              example: false
            },
            error: {
              type: 'string',
              example: 'Error message'
            }
          }
        },
        User: {
          type: 'object',
          required: ['name', 'email'],
          properties: {
            id: {
              type: 'string',
              description: 'User ID'
            },
            name: {
              type: 'string',
              description: 'User name'
            },
            email: {
              type: 'string',
              format: 'email',
              description: 'User email'
            },
            role: {
              type: 'string',
              enum: ['user', 'admin'],
              description: 'User role'
            },
            createdAt: {
              type: 'string',
              format: 'date-time',
              description: 'Creation timestamp'
            },
            updatedAt: {
              type: 'string',
              format: 'date-time',
              description: 'Update timestamp'
            }
          }
        }
      }
    }
  },
  apis: ['./src/routes/*.js', './src/controllers/*.js']
};

const specs = swaggerJsdoc(options);

module.exports = specs;
```

### 3. Update app.js to serve Swagger UI
```javascript
const swaggerUi = require('swagger-ui-express');
const swaggerSpec = require('./config/swagger');

// Swagger documentation
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec, {
  explorer: true,
  customCss: '.swagger-ui .topbar { display: none }',
  customSiteTitle: 'API Documentation'
}));
```

### 4. Create Example Controller with Swagger Comments (src/controllers/userController.js)
```javascript
const User = require('../models/User');
const asyncHandler = require('../utils/asyncHandler');

/**
 * @swagger
 * /users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *         description: Page number
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *         description: Number of users per page
 *     responses:
 *       200:
 *         description: List of users
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 count:
 *                   type: integer
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 *       401:
 *         description: Unauthorized
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Error'
 */
exports.getUsers = asyncHandler(async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip = (page - 1) * limit;

  const users = await User.find().skip(skip).limit(limit);
  const total = await User.countDocuments();

  res.json({
    success: true,
    count: users.length,
    total,
    page,
    pages: Math.ceil(total / limit),
    data: users
  });
});

/**
 * @swagger
 * /users/{id}:
 *   get:
 *     summary: Get user by ID
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *         description: User ID
 *     responses:
 *       200:
 *         description: User details
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   $ref: '#/components/schemas/User'
 *       404:
 *         description: User not found
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Error'
 */
exports.getUser = asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) {
    return res.status(404).json({ success: false, error: 'User not found' });
  }
  res.json({ success: true, data: user });
});

/**
 * @swagger
 * /users:
 *   post:
 *     summary: Create new user
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - name
 *               - email
 *               - password
 *             properties:
 *               name:
 *                 type: string
 *                 example: John Doe
 *               email:
 *                 type: string
 *                 format: email
 *                 example: john@example.com
 *               password:
 *                 type: string
 *                 format: password
 *                 example: password123
 *               role:
 *                 type: string
 *                 enum: [user, admin]
 *                 example: user
 *     responses:
 *       201:
 *         description: User created successfully
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   $ref: '#/components/schemas/User'
 *       400:
 *         description: Bad request
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Error'
 */
exports.createUser = asyncHandler(async (req, res) => {
  const user = await User.create(req.body);
  res.status(201).json({ success: true, data: user });
});

/**
 * @swagger
 * /users/{id}:
 *   put:
 *     summary: Update user
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               name:
 *                 type: string
 *                 example: Jane Doe
 *               email:
 *                 type: string
 *                 format: email
 *                 example: jane@example.com
 *               role:
 *                 type: string
 *                 enum: [user, admin]
 *                 example: admin
 *     responses:
 *       200:
 *         description: User updated successfully
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   $ref: '#/components/schemas/User'
 *       404:
 *         description: User not found
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Error'
 */
exports.updateUser = asyncHandler(async (req, res) => {
  const user = await User.findByIdAndUpdate(
    req.params.id,
    req.body,
    { new: true, runValidators: true }
  );
  if (!user) {
    return res.status(404).json({ success: false, error: 'User not found' });
  }
  res.json({ success: true, data: user });
});

/**
 * @swagger
 * /users/{id}:
 *   delete:
 *     summary: Delete user
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: User deleted successfully
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   type: object
 *       404:
 *         description: User not found
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Error'
 */
exports.deleteUser = asyncHandler(async (req, res) => {
  const user = await User.findByIdAndDelete(req.params.id);
  if (!user) {
    return res.status(404).json({ success: false, error: 'User not found' });
  }
  res.json({ success: true, data: {} });
});
```

### 5. Create Export Script (scripts/swagger.js)
```javascript
const fs = require('fs');
const path = require('path');
const swaggerSpec = require('../src/config/swagger');

const outputPath = path.join(__dirname, '..', 'swagger.json');

fs.writeFileSync(outputPath, JSON.stringify(swaggerSpec, null, 2));
console.log('Swagger specification exported to:', outputPath);
```

## Features

✅ **OpenAPI 3.0 specification**
✅ **Interactive UI** with Swagger UI
✅ **Authentication/Authorization** examples (JWT Bearer)
✅ **Request/Response schemas**
✅ **Error handling documentation**
✅ **Pagination support**
✅ **Common schemas reusability**
✅ **API grouping by tags**
✅ **Export to JSON** for API clients
✅ **Multiple environments** (dev, prod)

## Usage

### 1. Install Dependencies
```bash
npm install swagger-ui-express swagger-jsdoc
```

### 2. Access Documentation

**Development:**
```
http://localhost:3000/api-docs
```

**Production:**
```
https://api.example.com/api-docs
```

### 3. Export OpenAPI Spec
```bash
npm run swagger
```

This creates `swagger.json` in the project root, which can be used with API clients like Postman, Insomnia, or generated code.

## Documentation Structure

### Swagger Tags
Group your endpoints by functionality:
- `Users` - User management
- `Products` - Product operations
- `Orders` - Order processing
- `Auth` - Authentication

### Common Components

Reuse schemas and definitions:
```yaml
components:
  schemas:
    Error:
      type: object
      properties:
        success: boolean
        error: string
    User:
      type: object
      properties:
        id: string
        name: string
```

## Adding Swagger to New Endpoints

### Basic CRUD Pattern

```javascript
/**
 * @swagger
 * /products:
 *   get:
 *     summary: Get all products
 *     tags: [Products]
 *     responses:
 *       200:
 *         description: Success
 */
exports.getProducts = async (req, res) => {
  // your code
};
```

### With Parameters

```javascript
/**
 * @swagger
 * /products/{id}:
 *   get:
 *     summary: Get product by ID
 *     tags: [Products]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 */
exports.getProduct = async (req, res) => {
  // your code
};
```

### With Authentication

```javascript
/**
 * @swagger
 * /products:
 *   post:
 *     summary: Create product
 *     tags: [Products]
 *     security:
 *       - bearerAuth: []
 */
exports.createProduct = async (req, res) => {
  // your code
};
```

## Best Practices

1. **Use meaningful summaries** and descriptions
2. **Include examples** for all request/response bodies
3. **Document all errors** with appropriate status codes
4. **Use tags** to group related endpoints
5. **Reuse schemas** instead of duplicating definitions
6. **Add authentication** to protected endpoints
7. **Include pagination** parameters for list endpoints
8. **Document query parameters** for filtering and sorting

## Integration with API Clients

### Postman
1. Import `swagger.json` to Postman
2. Use "Import" → "Upload Files"
3. Postman will generate requests from your spec

### Code Generation

```bash
# Generate client SDKs
npx @openapitools/openapi-generator-cli generate \
  -i swagger.json \
  -g javascript \
  -o ./client-sdk

# Generate server stubs
npx @openapitools/openapi-generator-cli generate \
  -i swagger.json \
  -g express \
  -o ./server-stub
```

## Next Steps

1. Add Swagger comments to all your endpoints
2. Create additional schemas for your data models
3. Set up automated testing using the spec
4. Generate API clients for frontend
5. Share spec with team members
6. Set up CI to validate spec changes

## Advanced Features

### Multiple Servers
```javascript
servers: [
  {
    url: 'http://localhost:3000',
    description: 'Development'
  },
  {
    url: 'https://staging-api.example.com',
    description: 'Staging'
  },
  {
    url: 'https://api.example.com',
    description: 'Production'
  }
]
```

### Reusable Parameters
```yaml
components:
  parameters:
    PageParam:
      in: query
      name: page
      schema:
        type: integer
      description: Page number
```

### Security Schemes
```yaml
components:
  securitySchemes:
    apiKey:
      type: apiKey
      in: header
      name: X-API-Key
    oauth2:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://example.com/oauth/authorize
          tokenUrl: https://example.com/oauth/token
```

## Troubleshooting

**Swagger UI not loading**
- Check if dependencies are installed
- Verify route is registered in app.js
- Check console for errors

**Spec not updating**
- Restart the server
- Check file paths in swagger configuration
- Verify comments are in correct files
