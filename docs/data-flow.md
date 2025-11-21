# Data Flow Diagrams

## Overview
This document provides detailed sequence diagrams for all major workflows in the platform.

---

## 1. User Registration & KYC Flow

```
User                Frontend            Backend             Sumsub              Blockchain
  |                    |                   |                   |                      |
  | Register           |                   |                   |                      |
  |------------------->|                   |                   |                      |
  |                    | POST /auth/register                   |                      |
  |                    |------------------>|                   |                      |
  |                    |                   | Create user       |                      |
  |                    |                   | Hash password     |                      |
  |                    |                   | Store in DB       |                      |
  |                    |<------------------|                   |                      |
  |<-------------------|                   |                   |                      |
  | Email sent         |                   |                   |                      |
  |                    |                   |                   |                      |
  | Verify email       |                   |                   |                      |
  |------------------->|                   |                   |                      |
  |                    | POST /auth/verify |                   |                      |
  |                    |------------------>|                   |                      |
  |                    |<------------------|                   |                      |
  |<-------------------|                   |                   |                      |
  |                    |                   |                   |                      |
  | Start KYC          |                   |                   |                      |
  |------------------->|                   |                   |                      |
  |                    | POST /kyc/initiate|                   |                      |
  |                    |------------------>|                   |                      |
  |                    |                   | Create session    |                      |
  |                    |                   |------------------>|                      |
  |                    |                   |<------------------|                      |
  |                    |<------------------|                   |                      |
  |<-------------------|                   |                   |                      |
  | Sumsub iframe      |                   |                   |                      |
  |                    |                   |                   |                      |
  | Upload documents   |                   |                   |                      |
  |--------------------------------------------------->|                      |
  |                    |                   |                   |                      |
  |                    |                   | Webhook: Approved |                      |
  |                    |                   |<------------------|                      |
  |                    |                   | Update KYC status |                      |
  |                    |                   | Sync to blockchain|                      |
  |                    |                   |------------------------------------->|
  |                    |                   |                   |                      | Register
  |                    |                   |                   |                      | Identity
  |                    |                   |<-------------------------------------|
  |                    |                   | Notify user       |                      |
  |<---------------------------------------|                   |                      |
  | KYC Approved       |                   |                   |                      |
```

---

## 2. Property Creation & Tokenization Flow

```
Prop Manager        Frontend            Backend             AWS S3              Blockchain
  |                    |                   |                   |                      |
  | Create property    |                   |                   |                      |
  |------------------->|                   |                   |                      |
  |                    | POST /properties  |                   |                      |
  |                    |------------------>|                   |                      |
  |                    |                   | Create (DRAFT)    |                      |
  |                    |                   | Store in DB       |                      |
  |                    |<------------------|                   |                      |
  |<-------------------|                   |                   |                      |
  |                    |                   |                   |                      |
  | Upload documents   |                   |                   |                      |
  |------------------->|                   |                   |                      |
  |                    | POST /properties/:id/documents        |                      |
  |                    |------------------>|                   |                      |
  |                    |                   | Upload to S3      |                      |
  |                    |                   |------------------>|                      |
  |                    |                   |<------------------|                      |
  |                    |                   | Store URL in DB   |                      |
  |                    |<------------------|                   |                      |
  |<-------------------|                   |                   |                      |
  |                    |                   |                   |                      |
  | Submit for approval|                   |                   |                      |
  |------------------->|                   |                   |                      |
  |                    | POST /properties/:id/submit           |                      |
  |                    |------------------>|                   |                      |
  |                    |                   | Update status     |                      |
  |                    |                   | (PENDING_APPROVAL)|                      |
  |                    |                   | Notify admin      |                      |
  |                    |<------------------|                   |                      |
  |<-------------------|                   |                   |                      |

Admin                  |                   |                   |                      |
  |                    |                   |                   |                      |
  | Review property    |                   |                   |                      |
  |------------------->|                   |                   |                      |
  |                    | GET /properties/:id                   |                      |
  |                    |------------------>|                   |                      |
  |                    |                   | Fetch from DB     |                      |
  |                    |                   | Fetch docs from S3|                      |
  |                    |<------------------|                   |                      |
  |<-------------------|                   |                   |                      |
  |                    |                   |                   |                      |
  | Approve property   |                   |                   |                      |
  |------------------->|                   |                   |                      |
  |                    | POST /properties/:id/approve          |                      |
  |                    |------------------>|                   |                      |
  |                    |                   | Update status     |                      |
  |                    |                   | (APPROVED)        |                      |
  |                    |<------------------|                   |                      |
  |<-------------------|                   |                   |                      |
  |                    |                   |                   |                      |
  | Trigger tokenization                   |                   |                      |
  |------------------->|                   |                   |                      |
  |                    | POST /tokens/mint/:propertyId         |                      |
  |                    |------------------>|                   |                      |
  |                    |                   | Deploy ERC-3643   |                      |
  |                    |                   |------------------------------------->|
  |                    |                   |                   |                      | Deploy
  |                    |                   |                   |                      | contract
  |                    |                   |<-------------------------------------|
  |                    |                   | Store contract addr                      |
  |                    |                   | Update status     |                      |
  |                    |                   | (TOKENIZED)       |                      |
  |                    |<------------------|                   |                      |
  |<-------------------|                   |                   |                      |
  | Tokenization complete                  |                   |                      |
```

---

## 3. Token Purchase Flow

```
Investor            Frontend            Backend             Blockchain
  |                    |                   |                      |
  | Browse properties  |                   |                      |
  |------------------->|                   |                      |
  |                    | GET /properties   |                      |
  |                    |------------------>|                      |
  |                    |<------------------|                      |
  |<-------------------|                   |                      |
  |                    |                   |                      |
  | View property details                  |                      |
  |------------------->|                   |                      |
  |                    | GET /properties/:id                      |
  |                    |------------------>|                      |
  |                    |<------------------|                      |
  |<-------------------|                   |                      |
  |                    |                   |                      |
  | Click "Buy Tokens" |                   |                      |
  |------------------->|                   |                      |
  |                    | Check KYC status  |                      |
  |                    |------------------>|                      |
  |                    |<------------------|                      |
  |                    | KYC approved      |                      |
  |                    |                   |                      |
  | Connect wallet     |                   |                      |
  |------------------->|                   |                      |
  |                    | Request connection|                      |
  |                    |------------------------------------->|
  |<-------------------------------------------------|
  | Wallet connected   |                   |                      |
  |                    |                   |                      |
  | Approve USDC spend |                   |                      |
  |------------------->|                   |                      |
  |                    | approve(spender, amount)                 |
  |                    |------------------------------------->|
  |                    |                   |                      | Tx submitted
  |                    |                   |                      | Tx confirmed
  |                    |<-------------------------------------|
  |<-------------------|                   |                      |
  | Approval confirmed |                   |                      |
  |                    |                   |                      |
  | Confirm purchase   |                   |                      |
  |------------------->|                   |                      |
  |                    | POST /marketplace/buy                    |
  |                    |------------------>|                      |
  |                    |                   | Validate compliance  |
  |                    |                   | Execute transfer     |
  |                    |                   |--------------------->|
  |                    |                   |                      | transferFrom
  |                    |                   |                      | Compliance
  |                    |                   |                      | checks
  |                    |                   |<---------------------|
  |                    |                   | Listen Transfer event|
  |                    |                   | Update DB            |
  |                    |<------------------|                      |
  |<-------------------|                   |                      |
  | Purchase successful|                   |                      |
```

---

## 4. Rent Payment → Dividend Distribution Flow

```
Tenant    Stripe    Backend    Circle    Blockchain    Investor
  |         |          |          |            |            |
  | Click   |          |          |            |            |
  | payment |          |          |            |            |
  | link    |          |          |            |            |
  |-------->|          |          |            |            |
  |         | Checkout |          |            |            |
  |<--------|          |          |            |            |
  |         |          |          |            |            |
  | Pay     |          |          |            |            |
  | $1000   |          |          |            |            |
  |-------->|          |          |            |            |
  |         | Webhook  |          |            |            |
  |         |--------->|          |            |            |
  |         |          | Verify   |            |            |
  |         |          | payment  |            |            |
  |         |          | Store in |            |            |
  |         |          | DB       |            |            |
  |         |          |          |            |            |
  |         |          | Convert  |            |            |
  |         |          | $1000→   |            |            |
  |         |          | USDC     |            |            |
  |         |          |--------->|            |            |
  |         |          |          | Mint USDC  |            |
  |         |          |<---------|            |            |
  |         |          | Receive  |            |            |
  |         |          | 1000 USDC|            |            |
  |         |          |          |            |            |

Admin                  |          |            |            |
  |                    |          |            |            |
  | Review income      |          |            |            |
  | statement          |          |            |            |
  |------------------->|          |            |            |
  |                    | Calculate|            |            |
  |                    | net income           |            |
  |                    | (deduct  |            |            |
  |                    | fees)    |            |            |
  |<-------------------|          |            |            |
  |                    |          |            |            |
  | Trigger dividend   |          |            |            |
  |------------------->|          |            |            |
  |                    | Call     |            |            |
  |                    | Dividend |            |            |
  |                    | Contract |            |            |
  |                    |---------------------->|            |
  |                    |          |            | Snapshot   |
  |                    |          |            | holders    |
  |                    |          |            | Distribute |
  |                    |          |            | USDC       |
  |                    |          |            |----------->|
  |                    |          |            |            | Receive
  |                    |          |            |            | dividend
  |                    |<----------------------|            |
  |                    | Listen   |            |            |
  |                    | events   |            |            |
  |                    | Update DB|            |            |
  |<-------------------|          |            |            |
  | Distribution       |          |            |            |
  | complete           |          |            |            |
```

---

## 5. Marketplace Trading Flow (Off-chain Order Book)

```
Seller              Frontend            Backend             Blockchain
  |                    |                   |                      |
  | Post SELL order    |                   |                      |
  | (100 @ $10.50)     |                   |                      |
  |------------------->|                   |                      |
  |                    | POST /marketplace/orders                 |
  |                    |------------------>|                      |
  |                    |                   | Validate seller      |
  |                    |                   | has tokens           |
  |                    |                   |--------------------->|
  |                    |                   |<---------------------|
  |                    |                   | Store order in DB    |
  |                    |<------------------|                      |
  |<-------------------|                   |                      |
  | Order posted       |                   |                      |

Buyer                  |                   |                      |
  |                    |                   |                      |
  | Post BUY order     |                   |                      |
  | (50 @ $10.50)      |                   |                      |
  |------------------->|                   |                      |
  |                    | POST /marketplace/orders                 |
  |                    |------------------>|                      |
  |                    |                   | Validate buyer       |
  |                    |                   | has USDC             |
  |                    |                   |--------------------->|
  |                    |                   |<---------------------|
  |                    |                   |                      |
  |                    |                   | Matching engine      |
  |                    |                   | finds match          |
  |                    |                   |                      |
  |                    |                   | Validate compliance  |
  |                    |                   | (KYC, accredited)    |
  |                    |                   |                      |
  |                    |                   | Execute trade        |
  |                    |                   |--------------------->|
  |                    |                   |                      | Transfer
  |                    |                   |                      | 50 tokens
  |                    |                   |                      | Transfer
  |                    |                   |                      | 525 USDC
  |                    |                   |<---------------------|
  |                    |                   | Listen events        |
  |                    |                   | Update orders table  |
  |                    |                   | Record trade         |
  |                    |<------------------|                      |
  |<-------------------|                   |                      |
  | Trade confirmed    |                   |                      |

Seller                 |                   |                      |
  |<---------------------------------------|                      |
  | Notification:      |                   |                      |
  | 50 tokens sold     |                   |                      |
  | for 525 USDC       |                   |                      |
```

---

## 6. Property Exit & Token Burning Flow

```
Admin               Frontend            Backend             Blockchain          Investors
  |                    |                   |                      |                  |
  | Initiate property  |                   |                      |                  |
  | exit               |                   |                      |                  |
  |------------------->|                   |                      |                  |
  |                    | POST /properties/:id/liquidate           |                  |
  |                    |------------------>|                      |                  |
  |                    |                   | Update status        |                  |
  |                    |                   | (LIQUIDATING)        |                  |
  |                    |                   | Notify investors     |                  |
  |                    |                   |--------------------------------------------->|
  |                    |<------------------|                      |                  |
  |<-------------------|                   |                      |                  |
  |                    |                   |                      |                  |
  | Property sold      |                   |                      |                  |
  | Calculate payout   |                   |                      |                  |
  |------------------->|                   |                      |                  |
  |                    | POST /tokens/burn/:tokenId               |                  |
  |                    |------------------>|                      |                  |
  |                    |                   | Calculate per-token  |                  |
  |                    |                   | payout               |                  |
  |                    |                   |                      |                  |
  |                    |                   | Burn tokens          |                  |
  |                    |                   |--------------------->|                  |
  |                    |                   |                      | Burn all         |
  |                    |                   |                      | tokens           |
  |                    |                   |<---------------------|                  |
  |                    |                   |                      |                  |
  |                    |                   | Distribute proceeds  |                  |
  |                    |                   |--------------------->|                  |
  |                    |                   |                      | Send USDC        |
  |                    |                   |                      | to holders       |
  |                    |                   |                      |----------------->|
  |                    |                   |<---------------------|                  |
  |                    |                   | Update DB            |                  |
  |                    |                   | Update status (CLOSED)                  |
  |                    |<------------------|                      |                  |
  |<-------------------|                   |                      |                  |
  | Exit complete      |                   |                      |                  |
```

---

## 7. Event Synchronization Flow

```
Blockchain          Event Listener      Event Queue         Event Processor     Database
  |                      |                   |                      |                  |
  | Transfer event       |                   |                      |                  |
  |--------------------->|                   |                      |                  |
  |                      | Parse event       |                      |                  |
  |                      | Add to queue      |                      |                  |
  |                      |------------------>|                      |                  |
  |                      |                   |                      |                  |
  |                      |                   | Process event        |                  |
  |                      |                   |--------------------->|                  |
  |                      |                   |                      | Validate         |
  |                      |                   |                      | Transform        |
  |                      |                   |                      | Update DB        |
  |                      |                   |                      |----------------->|
  |                      |                   |                      |<-----------------|
  |                      |                   |<---------------------|                  |
  |                      |<------------------|                      |                  |
  |                      |                   |                      |                  |
  | DividendPaid event   |                   |                      |                  |
  |--------------------->|                   |                      |                  |
  |                      | Parse event       |                      |                  |
  |                      | Add to queue      |                      |                  |
  |                      |------------------>|                      |                  |
  |                      |                   | Process event        |                  |
  |                      |                   |--------------------->|                  |
  |                      |                   |                      | Update dividends |                  |
  |                      |                   |                      |----------------->|
  |                      |                   |                      | Notify investors |                  |
  |                      |                   |<---------------------|                  |
  |                      |<------------------|                      |                  |
```

---

## Key Takeaways

1. **Backend as Orchestrator**: Backend coordinates between Web2 (database, Stripe, Circle) and Web3 (blockchain)
2. **Event-Driven Sync**: Blockchain events trigger database updates to maintain consistency
3. **Compliance First**: All token transfers go through compliance checks (KYC, accredited status)
4. **Off-chain Matching**: Orders stored in DB, matched off-chain, executed on-chain for efficiency
5. **Async Processing**: Heavy operations (dividend distribution, tokenization) use job queues
