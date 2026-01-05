---
description: "Setup CI/CD pipeline with GitHub Actions or GitLab CI"
argument-hint: "<platform>"
---

# Setup CI/CD Pipeline

This command creates a complete CI/CD pipeline for continuous integration and deployment.

## Usage

```bash
/nodejs-api:setup-ci github          # Setup GitHub Actions
/nodejs-api:setup-ci gitlab          # Setup GitLab CI
/nodejs-api:setup-ci all             # Setup both platforms
```

## GitHub Actions

### What It Creates (.github/workflows/ci.yml)
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  NODE_VERSION: '20.x'
  MONGODB_URI: mongodb://localhost:27017/testdb

jobs:
  # Code Quality Checks
  lint:
    name: Lint and Format Check
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Check code formatting
        run: npm run format:check

  # Run Tests
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    needs: lint

    services:
      mongodb:
        image: mongo:7.0
        ports:
          - 27017:27017
        options: >-
          --health-cmd "mongosh --eval 'db.runCommand(\"ping\").ok'"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
        env:
          TEST_MONGODB_URI: ${{ env.MONGODB_URI }}

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella
          fail_ci_if_error: false

  # Build Docker Image
  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: test
    if: github.event_name == 'push'

    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ secrets.DOCKER_USERNAME }}/nodejs-api
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # Security Scanning
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ needs.build.outputs.image-tag }}
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

  # Deploy to Production
  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [test, security]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.DEPLOY_KEY }}
          script: |
            cd /app/nodejs-api
            git pull origin main
            docker-compose pull
            docker-compose up -d
            docker system prune -f

      - name: Notify deployment
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Deployment to production completed'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        if: always()

  # Deploy to Staging
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [test, security]
    if: github.ref == 'refs/heads/develop' && github.event_name == 'push'

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to staging
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_KEY }}
          script: |
            cd /app/nodejs-api
            git pull origin develop
            docker-compose pull
            docker-compose up -d
```

### GitHub Workflows for Release (.github/workflows/release.yml)
```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          draft: false
          prerelease: false
```

## GitLab CI

### What It Creates (.gitlab-ci.yml)
```yaml
stages:
  - lint
  - test
  - build
  - security
  - deploy

variables:
  NODE_VERSION: "20"
  DOCKER_IMAGE: ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHORT_SHA}
  MONGODB_URI: mongodb://mongo:27017/testdb

# Lint Stage
lint:
  stage: lint
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm run lint
    - npm run format:check
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  only:
    - branches
    - merge_requests

# Test Stage
test:
  stage: test
  image: node:${NODE_VERSION}
  services:
    - mongo:7.0

  variables:
    TEST_MONGODB_URI: mongodb://mongo:27017/testdb

  script:
    - npm ci
    - npm test

  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'

  artifacts:
    when: always
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
      junit:
        - junit.xml
    paths:
      - coverage/

  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/

  only:
    - branches
    - merge_requests

# Build Stage
build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind

  before_script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login -u "$CI_REGISTRY_USER" --password-stdin $CI_REGISTRY

  script:
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE
    - docker tag $DOCKER_IMAGE ${CI_REGISTRY_IMAGE}:latest
    - docker push ${CI_REGISTRY_IMAGE}:latest

  only:
    - main
    - develop

# Security Scan
security:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $DOCKER_IMAGE
  allow_failure: true
  only:
    - main
    - develop

# Deploy to Staging
deploy_staging:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$STAGING_SSH_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan $STAGING_HOST >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts

    - ssh $STAGING_USER@$STAGING_HOST "cd /app/nodejs-api && git pull origin develop && docker-compose pull && docker-compose up -d && docker system prune -f"

  only:
    - develop
  when: manual

# Deploy to Production
deploy_production:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$PRODUCTION_SSH_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan $PRODUCTION_HOST >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts

    - ssh $PRODUCTION_USER@$PRODUCTION_HOST "cd /app/nodejs-api && git pull origin main && docker-compose pull && docker-compose up -d && docker system prune -f"

  only:
    - main
  when: manual
```

## Setup Instructions

### GitHub Actions

1. **Add Secrets** (Settings → Secrets and variables → Actions)
```bash
# Docker Hub
DOCKER_USERNAME=your-dockerhub-username
DOCKER_PASSWORD=your-dockerhub-password

# Deployment Server
DEPLOY_HOST=your-server.com
DEPLOY_USER=deploy-user
DEPLOY_KEY=private-ssh-key

# Slack Notifications (optional)
SLACK_WEBHOOK=your-slack-webhook-url
```

2. **Enable Workflows**
- Workflows automatically run on push/PR
- Manual triggers available in Actions tab

3. **Monitor Runs**
- Go to Actions tab in GitHub
- View logs, artifacts, and status

### GitLab CI

1. **Add Variables** (Settings → CI/CD → Variables)
```bash
# GitLab Container Registry
CI_REGISTRY=registry.gitlab.com
CI_REGISTRY_USER=gitlab-ci-token
CI_REGISTRY_PASSWORD=$CI_JOB_TOKEN

# Staging Server
STAGING_HOST=your-staging-server.com
STAGING_USER=deploy-user
STAGING_SSH_KEY=private-ssh-key

# Production Server
PRODUCTION_HOST=your-production-server.com
PRODUCTION_USER=deploy-user
PRODUCTION_SSH_KEY=private-ssh-key
```

2. **Enable Container Registry**
- Settings → General → Visibility, project features, permissions
- Enable Container Registry

3. **Monitor Pipelines**
- Go to CI/CD → Pipelines
- View logs and job status

## Features

✅ **Automated linting** and code quality checks
✅ **Automated testing** on every commit
✅ **Code coverage** reporting
✅ **Docker image** building and pushing
✅ **Security scanning** with Trivy
✅ **Automated deployment** to staging/production
✅ **Slack notifications** (GitHub)
✅ **Manual deployment** gates (GitLab)
✅ **Artifact management**
✅ **Cache optimization** for faster builds

## Environment-Specific Deployments

### Development
```yaml
# Runs on every push to any branch
on: [push]
```

### Staging
```yaml
# Runs on pushes to develop branch
if: github.ref == 'refs/heads/develop'
```

### Production
```yaml
# Runs on pushes to main branch
if: github.ref == 'refs/heads/main'
when: manual  # Requires manual approval
```

## Deployment Strategies

### Blue-Green Deployment
```yaml
- name: Deploy to production (Blue-Green)
  run: |
    docker-compose up -d --scale app=2 --no-recreate
    sleep 30
    docker-compose up -d --scale app=1
```

### Rolling Update
```yaml
- name: Rolling update
  run: |
    docker-compose pull
    docker-compose up -d --no-deps --scale app=2
    docker-compose up -d --no-deps --scale app=1
```

### Canary Deployment
```yaml
- name: Canary deployment
  run: |
    # Deploy to canary server first
    ssh canary-server "docker pull && docker-compose up -d"
    # Monitor metrics before full rollout
```

## Monitoring and Notifications

### Slack Notifications
```yaml
- name: Notify on Slack
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
  if: always()
```

### Email Notifications
```yaml
- name: Send email
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: smtp.gmail.com
    server_port: 465
    username: ${{ secrets.EMAIL_USERNAME }}
    password: ${{ secrets.EMAIL_PASSWORD }}
    subject: 'CI/CD Pipeline Status'
    body: |
      Pipeline ${{ job.status }}
```

## Next Steps

1. Configure secrets/variables in your CI/CD platform
2. Test the pipeline with a feature branch
3. Set up monitoring and logging
4. Configure rollback procedures
5. Add performance benchmarks to pipeline
6. Set up database migrations in deployment
7. Implement canary deployments

## Best Practices

1. **Use feature branches** for development
2. **Require approvals** for production deployments
3. **Test on staging** before production
4. **Monitor deployment** health
5. **Keep pipelines fast** (under 10 minutes)
6. **Use caching** to speed up builds
7. **Review logs** regularly
8. **Implement rollback** procedures
9. **Version tag** production releases
10. **Security scan** every deployment
