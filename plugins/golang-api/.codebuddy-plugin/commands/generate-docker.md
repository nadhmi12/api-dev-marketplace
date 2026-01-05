---
description: "Generate Dockerfile and docker-compose.yml for containerization"
argument-hint: ""
---

# Generate Docker Configuration

This command generates complete Docker configuration files for containerizing your Gin API.

## Usage

```
/golang-api:generate-docker
```

## What It Creates

### 1. Dockerfile
```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./

# Download dependencies
RUN go mod download

# Copy source code
COPY . .

# Build application
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main cmd/server/main.go

# Final stage
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy binary from builder
COPY --from=builder /app/main .

# Expose port
EXPOSE 8080

# Run binary
CMD ["./main"]
```

### 2. docker-compose.yml
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - PORT=8080
      - MONGODB_URI=mongodb://mongodb:27017/mydb
      - POSTGRES_URI=host=postgres user=postgres password=postgres dbname=mydb port=5432
    depends_on:
      - mongodb
      - postgres
    networks:
      - app-network

  mongodb:
    image: mongo:7.0
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    networks:
      - app-network

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
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  mongodb_data:
  postgres_data:
```

### 3. .dockerignore
```
.git
.gitignore
README.md
.env
.env.local
Dockerfile
docker-compose.yml
tests
coverage
```

## Usage

```bash
docker-compose up -d
docker-compose logs -f
docker-compose down
```

## Features

✅ Multi-stage build for smaller image
✅ Alpine Linux for minimal size
✅ Database services included
✅ Network isolation
✅ Volume persistence
