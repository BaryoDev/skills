---
name: baryo-devops
description: DevOps and Infrastructure as Code standards for CI/CD, containerization, and deployment.
---

# BaryoDev DevOps Standard

You are the DevOps Engineer. **Automate everything, trust nothing.**

## CI/CD Pipeline

### Pipeline Stages
```yaml
stages:
  - lint      # Code quality
  - test      # Unit + integration tests
  - security  # Vulnerability scanning
  - build     # Build artifacts
  - deploy    # Deploy to environment
```

### GitHub Actions Example
```yaml
name: CI/CD
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm audit

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

## Containerization

### Dockerfile Best Practices
```dockerfile
# Multi-stage build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
CMD ["node", "dist/index.js"]
```

## Infrastructure as Code

### Terraform Pattern
```hcl
resource "aws_ecs_service" "api" {
  name            = "api-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.api.arn
  desired_count   = 3
  
  deployment_configuration {
    maximum_percent         = 200
    minimum_healthy_percent = 100
  }
}
```

## Environment Parity
- Dev ≈ Staging ≈ Production
- Same containers, different configs
- Feature flags for differences

## Deployment Strategies
- **Blue-Green**: Instant rollback capability
- **Canary**: Gradual rollout (1% → 10% → 100%)
- **Rolling**: Replace instances one by one

## Checklist
- [ ] All infrastructure as code
- [ ] CI runs on every PR
- [ ] Tests must pass before merge
- [ ] Security scanning in pipeline
- [ ] Automated deployments
- [ ] Rollback mechanism tested
- [ ] Secrets in vault, not code
