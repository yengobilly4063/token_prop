# Infrastructure & Architecture

## High-Level System Architecture

```
                         USERS
                           |
                           v
              +------------------------+
              |   Frontend (Next.js)   |
              |   - Vercel Hosting     |
              +------------------------+
                           |
                           | REST API
                           v
              +------------------------+
              |   Backend (NestJS)     |
              |   - Renbder            |
              +------------------------+
                     |           |
          +----------+           +----------+
          |                                |
          v                                v
  +---------------+              +-------------------+
  |  PostgreSQL   |              |  Blockchain       |
  |  - Render     |              |  - Polygon/Base   |
  +---------------+              +-------------------+
          |
          v
  +---------------+
  | External APIs |
  | - Stripe      |
  | - Circle      |
  | - Sumsub      |
  +---------------+
```

## Component Layers

### 1. Frontend Layer (Next.js on Vercel)
- **Investor Dashboard**: Portfolio, marketplace, transactions
- **Property Manager Dashboard**: Property CRUD, rent links
- **Admin Dashboard**: Approvals, tokenization, dividends
- **Tenant Portal**: Rent payments
- **Wallet Integration**: MetaMask, WalletConnect

### 2. Backend Layer (NestJS on AWS)

**Core Modules:**
```
AuthModule       → Session management, authentication
KYCModule        → Sumsub/Verrif integration, identity verification
PropertyModule   → Property lifecycle, CRUD operations
TokenModule      → Minting, burning, supply management
MarketplaceModule → Order book, matching engine
DividendModule   → Calculation, distribution
RentModule       → Payment links, reconciliation
PaymentsModule   → Stripe, Circle integration
BlockchainModule → Web3 provider, event listeners
ComplianceModule → Transfer validation, reporting
```

### 3. Database Layer (PostgreSQL)

**Schema Groups:**
- **Users & Auth**: users, sessions, kyc_verifications
- **Properties**: properties, property_documents, valuations
- **Tokens**: property_tokens, token_holders
- **Marketplace**: orders, trades
- **Dividends**: distributions, payments
- **Rent**: rent_payments, payment_links
- **Transactions**: blockchain_transactions, fiat_transactions

### 4. Blockchain Layer (Polygon/Base)

**Smart Contracts:**
```
Identity Registry (shared)
  └─> Compliance Contract (shared)
        └─> Property Token 1 (ERC-3643)
        └─> Property Token 2 (ERC-3643)
        └─> Property Token 3 (ERC-3643)
              └─> Dividend Contract
              └─> Marketplace Contract
              └─> Treasury Contract
```

## Key Data Flows

### Flow 1: Property Tokenization
```
1. Property Manager creates property (DRAFT)
2. Uploads documents to S3
3. Submits for approval (PENDING_APPROVAL)
4. Admin reviews and approves (APPROVED)
5. Admin triggers tokenization
6. Backend deploys ERC-3643 contract
7. Contract address stored in DB (TOKENIZED)
8. Property ready for investment
```

### Flow 2: Token Purchase
```
1. Investor browses properties
2. Clicks "Buy Tokens"
3. Backend validates KYC status
4. Investor approves USDC spend (MetaMask)
5. Backend executes transfer on-chain
6. Smart contract enforces compliance
7. Event listener updates DB
8. Investor sees tokens in portfolio
```

### Flow 3: Rent → Dividend
```
1. Tenant pays rent via Stripe ($1000)
2. Stripe webhook confirms payment
3. Backend converts $1000 → USDC via Circle
4. Admin reviews income statement
5. Admin triggers dividend distribution
6. Backend calls Dividend Contract
7. Contract snapshots token holders
8. Contract distributes USDC proportionally
9. Event listener updates DB
10. Investors see USDC in wallets
```

### Flow 4: Marketplace Trading
```
1. Seller posts SELL order (off-chain, stored in DB)
2. Buyer posts BUY order
3. Backend matching engine finds match
4. Backend validates compliance (KYC, accredited)
5. Backend executes trade on-chain
6. Smart contract transfers tokens + USDC
7. Event listener updates DB
8. Both parties notified
```

## Event-Driven Synchronization

**Blockchain Events → Backend:**
```typescript
Transfer Event
  → Update token_holders table
  → Update investor portfolios

DividendDistributed Event
  → Update dividend_payments table
  → Notify investors

TradeExecuted Event
  → Update trades table
  → Update order statuses

PropertyTokenized Event
  → Update properties with contract address
  → Change state to TOKENIZED
```

**Event Processing:**
```
Blockchain → Event Listener → Event Queue (Redis)
  → Event Processor → Database Update → Notifications
```

## Deployment Architecture

### Production Environment
```
Cloudflare (CDN, DDoS, WAF)
  |
  +-- Vercel (Frontend)
  |     └─> Next.js App
  |
  +-- AWS (Backend)
  |     ├─> ECS/EC2 (NestJS)
  |     ├─> RDS (PostgreSQL)
  |     ├─> S3 (Documents)
  |     └─> Redis (Cache/Queue)
  |
  +-- Polygon/Base (Blockchain)
        └─> Alchemy RPC
```

### External Services
- **Stripe**: Fiat rent payments
- **Circle**: Fiat ↔ USDC conversion
- **Sumsub/Persona**: KYC/AML verification
- **Alchemy/Infura**: Blockchain RPC
- **SendGrid**: Email notifications
- **Sentry**: Error tracking

## Security Architecture

### Authentication
- **Session-based**: httpOnly cookies, stored in PostgreSQL
- **CSRF protection**: Token validation
- **Rate limiting**: API throttling
- **Web3 login**: Optional signature-based auth

### Data Security
- **Encryption at rest**: RDS encryption, S3 encryption
- **Encryption in transit**: TLS/SSL everywhere
- **Secret management**: AWS Secrets Manager
- **WAF**: Cloudflare Web Application Firewall

### Smart Contract Security
- **Audits**: Third-party security audits (Certik, OpenZeppelin)
- **Testing**: Comprehensive test coverage (Hardhat)
- **Upgradability**: Proxy pattern for critical contracts
- **Multi-sig**: Admin operations require multiple signatures

## Scaling Strategy

### Horizontal Scaling
- **Backend**: Multiple ECS tasks behind ALB
- **Database**: Read replicas for queries
- **Redis**: Cluster mode

### Caching
- **Redis**: Sessions, property listings, token prices
- **CDN**: Static assets, images
- **Browser**: API response caching

### Performance Optimization
- **Database indexing**: On frequently queried fields
- **Connection pooling**: Prisma connection pool
- **Lazy loading**: Frontend code splitting
- **Image optimization**: Next.js Image component

## Monitoring & Observability

### Application Monitoring
- **Sentry**: Error tracking and alerting
- **CloudWatch**: AWS infrastructure metrics
- **Datadog**: Full-stack observability (optional)

### Blockchain Monitoring
- **Alchemy**: Transaction monitoring
- **Custom alerts**: Failed transactions, gas spikes
- **Event logs**: All smart contract events

### Business Metrics
- **Mixpanel/Amplitude**: User behavior analytics
- **Custom dashboards**: Properties tokenized, trades executed, dividends distributed

## Disaster Recovery

### Backup Strategy
- **Database**: Automated daily backups (RDS)
- **Documents**: S3 versioning enabled
- **Smart contracts**: Immutable on blockchain

### Failover
- **Database**: Multi-AZ deployment
- **Backend**: Auto-scaling with health checks
- **RPC**: Multiple providers (Alchemy + Infura)

### Incident Response
- **Runbooks**: Documented procedures
- **On-call rotation**: 24/7 coverage
- **Post-mortems**: After major incidents

## Development Environments

### Local Development
- **Frontend**: `npm run dev` (localhost:3000)
- **Backend**: `npm run start:dev` (localhost:4000)
- **Database**: Docker PostgreSQL
- **Blockchain**: Hardhat local node

### Staging Environment
- **Frontend**: Vercel preview deployment
- **Backend**: AWS staging environment
- **Database**: Separate RDS instance
- **Blockchain**: Polygon Mumbai testnet

### Production Environment
- **Frontend**: Vercel production
- **Backend**: AWS production
- **Database**: Production RDS
- **Blockchain**: Polygon mainnet

## CI/CD Pipeline

**GitHub Actions Workflow:**
```
1. Code push to branch
2. Run linters (ESLint, Prettier)
3. Run unit tests (Jest)
4. Run integration tests
5. Build Docker image (backend)
6. Push to ECR (AWS)
7. Deploy to staging
8. Run E2E tests (Playwright)
9. Manual approval for production
10. Deploy to production
11. Run smoke tests
```

**Smart Contract Deployment:**
```
1. Write contract code
2. Write tests (Hardhat)
3. Run security analysis (Slither, Mythril)
4. Deploy to testnet
5. Verify on block explorer
6. Third-party audit
7. Deploy to mainnet
8. Verify on block explorer
9. Update backend with contract addresses
```

See `ci-cd-pipelines.md` for detailed pipeline configuration.
