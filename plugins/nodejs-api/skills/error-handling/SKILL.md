---
description: "Comprehensive error handling strategies including custom error classes, async error handling, validation errors, and error logging"
---

# Error Handling Strategies

This skill provides comprehensive guidance on implementing robust error handling in Express.js applications.

## Error Classes

### 1. Custom Error Class

```javascript
// utils/AppError.js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
    this.isOperational = true;

    Error.captureStackTrace(this, this.constructor);
  }
}

module.exports = AppError;
```

### 2. Specific Error Classes

```javascript
// utils/errors.js
const AppError = require('./AppError');

class ValidationError extends AppError {
  constructor(message) {
    super(message, 400);
    this.name = 'ValidationError';
  }
}

class AuthenticationError extends AppError {
  constructor(message = 'Authentication failed') {
    super(message, 401);
    this.name = 'AuthenticationError';
  }
}

class AuthorizationError extends AppError {
  constructor(message = 'Not authorized') {
    super(message, 403);
    this.name = 'AuthorizationError';
  }
}

class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(message, 404);
    this.name = 'NotFoundError';
  }
}

class ConflictError extends AppError {
  constructor(message = 'Resource already exists') {
    super(message, 409);
    this.name = 'ConflictError';
  }
}

class DatabaseError extends AppError {
  constructor(message = 'Database operation failed') {
    super(message, 500);
    this.name = 'DatabaseError';
  }
}

module.exports = {
  ValidationError,
  AuthenticationError,
  AuthorizationError,
  NotFoundError,
  ConflictError,
  DatabaseError
};
```

## Async Error Handling

### 1. Async Handler Wrapper

```javascript
// utils/asyncHandler.js
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

module.exports = asyncHandler;
```

### 2. Usage in Controllers

```javascript
const asyncHandler = require('../utils/asyncHandler');
const { NotFoundError } = require('../utils/errors');
const User = require('../models/User');

exports.getUser = asyncHandler(async (req, res, next) => {
  const user = await User.findById(req.params.id);

  if (!user) {
    return next(new NotFoundError('User not found'));
  }

  res.json({ success: true, data: user });
});

exports.createUser = asyncHandler(async (req, res, next) => {
  try {
    const user = await User.create(req.body);
    res.status(201).json({ success: true, data: user });
  } catch (error) {
    if (error.code === 11000) {
      return next(new ConflictError('Email already exists'));
    }
    next(error);
  }
});
```

## Validation Errors

### 1. Express-Validator Integration

```javascript
// validators/userValidator.js
const { body, param, query, validationResult } = require('express-validator');

const validate = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    const errorMessages = errors.array().map(err => ({
      field: err.path,
      message: err.msg,
      value: err.value
    }));

    return res.status(400).json({
      success: false,
      error: 'Validation failed',
      details: errorMessages
    });
  }
  next();
};

exports.validateUser = [
  body('name')
    .trim()
    .notEmpty()
    .withMessage('Name is required')
    .isLength({ min: 2, max: 50 })
    .withMessage('Name must be between 2 and 50 characters'),

  body('email')
    .trim()
    .notEmpty()
    .withMessage('Email is required')
    .isEmail()
    .withMessage('Invalid email format')
    .normalizeEmail(),

  body('password')
    .notEmpty()
    .withMessage('Password is required')
    .isLength({ min: 6, max: 100 })
    .withMessage('Password must be between 6 and 100 characters')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .withMessage('Password must contain uppercase, lowercase, and number'),

  body('role')
    .optional()
    .isIn(['user', 'admin'])
    .withMessage('Invalid role'),

  validate
];

exports.validateUserId = [
  param('id')
    .isMongoId()
    .withMessage('Invalid user ID'),

  validate
];
```

### 2. Custom Validation

```javascript
// validators/validators.js
const { body, validationResult } = require('express-validator');

const validateRequest = (req, res, next) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    const formattedErrors = errors.array().reduce((acc, error) => {
      acc[error.path] = error.msg;
      return acc;
    }, {});

    return res.status(400).json({
      success: false,
      error: 'Validation Error',
      fields: formattedErrors
    });
  }

  next();
};

// Custom validator for unique email
const isUniqueEmail = async (value) => {
  const user = await User.findOne({ email: value });
  if (user) {
    throw new Error('Email already exists');
  }
  return true;
};

// Usage
exports.validateRegister = [
  body('email')
    .custom(isUniqueEmail)
    .withMessage('Email already registered'),
  validateRequest
];
```

## Global Error Handler

```javascript
// middleware/errorHandler.js
const AppError = require('../utils/AppError');

const errorHandler = (err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;

  // Log error
  console.error(err);

  // Mongoose bad ObjectId
  if (err.name === 'CastError') {
    const message = 'Resource not found';
    error = new AppError(message, 404);
  }

  // Mongoose duplicate key
  if (err.code === 11000) {
    const field = Object.keys(err.keyValue)[0];
    const message = `${field} already exists`;
    error = new AppError(message, 400);
  }

  // Mongoose validation error
  if (err.name === 'ValidationError') {
    const message = Object.values(err.errors).map(val => val.message);
    error = new AppError(message.join(', '), 400);
  }

  // JWT errors
  if (err.name === 'JsonWebTokenError') {
    const message = 'Invalid token';
    error = new AppError(message, 401);
  }

  if (err.name === 'TokenExpiredError') {
    const message = 'Token expired';
    error = new AppError(message, 401);
  }

  // Operational, trusted errors: send message to client
  if (process.env.NODE_ENV === 'development') {
    return res.status(error.statusCode || 500).json({
      success: false,
      error: error.message || 'Server Error',
      stack: error.stack
    });
  }

  // Programming or other unknown errors: don't leak details
  res.status(error.statusCode || 500).json({
    success: false,
    error: 'An error occurred. Please try again later.'
  });
};

module.exports = errorHandler;
```

## 404 Handler

```javascript
// middleware/notFound.js
const notFound = (req, res, next) => {
  const error = new AppError(`Not Found - ${req.originalUrl}`, 404);
  next(error);
};

module.exports = notFound;
```

## Error Logging

### 1. Winston Logger

```javascript
// utils/logger.js
const winston = require('winston');
const { format } = winston;

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: format.combine(
    format.timestamp(),
    format.errors({ stack: true }),
    format.json()
  ),
  defaultMeta: { service: 'api-server' },
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' })
  ]
});

if (process.env.NODE_ENV === 'development') {
  logger.add(new winston.transports.Console({
    format: format.combine(
      format.colorize(),
      format.simple()
    )
  }));
}

// Log errors
logger.error('Database connection failed', {
  error: err.message,
  stack: err.stack,
  timestamp: new Date().toISOString()
});

module.exports = logger;
```

### 2. Request Error Logger

```javascript
// middleware/errorLogger.js
const logger = require('../utils/logger');

const errorLogger = (err, req, res, next) => {
  logger.error('Request Error', {
    method: req.method,
    url: req.url,
    ip: req.ip,
    userAgent: req.get('user-agent'),
    statusCode: err.statusCode || 500,
    errorMessage: err.message,
    stack: err.stack,
    timestamp: new Date().toISOString()
  });

  next(err);
};

module.exports = errorLogger;
```

## Database Error Handling

### 1. MongoDB Errors

```javascript
// Handle MongoDB specific errors
const handleDatabaseError = (error) => {
  if (error.name === 'MongoServerError') {
    // Duplicate key error
    if (error.code === 11000) {
      const field = Object.keys(error.keyPattern)[0];
      throw new ConflictError(`${field} already exists`);
    }
  }

  if (error.name === 'MongooseError') {
    throw new DatabaseError('Database error occurred');
  }

  if (error.name === 'CastError') {
    throw new NotFoundError('Invalid ID format');
  }

  if (error.name === 'ValidationError') {
    throw new ValidationError(error.message);
  }

  throw new AppError('Database operation failed', 500);
};

// Usage in controller
exports.createProduct = asyncHandler(async (req, res, next) => {
  try {
    const product = await Product.create(req.body);
    res.status(201).json({ success: true, data: product });
  } catch (error) {
    handleDatabaseError(error);
    next(error);
  }
});
```

### 2. PostgreSQL Errors (Sequelize)

```javascript
// Handle PostgreSQL specific errors
const handlePostgresError = (error) => {
  // Unique constraint violation
  if (error.name === 'SequelizeUniqueConstraintError') {
    const field = error.errors[0].path;
    throw new ConflictError(`${field} already exists`);
  }

  // Foreign key constraint
  if (error.name === 'SequelizeForeignKeyConstraintError') {
    throw new ValidationError('Invalid reference');
  }

  // Validation error
  if (error.name === 'SequelizeValidationError') {
    const messages = error.errors.map(e => e.message);
    throw new ValidationError(messages.join(', '));
  }

  throw new DatabaseError('Database error occurred');
};
```

## API Error Responses

### 1. Standard Error Response Format

```javascript
// Error response structure
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ],
    "timestamp": "2024-01-15T10:30:00Z",
    "path": "/api/users"
  }
}
```

### 2. Error Handler with Response Format

```javascript
const errorHandler = (err, req, res, next) => {
  const response = {
    success: false,
    error: {
      code: err.name || 'INTERNAL_ERROR',
      message: err.message || 'An error occurred',
      timestamp: new Date().toISOString(),
      path: req.originalUrl
    }
  };

  // Add details in development
  if (process.env.NODE_ENV === 'development') {
    response.error.stack = err.stack;
    response.error.details = err.details || [];
  }

  res.status(err.statusCode || 500).json(response);
};
```

## Testing Error Handling

### 1. Unit Tests

```javascript
// tests/unit/errors.test.js
const { ValidationError, NotFoundError } = require('../../src/utils/errors');

describe('Error Classes', () => {
  describe('ValidationError', () => {
    it('should create validation error with correct status code', () => {
      const error = new ValidationError('Invalid input');
      expect(error.statusCode).toBe(400);
      expect(error.name).toBe('ValidationError');
      expect(error.message).toBe('Invalid input');
    });
  });

  describe('NotFoundError', () => {
    it('should create not found error with correct status code', () => {
      const error = new NotFoundError('User not found');
      expect(error.statusCode).toBe(404);
      expect(error.name).toBe('NotFoundError');
    });
  });
});
```

### 2. Integration Tests

```javascript
// tests/integration/errors.test.js
const request = require('supertest');
const app = require('../../src/app');

describe('Error Handling', () => {
  it('should return 404 for invalid route', async () => {
    const res = await request(app).get('/api/nonexistent');
    expect(res.statusCode).toBe(404);
    expect(res.body.success).toBe(false);
  });

  it('should return validation error for invalid data', async () => {
    const res = await request(app)
      .post('/api/users')
      .send({
        name: '',
        email: 'invalid-email'
      });
    expect(res.statusCode).toBe(400);
    expect(res.body.success).toBe(false);
    expect(res.body.error).toHaveProperty('details');
  });

  it('should return 500 for internal server error', async () => {
    const res = await request(app).get('/api/error');
    expect(res.statusCode).toBe(500);
    expect(res.body.success).toBe(false);
  });
});
```

## Best Practices

1. ✅ **Use custom error classes** for different error types
2. ✅ **Wrap async functions** with error handlers
3. ✅ **Validate all inputs** before processing
4. ✅ **Provide meaningful error messages**
5. ✅ **Log errors** for debugging and monitoring
6. ✅ **Don't expose sensitive information** in production
7. ✅ **Handle database errors** appropriately
8. ✅ **Return consistent error formats**
9. ✅ **Test error scenarios** thoroughly
10. ✅ **Use appropriate HTTP status codes**

## HTTP Status Codes Guide

```javascript
// 2xx Success
200 OK              - Request succeeded
201 Created         - Resource created
204 No Content      - Request succeeded, no content returned

// 4xx Client Errors
400 Bad Request     - Invalid request
401 Unauthorized    - Authentication required
403 Forbidden       - Access denied
404 Not Found       - Resource not found
409 Conflict        - Resource already exists
422 Unprocessable Entity - Validation error
429 Too Many Requests - Rate limit exceeded

// 5xx Server Errors
500 Internal Server Error - Unexpected error
502 Bad Gateway     - Invalid response from upstream
503 Service Unavailable - Service temporarily unavailable
504 Gateway Timeout - Upstream timeout
```
