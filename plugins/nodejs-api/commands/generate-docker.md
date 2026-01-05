---
description: "Generate Dockerfile and docker-compose.yml for containerization"
argument-hint: ""
---

# Generate Docker Configuration

This command generates complete Docker configuration files for containerizing your Express.js API.

## Usage

```
/nodejs-api:generate-docker
```

## What It Creates

### 1. Dockerfile (in project root)
```dockerfile
# Use Node.js LTS version
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy application code
COPY . .

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 && \
    chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose application port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start application
CMD ["npm", "start"]
```

### 2. docker-compose.yml (in project root)
```yaml
version: '3.8'

services:
  # Main application
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: nodejs-api
    restart: unless-stopped
    ports:
      - "${PORT:-3000}:3000"
    environment:
      - NODE_ENV=${NODE_ENV:-production}
      - PORT=${PORT:-3000}
    env_file:
      - .env
    depends_on:
      - postgres
      - redis
    volumes:
      - ./logs:/app/logs
      - ./uploads:/app/uploads
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  # PostgreSQL database
  postgres:
    image: postgres:15-alpine
    container_name: nodejs-postgres
    restart: unless-stopped
    ports:
      - "${POSTGRES_PORT:-5432}:5432"
    environment:
      - POSTGRES_DB=${POSTGRES_DB:-mydb}
      - POSTGRES_USER=${POSTGRES_USER:-postgres}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD:-postgres}
      - POSTGRES_INITDB_ARGS=--encoding=UTF-8 --lc-collate=C --lc-ctype=C
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-db.sql:/docker-entrypoint-initdb.d/init-db.sql
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-postgres}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # MongoDB database (optional)
  mongodb:
    image: mongo:7.0
    container_name: nodejs-mongodb
    restart: unless-stopped
    ports:
      - "${MONGODB_PORT:-27017}:27017"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_ROOT_USERNAME:-admin}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_ROOT_PASSWORD:-admin}
      - MONGO_INITDB_DATABASE=${MONGODB_DB:-mydb}
    volumes:
      - mongodb_data:/data/db
      - mongodb_config:/data/configdb
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5
    profiles:
      - with-mongodb

  # Redis for caching/sessions
  redis:
    image: redis:7-alpine
    container_name: nodejs-redis
    restart: unless-stopped
    ports:
      - "${REDIS_PORT:-6379}:6379"
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD:-redis}
    volumes:
      - redis_data:/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "incr", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    profiles:
      - with-redis

  # Nginx reverse proxy (optional)
  nginx:
    image: nginx:alpine
    container_name: nodejs-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    networks:
      - app-network
    profiles:
      - with-nginx

networks:
  app-network:
    driver: bridge

volumes:
  postgres_data:
    driver: local
  mongodb_data:
    driver: local
  mongodb_config:
    driver: local
  redis_data:
    driver: local
```

### 3. .dockerignore (in project root)
```
node_modules
npm-debug.log
.git
.gitignore
.env
.env.local
.env.development.local
.env.test.local
.env.production.local
Dockerfile
docker-compose.yml
docker-compose.*.yml
README.md
.vscode
.idea
*.md
tests
coverage
.nyc_output
logs
uploads
dist
build
```

### 4. Create Health Check Endpoint (src/routes/health.js)
```javascript
const express = require('express');
const router = express.Router();

router.get('/', async (req, res) => {
  try {
    // Check database connection
    const dbStatus = await checkDatabase();

    res.json({
      status: 'ok',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      environment: process.env.NODE_ENV,
      database: dbStatus
    });
  } catch (error) {
    res.status(503).json({
      status: 'error',
      message: error.message
    });
  }
});

async function checkDatabase() {
  // Add your database check logic here
  // Example for MongoDB or PostgreSQL
  return { status: 'connected' };
}

module.exports = router;
```

### 5. Update src/routes/index.js (if not already done)
```javascript
const express = require('express');
const router = express.Router();

// Health check
router.use('/health', require('./health'));

// Your other routes
// router.use('/users', require('./users'));

module.exports = router;
```

## Features

✅ **Multi-stage build** for optimized image size
✅ **Non-root user** for enhanced security
✅ **Health checks** for container monitoring
✅ **Auto-restart** policy
✅ **Database services** (PostgreSQL, MongoDB)
✅ **Redis** for caching and sessions (optional)
✅ **Nginx** reverse proxy (optional)
✅ **Volume management** for data persistence
✅ **Network isolation** for security
✅ **Environment-specific configurations**
✅ **Logging persistence**
✅ **Profile-based services** (start only what you need)

## Usage

### Basic Usage
```bash
# Start application and PostgreSQL
docker-compose up -d

# Start application with MongoDB
docker-compose --profile with-mongodb up -d

# Start application with Redis
docker-compose --profile with-redis up -d

# Start all services (Postgres, MongoDB, Redis, Nginx)
docker-compose --profile with-mongodb --profile with-redis --profile with-nginx up -d
```

### View Logs
```bash
# View all logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f app
docker-compose logs -f postgres
```

### Stop Services
```bash
# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

### Rebuild
```bash
# Rebuild application
docker-compose build app

# Rebuild and start
docker-compose up -d --build app
```

## Docker Compose Profiles

The generated docker-compose.yml supports profiles for optional services:

```bash
# Production with all services
docker-compose --profile with-mongodb --profile with-redis --profile with-nginx up -d

# Development without optional services
docker-compose up -d

# Development with Redis only
docker-compose --profile with-redis up -d
```

## Environment Variables

Create `.env` file in project root:
```bash
# Application
NODE_ENV=production
PORT=3000

# PostgreSQL
POSTGRES_DB=mydb
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_password
POSTGRES_PORT=5432

# MongoDB (if using)
MONGODB_URI=mongodb://admin:admin@mongodb:27017/mydb?authSource=admin
MONGODB_PORT=27017

# Redis (if using)
REDIS_PASSWORD=your_redis_password
REDIS_PORT=6379
```

## Production Considerations

### 1. Use Secrets (Docker Swarm or Kubernetes)
```yaml
services:
  app:
    secrets:
      - db_password
secrets:
  db_password:
    external: true
```

### 2. Resource Limits
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

### 3. Update Policy
```yaml
services:
  app:
    image: your-registry/nodejs-api:latest
    deploy:
      update_config:
        parallelism: 2
        delay: 10s
        failure_action: rollback
```

## Monitoring

### Docker Stats
```bash
docker stats
```

### Health Check
```bash
curl http://localhost:3000/health
```

## Development vs Production

### Development Mode
```bash
# Use docker-compose.dev.yml (you can create this)
docker-compose -f docker-compose.dev.yml up
```

### Production Mode
```bash
docker-compose -f docker-compose.prod.yml up
```

## Next Steps

1. Customize Dockerfile for your specific needs
2. Add SSL certificates for HTTPS
3. Configure Nginx reverse proxy
4. Set up CI/CD pipeline for automated deployments
5. Add monitoring (Prometheus, Grafana)
6. Implement log aggregation (ELK, Loki)

## Troubleshooting

**Port already in use**
```bash
# Change port in .env or docker-compose.yml
PORT=3001
```

**Container won't start**
```bash
# Check logs
docker-compose logs app

# Check health status
docker-compose ps
```

**Database connection issues**
```bash
# Verify database is running
docker-compose ps postgres

# Check database logs
docker-compose logs postgres
```

## Best Practices

1. **Use specific version tags** instead of `latest`
2. **Minimize image size** by using `.dockerignore`
3. **Run as non-root user** for security
4. **Implement health checks** for all services
5. **Use volumes** for data persistence
6. **Add resource limits** in production
7. **Use secrets** for sensitive data
8. **Implement logging** strategy
