# Technology Stack

## Overview
This document outlines the complete technology stack for the Real Estate Tokenization Platform, including blockchain, backend, frontend, and infrastructure components.

---

## Blockchain Layer

### Network
**Primary: Polygon (Recommended)**
- Low transaction fees (~$0.01 per transaction)
- EVM-compatible (Ethereum tooling works)
- Mature ecosystem with good documentation
- Fast block times (~2 seconds)
- Strong institutional adoption

**Alternative: Base**
- Coinbase-backed L2 on Ethereum
- Growing ecosystem
- Similar low fees
- Good for US-focused projects

### Token Standard
**ERC-3643 (T-REX Protocol)** ✅ CONFIRMED
- Purpose-built for security tokens
- Built-in compliance layer (Identity Registry + Compliance Rules)
- Transfer restrictions enforced at smart contract level
- Regulatory-friendly and battle-tested
- Used by Tokeny and major security token platforms

**Why ERC-3643 over alternatives:**
- ❌ ERC-20: No compliance features
- ❌ ERC-1155: Would require custom compliance layer
- ✅ ERC-3643: Industry standard for tokenized securities

### Smart Contract Suite

```
Contracts:
├── Identity Registry (shared across all properties)
│   └── Maps wallet addresses → verified identities
├── Compliance Contract (shared, reusable rules)
│   └── Transfer restrictions, investor limits, lockup periods
├── Property Registry Contract
│   └── Maps propertyId → token contract address
├── Property Token Contracts (ERC-3643, one per property)
│   └── Represents fractional ownership of specific property
├── Dividend Distribution Contract
│   └── Receives USDC, snapshots holders, distributes proportionally
├── Marketplace Contract
│   └── Executes trades with compliance checks
└── Treasury/Escrow Contract
    └── Holds funds before distribution, manages reserves
```

### Web3 Infrastructure
- **RPC Provider**: Alchemy (primary) or Infura (backup)
- **Web3 Library**: ethers.js v6
- **Wallet Integration**: 
  - MetaMask
  - WalletConnect v2
  - Coinbase Wallet

### Stablecoin
- **USDC (USD Coin)** - Circle-issued
- Used for: rent payments, dividends, token trading
- Native on Polygon, easy fiat on/off-ramp via Circle API

---

## Backend Stack

### Framework
**NestJS (Node.js + TypeScript)**
- Modular architecture
- Built-in dependency injection
- Strong typing with TypeScript
- Excellent for microservices
- Great testing support

### Database
**PostgreSQL 15+**
- ACID compliance for financial data
- Strong relational model for complex queries
- JSON support for flexible metadata
- Excellent performance and reliability

### ORM
**Prisma**
- Type-safe database client
- Auto-generated types from schema
- Migration management
- Great developer experience

### Backend Modules

```typescript
Backend Architecture:
├── AuthModule
│   ├── Session-based authentication
│   ├── JWT tokens (optional for API access)
│   ├── Password hashing (bcrypt)
│   └── Optional Web3 signature login
├── KYCModule
│   ├── Sumsub integration (primary)
│   ├── Persona integration (alternative)
│   ├── Identity verification workflow
│   └── Sync KYC status → Identity Registry (blockchain)
├── PropertyModule
│   ├── Property CRUD operations
│   ├── Property lifecycle state machine
│   ├── Document management (S3/Firebase)
│   └── Property valuation tracking
├── TokenModule
│   ├── Token minting (deploy ERC-3643 contracts)
│   ├── Token burning (property exit)
│   └── Supply management
├── MarketplaceModule
│   ├── Order book management (PostgreSQL)
│   ├── Order matching engine
│   ├── Trade execution (on-chain)
│   └── Price discovery and analytics
├── DividendModule
│   ├── Dividend calculation workflow
│   ├── Income statement preparation
│   ├── Distribution triggering
│   └── Payout tracking
├── RentModule
│   ├── Rent invoice generation
│   ├── Payment link creation (Stripe)
│   ├── Payment reconciliation
│   └── Tenant payment tracking
├── PaymentsModule
│   ├── Stripe integration (fiat payments)
│   ├── Circle API (fiat ↔ USDC conversion)
│   ├── Payment queue with retry logic
│   └── Transaction history
├── BlockchainModule
│   ├── Web3 provider management
│   ├── Contract interaction layer
│   ├── Event listener service (watches blockchain events)
│   ├── Sync service (reconciles DB with blockchain)
│   └── Transaction queue (retry failed txns)
└── ComplianceModule
    ├── Transfer validation
    ├── Accredited investor checks
    ├── Geographic restrictions
    └── Regulatory reporting
```

### API Architecture
**RESTful API**
- Standard HTTP methods (GET, POST, PUT, DELETE)
- JSON request/response format
- API versioning (/api/v1/...)
- Rate limiting and throttling

**Authentication**
- Session-based (cookies)
- Sessions stored in PostgreSQL
- Token ID hashed and stored in httpOnly cookies
- CSRF protection

---

## Frontend Stack

### Framework
**Next.js 14+ (App Router)**
- React-based with server components
- File-based routing
- API routes for BFF pattern
- Excellent SEO and performance
- TypeScript support

### UI Framework
**Modern Component Library**
- **shadcn/ui** - Accessible, customizable components
- **Radix UI** - Unstyled, accessible primitives
- **TailwindCSS** - Utility-first styling
- **Lucide Icons** - Modern icon library

### State Management
- **React Context** - For global app state
- **TanStack Query (React Query)** - Server state management
- **Zustand** - Lightweight client state (if needed)

### Web3 Integration
- **wagmi** - React hooks for Ethereum
- **viem** - TypeScript Ethereum library
- **RainbowKit/ConnectWallet** - Wallet connection UI

### Frontend Pages/Dashboards

```
Application Structure:
├── Public Pages
│   ├── Landing page
│   ├── Property marketplace (browse)
│   ├── Login/Register
│   └── About/Legal pages
├── Investor Dashboard
│   ├── Portfolio overview
│   ├── Property details
│   ├── Buy/Sell tokens (marketplace)
│   ├── Transaction history
│   ├── Dividend history
│   └── Wallet management
├── Property Manager Dashboard
│   ├── Property listings (create/edit)
│   ├── Property details view
│   ├── Investor list for owned properties
│   ├── Rent payment link generation
│   └── Transaction history
├── Admin Dashboard
│   ├── Property approval workflow
│   ├── Tokenization trigger
│   ├── Dividend distribution trigger
│   ├── User management
│   ├── System analytics
│   └── Compliance monitoring
└── Tenant Portal
    ├── Rent payment page
    ├── Payment history
    └── Property details
```

---

## Payment Infrastructure

### Fiat Payments
**Stripe/Moonpay**
- Rent payments from tenants
- Payment links (one-time use)
- Webhook/(FE-BE onSuccess confirmation) integration for payment confirmation
- PCI-compliant

### Fiat ↔ Crypto Bridge
**Circle API**
- USDC minting/redemption
- Fiat → USDC conversion
- USDC → Fiat conversion
- Bank account integration
- Wire transfer support

### Payment Flow
```
Tenant Rent Payment:
1. Tenant receives Stripe payment link
2. Tenant pays in fiat (USD/EUR)
3. Stripe webhook → Backend confirms payment
4. Backend converts fiat → USDC via Circle
5. Backend sends USDC to Dividend Contract
6. Dividend Contract distributes to token holders

Investor Withdrawal:
1. Investor requests fiat withdrawal
2. Backend converts USDC → fiat via Circle
3. Backend initiates bank transfer
4. Investor receives fiat in bank account
```

---

## Infrastructure & DevOps

### Hosting
**Backend:**
- **Primary**: AWS (EC2, ECS, or Lambda)
- **Alternative**: Railway, Render, or DigitalOcean

**Frontend:**
- **Vercel** (recommended for Next.js)
- **Alternative**: Netlify, AWS Amplify

**Database:**
- **AWS RDS** (PostgreSQL managed service)
- **Alternative**: Supabase, Neon

### File Storage
**Property Documents:**
- **AWS S3** (primary)
- **Alternative**: Firebase Storage
- Store: property deeds, appraisals, insurance docs

### Monitoring & Logging
- **Sentry** - Error tracking
- **LogRocket** - Session replay (frontend)
- **CloudWatch** - AWS infrastructure monitoring
- **Datadog** - Full-stack observability (optional)
- **PostHog** (optional)

### Security
- **SSL/TLS** - HTTPS everywhere
- **WAF** - Web Application Firewall (AWS WAF or Cloudflare)
- **DDoS Protection** - Cloudflare
- **Secret Management** - AWS Secrets Manager or HashiCorp Vault
- **Rate Limiting** - API throttling

---

## Development Tools

### Version Control
- **Git** + **GitHub**
- Branch protection rules
- Pull request reviews

### Testing
**Backend:**
- **Jest** - Unit tests
- **Supertest** - API integration tests
- **Hardhat** - Smart contract testing

**Frontend:**
- **Jest** - Unit tests
- **React Testing Library** - Component tests
- **Playwright** - E2E tests

**Smart Contracts:**
- **Hardhat** - Development environment
- **Waffle** - Contract testing
- **Slither** - Security analysis
- **Mythril** - Vulnerability scanner

### Code Quality
- **ESLint** - Linting
- **Prettier** - Code formatting
- **Husky** - Git hooks
- **lint-staged** - Pre-commit checks

### CI/CD
- **GitHub Actions** (recommended)
- **Alternative**: GitLab CI, CircleCI

---

## Third-Party Services

### KYC/AML
- **Sumsub** (primary) - Identity verification
- **Persona** (alternative)
- **ComplyAdvantage** - AML screening (optional)

### Communication
- **MailSender** or **SendGrid** or **AWS SES** - Transactional emails
- **Twilio** - SMS notifications (optional)

### Analytics
- **Mixpanel** or **Amplitude** - Product analytics
- **Google Analytics** - Web analytics

---

## Marketplace Architecture

### Approach: Hybrid Order Book ✅ CONFIRMED
**Off-chain matching, On-chain settlement**

**Phase 1 (MVP):**
- Orders stored in PostgreSQL
- Matching engine runs in backend
- Compliance validation before execution
- Trade execution on-chain via Marketplace Contract

**Benefits:**
- ✅ Fast order posting (no gas fees)
- ✅ Complex order types (limit, market, stop-loss)
- ✅ Easy to modify matching logic
- ✅ Lower costs for users
- ✅ Compliance checks before execution

**Phase 2 (Future):**
- Add on-chain order book for transparency
- Hybrid approach: users choose speed vs trustlessness

**Why NOT AMM:**
- ❌ Can't enforce KYC on liquidity providers
- ❌ High slippage for illiquid assets
- ❌ Regulatory risk for securities

---

## Property Lifecycle States ✅ APPROVED

```
State Machine:
DRAFT
  ↓ (Property Manager submits)
PENDING_APPROVAL
  ↓ (Admin reviews documents)
APPROVED
  ↓ (Admin triggers tokenization)
TOKENIZED
  ↓ (Investors buy tokens)
FUNDED
  ↓ (Minimum investment reached)
ACTIVE
  ↓ (Generating rent, distributing dividends)
LIQUIDATING
  ↓ (Property being sold)
CLOSED
  (Tokens burned, investors paid out)
```

---

## Summary

This tech stack provides:
- ✅ **Compliance-first** approach with ERC-3643
- ✅ **Scalable** architecture with NestJS + PostgreSQL
- ✅ **Modern** frontend with Next.js + TailwindCSS
- ✅ **Secure** payment flows with Stripe + Circle
- ✅ **Reliable** blockchain integration with Polygon
- ✅ **Production-ready** infrastructure with AWS

All technology choices are battle-tested, well-documented, and suitable for a regulated financial platform.
