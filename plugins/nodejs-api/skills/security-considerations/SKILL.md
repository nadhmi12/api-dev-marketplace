---
description: "Security best practices including authentication, authorization, data validation, rate limiting, and protection against common vulnerabilities"
---

# Security Considerations

This skill provides comprehensive security guidance for Express.js applications to protect against common vulnerabilities.

## Authentication & Authorization

### 1. Password Security

```javascript
const bcrypt = require('bcrypt');
const crypto = require('crypto');

// Hash password
const hashPassword = async (password) => {
  const salt = await bcrypt.genSalt(12);
  return await bcrypt.hash(password, salt);
};

// Compare password
const comparePassword = async (password, hashedPassword) => {
  return await bcrypt.compare(password, hashedPassword);
};

// Usage in User model
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) {
    return next();
  }
  this.password = await hashPassword(this.password);
  next();
});

// Method in User model
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await comparePassword(candidatePassword, this.password);
};
```

### 2. JWT Implementation

```javascript
const jwt = require('jsonwebtoken');

const generateToken = (payload, expiresIn = '7d') => {
  return jwt.sign(payload, process.env.JWT_SECRET, {
    expiresIn,
    issuer: 'api.example.com',
    audience: 'api.example.com'
  });
};

const verifyToken = (token) => {
  return jwt.verify(token, process.env.JWT_SECRET, {
    issuer: 'api.example.com',
    audience: 'api.example.com'
  });
};

// Protect route middleware
const protect = async (req, res, next) => {
  let token;

  if (req.headers.authorization?.startsWith('Bearer')) {
    token = req.headers.authorization.split(' ')[1];
  }

  if (!token) {
    return res.status(401).json({
      success: false,
      error: 'Not authorized to access this route'
    });
  }

  try {
    const decoded = verifyToken(token);
    const user = await User.findById(decoded.id);

    if (!user) {
      return res.status(401).json({
        success: false,
        error: 'User no longer exists'
      });
    }

    // Check if password changed after token was issued
    if (user.passwordChangedAt && decoded.iat < Math.floor(user.passwordChangedAt / 1000)) {
      return res.status(401).json({
        success: false,
        error: 'Password has been changed. Please login again.'
      });
    }

    req.user = user;
    next();
  } catch (error) {
    return res.status(401).json({
      success: false,
      error: 'Not authorized to access this route'
    });
  }
};

// Authorization middleware
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        success: false,
        error: 'Not authorized to perform this action'
      });
    }
    next();
  };
};
```

### 3. Password Reset

```javascript
const crypto = require('crypto');

// Generate reset token
const generateResetToken = () => {
  const resetToken = crypto.randomBytes(32).toString('hex');
  const hashedToken = crypto
    .createHash('sha256')
    .update(resetToken)
    .digest('hex');

  return {
    resetToken,
    hashedToken,
    expireDate: Date.now() + 10 * 60 * 1000 // 10 minutes
  };
};

// Save reset token to user
userSchema.methods.createPasswordResetToken = function() {
  const { resetToken, hashedToken, expireDate } = generateResetToken();

  this.passwordResetToken = hashedToken;
  this.passwordResetExpires = expireDate;

  return resetToken;
};

// Reset password route
exports.resetPassword = async (req, res, next) => {
  const hashedToken = crypto
    .createHash('sha256')
    .update(req.params.token)
    .digest('hex');

  const user = await User.findOne({
    passwordResetToken: hashedToken,
    passwordResetExpires: { $gt: Date.now() }
  });

  if (!user) {
    return next(new AppError('Token is invalid or has expired', 400));
  }

  user.password = req.body.password;
  user.passwordResetToken = undefined;
  user.passwordResetExpires = undefined;
  await user.save();

  const token = generateToken(user.id);

  res.json({
    success: true,
    token
  });
};
```

## Input Validation

### 1. Express-Validator

```javascript
const { body, param, query, validationResult } = require('express-validator');

// Sanitization and validation
const validateInput = (req, res, next) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      error: 'Validation failed',
      details: errors.array()
    });
  }

  next();
};

// User registration validation
exports.validateRegister = [
  body('name')
    .trim()
    .notEmpty()
    .withMessage('Name is required')
    .isLength({ min: 2, max: 50 })
    .withMessage('Name must be between 2 and 50 characters')
    .escape(),

  body('email')
    .trim()
    .notEmpty()
    .withMessage('Email is required')
    .isEmail()
    .withMessage('Invalid email format')
    .normalizeEmail()
    .toLowerCase(),

  body('password')
    .isLength({ min: 8, max: 128 })
    .withMessage('Password must be at least 8 characters')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
    .withMessage('Password must contain uppercase, lowercase, number, and special character'),

  body('confirmPassword')
    .custom((value, { req }) => {
      if (value !== req.body.password) {
        throw new Error('Passwords do not match');
      }
      return true;
    }),

  validateInput
];
```

### 2. MongoDB Injection Prevention

```javascript
// Sanitize user input
const mongoSanitize = require('express-mongo-sanitize');

app.use(mongoSanitize({
  onSanitize: ({ req, key, value }) => {
    console.warn(`Mongo injection attempt detected: ${key} = ${value}`);
  }
}));

// Validate MongoDB ObjectIds
const isValidObjectId = (id) => {
  return mongoose.Types.ObjectId.isValid(id);
};

// Usage
exports.getUser = async (req, res, next) => {
  if (!isValidObjectId(req.params.id)) {
    return next(new AppError('Invalid ID', 400));
  }

  const user = await User.findById(req.params.id);
  // ...
};
```

### 3. SQL Injection Prevention (PostgreSQL)

```javascript
// Use parameterized queries with Sequelize
const { Op } = require('sequelize');

// Safe query
const users = await User.findAll({
  where: {
    email: req.body.email, // Parameterized automatically
    name: {
      [Op.like]: `%${req.query.search}%` // Still safe with Sequelize
    }
  }
});

// Never use string interpolation
// ❌ Bad
// const query = `SELECT * FROM users WHERE email = '${req.body.email}'`;

// ✅ Good - Parameterized
// const query = 'SELECT * FROM users WHERE email = ?';
// await sequelize.query(query, { replacements: [req.body.email] });
```

## XSS Protection

```javascript
const xss = require('xss-clean');
const helmet = require('helmet');

// Sanitize HTML/JS in request body
app.use(xss());

// Set security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"]
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));

// Additional XSS protection middleware
const xssProtection = (req, res, next) => {
  const sanitize = (obj) => {
    if (typeof obj === 'string') {
      return obj.replace(/</g, '&lt;').replace(/>/g, '&gt;');
    }
    if (typeof obj === 'object' && obj !== null) {
      for (const key in obj) {
        obj[key] = sanitize(obj[key]);
      }
    }
    return obj;
  };

  if (req.body) {
    req.body = sanitize(req.body);
  }
  if (req.query) {
    req.query = sanitize(req.query);
  }

  next();
};

app.use(xssProtection);
```

## Rate Limiting

### 1. General Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

// API rate limiter
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per 15 minutes
  message: {
    success: false,
    error: 'Too many requests from this IP, please try again later.'
  },
  standardHeaders: true,
  legacyHeaders: false,
  handler: (req, res) => {
    res.status(429).json({
      success: false,
      error: 'Too many requests from this IP, please try again later.'
    });
  }
});

app.use('/api', apiLimiter);
```

### 2. Strict Rate Limiting for Sensitive Endpoints

```javascript
// Login rate limiter
const loginLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // 5 login attempts per hour
  message: {
    success: false,
    error: 'Too many login attempts, please try again in an hour.'
  },
  skipSuccessfulRequests: true, // Don't count successful logins
  keyGenerator: (req) => {
    // Use IP + email for more specific limiting
    return req.ip + req.body.email;
  }
});

app.post('/api/auth/login', loginLimiter, login);

// Password reset rate limiter
const resetLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 3, // 3 reset attempts per hour
  message: {
    success: false,
    error: 'Too many password reset attempts, please try again later.'
  }
});

app.post('/api/auth/forgot-password', resetLimiter, forgotPassword);
```

### 3. DDoS Protection

```javascript
// Express-slow-down to slow down excessive requests
const slowDown = require('express-slow-down');

const speedLimiter = slowDown({
  windowMs: 15 * 60 * 1000,
  delayAfter: 50, // Allow 50 requests per 15 minutes at full speed
  delayMs: 500, // Add 500ms delay after 50 requests
  maxDelayMs: 20000, // Maximum delay of 20 seconds
  skipSuccessfulRequests: true
});

app.use('/api', speedLimiter);
```

## CORS Configuration

```javascript
const cors = require('cors');

// Allowed origins
const allowedOrigins = [
  'https://example.com',
  'https://www.example.com',
  'http://localhost:3000'
];

// CORS configuration
const corsOptions = {
  origin: (origin, callback) => {
    // Allow requests with no origin (like mobile apps, curl, etc.)
    if (!origin) return callback(null, true);

    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true, // Allow cookies
  optionsSuccessStatus: 200, // Some legacy browsers (IE11) choke on 204
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Total-Count']
};

if (process.env.NODE_ENV === 'development') {
  // Allow all origins in development
  app.use(cors());
} else {
  // Strict CORS in production
  app.use(cors(corsOptions));
}

// Handle CORS errors
app.use((err, req, res, next) => {
  if (err.message === 'Not allowed by CORS') {
    return res.status(403).json({
      success: false,
      error: 'CORS policy violation'
    });
  }
  next(err);
});
```

## HTTPS & SSL/TLS

```javascript
// Force HTTPS in production
if (process.env.NODE_ENV === 'production') {
  app.use((req, res, next) => {
    if (req.header('x-forwarded-proto') !== 'https') {
      return res.redirect(`https://${req.header('host')}${req.url}`);
    }
    next();
  });

  // HTTPS redirect middleware
  const forceSSL = require('force-ssl-heroku');
  app.use(forceSSL);
}
```

## File Upload Security

```javascript
const multer = require('multer');
const path = require('path');

// File filter
const fileFilter = (req, file, cb) => {
  const allowedTypes = /jpeg|jpg|png|gif|pdf|doc|docx/;

  const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
  const mimetype = allowedTypes.test(file.mimetype);

  if (extname && mimetype) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type. Only images and PDFs are allowed.'));
  }
};

// Multer configuration
const upload = multer({
  dest: 'uploads/',
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB limit
    files: 5 // Maximum 5 files
  },
  fileFilter: fileFilter,
  storage: multer.diskStorage({
    destination: (req, file, cb) => {
      cb(null, 'uploads/');
    },
    filename: (req, file, cb) => {
      const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1e9);
      cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
    }
  })
});

// Virus scanning (using clamav)
const NodeClam = require('clamav.js');
const clamscan = new NodeClam().init({
  clamdscan: {
    host: '127.0.0.1',
    port: 3310
  }
});

exports.uploadFile = async (req, res, next) => {
  try {
    upload.single('file')(req, res, async (err) => {
      if (err) {
        return next(new AppError(err.message, 400));
      }

      // Scan for viruses
      const scanResult = await clamscan.scanFile(req.file.path);
      if (scanResult.isInfected) {
        fs.unlinkSync(req.file.path);
        return next(new AppError('File is infected', 400));
      }

      res.json({
        success: true,
        data: {
          file: req.file
        }
      });
    });
  } catch (error) {
    next(error);
  }
};
```

## Session Security

```javascript
const session = require('express-session');
const MongoStore = require('connect-mongo');

// Session configuration
app.use(session({
  name: 'sessionId',
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production', // HTTPS only in production
    httpOnly: true, // Prevent XSS
    maxAge: 24 * 60 * 60 * 1000, // 24 hours
    sameSite: 'strict' // CSRF protection
  },
  store: MongoStore.create({
    mongoUrl: process.env.MONGODB_URI,
    ttl: 24 * 60 * 60, // 24 hours
    crypto: {
      secret: process.env.SESSION_SECRET
    }
  })
}));
```

## Security Headers

```javascript
app.use(helmet());

// Custom security headers
app.use((req, res, next) => {
  // X-Content-Type-Options
  res.setHeader('X-Content-Type-Options', 'nosniff');

  // X-Frame-Options
  res.setHeader('X-Frame-Options', 'DENY');

  // X-XSS-Protection
  res.setHeader('X-XSS-Protection', '1; mode=block');

  // Referrer-Policy
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');

  // Permissions-Policy
  res.setHeader('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');

  // Content-Type
  res.setHeader('Content-Type', 'application/json; charset=utf-8');

  next();
});
```

## Environment Variables

```javascript
// Validate required environment variables
const requiredEnvVars = [
  'NODE_ENV',
  'PORT',
  'MONGODB_URI',
  'JWT_SECRET',
  'SESSION_SECRET'
];

const checkEnvVars = () => {
  requiredEnvVars.forEach((varName) => {
    if (!process.env[varName]) {
      throw new Error(`Missing required environment variable: ${varName}`);
    }
  });

  // Validate JWT secret length
  if (process.env.JWT_SECRET.length < 32) {
    throw new Error('JWT_SECRET must be at least 32 characters');
  }
};

checkEnvVars();
```

## Security Best Practices Checklist

1. ✅ **Use HTTPS** in production
2. ✅ **Implement rate limiting** for all endpoints
3. ✅ **Validate and sanitize** all user inputs
4. ✅ **Use parameterized queries** to prevent SQL injection
5. ✅ **Hash passwords** with bcrypt (10+ rounds)
6. ✅ **Use JWT** for stateless authentication
7. ✅ **Set security headers** with Helmet
8. ✅ **Configure CORS** properly
9. ✅ **