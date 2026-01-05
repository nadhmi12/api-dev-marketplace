# API Development Marketplace

A comprehensive plugin marketplace for CodeBuddy Code that provides powerful tools for API development in Node.js and Golang.

## 🚀 Features

- **Node.js/Express Plugin**: Complete toolkit for building Express.js APIs
- **Golang/Gin Plugin**: Full-featured toolkit for Gin web framework APIs
- **Database Integration**: Support for MongoDB and PostgreSQL
- **Docker Support**: Ready-to-use containerization configurations
- **Testing Frameworks**: Comprehensive testing setup with Jest (Node.js) and Go testing
- **API Documentation**: Swagger/OpenAPI integration for both frameworks
- **CI/CD Pipelines**: GitHub Actions and GitLab CI configurations
- **Best Practices Skills**: Built-in guidance for development patterns

## 📦 Plugins

### 1. nodejs-api

A complete Node.js API development toolkit with Express.js.

**Commands:**
- `init-express-api` - Initialize new Express.js project
- `add-endpoint` - Generate CRUD endpoints
- `setup-mongodb` - Add MongoDB integration
- `setup-postgresql` - Add PostgreSQL integration
- `generate-docker` - Generate Docker configuration
- `generate-swagger` - Add Swagger documentation
- `setup-jest` - Setup Jest testing framework
- `run-tests` - Run tests with options
- `setup-ci` - Setup CI/CD pipeline

**Skills:**
- express-best-practices
- api-design-patterns
- error-handling
- security-considerations

**Agents:**
- api-generator - Generate Express.js APIs from natural language
- database-designer - Design database schemas and relationships

**Features:**
- ✅ Express.js 4.x with middleware
- ✅ Environment configuration
- ✅ Error handling middleware
- ✅ Security (Helmet, CORS, rate limiting)
- ✅ Database integration (MongoDB, PostgreSQL)
- ✅ Docker support
- ✅ Jest testing framework
- ✅ Swagger/OpenAPI documentation
- ✅ CI/CD pipelines (GitHub Actions, GitLab CI)

### 2. golang-api

A complete Golang API development toolkit with Gin framework.

**Commands:**
- `init-gin-api` - Initialize new Gin project
- `add-endpoint` - Generate CRUD endpoints
- `setup-mongodb` - Add MongoDB integration
- `setup-postgresql` - Add PostgreSQL integration
- `generate-docker` - Generate Docker configuration
- `generate-swagger` - Add Swagger documentation
- `setup-gotest` - Setup Go testing framework
- `run-tests` - Run tests
- `setup-ci` - Setup CI/CD pipeline

**Skills:**
- go-best-practices
- api-design-patterns
- error-handling
- security-considerations

**Agents:**
- api-generator - Generate Gin APIs from natural language
- database-designer - Design database schemas and relationships

**Features:**
- ✅ Gin web framework v1.10
- ✅ Environment configuration with Viper
- ✅ MongoDB driver integration
- ✅ PostgreSQL with GORM
- ✅ Docker multi-stage builds
- ✅ Go testing framework
- ✅ Swagger/OpenAPI with Swag
- ✅ CI/CD pipelines

## 🔧 Installation

### Add Marketplace to CodeBuddy Code

```bash
codebuddy plugin marketplace add https://github.com/your-org/api-dev-marketplace
```

### Install Plugins

```bash
# Install Node.js API plugin
codebuddy plugin install nodejs-api

# Install Golang API plugin
codebuddy plugin install golang-api

# Or install both
codebuddy plugin install nodejs-api golang-api
```

## 📚 Usage Examples

### Node.js/Express

#### Initialize a new Express API
```bash
/nodejs-api:init-express-api my-api
cd my-api
npm install
npm run dev
```

#### Add database support
```bash
/nodejs-api:setup-mongodb
# or
/nodejs-api:setup-postgresql
```

#### Generate CRUD endpoints
```bash
/nodejs-api:add-endpoint users
```

#### Add Docker support
```bash
/nodejs-api:generate-docker
docker-compose up
```

#### Add API documentation
```bash
/nodejs-api:generate-swagger
# Visit http://localhost:3000/api-docs
```

### Golang/Gin

#### Initialize a new Gin API
```bash
/golang-api:init-gin-api my-api
cd my-api
go mod download
go run cmd/server/main.go
```

#### Add database support
```bash
/golang-api:setup-mongodb
# or
/golang-api:setup-postgresql
```

#### Generate CRUD endpoints
```bash
/golang-api:add-endpoint users
```

#### Add Docker support
```bash
/golang-api:generate-docker
docker-compose up
```

#### Add API documentation
```bash
/golang-api:generate-swagger
swag init
# Visit http://localhost:8080/swagger/index.html
```

## 🎯 Features Overview

### Project Initialization
- Complete folder structure
- Best practice configurations
- Environment variable setup
- Security middleware
- Error handling
- Logging system

### Database Support
- **MongoDB**: Official drivers, connection pooling, CRUD operations
- **PostgreSQL**: GORM ORM, relationships, migrations, validation

### API Design
- RESTful endpoints
- CRUD operations
- Pagination, filtering, sorting
- Input validation
- Error handling

### Testing
- **Node.js**: Jest, Supertest, coverage reports
- **Golang**: Testing package, race detection, benchmarks

### Documentation
- Swagger/OpenAPI 3.0
- Interactive API explorer
- Auto-generated from code comments

### Docker
- Multi-stage builds
- Database services
- Volume management
- Network isolation
- Environment-specific configurations

### CI/CD
- GitHub Actions
- GitLab CI
- Automated testing
- Docker image building
- Deployment pipelines

## 🔒 Security Features

- Password hashing (bcrypt)
- JWT authentication
- Rate limiting
- Input validation
- SQL injection prevention
- XSS protection
- CORS configuration
- Security headers (Helmet)

## 📊 Project Structure

### Node.js/Express
```
project/
├── src/
│   ├── config/
│   ├── controllers/
│   ├── middleware/
│   ├── models/
│   ├── routes/
│   ├── services/
│   └── utils/
├── tests/
│   ├── integration/
│   └── unit/
├── .env.example
├── Dockerfile
├── docker-compose.yml
└── package.json
```

### Golang/Gin
```
project/
├── api/
│   ├── controllers/
│   ├── middleware/
│   ├── models/
│   ├── routes/
│   ├── services/
│   └── database/
├── cmd/
│   └── server/
│       └── main.go
├── configs/
├── tests/
├── .env.example
├── Dockerfile
├── docker-compose.yml
└── go.mod
```

## 🛠️ Development

### Prerequisites

- Node.js 18+ (for Node.js plugin)
- Go 1.21+ (for Golang plugin)
- Docker (optional, for containerization)
- MongoDB (optional, for MongoDB support)
- PostgreSQL (optional, for PostgreSQL support)

### Quick Start

1. Add the marketplace to CodeBuddy Code
2. Install the desired plugin
3. Run initialization command
4. Follow the setup instructions
5. Start building your API!

## 📖 Documentation

Each command includes comprehensive documentation with:
- Detailed descriptions
- Code examples
- Configuration options
- Best practices
- Troubleshooting tips

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

MIT License - see LICENSE file for details

## 🙋 Support

For issues and questions:
- Create an issue on GitHub
- Check the documentation for troubleshooting
- Review the skills for best practices

## 🌟 Features Coming Soon

- [ ] GraphQL support
- [ ] WebSocket support
- [ ] Microservices patterns
- [ ] API Gateway configurations
- [ ] Monitoring and observability
- [ ] Performance optimization guides
- [ ] Database migrations
- [ ] API versioning strategies
- [ ] Authentication providers (OAuth, SAML)
- [ ] Real-time features

## 📝 Changelog

### Version 1.0.0 (2024-01-15)
- Initial release
- Node.js/Express plugin
- Golang/Gin plugin
- MongoDB and PostgreSQL support
- Docker integration
- Testing frameworks
- Swagger documentation
- CI/CD pipelines

## 🔗 Links

- [Node.js Plugin Documentation](plugins/nodejs-api/.codebuddy-plugin/)
- [Golang Plugin Documentation](plugins/golang-api/.codebuddy-plugin/)
- [CodeBuddy Code Documentation](https://docs.codebuddy.example.com)

---

Made with ❤️ by the API Dev Team
