# CI/CD Pipelines

## Overview
Automated deployment pipelines for frontend, backend, and smart contracts using GitHub Actions.

## Pipeline Architecture

```
Git Push → GitHub Actions → Tests → Build → Deploy
```

## 1. Frontend Pipeline (Next.js)

**File:** `.github/workflows/frontend.yml`

```yaml
name: Frontend CI/CD

on:
  push:
    branches: [main, develop]
    paths:
      - 'frontend/**'
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: cd frontend && npm ci
      - run: cd frontend && npm run lint
      - run: cd frontend && npm run test
      - run: cd frontend && npm run build

  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          scope: ${{ secrets.VERCEL_ORG_ID }}

  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
          scope: ${{ secrets.VERCEL_ORG_ID }}
```

## 2. Backend Pipeline (NestJS)

**File:** `.github/workflows/backend.yml`

```yaml
name: Backend CI/CD

on:
  push:
    branches: [main, develop]
    paths:
      - 'backend/**'
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: token_prop_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: cd backend && npm ci
      - run: cd backend && npm run lint
      - run: cd backend && npm run test
      - run: cd backend && npm run test:e2e

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: docker/setup-buildx-action@v2
      - uses: docker/login-action@v2
        with:
          registry: ${{ secrets.AWS_ECR_REGISTRY }}
          username: ${{ secrets.AWS_ACCESS_KEY_ID }}
          password: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      - uses: docker/build-push-action@v4
        with:
          context: ./backend
          push: true
          tags: ${{ secrets.AWS_ECR_REGISTRY }}/token-prop-backend:${{ github.sha }}

  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - run: |
          aws ecs update-service \
            --cluster token-prop-staging \
            --service backend \
            --force-new-deployment

  deploy-production:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://api.tokenprop.com
    steps:
      - uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - run: |
          aws ecs update-service \
            --cluster token-prop-production \
            --service backend \
            --force-new-deployment
```

## 3. Smart Contracts Pipeline

**File:** `.github/workflows/contracts.yml`

```yaml
name: Smart Contracts CI/CD

on:
  push:
    branches: [main, develop]
    paths:
      - 'contracts/**'
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: cd contracts && npm ci
      - run: cd contracts && npm run compile
      - run: cd contracts && npm run test
      - run: cd contracts && npm run coverage

  security-analysis:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: cd contracts && npm ci
      - run: cd contracts && npx slither .
      - run: cd contracts && npx mythril analyze contracts/*.sol

  deploy-testnet:
    needs: [test, security-analysis]
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: cd contracts && npm ci
      - run: |
          cd contracts && npx hardhat run scripts/deploy.ts \
            --network mumbai
        env:
          PRIVATE_KEY: ${{ secrets.TESTNET_PRIVATE_KEY }}
          ALCHEMY_API_KEY: ${{ secrets.ALCHEMY_API_KEY }}

  deploy-mainnet:
    needs: [test, security-analysis]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: mainnet
    steps:
      - uses: actions/checkout@v3
      - run: cd contracts && npm ci
      - run: |
          cd contracts && npx hardhat run scripts/deploy.ts \
            --network polygon
        env:
          PRIVATE_KEY: ${{ secrets.MAINNET_PRIVATE_KEY }}
          ALCHEMY_API_KEY: ${{ secrets.ALCHEMY_API_KEY }}
```

## 4. Database Migrations

**File:** `.github/workflows/migrations.yml`

```yaml
name: Database Migrations

on:
  push:
    branches: [main]
    paths:
      - 'backend/prisma/**'

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: cd backend && npm ci
      - run: cd backend && npx prisma migrate deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

## Environment Variables

### GitHub Secrets

**Frontend:**
- `VERCEL_TOKEN`
- `VERCEL_ORG_ID`
- `VERCEL_PROJECT_ID`

**Backend:**
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_ECR_REGISTRY`
- `DATABASE_URL`

**Smart Contracts:**
- `TESTNET_PRIVATE_KEY`
- `MAINNET_PRIVATE_KEY`
- `ALCHEMY_API_KEY`
- `POLYGONSCAN_API_KEY`

## Deployment Environments

### Staging
- **Branch:** `develop`
- **Frontend:** Vercel preview
- **Backend:** AWS ECS staging cluster
- **Database:** RDS staging instance
- **Blockchain:** Polygon Mumbai testnet

### Production
- **Branch:** `main`
- **Frontend:** Vercel production
- **Backend:** AWS ECS production cluster
- **Database:** RDS production instance
- **Blockchain:** Polygon mainnet

## Deployment Process

### 1. Feature Development
```bash
git checkout -b feature/new-feature
# Make changes
git commit -m "Add new feature"
git push origin feature/new-feature
# Create PR to develop
```

### 2. Staging Deployment
```bash
# Merge PR to develop
git checkout develop
git pull
# Automatic deployment to staging
```

### 3. Production Deployment
```bash
# Create release PR from develop to main
# Review and approve
# Merge to main
# Automatic deployment to production
```

## Rollback Strategy

### Frontend (Vercel)
```bash
vercel rollback <deployment-url>
```

### Backend (AWS ECS)
```bash
aws ecs update-service \
  --cluster token-prop-production \
  --service backend \
  --task-definition backend:previous-version
```

### Database (Prisma)
```bash
# Rollback migration
npx prisma migrate resolve --rolled-back <migration-name>
```

### Smart Contracts
- Use proxy pattern for upgradeable contracts
- Deploy new implementation
- Update proxy to point to previous implementation if needed

## Monitoring & Alerts

### Post-Deployment Checks
1. Health check endpoints
2. Smoke tests
3. Error rate monitoring (Sentry)
4. Performance metrics (CloudWatch)

### Alerts
- Deployment failures → Slack notification
- Test failures → Block deployment
- High error rate → Auto-rollback

## Best Practices

1. **Never commit secrets** - Use GitHub Secrets
2. **Run tests before deploy** - Automated in pipeline
3. **Use staging environment** - Test before production
4. **Monitor deployments** - Set up alerts
5. **Have rollback plan** - Document and test
6. **Database migrations** - Always backward compatible
7. **Smart contract audits** - Before mainnet deployment
