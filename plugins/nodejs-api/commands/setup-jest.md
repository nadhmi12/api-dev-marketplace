---
description: "Setup Jest testing framework with configuration and sample tests"
argument-hint: ""
---

# Setup Jest Testing

This command configures Jest testing framework for your Express.js API with sample tests.

## Usage

```
/nodejs-api:setup-jest
```

## What It Adds

### 1. Update package.json
```json
{
  "devDependencies": {
    "jest": "^29.7.0",
    "supertest": "^6.3.3",
    "nodemon": "^3.0.2"
  },
  "scripts": {
    "test": "NODE_ENV=test jest --coverage",
    "test:watch": "NODE_ENV=test jest --watch",
    "test:verbose": "NODE_ENV=test jest --verbose",
    "test:unit": "NODE_ENV=test jest tests/unit",
    "test:integration": "NODE_ENV=test jest tests/integration",
    "test:coverage": "NODE_ENV=test jest --coverage --coverageReporters=text"
  }
}
```

### 2. Create Jest Configuration (jest.config.js)
```javascript
module.exports = {
  testEnvironment: 'node',
  coverageDirectory: 'coverage',
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/config/**',
    '!src/middleware/**/*.js',
    '!**/node_modules/**'
  ],
  testMatch: [
    '**/tests/**/*.test.js'
  ],
  setupFilesAfterEnv: ['./tests/setup.js'],
  testTimeout: 10000,
  verbose: true,
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1'
  }
};
```

### 3. Create Test Setup File (tests/setup.js)
```javascript
const mongoose = require('mongoose');

// Mock console methods to reduce test noise
global.console = {
  ...console,
  log: jest.fn(),
  debug: jest.fn(),
  info: jest.fn(),
  warn: jest.fn(),
  error: jest.fn(),
};

// Setup database connection for tests
beforeAll(async () => {
  const MONGODB_URI = process.env.TEST_MONGODB_URI || 'mongodb://localhost:27017/testdb';
  await mongoose.connect(MONGODB_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  });
});

// Cleanup database after tests
afterAll(async () => {
  await mongoose.connection.dropDatabase();
  await mongoose.connection.close();
});

// Clean database after each test
afterEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});
```

### 4. Create Test Directory Structure
```
tests/
├── setup.js
├── unit/
│   ├── controllers/
│   │   └── userController.test.js
│   ├── models/
│   │   └── user.test.js
│   └── utils/
│       └── asyncHandler.test.js
└── integration/
    └── api.test.js
```

### 5. Sample Unit Test (tests/unit/models/user.test.js)
```javascript
const User = require('../../../src/models/User');

describe('User Model', () => {
  describe('Validation', () => {
    it('should create a user with valid fields', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'password123'
      };

      const user = await User.create(userData);

      expect(user.name).toBe(userData.name);
      expect(user.email).toBe(userData.email);
      expect(user.password).not.toBe(userData.password); // Should be hashed
      expect(user.role).toBe('user');
    });

    it('should require name field', async () => {
      const user = new User({
        email: 'test@example.com',
        password: 'password123'
      });

      await expect(user.save()).rejects.toThrow();
    });

    it('should require email field', async () => {
      const user = new User({
        name: 'John Doe',
        password: 'password123'
      });

      await expect(user.save()).rejects.toThrow();
    });

    it('should validate email format', async () => {
      const user = new User({
        name: 'John Doe',
        email: 'invalid-email',
        password: 'password123'
      });

      await expect(user.save()).rejects.toThrow();
    });

    it('should enforce unique email', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'password123'
      };

      await User.create(userData);
      await expect(User.create(userData)).rejects.toThrow();
    });

    it('should require password with minimum 6 characters', async () => {
      const user = new User({
        name: 'John Doe',
        email: 'john@example.com',
        password: '123'
      });

      await expect(user.save()).rejects.toThrow();
    });
  });

  describe('Methods', () => {
    it('should match valid password', async () => {
      const user = await User.create({
        name: 'John Doe',
        email: 'john@example.com',
        password: 'password123'
      });

      const isMatch = await user.matchPassword('password123');
      expect(isMatch).toBe(true);
    });

    it('should not match invalid password', async () => {
      const user = await User.create({
        name: 'John Doe',
        email: 'john@example.com',
        password: 'password123'
      });

      const isMatch = await user.matchPassword('wrongpassword');
      expect(isMatch).toBe(false);
    });
  });
});
```

### 6. Sample Unit Test (tests/unit/controllers/userController.test.js)
```javascript
const User = require('../../../src/models/User');
const {
  getUsers,
  getUser,
  createUser,
  updateUser,
  deleteUser
} = require('../../../src/controllers/userController');

describe('User Controller', () => {
  let mockRequest, mockResponse, nextFunction;

  beforeEach(() => {
    mockRequest = {
      params: {},
      body: {},
      user: {}
    };
    mockResponse = {
      status: jest.fn().mockReturnThis(),
      json: jest.fn()
    };
    nextFunction = jest.fn();
  });

  describe('getUsers', () => {
    it('should get all users', async () => {
      const users = [
        await User.create({ name: 'User 1', email: 'user1@test.com', password: 'pass123' }),
        await User.create({ name: 'User 2', email: 'user2@test.com', password: 'pass123' })
      ];

      mockRequest.query = {};

      await getUsers(mockRequest, mockResponse, nextFunction);

      expect(mockResponse.status).not.toHaveBeenCalled();
      expect(mockResponse.json).toHaveBeenCalledWith(
        expect.objectContaining({
          success: true,
          count: users.length
        })
      );
    });

    it('should handle pagination', async () => {
      await User.create({ name: 'User 1', email: 'user1@test.com', password: 'pass123' });
      await User.create({ name: 'User 2', email: 'user2@test.com', password: 'pass123' });
      await User.create({ name: 'User 3', email: 'user3@test.com', password: 'pass123' });

      mockRequest.query = { page: 1, limit: 2 };

      await getUsers(mockRequest, mockResponse, nextFunction);

      expect(mockResponse.json).toHaveBeenCalledWith(
        expect.objectContaining({
          success: true,
          count: 2
        })
      );
    });
  });

  describe('getUser', () => {
    it('should get user by ID', async () => {
      const user = await User.create({
        name: 'Test User',
        email: 'test@test.com',
        password: 'pass123'
      });

      mockRequest.params.id = user._id;

      await getUser(mockRequest, mockResponse, nextFunction);

      expect(mockResponse.json).toHaveBeenCalledWith(
        expect.objectContaining({
          success: true,
          data: expect.objectContaining({
            name: 'Test User'
          })
        })
      );
    });

    it('should return 404 for non-existent user', async () => {
      mockRequest.params.id = '507f1f77bcf86cd799439011';

      await getUser(mockRequest, mockResponse, nextFunction);

      expect(mockResponse.status).toHaveBeenCalledWith(404);
      expect(mockResponse.json).toHaveBeenCalledWith(
        expect.objectContaining({
          success: false,
          error: 'User not found'
        })
      );
    });
  });

  describe('createUser', () => {
    it('should create new user', async () => {
      mockRequest.body = {
        name: 'New User',
        email: 'new@test.com',
        password: 'pass123'
      };

      await createUser(mockRequest, mockResponse, nextFunction);

      expect(mockResponse.status).toHaveBeenCalledWith(201);
      expect(mockResponse.json).toHaveBeenCalledWith(
        expect.objectContaining({
          success: true,
          data: expect.objectContaining({
            name: 'New User',
            email: 'new@test.com'
          })
        })
      );
    });
  });

  describe('updateUser', () => {
    it('should update user', async () => {
      const user = await User.create({
        name: 'Original Name',
        email: 'original@test.com',
        password: 'pass123'
      });

      mockRequest.params.id = user._id;
      mockRequest.body = { name: 'Updated Name' };

      await updateUser(mockRequest, mockResponse, nextFunction);

      expect(mockResponse.json).toHaveBeenCalledWith(
        expect.objectContaining({
          success: true,
          data: expect.objectContaining({
            name: 'Updated Name'
          })
        })
      );
    });

    it('should return 404 for non-existent user', async () => {
      mockRequest.params.id = '507f1f77bcf86cd799439011';
      mockRequest.body = { name: 'Updated Name' };

      await updateUser(mockRequest, mockResponse, nextFunction);

      expect(mockResponse.status).toHaveBeenCalledWith(404);
    });
  });

  describe('deleteUser', () => {
    it('should delete user', async () => {
      const user = await User.create({
        name: 'Test User',
        email: 'test@test.com',
        password: 'pass123'
      });

      mockRequest.params.id = user._id;

      await deleteUser(mockRequest, mockResponse, nextFunction);

      expect(mockResponse.json).toHaveBeenCalledWith(
        expect.objectContaining({
          success: true
        })
      );

      const deletedUser = await User.findById(user._id);
      expect(deletedUser).toBeNull();
    });
  });
});
```

### 7. Sample Integration Test (tests/integration/api.test.js)
```javascript
const request = require('supertest');
const app = require('../../src/app');

describe('API Integration Tests', () => {
  describe('Health Check', () => {
    it('should return 200 for health check', async () => {
      const res = await request(app).get('/health');
      expect(res.statusCode).toBe(200);
      expect(res.body).toHaveProperty('status', 'ok');
    });
  });

  describe('Authentication', () => {
    let authToken;

    it('should register a new user', async () => {
      const res = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'Test User',
          email: 'test@example.com',
          password: 'password123'
        });

      expect(res.statusCode).toBe(201);
      expect(res.body).toHaveProperty('token');
      authToken = res.body.token;
    });

    it('should login with valid credentials', async () => {
      const res = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'password123'
        });

      expect(res.statusCode).toBe(200);
      expect(res.body).toHaveProperty('token');
    });

    it('should not login with invalid credentials', async () => {
      const res = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'wrongpassword'
        });

      expect(res.statusCode).toBe(401);
    });
  });

  describe('User Endpoints', () => {
    let userId;

    beforeAll(async () => {
      const res = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'Integration Test User',
          email: 'integration@test.com',
          password: 'password123'
        });
      authToken = res.body.token;
    });

    it('should get all users', async () => {
      const res = await request(app)
        .get('/api/users')
        .set('Authorization', `Bearer ${authToken}`);

      expect(res.statusCode).toBe(200);
      expect(res.body).toHaveProperty('success', true);
      expect(res.body).toHaveProperty('data');
      expect(Array.isArray(res.body.data)).toBe(true);
    });

    it('should create a new user', async () => {
      const res = await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          name: 'New User',
          email: 'newuser@test.com',
          password: 'password123'
        });

      expect(res.statusCode).toBe(201);
      expect(res.body).toHaveProperty('success', true);
      expect(res.body.data).toHaveProperty('name', 'New User');
      userId = res.body.data._id;
    });

    it('should get a single user', async () => {
      const res = await request(app)
        .get(`/api/users/${userId}`)
        .set('Authorization', `Bearer ${authToken}`);

      expect(res.statusCode).toBe(200);
      expect(res.body.data).toHaveProperty('_id', userId);
    });

    it('should update a user', async () => {
      const res = await request(app)
        .put(`/api/users/${userId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          name: 'Updated Name'
        });

      expect(res.statusCode).toBe(200);
      expect(res.body.data).toHaveProperty('name', 'Updated Name');
    });

    it('should delete a user', async () => {
      const res = await request(app)
        .delete(`/api/users/${userId}`)
        .set('Authorization', `Bearer ${authToken}`);

      expect(res.statusCode).toBe(200);
    });
  });
});
```

### 8. Update .gitignore
```
# Add to existing .gitignore
coverage/
*.lcov
```

## Features

✅ **Jest** as the test runner
✅ **Supertest** for API testing
✅ **Unit tests** for models and controllers
✅ **Integration tests** for API endpoints
✅ **Code coverage** reports
✅ **Database cleanup** between tests
✅ **Mock environment** for tests
✅ **Watch mode** for development
✅ **Separated test suites**

## Usage

### Run All Tests
```bash
npm test
```

### Run Tests in Watch Mode
```bash
npm run test:watch
```

### Run Only Unit Tests
```bash
npm run test:unit
```

### Run Only Integration Tests
```bash
npm run test:integration
```

### Run Tests with Coverage
```bash
npm run test:coverage
```

### Run Specific Test File
```bash
npx jest tests/unit/models/user.test.js
```

### Run Tests Matching Pattern
```bash
npxjest tests/unit --testNamePattern="User Model"
```

## Writing Tests

### Test Structure
```javascript
describe('Feature Name', () => {
  describe('Specific Behavior', () => {
    it('should do something', () => {
      // Arrange
      const input = 'value';

      // Act
      const result = functionUnderTest(input);

      // Assert
      expect(result).toBe('expected');
    });
  });
});
```

### Common Matchers
```javascript
// Equality
expect(value).toBe(expected);
expect(value).toEqual(expected);

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();

// Numbers
expect(value).toBeGreaterThan(10);
expect(value).toBeLessThanOrEqual(100);

// Strings
expect(value).toMatch(/regex/);
expect(value).toContain('substring');

// Arrays/Objects
expect(array).toContain(item);
expect(object).toHaveProperty('key', value);

// Async
await expect(promise).resolves.toBe(value);
await expect(promise).rejects.toThrow();

// Function calls
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith(arg1, arg2);
```

### Mocking
```javascript
// Mock functions
const mockFn = jest.fn();
mockFn('arg');
expect(mockFn).toHaveBeenCalledWith('arg');

// Mock modules
jest.mock('../src/models/User');
const User = require('../src/models/User');
User.find.mockResolvedValue([]);

// Mock return values
User.create.mockReturnValue({
  name: 'Test',
  save: jest.fn()
});
```

## Best Practices

1. **Arrange-Act-Assert** pattern for clarity
2. **Descriptive test names** that explain what they test
3. **One assertion per test** when possible
4. **Test edge cases** and error conditions
5. **Use beforeEach** for common setup
6. **Avoid sharing state** between tests
7. **Mock external dependencies**
8. **Keep tests fast** and isolated
9. **Target 80%+ code coverage**

## CI/CD Integration

### GitHub Actions (.github/workflows/test.yml)
```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      mongodb:
        image: mongo:7.0
        ports:
          - 27017:27017

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
        env:
          TEST_MONGODB_URI: mongodb://localhost:27017/testdb
```

## Next Steps

1. Add tests for all your endpoints
2. Mock external services (email, SMS, etc.)
3. Add performance tests
4. Set up CI/CD to run tests automatically
5. Generate coverage reports in HTML format
6. Add API contract testing (Pact)

## Troubleshooting

**Tests timing out**
- Increase timeout in jest.config.js
- Check for hanging promises
- Ensure database cleanup is working

**Database connection issues**
- Verify TEST_MONGODB_URI is set
- Check MongoDB service is running
- Ensure cleanup happens between tests

**Coverage not generating**
- Check collectCoverageFrom paths
- Ensure tests