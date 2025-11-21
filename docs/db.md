# Database Schema (PostgreSQL)

## Overview
This document defines the complete PostgreSQL database schema using Prisma ORM syntax.

## Schema Diagram

```
users
  ├─> sessions
  ├─> kyc_verifications
  ├─> properties (as property_manager)
  ├─> token_holders (as investor)
  ├─> orders (as trader)
  ├─> rent_payments (as tenant)
  └─> dividend_payments (as investor)

properties
  ├─> property_documents
  ├─> property_valuations
  ├─> property_tokens
  ├─> rent_payments
  ├─> payment_links
  └─> dividend_distributions

property_tokens
  ├─> token_holders
  ├─> orders
  ├─> trades
  └─> dividend_distributions
```

## Prisma Schema

```prisma
// schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ============================================
// USERS & AUTHENTICATION
// ============================================

enum UserRole {
  ADMIN
  PROPERTY_MANAGER
  INVESTOR
  TENANT
}

enum KYCStatus {
  PENDING
  APPROVED
  REJECTED
  EXPIRED
}

model User {
  id                String   @id @default(uuid())
  email             String   @unique
  passwordHash      String
  role              UserRole
  firstName         String?
  lastName          String?
  walletAddress     String?  @unique
  phoneNumber       String?
  isActive          Boolean  @default(true)
  createdAt         DateTime @default(now())
  updatedAt         DateTime @updatedAt

  // Relations
  sessions              Session[]
  kycVerification       KYCVerification?
  managedProperties     Property[]         @relation("PropertyManager")
  tokenHoldings         TokenHolder[]
  orders                Order[]
  trades                Trade[]            @relation("Trader")
  rentPayments          RentPayment[]
  dividendPayments      DividendPayment[]

  @@index([email])
  @@index([walletAddress])
  @@index([role])
}

model Session {
  id           String   @id @default(uuid())
  userId       String
  sessionToken String   @unique
  expiresAt    DateTime
  createdAt    DateTime @default(now())

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([sessionToken])
  @@index([expiresAt])
}

model KYCVerification {
  id                String    @id @default(uuid())
  userId            String    @unique
  status            KYCStatus @default(PENDING)
  provider          String    // "sumsub" or "persona"
  externalId        String?   // Provider's verification ID
  isAccredited      Boolean   @default(false)
  accreditedUntil   DateTime?
  country           String?
  documentType      String?
  verifiedAt        DateTime?
  rejectionReason   String?
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([status])
}

// ============================================
// PROPERTIES
// ============================================

enum PropertyStatus {
  DRAFT
  PENDING_APPROVAL
  APPROVED
  TOKENIZED
  FUNDED
  ACTIVE
  LIQUIDATING
  CLOSED
}

enum PropertyType {
  RESIDENTIAL
  COMMERCIAL
  INDUSTRIAL
  MIXED_USE
}

model Property {
  id                    String         @id @default(uuid())
  propertyManagerId     String
  status                PropertyStatus @default(DRAFT)
  propertyType          PropertyType
  
  // Basic Info
  name                  String
  description           String
  address               String
  city                  String
  state                 String
  country               String
  zipCode               String
  
  // Financial Info
  purchasePrice         Decimal        @db.Decimal(18, 2)
  currentValuation      Decimal        @db.Decimal(18, 2)
  totalTokens           Int
  tokenPrice            Decimal        @db.Decimal(18, 2)
  minimumInvestment     Decimal        @db.Decimal(18, 2)
  
  // Rental Info
  monthlyRent           Decimal?       @db.Decimal(18, 2)
  annualReturn          Decimal?       @db.Decimal(5, 2) // Percentage
  
  // Distribution
  distributionPeriod    String?        // "MONTHLY", "QUARTERLY", "ANNUALLY"
  managementFeePercent  Decimal        @db.Decimal(5, 2)
  
  // Metadata
  imageUrls             String[]       // Array of S3 URLs
  metadata              Json?          // Flexible JSON field
  
  // Timestamps
  approvedAt            DateTime?
  tokenizedAt           DateTime?
  fundedAt              DateTime?
  closedAt              DateTime?
  createdAt             DateTime       @default(now())
  updatedAt             DateTime       @updatedAt

  // Relations
  propertyManager       User                    @relation("PropertyManager", fields: [propertyManagerId], references: [id])
  documents             PropertyDocument[]
  valuations            PropertyValuation[]
  token                 PropertyToken?
  rentPayments          RentPayment[]
  paymentLinks          PaymentLink[]
  dividendDistributions DividendDistribution[]

  @@index([propertyManagerId])
  @@index([status])
  @@index([propertyType])
}

model PropertyDocument {
  id         String   @id @default(uuid())
  propertyId String
  type       String   // "DEED", "APPRAISAL", "INSURANCE", "LEASE"
  fileName   String
  fileUrl    String   // S3 URL
  fileSize   Int      // Bytes
  uploadedAt DateTime @default(now())

  property Property @relation(fields: [propertyId], references: [id], onDelete: Cascade)

  @@index([propertyId])
}

model PropertyValuation {
  id            String   @id @default(uuid())
  propertyId    String
  valuationDate DateTime
  amount        Decimal  @db.Decimal(18, 2)
  appraiser     String?
  notes         String?
  createdAt     DateTime @default(now())

  property Property @relation(fields: [propertyId], references: [id], onDelete: Cascade)

  @@index([propertyId])
  @@index([valuationDate])
}

// ============================================
// TOKENS
// ============================================

model PropertyToken {
  id                String   @id @default(uuid())
  propertyId        String   @unique
  contractAddress   String   @unique
  tokenSymbol       String
  totalSupply       Int
  circulatingSupply Int      @default(0)
  deployedAt        DateTime @default(now())
  
  // Relations
  property              Property                 @relation(fields: [propertyId], references: [id])
  holders               TokenHolder[]
  orders                Order[]
  trades                Trade[]
  dividendDistributions DividendDistribution[]

  @@index([contractAddress])
  @@index([propertyId])
}

model TokenHolder {
  id              String   @id @default(uuid())
  tokenId         String
  userId          String
  balance         Int
  averageCostBasis Decimal @db.Decimal(18, 2) // For tax purposes
  firstPurchaseAt DateTime @default(now())
  lastUpdatedAt   DateTime @updatedAt

  token PropertyToken @relation(fields: [tokenId], references: [id])
  user  User          @relation(fields: [userId], references: [id])

  @@unique([tokenId, userId])
  @@index([tokenId])
  @@index([userId])
}

// ============================================
// MARKETPLACE
// ============================================

enum OrderType {
  BUY
  SELL
}

enum OrderKind {
  MARKET
  LIMIT
}

enum OrderStatus {
  OPEN
  PARTIALLY_FILLED
  FILLED
  CANCELLED
  EXPIRED
}

model Order {
  id             String      @id @default(uuid())
  tokenId        String
  userId         String
  orderType      OrderType
  orderKind      OrderKind
  quantity       Int
  pricePerToken  Decimal     @db.Decimal(18, 2)
  filledQuantity Int         @default(0)
  status         OrderStatus @default(OPEN)
  expiresAt      DateTime?
  createdAt      DateTime    @default(now())
  updatedAt      DateTime    @updatedAt

  token  PropertyToken @relation(fields: [tokenId], references: [id])
  user   User          @relation(fields: [userId], references: [id])
  trades Trade[]

  @@index([tokenId])
  @@index([userId])
  @@index([status])
  @@index([orderType])
  @@index([createdAt])
}

enum TradeStatus {
  PENDING
  CONFIRMED
  FAILED
}

model Trade {
  id             String      @id @default(uuid())
  tokenId        String
  buyOrderId     String
  sellOrderId    String
  buyerId        String
  sellerId       String
  quantity       Int
  pricePerToken  Decimal     @db.Decimal(18, 2)
  totalAmount    Decimal     @db.Decimal(18, 2)
  txHash         String?     // Blockchain transaction hash
  status         TradeStatus @default(PENDING)
  executedAt     DateTime    @default(now())
  confirmedAt    DateTime?

  token      PropertyToken @relation(fields: [tokenId], references: [id])
  buyOrder   Order         @relation(fields: [buyOrderId], references: [id])
  buyer      User          @relation("Trader", fields: [buyerId], references: [id])

  @@index([tokenId])
  @@index([buyerId])
  @@index([sellerId])
  @@index([status])
  @@index([executedAt])
}

// ============================================
// DIVIDENDS
// ============================================

enum DistributionStatus {
  PENDING
  PROCESSING
  COMPLETED
  FAILED
}

model DividendDistribution {
  id                String             @id @default(uuid())
  propertyId        String
  tokenId           String
  periodStart       DateTime
  periodEnd         DateTime
  totalAmount       Decimal            @db.Decimal(18, 2) // Total USDC to distribute
  rentCollected     Decimal            @db.Decimal(18, 2)
  managementFees    Decimal            @db.Decimal(18, 2)
  platformFees      Decimal            @db.Decimal(18, 2)
  netAmount         Decimal            @db.Decimal(18, 2)
  snapshotBlockNumber Int?
  txHash            String?
  status            DistributionStatus @default(PENDING)
  distributedAt     DateTime?
  createdAt         DateTime           @default(now())

  property Property      @relation(fields: [propertyId], references: [id])
  token    PropertyToken @relation(fields: [tokenId], references: [id])
  payments DividendPayment[]

  @@index([propertyId])
  @@index([tokenId])
  @@index([status])
  @@index([periodEnd])
}

model DividendPayment {
  id             String   @id @default(uuid())
  distributionId String
  userId         String
  tokenBalance   Int      // Snapshot of user's token balance
  amount         Decimal  @db.Decimal(18, 2) // USDC amount
  txHash         String?
  paidAt         DateTime @default(now())

  distribution DividendDistribution @relation(fields: [distributionId], references: [id])
  user         User                 @relation(fields: [userId], references: [id])

  @@index([distributionId])
  @@index([userId])
  @@index([paidAt])
}

// ============================================
// RENT PAYMENTS
// ============================================

enum PaymentStatus {
  PENDING
  COMPLETED
  FAILED
  REFUNDED
}

enum PaymentMethod {
  FIAT
  CRYPTO
}

model RentPayment {
  id              String        @id @default(uuid())
  propertyId      String
  tenantId        String?
  amount          Decimal       @db.Decimal(18, 2)
  currency        String        @default("USD")
  paymentMethod   PaymentMethod
  status          PaymentStatus @default(PENDING)
  
  // Stripe Info
  stripePaymentId String?       @unique
  
  // Circle Info (if converted to USDC)
  usdcAmount      Decimal?      @db.Decimal(18, 2)
  circleTransferId String?
  
  // Metadata
  paymentDate     DateTime
  dueDate         DateTime?
  notes           String?
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt

  property Property @relation(fields: [propertyId], references: [id])
  tenant   User?    @relation(fields: [tenantId], references: [id])

  @@index([propertyId])
  @@index([tenantId])
  @@index([status])
  @@index([paymentDate])
}

model PaymentLink {
  id         String    @id @default(uuid())
  propertyId String
  linkId     String    @unique // Short unique ID for URL
  amount     Decimal   @db.Decimal(18, 2)
  isActive   Boolean   @default(true)
  usedBy     String?   // Tenant user ID
  usedAt     DateTime?
  expiresAt  DateTime?
  createdAt  DateTime  @default(now())

  property Property @relation(fields: [propertyId], references: [id])

  @@index([propertyId])
  @@index([linkId])
  @@index([isActive])
}

// ============================================
// BLOCKCHAIN TRANSACTIONS
// ============================================

enum TransactionType {
  TOKEN_MINT
  TOKEN_BURN
  TOKEN_TRANSFER
  DIVIDEND_DISTRIBUTION
  TRADE_EXECUTION
}

enum TransactionStatus {
  PENDING
  CONFIRMED
  FAILED
}

model BlockchainTransaction {
  id          String            @id @default(uuid())
  txHash      String            @unique
  type        TransactionType
  from        String            // Wallet address
  to          String            // Wallet address
  amount      String?           // Token amount or USDC amount
  gasUsed     String?
  gasPrice    String?
  blockNumber Int?
  status      TransactionStatus @default(PENDING)
  metadata    Json?             // Flexible JSON for additional data
  createdAt   DateTime          @default(now())
  confirmedAt DateTime?

  @@index([txHash])
  @@index([type])
  @@index([status])
  @@index([from])
  @@index([to])
}
```

## Key Relationships

### User Relationships
- **One-to-Many**: User → Sessions
- **One-to-One**: User → KYCVerification
- **One-to-Many**: User (Property Manager) → Properties
- **One-to-Many**: User (Investor) → TokenHoldings
- **One-to-Many**: User (Investor) → Orders
- **One-to-Many**: User (Tenant) → RentPayments

### Property Relationships
- **One-to-Many**: Property → PropertyDocuments
- **One-to-Many**: Property → PropertyValuations
- **One-to-One**: Property → PropertyToken
- **One-to-Many**: Property → RentPayments
- **One-to-Many**: Property → DividendDistributions

### Token Relationships
- **One-to-Many**: PropertyToken → TokenHolders
- **One-to-Many**: PropertyToken → Orders
- **One-to-Many**: PropertyToken → Trades
- **One-to-Many**: PropertyToken → DividendDistributions

## Indexes

Critical indexes for performance:

```sql
-- User lookups
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_wallet ON users(wallet_address);
CREATE INDEX idx_users_role ON users(role);

-- Session validation
CREATE INDEX idx_sessions_token ON sessions(session_token);
CREATE INDEX idx_sessions_expires ON sessions(expires_at);

-- Property queries
CREATE INDEX idx_properties_status ON properties(status);
CREATE INDEX idx_properties_manager ON properties(property_manager_id);

-- Token lookups
CREATE INDEX idx_tokens_contract ON property_tokens(contract_address);
CREATE INDEX idx_token_holders_user ON token_holders(user_id);

-- Marketplace queries
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_token ON orders(token_id);
CREATE INDEX idx_trades_executed ON trades(executed_at);

-- Dividend queries
CREATE INDEX idx_dividends_property ON dividend_distributions(property_id);
CREATE INDEX idx_dividend_payments_user ON dividend_payments(user_id);

-- Blockchain transactions
CREATE INDEX idx_blockchain_tx_hash ON blockchain_transactions(tx_hash);
```

## Migrations

Generate and run migrations:

```bash
# Generate migration
npx prisma migrate dev --name init

# Apply migration to production
npx prisma migrate deploy

# Generate Prisma Client
npx prisma generate
```

## Seed Data

Example seed script for development:

```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client';
import * as bcrypt from 'bcrypt';

const prisma = new PrismaClient();

async function main() {
  // Create Admin user
  const admin = await prisma.user.create({
    data: {
      email: 'admin@tokenprop.com',
      passwordHash: await bcrypt.hash('admin123', 10),
      role: 'ADMIN',
      firstName: 'Admin',
      lastName: 'User',
      isActive: true,
    },
  });

  // Create Property Manager
  const propertyManager = await prisma.user.create({
    data: {
      email: 'manager@tokenprop.com',
      passwordHash: await bcrypt.hash('manager123', 10),
      role: 'PROPERTY_MANAGER',
      firstName: 'Property',
      lastName: 'Manager',
      isActive: true,
    },
  });

  // Create Investor
  const investor = await prisma.user.create({
    data: {
      email: 'investor@tokenprop.com',
      passwordHash: await bcrypt.hash('investor123', 10),
      role: 'INVESTOR',
      firstName: 'Test',
      lastName: 'Investor',
      walletAddress: '0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb',
      isActive: true,
      kycVerification: {
        create: {
          status: 'APPROVED',
          provider: 'sumsub',
          isAccredited: true,
          country: 'US',
          verifiedAt: new Date(),
        },
      },
    },
  });

  console.log({ admin, propertyManager, investor });
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

Run seed:
```bash
npx prisma db seed
```

## Backup & Restore

```bash
# Backup
pg_dump -h <host> -U <user> -d token_prop > backup.sql

# Restore
psql -h <host> -U <user> -d token_prop < backup.sql
```
