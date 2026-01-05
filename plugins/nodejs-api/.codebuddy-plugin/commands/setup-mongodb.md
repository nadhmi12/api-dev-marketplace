---
description: "Add MongoDB integration with Mongoose to your Express API"
argument-hint: ""
---

# Setup MongoDB

This command adds MongoDB integration with Mongoose to your Express.js API project.

## Usage

```
/nodejs-api:setup-mongodb
```

## What It Adds

### 1. Update package.json
```json
{
  "dependencies": {
    "mongoose": "^8.0.0"
  }
}
```

### 2. Create Database Configuration (src/config/database.js)
```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });

    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`Error: ${error.message}`);
    process.exit(1);
  }
};

module.exports = connectDB;
```

### 3. Update app.js
```javascript
const connectDB = require('./config/database');

// Connect to MongoDB
connectDB();
```

### 4. Update .env.example
```
MONGODB_URI=mongodb://localhost:27017/mydb
```

### 5. Create Sample Model (src/models/User.js)
```javascript
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Please add a name']
  },
  email: {
    type: String,
    required: [true, 'Please add an email'],
    unique: true,
    lowercase: true,
    trim: true
  },
  password: {
    type: String,
    required: [true, 'Please add a password'],
    minlength: 6,
    select: false
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('User', userSchema);
```

### 6. Update docker-compose.yml
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - MONGODB_URI=mongodb://mongodb:27017/mydb
    depends_on:
      - mongodb

  mongodb:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db

volumes:
  mongodb_data:
```

## Features

✅ **Mongoose ODM** for MongoDB schema modeling
✅ **Connection management** with error handling
✅ **Environment-based configuration**
✅ **Docker integration** with MongoDB container
✅ **Sample User model** with validation
✅ **Automatic connection on app startup**

## Setup Instructions

### 1. Install Dependencies
```bash
npm install mongoose
```

### 2. Configure Environment
```bash
# Copy .env.example to .env
cp .env.example .env

# Edit .env and add your MongoDB URI
MONGODB_URI=mongodb://localhost:27017/mydb
```

### 3. Start MongoDB

**Option A: Local Installation**
```bash
# Install MongoDB
sudo apt-get install mongodb

# Start MongoDB
sudo systemctl start mongodb
```

**Option B: Docker**
```bash
docker-compose up mongodb
```

**Option C: MongoDB Atlas (Cloud)**
```bash
# Get your connection string from MongoDB Atlas
MONGODB_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/mydb
```

## Usage Examples

### Creating a Model
```javascript
// src/models/Product.js
const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
  name: { type: String, required: true },
  price: { type: Number, required: true },
  description: String
});

module.exports = mongoose.model('Product', productSchema);
```

### Using in Controller
```javascript
const Product = require('../models/Product');

exports.getProducts = async (req, res) => {
  const products = await Product.find();
  res.json({ success: true, data: products });
};

exports.createProduct = async (req, res) => {
  const product = await Product.create(req.body);
  res.status(201).json({ success: true, data: product });
};
```

## Advanced Features

### Schema Validation
```javascript
const schema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    unique: true,
    match: [/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/, 'Please add a valid email']
  }
});
```

### Middleware
```javascript
// Pre-save middleware
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) {
    next();
  }
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
});

// Virtual fields
userSchema.virtual('fullname').get(function() {
  return `${this.firstName} ${this.lastName}`;
});
```

### Indexes
```javascript
userSchema.index({ email: 1 });
userSchema.index({ createdAt: -1 });
```

## Testing

```javascript
// tests/unit/model.test.js
const mongoose = require('mongoose');
const User = require('../../src/models/User');

describe('User Model', () => {
  it('should create a user', async () => {
    const user = await User.create({
      name: 'Test User',
      email: 'test@example.com',
      password: 'password123'
    });
    expect(user.name).toBe('Test User');
  });
});
```

## Next Steps

1. Run `/nodejs-api:add-endpoint <resource>` to create models with MongoDB
2. Add indexes for frequently queried fields
3. Implement relationships between models (referencing or embedding)
4. Add validation and sanitization
5. Set up database seeding for development

## Troubleshooting

**Connection Error**
```
Error: connect ECONNREFUSED 127.0.0.1:27017
```
- Make sure MongoDB is running
- Check your MONGODB_URI in .env

**Authentication Error**
```
Error: Authentication failed
```
- Verify username and password in connection string
- Check if IP whitelist includes your address (Atlas)
