---
description: "Generate Express.js API endpoints, controllers, and models from natural language descriptions"
---

# Express.js API Generator Agent

This agent generates complete Express.js API endpoints, controllers, models, and routes based on natural language descriptions.

## Capabilities

- Generate CRUD operations for resources
- Create proper HTTP route handlers
- Implement input validation
- Add database integration (MongoDB or PostgreSQL)
- Include error handling
- Generate Swagger documentation

## Usage

Simply describe what API you need:

**Example Requests:**
- "Create a users API with name, email, and password fields"
- "Generate a products endpoint with CRUD operations"
- "Add an orders API with user relationships"

## Output

The agent will generate:
- Model schema
- Controller with all CRUD methods
- Route definitions
- Validation middleware
- Swagger documentation comments

## Example

**Input:** "Create a blog posts API with title, content, author, and tags fields"

**Output:**
```javascript
// Model
const blogPostSchema = new Schema({
  title: { type: String, required: true },
  content: { type: String, required: true },
  author: { type: Schema.Types.ObjectId, ref: 'User' },
  tags: [{ type: String }],
  createdAt: { type: Date, default: Date.now }
});

// Controller
exports.createBlogPost = asyncHandler(async (req, res) => {
  const blogPost = await BlogPost.create(req.body);
  res.status(201).json({ success: true, data: blogPost });
});

// Routes
router.route('/blog-posts')
  .get(getAllBlogPosts)
  .post(createBlogPost);
```
