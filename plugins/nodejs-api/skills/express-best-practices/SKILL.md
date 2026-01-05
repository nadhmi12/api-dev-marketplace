---
description: "Best practices for developing Express.js applications including middleware, error handling, security, and performance optimization"
---

# Express.js Best Practices

This skill provides comprehensive guidance on developing production-ready Express.js applications following industry best practices.

## Core Principles

### 1. Middleware Architecture

**Use middleware efficiently and in the correct order:**

```javascript
// 1. Security headers (first)
app.use(helmet());
app.use(cors());

// 2. Logging (early)
app.use(morgan('combined'));

// 3. Body parsing (before routes)
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// 4. Static files (if needed)
app.use(express.static('public'));

// 5. API routes
app.use('/api', apiRoutes);

// 6. Error handling (last)
app.use(notFound);
app.use(errorHandler);
```

### 2. Error Handling

**Implement comprehensive error handling:**

```javascript
// Custom error class
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
    this.isOperational = true;

    Error.captureStackTrace(this, this.constructor);
  }
}

// Async error wrapper
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Global error handler
const errorHandler = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  if (process.env.NODE_ENV === 'development') {
    res.status(err.statusCode).json({
      success: false,
      error: err,
      message: err.message,
      stack: err.stack
    });
  } else {
    res.status(err.statusCode).json({
      success: false,
      message: err.message
    });
  }
};

// Usage in controller
exports.getUser = asyncHandler(async (req, res, next) => {
  const user = await User.findById(req.params.id);

  if (!user) {
    return next(new AppError('User not found', 404));
  }

  res.json({ success: true, data: user });
});
```

### 3. Security Best Practices

**Implement robust security measures:**

```javascript
// 1. Set security headers with Helmet
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"]
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));

// 2. Rate limiting
const rateLimiter = require('express-rate-limit');
const apiLimiter = rateLimiter({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP, please try again later.'
});
app.use('/api', apiLimiter);

// 3. Input validation
const { body, param, query, validationResult } = require('express-validator');

const validateRequest = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      errors: errors.array()
    });
  }
  next();
};

// Validation middleware
exports.validateUser = [
  body('name').trim().notEmpty().withMessage('Name is required'),
  body('email').isEmail().normalizeEmail().withMessage('Valid email required'),
  body('password').isLength({ min: 6 }).withMessage('Password must be at least 6 characters'),
  validateRequest
];

// 4. Sanitize user input
const xss = require('xss-clean');
app.use(xss());

// 5. Prevent parameter pollution
app.use(hpp({
  whitelist: ['price', 'rating', 'category']
}));
```

### 4. Environment Configuration

**Use environment variables:**

```javascript
const dotenv = require('dotenv');
dotenv.config();

const config = {
  env: process.env.NODE_ENV || 'development',
  port: parseInt(process.env.PORT, 10) || 3000,
  mongodb: {
    uri: process.env.MONGODB_URI
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRE || '30d'
  },
  cors: {
    origin: process.env.CORS_ORIGIN || '*'
  }
};

module.exports = config;
```

## Performance Optimization

### 1. Compression

```javascript
const compression = require('compression');
app.use(compression());
```

### 2. Response Time

```javascript
const responseTime = require('response-time');
app.use(responseTime());
```

### 3. Caching

```javascript
const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 600 }); // 10 minutes

const cacheMiddleware = (duration) => {
  return (req, res, next) => {
    const key = req.originalUrl || req.url;
    const cached = cache.get(key);

    if (cached) {
      return res.json(cached);
    }

    res.sendResponse = res.json;
    res.json = (body) => {
      cache.set(key, body, duration);
      res.sendResponse(body);
    };
    next();
  };
};

// Usage
app.get('/api/users', cacheMiddleware(300), getUsers);
```

### 4. Connection Pooling

```javascript
// MongoDB connection
const mongoose = require('mongoose');
mongoose.connect(process.env.MONGODB_URI, {
  maxPoolSize: 10,
  minPoolSize: 5,
  serverSelectionTimeoutMS: 5000,
  socketTimeoutMS: 45000
});
```

### 5. Query Optimization

```javascript
// Use select to limit returned fields
const users = await User.find().select('name email');

// Use lean for better performance
const users = await User.find().lean();

// Use indexes in schema
userSchema.index({ email: 1 });
userSchema.index({ createdAt: -1 });

// Pagination
const page = parseInt(req.query.page) || 1;
const limit = parseInt(req.query.limit) || 10;
const skip = (page - 1) * limit;

const users = await User.find().skip(skip).limit(limit);
```

## Structure Best Practices

### 1. Separation of Concerns

```
src/
├── config/          # Configuration files
├── controllers/     # Request handlers
├── models/          # Database models
├── routes/          # Route definitions
├── services/        # Business logic
├── middleware/      # Custom middleware
├── utils/           # Utility functions
├── validators/      # Request validators
└── app.js          # App configuration
```

### 2. Route Organization

```javascript
// routes/index.js
const express = require('express');
const router = express.Router();

router.use('/users', require('./users'));
router.use('/products', require('./products'));
router.use('/orders', require('./orders'));

module.exports = router;
```

### 3. Controller Pattern

```javascript
// controllers/userController.js
const User = require('../models/User');
const asyncHandler = require('../utils/asyncHandler');

// @desc    Get all users
// @route   GET /api/users
// @access  Private/Admin
exports.getUsers = asyncHandler(async (req, res) => {
  const users = await User.find();
  res.json({ success: true, count: users.length, data: users });
});
```

### 4. Service Layer

```javascript
// services/userService.js
const User = require('../models/User');

class UserService {
  async getAllUsers(filters = {}) {
    return await User.find(filters);
  }

  async getUserById(id) {
    return await User.findById(id);
  }

  async createUser(userData) {
    return await User.create(userData);
  }

  async updateUser(id, userData) {
    return await User.findByIdAndUpdate(id, userData, { new: true });
  }

  async deleteUser(id) {
    return await User.findByIdAndDelete(id);
  }
}

module.exports = new UserService();
```

## Logging Best Practices

```javascript
// utils/logger.js
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.NODE_ENV === 'development' ? 'debug' : 'info',
  format: winston.format.combine(
    winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' })
  ]
});

if (process.env.NODE_ENV === 'development') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

module.exports = logger;

// Usage
logger.info('Server started', { port: 3000 });
logger.error('Database connection failed', { error: err.message });
logger.debug('User request', { userId: req.user.id, endpoint: req.url });
```

## API Design Principles

### 1. RESTful Design

```javascript
// GET /api/users           - Get all users
// GET /api/users/:id       - Get specific user
// POST /api/users          - Create user
// PUT /api/users/:id       - Update user (full)
// PATCH /api/users/:id     - Update user (partial)
// DELETE /api/users/:id    - Delete user
```

### 2. Consistent Response Format

```javascript
// Success response
{
  "success": true,
  "data": { ... },
  "message": "Operation successful"
}

// Error response
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE"
}

// List response with pagination
{
  "success": true,
  "count": 100,
  "page": 1,
  "pages": 10,
  "data": [...]
}
```

### 3. HTTP Status Codes

```javascript
200 OK                    - Request succeeded
201 Created               - Resource created
204 No Content            - Successful deletion
400 Bad Request           - Invalid request
401 Unauthorized          - Authentication required
403 Forbidden             - Access denied
404 Not Found             - Resource not found
409 Conflict              - Resource already exists
422 Unprocessable Entity  - Validation error
429 Too Many Requests    - Rate limit exceeded
500 Internal Server Error - Server error
```

## Testing Best Practices

```javascript
// Unit test for controller
describe('UserController', () => {
  it('should get user by ID', async () => {
    const req = { params: { id: '123' } };
    const res = {
      status: jest.fn().mockReturnThis(),
      json: jest.fn()
    };

    await getUser(req, res);

    expect(res.status).toHaveBeenCalledWith(200);
    expect(res.json).toHaveBeenCalledWith(
      expect.objectContaining({ success: true })
    );
  });
});

// Integration test
describe('User API', () => {
  it('should create new user', async () => {
    const res = await request(app)
      .post('/api/users')
      .send({
        name: 'Test User',
        email: 'test@example.com',
        password: 'password123'
      });

    expect(res.statusCode).toBe(201);
    expect(res.body).toHaveProperty('success', true);
  });
});
```

## Deployment Best Practices

### 1. Process Management

```javascript
// Use PM2 for production
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'api-server',
    script: './src/server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss'
  }]
};
```

### 2. Health Checks

```javascript
app.get('/health', async (req, res) => {
  try {
    // Check database
    await mongoose.connection.db.admin().ping();

    res.status(200).json({
      status: 'ok',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      environment: process.env.NODE_ENV
    });
  } catch (error) {
    res.status(503).json({
      status: 'error',
      message: error.message
    });
  }
});
```

### 3. Graceful Shutdown

```javascript
const gracefulShutdown = async () => {
  console.log('Received shutdown signal...');

  try {
    await mongoose.connection.close();
    console.log('Database connection closed.');

    server.close(() => {
      console.log('Server closed.');
      process.exit(0);
    });
  } catch (error) {
    console.error('Error during shutdown:', error);
    process.exit(1);
  }
};

process.on('SIGTERM', gracefulShutdown);
process.on('SIGINT', gracefulShutdown);
```

## Key Takeaways

1. ✅ **Use middleware** in the correct order
2. ✅ **Implement comprehensive error handling**
3. ✅ **Apply security measures** (helmet, rate limiting, validation)
4. ✅ **Use environment variables** for configuration
5. ✅ **Optimize performance** (caching, indexing, pagination)
6. ✅ **Follow RESTful principles**
7. ✅ **Use consistent response formats**
8. ✅ **Implement proper logging**
9. ✅ **Write tests** for all endpoints
10. ✅ **Use process managers** (PM2) in production
