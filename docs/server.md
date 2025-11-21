# Backend Server (NestJS)

## Overview
The backend is built with NestJS, providing a modular REST API that orchestrates business logic, database operations, blockchain interactions, and external service integrations.

## Project Structure

```
src/
├── main.ts
├── app.module.ts
├── config/
│   ├── database.config.ts
│   ├── blockchain.config.ts
│   └── env.validation.ts
├── common/
│   ├── guards/
│   ├── decorators/
│   ├── filters/
│   └── interceptors/
├── modules/
│   ├── auth/
│   ├── kyc/
│   ├── property/
│   ├── token/
│   ├── marketplace/
│   ├── dividend/
│   ├── rent/
│   ├── payments/
│   ├── blockchain/
│   └── compliance/
└── prisma/
    └── schema.prisma
```

## Core Modules

### 1. AuthModule

**Responsibilities:**
- User registration and login
- Session management
- Password hashing and validation
- Optional Web3 signature authentication

**Endpoints:**
```typescript
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
GET    /api/v1/auth/me
POST   /api/v1/auth/web3/nonce
POST   /api/v1/auth/web3/verify
```

**Key Services:**
- `AuthService`: Core authentication logic
- `SessionService`: Session CRUD operations
- `PasswordService`: Hashing and validation

### 2. KYCModule

**Responsibilities:**
- KYC verification workflow
- Sumsub/Persona integration
- Sync KYC status to blockchain Identity Registry
- Accredited investor verification

**Endpoints:**
```typescript
GET    /api/v1/kyc/status
POST   /api/v1/kyc/initiate
POST   /api/v1/kyc/webhook        // Sumsub callback
GET    /api/v1/kyc/accredited
```

**Key Services:**
- `KYCService`: Verification workflow
- `SumsubService`: Sumsub API integration
- `IdentitySyncService`: Sync to blockchain

### 3. PropertyModule

**Responsibilities:**
- Property CRUD operations
- Property lifecycle state management
- Document upload/retrieval
- Property valuation tracking

**Endpoints:**
```typescript
GET    /api/v1/properties
GET    /api/v1/properties/:id
POST   /api/v1/properties          // Property Manager only
PUT    /api/v1/properties/:id      // Property Manager only
DELETE /api/v1/properties/:id      // Admin only
POST   /api/v1/properties/:id/documents
GET    /api/v1/properties/:id/documents
POST   /api/v1/properties/:id/submit      // Submit for approval
POST   /api/v1/properties/:id/approve     // Admin only
POST   /api/v1/properties/:id/reject      // Admin only
```

**Key Services:**
- `PropertyService`: CRUD operations
- `PropertyLifecycleService`: State transitions
- `PropertyDocumentService`: S3 upload/download

### 4. TokenModule

**Responsibilities:**
- Token minting (deploy ERC-3643 contracts)
- Token burning (property exit)
- Supply management
- Token metadata

**Endpoints:**
```typescript
POST   /api/v1/tokens/mint/:propertyId    // Admin only
POST   /api/v1/tokens/burn/:tokenId       // Admin only
GET    /api/v1/tokens/:tokenId
GET    /api/v1/tokens/:tokenId/holders
GET    /api/v1/tokens/:tokenId/supply
```

**Key Services:**
- `TokenService`: Token operations
- `MintingService`: Deploy ERC-3643 contracts
- `BurningService`: Burn tokens and payout

### 5. MarketplaceModule

**Responsibilities:**
- Order book management
- Order matching engine
- Trade execution
- Price discovery

**Endpoints:**
```typescript
GET    /api/v1/marketplace/orders
POST   /api/v1/marketplace/orders
DELETE /api/v1/marketplace/orders/:id
GET    /api/v1/marketplace/trades
GET    /api/v1/marketplace/orderbook/:tokenId
GET    /api/v1/marketplace/price/:tokenId
```

**Key Services:**
- `OrderService`: Order CRUD
- `MatchingEngineService`: Match buy/sell orders
- `TradeExecutionService`: Execute on-chain

### 6. DividendModule

**Responsibilities:**
- Dividend calculation
- Income statement preparation
- Distribution triggering
- Payout tracking

**Endpoints:**
```typescript
GET    /api/v1/dividends/property/:propertyId
POST   /api/v1/dividends/calculate/:propertyId  // Admin only
POST   /api/v1/dividends/distribute/:id         // Admin only
GET    /api/v1/dividends/investor/:userId
```

**Key Services:**
- `DividendService`: Distribution workflow
- `CalculationService`: Net income calculation
- `DistributionService`: On-chain distribution

### 7. RentModule

**Responsibilities:**
- Rent invoice generation
- Payment link creation
- Payment reconciliation
- Tenant payment tracking

**Endpoints:**
```typescript
POST   /api/v1/rent/payment-link/:propertyId
GET    /api/v1/rent/payment-link/:linkId
POST   /api/v1/rent/payment/:linkId
GET    /api/v1/rent/payments/:propertyId
DELETE /api/v1/rent/payment-link/:linkId
```

**Key Services:**
- `RentService`: Rent operations
- `PaymentLinkService`: Generate/revoke links
- `ReconciliationService`: Match payments

### 8. PaymentsModule

**Responsibilities:**
- Stripe integration (fiat payments)
- Circle integration (fiat ↔ USDC)
- Payment queue with retry logic
- Transaction history

**Endpoints:**
```typescript
POST   /api/v1/payments/stripe/webhook
POST   /api/v1/payments/circle/convert
GET    /api/v1/payments/transactions
```

**Key Services:**
- `StripeService`: Stripe API integration
- `CircleService`: Circle API integration
- `PaymentQueueService`: Retry failed payments

### 9. BlockchainModule

**Responsibilities:**
- Web3 provider management
- Contract interaction layer
- Event listener service
- Sync service (DB ↔ blockchain)
- Transaction queue

**Endpoints:**
```typescript
GET    /api/v1/blockchain/transaction/:txHash
GET    /api/v1/blockchain/balance/:address
POST   /api/v1/blockchain/sync/:contractAddress
```

**Key Services:**
- `Web3Service`: Provider management
- `ContractService`: Contract interactions
- `EventListenerService`: Watch blockchain events
- `SyncService`: Reconcile DB with blockchain
- `TransactionQueueService`: Retry failed txns

### 10. ComplianceModule

**Responsibilities:**
- Transfer validation
- Accredited investor checks
- Geographic restrictions
- Regulatory reporting

**Endpoints:**
```typescript
POST   /api/v1/compliance/validate-transfer
GET    /api/v1/compliance/rules/:tokenId
GET    /api/v1/compliance/reports
```

**Key Services:**
- `ComplianceService`: Compliance logic
- `TransferValidatorService`: Pre-transfer checks
- `ReportingService`: Generate reports

## API Response Format

**Success Response:**
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation successful"
}
```

**Error Response:**
```json
{
  "success": false,
  "error": {
    "code": "PROPERTY_NOT_FOUND",
    "message": "Property with ID xyz not found",
    "details": { ... }
  }
}
```

## Authentication & Authorization

### Guards

**AuthGuard:**
```typescript
@UseGuards(AuthGuard)
@Get('/properties')
async getProperties() { ... }
```

**RolesGuard:**
```typescript
@UseGuards(AuthGuard, RolesGuard)
@Roles('ADMIN')
@Post('/properties/:id/approve')
async approveProperty() { ... }
```

**KYCGuard:**
```typescript
@UseGuards(AuthGuard, KYCGuard)
@Post('/marketplace/orders')
async createOrder() { ... }
```

### Decorators

**@CurrentUser:**
```typescript
@Get('/me')
async getProfile(@CurrentUser() user: User) {
  return user;
}
```

**@Roles:**
```typescript
@Roles('ADMIN', 'PROPERTY_MANAGER')
@Post('/properties')
async createProperty() { ... }
```

## Error Handling

**Custom Exceptions:**
```typescript
throw new PropertyNotFoundException(propertyId);
throw new InsufficientBalanceException();
throw new KYCNotApprovedException();
throw new InvalidOrderException();
```

**Global Exception Filter:**
```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    // Log error
    // Format response
    // Send to Sentry
  }
}
```

## Validation

**DTOs with class-validator:**
```typescript
export class CreatePropertyDto {
  @IsString()
  @IsNotEmpty()
  name: string;

  @IsEnum(PropertyType)
  propertyType: PropertyType;

  @IsNumber()
  @Min(0)
  purchasePrice: number;

  @IsInt()
  @Min(1)
  totalTokens: number;
}
```

## Background Jobs

**Bull Queue for async tasks:**
```typescript
// Event processing
@Process('blockchain-events')
async processEvent(job: Job) { ... }

// Dividend distribution
@Process('dividend-distribution')
async distributeDividends(job: Job) { ... }

// Payment conversion
@Process('payment-conversion')
async convertPayment(job: Job) { ... }
```

## Logging

**Winston Logger:**
```typescript
this.logger.log('Property created', { propertyId });
this.logger.warn('KYC verification pending', { userId });
this.logger.error('Blockchain transaction failed', { txHash, error });
```

## Testing

**Unit Tests:**
```bash
npm run test
```

**E2E Tests:**
```bash
npm run test:e2e
```

**Coverage:**
```bash
npm run test:cov
```

## Environment Variables

```env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/token_prop

# Blockchain
BLOCKCHAIN_RPC_URL=https://polygon-mainnet.g.alchemy.com/v2/...
BLOCKCHAIN_PRIVATE_KEY=0x...

# External Services
STRIPE_SECRET_KEY=sk_...
CIRCLE_API_KEY=...
SUMSUB_APP_TOKEN=...
ALCHEMY_API_KEY=...

# AWS
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_S3_BUCKET=token-prop-documents

# Application
PORT=4000
NODE_ENV=production
JWT_SECRET=...
SESSION_SECRET=...
```

## Running the Server

**Development:**
```bash
npm run start:dev
```

**Production:**
```bash
npm run build
npm run start:prod
```

**Docker:**
```bash
docker build -t token-prop-backend .
docker run -p 4000:4000 token-prop-backend
```
