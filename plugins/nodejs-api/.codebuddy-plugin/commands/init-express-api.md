---
description: "Initialize a new Express.js API project with best practices, folder structure, and configuration"
argument-hint: "<project-name>"
---

# Initialize Express.js API Project

This command creates a complete Express.js API project with production-ready structure and configuration.

## Usage

```
/nodejs-api:init-express-api my-api
```

## What It Creates

```
project-name/
├── src/
│   ├── config/
│   │   ├── database.js
│   │   └── index.js
│   ├── controllers/
│   │   └── .gitkeep
│   ├── middleware/
│   │   ├── auth.js
│   │   ├── errorHandler.js
│   │   └── validate.js
│   ├── models/
│   │   └── .gitkeep
│   ├── routes/
│   │   ├── index.js
│   │   └── .gitkeep
│   ├── services/
│   │   └── .gitkeep
│   ├── utils/
│   │   ├── logger.js
│   │   └── asyncHandler.js
│   ├── app.js
│   └── server.js
├── tests/
│   ├── integration/
│   │   └── .gitkeep
│   └── unit/
│       └── .gitkeep
├── .env.example
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── package.json
├── jest.config.js
└── README.md
```

## Features Included

✅ **Express.js 4.x** with middleware setup
✅ **Environment configuration** with dotenv
✅ **Error handling middleware** with proper HTTP status codes
✅ **Logging system** (winston or morgan)
✅ **Security headers** (helmet)
✅ **CORS support**
✅ **Request validation** middleware
✅ **Async/await error handling** utility
✅ **Docker setup** with Dockerfile and docker-compose.yml
✅ **Jest testing framework** configured
✅ **Git ignore** for Node.js projects
✅ **README** with setup instructions

## Quick Start

After running the command:

```bash
cd $1
npm install
npm run dev        # Start development server
npm test           # Run tests
npm run start      # Start production server
docker-compose up  # Start with Docker
```

## Database Options

By default, the project is set up without a database. Use these commands to add database support:

- `/nodejs-api:setup-mongodb` - Add MongoDB integration
- `/nodejs-api:setup-postgresql` - Add PostgreSQL integration

## Adding Endpoints

Use `/nodejs-api:add-endpoint <resource>` to quickly scaffold CRUD endpoints for a new resource.

## API Documentation

Run `/nodejs-api:generate-swagger` to add Swagger/OpenAPI documentation.

## Configuration

The project uses environment variables. Copy `.env.example` to `.env` and configure:

```
PORT=3000
NODE_ENV=development
MONGODB_URI=mongodb://localhost:27017/mydb
POSTGRES_URI=postgresql://user:password@localhost:5432/mydb
JWT_SECRET=your-secret-key
```

## Scripts

- `npm run dev` - Start development server with hot reload
- `npm run start` - Start production server
- `npm test` - Run all tests
- `npm test:watch` - Run tests in watch mode
- `npm run lint` - Run ESLint
- `npm run format` - Format code with Prettier
