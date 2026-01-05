---
description: "Initialize a new Gin API project with best practices, folder structure, and configuration"
argument-hint: "<project-name>"
---

# Initialize Gin API Project

This command creates a complete Gin API project with production-ready structure and configuration.

## Usage

```
/golang-api:init-gin-api my-api
```

## What It Creates

```
project-name/
├── api/
│   ├── controllers/
│   │   └── .gitkeep
│   ├── middleware/
│   │   ├── auth.go
│   │   ├── cors.go
│   │   └── logger.go
│   ├── models/
│   │   └── .gitkeep
│   ├── routes/
│   │   └── routes.go
│   ├── services/
│   │   └── .gitkeep
│   ├── utils/
│   │   ├── logger.go
│   │   └── response.go
│   └── database/
│       └── database.go
├── cmd/
│   └── server/
│       └── main.go
├── configs/
│   └── config.go
├── tests/
│   ├── integration/
│   │   └── .gitkeep
│   └── unit/
│       └── .gitkeep
├── .env.example
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── go.mod
├── go.sum
├── Makefile
└── README.md
```

## Features Included

✅ **Gin Web Framework** v1.10.x with middleware setup
✅ **Environment configuration** with viper
✅ **Error handling** with proper HTTP status codes
✅ **Logging system** (zap or logrus)
✅ **Security middleware** (CORS, auth)
✅ **Structured responses** for API consistency
✅ **Docker setup** with Dockerfile and docker-compose.yml
✅ **Testing framework** (testing package)
✅ **Git ignore** for Go projects
✅ **Makefile** for common commands
✅ **README** with setup instructions

## Quick Start

After running the command:

```bash
cd $1
go mod download
go run cmd/server/main.go          # Start development server
make test                          # Run tests
make build                         # Build binary
docker-compose up                  # Start with Docker
```

## Database Options

By default, the project is set up without a database. Use these commands to add database support:

- `/golang-api:setup-mongodb` - Add MongoDB integration
- `/golang-api:setup-postgresql` - Add PostgreSQL integration

## Adding Endpoints

Use `/golang-api:add-endpoint <resource>` to quickly scaffold CRUD endpoints for a new resource.

## API Documentation

Run `/golang-api:generate-swagger` to add Swagger/OpenAPI documentation.

## Configuration

The project uses environment variables. Copy `.env.example` to `.env` and configure:

```bash
PORT=8080
GIN_MODE=debug
MONGODB_URI=mongodb://localhost:27017/mydb
POSTGRES_URI=postgresql://user:password@localhost:5432/mydb
JWT_SECRET=your-secret-key
LOG_LEVEL=debug
```

## Makefile Commands

```bash
make run        # Run the server
make build      # Build the application
make test       # Run all tests
make test-cov   # Run tests with coverage
make docker     # Build Docker image
make docker-run # Run Docker container
make clean      # Clean build artifacts
```

## Project Structure

### cmd/server/main.go
Application entry point

### api/
Application logic
- **controllers/** - Request handlers
- **middleware/** - Middleware (auth, logging, etc.)
- **models/** - Data models
- **routes/** - Route definitions
- **services/** - Business logic
- **database/** - Database setup

### configs/
Configuration management

### utils/
Utility functions and helpers

## Environment Variables

```bash
# Server
PORT=8080
GIN_MODE=debug  # debug, release, test

# Database
MONGODB_URI=mongodb://localhost:27017/mydb
POSTGRES_URI=postgresql://user:password@localhost:5432/mydb

# Security
JWT_SECRET=your-jwt-secret-key-here
JWT_EXPIRE=24h

# Logging
LOG_LEVEL=debug  # debug, info, warn, error

# CORS
CORS_ALLOWED_ORIGINS=http://localhost:3000
```

## Examples

### Create a new API project
```bash
/golang-api:init-gin-api user-api
```

### Run the server
```bash
cd user-api
make run
# Server runs on http://localhost:8080
```

### Build the application
```bash
make build
./bin/user-api
```

### Run tests
```bash
make test
make test-cov
```

### Use Docker
```bash
make docker
make docker-run
```

## Next Steps

1. Run `/golang-api:setup-mongodb` or `/golang-api:setup-postgresql` to add database
2. Run `/golang-api:add-endpoint <resource>` to create CRUD endpoints
3. Run `/golang-api:generate-swagger` to add API documentation
4. Configure `.env` with your settings
5. Implement business logic in services
6. Write tests for your endpoints
7. Add authentication and authorization middleware
8. Deploy using Docker or your preferred method
