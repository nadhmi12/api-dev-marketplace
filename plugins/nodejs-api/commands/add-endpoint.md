---
description: "Generate CRUD endpoints for a resource with controller, model, and routes"
argument-hint: "<resource-name>"
---

# Add CRUD Endpoints

This command generates complete CRUD (Create, Read, Update, Delete) endpoints for a resource.

## Usage

```
/nodejs-api:add-endpoint users
/nodejs-api:add-endpoint products
/nodejs-api:add-endpoint orders
```

## What It Creates

### 1. Model (src/models/<Resource>.js)
```javascript
const mongoose = require('mongoose');

const { Schema } = mongoose;

const ResourceSchema = new Schema({
  name: { type: String, required: true },
  description: { type: String },
  // Additional fields based on resource
}, {
  timestamps: true
});

module.exports = mongoose.model('Resource', ResourceSchema);
```

### 2. Controller (src/controllers/<Resource>Controller.js)
```javascript
const Resource = require('../models/<Resource>');
const asyncHandler = require('../utils/asyncHandler');

// @desc    Get all resources
// @route   GET /api/resources
// @access  Public
exports.getResources = asyncHandler(async (req, res) => {
  const resources = await Resource.find();
  res.json({ success: true, count: resources.length, data: resources });
});

// @desc    Get single resource
// @route   GET /api/resources/:id
// @access  Public
exports.getResource = asyncHandler(async (req, res) => {
  const resource = await Resource.findById(req.params.id);
  if (!resource) {
    return res.status(404).json({ success: false, error: 'Resource not found' });
  }
  res.json({ success: true, data: resource });
});

// @desc    Create resource
// @route   POST /api/resources
// @access  Private
exports.createResource = asyncHandler(async (req, res) => {
  const resource = await Resource.create(req.body);
  res.status(201).json({ success: true, data: resource });
});

// @desc    Update resource
// @route   PUT /api/resources/:id
// @access  Private
exports.updateResource = asyncHandler(async (req, res) => {
  const resource = await Resource.findByIdAndUpdate(
    req.params.id,
    req.body,
    { new: true, runValidators: true }
  );
  if (!resource) {
    return res.status(404).json({ success: false, error: 'Resource not found' });
  }
  res.json({ success: true, data: resource });
});

// @desc    Delete resource
// @route   DELETE /api/resources/:id
// @access  Private
exports.deleteResource = asyncHandler(async (req, res) => {
  const resource = await Resource.findByIdAndDelete(req.params.id);
  if (!resource) {
    return res.status(404).json({ success: false, error: 'Resource not found' });
  }
  res.json({ success: true, data: {} });
});
```

### 3. Routes (src/routes/<Resource>.js)
```javascript
const express = require('express');
const router = express.Router();
const {
  getResources,
  getResource,
  createResource,
  updateResource,
  deleteResource
} = require('../controllers/<Resource>Controller');

router.route('/')
  .get(getResources)
  .post(createResource);

router.route('/:id')
  .get(getResource)
  .put(updateResource)
  .delete(deleteResource);

module.exports = router;
```

### 4. Update Main Routes (src/routes/index.js)
```javascript
const express = require('express');
const router = express.Router();
const resourceRoutes = require('./<Resource>');

router.use('/resources', resourceRoutes);

module.exports = router;
```

## Features

✅ **CRUD Operations**: GET, POST, PUT, DELETE
✅ **MongoDB Mongoose Integration** (if MongoDB is set up)
✅ **Proper Error Handling** with asyncHandler utility
✅ **RESTful API Design** following best practices
✅ **JSDoc Comments** for API documentation
✅ **Resource Naming** automatically pluralized

## API Endpoints Created

- `GET /api/resources` - Get all resources
- `GET /api/resources/:id` - Get single resource
- `POST /api/resources` - Create new resource
- `PUT /api/resources/:id` - Update resource
- `DELETE /api/resources/:id` - Delete resource

## Examples

```bash
# Create a users resource
/nodejs-api:add-endpoint users

# This creates:
# - src/models/User.js
# - src/controllers/userController.js
# - src/routes/users.js
# - Updates src/routes/index.js
```

## Customization

After generation, you can:
1. Add custom fields to the model
2. Add validation rules
3. Implement business logic in the controller
4. Add middleware for authentication/authorization
5. Add query parameters for filtering, pagination, sorting

## Authentication

To protect endpoints, add authentication middleware:

```javascript
const { protect } = require('../middleware/auth');

router.use('/resources', protect);
```

## Next Steps

1. Run `/nodejs-api:generate-swagger` to add API documentation
2. Add validation using the validation middleware
3. Implement business logic in the controller
4. Write tests for your endpoints
