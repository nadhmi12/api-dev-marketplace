---
description: "Add PostgreSQL integration with Sequelize to your Express API"
argument-hint: ""
---

# Setup PostgreSQL

This command adds PostgreSQL integration with Sequelize ORM to your Express.js API project.

## Usage

```
/nodejs-api:setup-postgresql
```

## What It Adds

### 1. Update package.json
```json
{
  "dependencies": {
    "pg": "^8.11.0",
    "sequelize": "^6.35.0",
    "sequelize-cli": "^6.6.0"
  }
}
```

### 2. Create Database Configuration (src/config/database.js)
```javascript
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize(
  process.env.POSTGRES_DB || 'mydb',
  process.env.POSTGRES_USER || 'postgres',
  process.env.POSTGRES_PASSWORD || 'password',
  {
    host: process.env.POSTGRES_HOST || 'localhost',
    port: process.env.POSTGRES_PORT || 5432,
    dialect: 'postgres',
    logging: process.env.NODE_ENV === 'development' ? console.log : false,
  }
);

const connectDB = async () => {
  try {
    await sequelize.authenticate();
    console.log('PostgreSQL connection has been established successfully.');
    await sequelize.sync(); // Sync models to database
  } catch (error) {
    console.error('Unable to connect to the database:', error);
    process.exit(1);
  }
};

module.exports = { sequelize, connectDB };
```

### 3. Create Models Directory Structure
```
src/models/
├── index.js          # Models initialization
├── .gitkeep
└── User.js           # Sample model
```

### 4. Create Sample Model (src/models/User.js)
```javascript
const { DataTypes } = require('sequelize');
const { sequelize } = require('../config/database');

const User = sequelize.define('User', {
  id: {
    type: DataTypes.UUID,
    defaultValue: DataTypes.UUIDV4,
    primaryKey: true
  },
  name: {
    type: DataTypes.STRING,
    allowNull: false,
    validate: {
      notEmpty: true
    }
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
    validate: {
      isEmail: true
    }
  },
  password: {
    type: DataTypes.STRING,
    allowNull: false,
    validate: {
      len: [6, 100]
    }
  },
  role: {
    type: DataTypes.ENUM('user', 'admin'),
    defaultValue: 'user'
  }
}, {
  tableName: 'users',
  timestamps: true,
  paranoid: true, // Soft deletes
});

module.exports = User;
```

### 5. Create Models Index (src/models/index.js)
```javascript
const fs = require('fs');
const path = require('path');
const { sequelize } = require('../config/database');

const db = {};

fs.readdirSync(__dirname)
  .filter(file => {
    return (file.indexOf('.') !== 0) && (file !== path.basename(__filename)) && (file.slice(-3) === '.js');
  })
  .forEach(file => {
    const model = require(path.join(__dirname, file));
    db[model.name] = model;
  });

Object.keys(db).forEach(modelName => {
  if (db[modelName].associate) {
    db[modelName].associate(db);
  }
});

db.sequelize = sequelize;
db.Sequelize = require('sequelize');

module.exports = db;
```

### 6. Update app.js
```javascript
const { connectDB } = require('./config/database');

// Connect to PostgreSQL
connectDB();
```

### 7. Update .env.example
```
POSTGRES_DB=mydb
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
```

### 8. Update docker-compose.yml
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - POSTGRES_HOST=postgres
      - POSTGRES_DB=mydb
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    depends_on:
      - postgres

  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Features

✅ **Sequelize ORM** for database modeling
✅ **Connection management** with environment variables
✅ **Model relationships** support (hasMany, belongsTo, etc.)
✅ **Validation** at the model level
✅ **Migrations support** via sequelize-cli
✅ **Soft deletes** with paranoid mode
✅ **Docker integration** with PostgreSQL container
✅ **UUID primary keys** by default

## Setup Instructions

### 1. Install Dependencies
```bash
npm install pg sequelize sequelize-cli
```

### 2. Configure Environment
```bash
# Copy .env.example to .env
cp .env.example .env

# Edit .env with your PostgreSQL configuration
POSTGRES_DB=mydb
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
```

### 3. Start PostgreSQL

**Option A: Local Installation**
```bash
# Install PostgreSQL
sudo apt-get install postgresql postgresql-contrib

# Start PostgreSQL
sudo systemctl start postgresql

# Create database
sudo -u postgres psql
CREATE DATABASE mydb;
\q
```

**Option B: Docker**
```bash
docker-compose up postgres
```

**Option C: Cloud Database (Neon, Supabase, AWS RDS)**
```bash
# Get your connection string from your cloud provider
POSTGRES_HOST=your-host
POSTGRES_PORT=5432
POSTGRES_DB=your-db
POSTGRES_USER=your-user
POSTGRES_PASSWORD=your-password
```

## Usage Examples

### Creating a Model
```javascript
// src/models/Product.js
const { DataTypes } = require('sequelize');
const { sequelize } = require('../config/database');

const Product = sequelize.define('Product', {
  name: {
    type: DataTypes.STRING,
    allowNull: false
  },
  price: {
    type: DataTypes.DECIMAL(10, 2),
    allowNull: false
  },
  description: {
    type: DataTypes.TEXT
  },
  stock: {
    type: DataTypes.INTEGER,
    defaultValue: 0
  }
});

module.exports = Product;
```

### Using in Controller
```javascript
const { Product } = require('../models');

exports.getProducts = async (req, res) => {
  const products = await Product.findAll();
  res.json({ success: true, data: products });
};

exports.getProduct = async (req, res) => {
  const product = await Product.findByPk(req.params.id);
  if (!product) {
    return res.status(404).json({ success: false, error: 'Product not found' });
  }
  res.json({ success: true, data: product });
};

exports.createProduct = async (req, res) => {
  const product = await Product.create(req.body);
  res.status(201).json({ success: true, data: product });
};

exports.updateProduct = async (req, res) => {
  const product = await Product.findByPk(req.params.id);
  if (!product) {
    return res.status(404).json({ success: false, error: 'Product not found' });
  }
  await product.update(req.body);
  res.json({ success: true, data: product });
};

exports.deleteProduct = async (req, res) => {
  const product = await Product.findByPk(req.params.id);
  if (!product) {
    return res.status(404).json({ success: false, error: 'Product not found' });
  }
  await product.destroy();
  res.json({ success: true, data: {} });
};
```

### Model Relationships
```javascript
// User model
User.hasMany(Order, { foreignKey: 'userId', as: 'orders' });

// Order model
Order.belongsTo(User, { foreignKey: 'userId', as: 'user' });
Order.hasMany(OrderItem, { foreignKey: 'orderId', as: 'items' });

// Querying with relations
const userWithOrders = await User.findByPk(userId, {
  include: [{
    model: Order,
    as: 'orders',
    include: ['items']
  }]
});
```

### Advanced Queries
```javascript
// Pagination
const products = await Product.findAndCountAll({
  limit: 10,
  offset: 0,
  order: [['createdAt', 'DESC']]
});

// Filtering
const products = await Product.findAll({
  where: {
    price: { [Op.gte]: 100 },
    stock: { [Op.gt]: 0 }
  }
});

// Searching
const products = await Product.findAll({
  where: {
    name: { [Op.iLike]: '%search term%' }
  }
});
```

## Migrations

### Initialize Migrations
```bash
npx sequelize-cli init
```

### Create Migration
```bash
npx sequelize-cli migration:generate --name create-users-table
```

### Run Migrations
```bash
npx sequelize-cli db:migrate
```

### Undo Migration
```bash
npx sequelize-cli db:migrate:undo
```

## Testing

```javascript
// tests/integration/product.test.js
const request = require('supertest');
const app = require('../../src/app');
const { Product } = require('../../src/models');

describe('Product Endpoints', () => {
  let productId;

  beforeEach(async () => {
    const product = await Product.create({
      name: 'Test Product',
      price: 99.99,
      description: 'Test description'
    });
    productId = product.id;
  });

  it('should get all products', async () => {
    const res = await request(app).get('/api/products');
    expect(res.statusCode).toBe(200);
    expect(res.body.success).toBe(true);
  });

  it('should get single product', async () => {
    const res = await request(app).get(`/api/products/${productId}`);
    expect(res.statusCode).toBe(200);
    expect(res.body.data.name).toBe('Test Product');
  });
});
```

## Next Steps

1. Run `/nodejs-api:add-endpoint <resource>` to create models with PostgreSQL
2. Set up migrations for database version control
3. Define model relationships
4. Add indexes for performance
5. Implement data seeding for development

## Troubleshooting

**Connection Error**
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```
- Make sure PostgreSQL is running
- Check your POSTGRES_HOST, POSTGRES_PORT in .env
- Verify database exists

**Authentication Error**
```
Error: password authentication failed
```
- Verify POSTGRES_USER and POSTGRES_PASSWORD
- Check pg_hba.conf if using local PostgreSQL
