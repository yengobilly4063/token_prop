
### Smart Contract

#### Recommendation: Use ERC-3643 since we're dealing with securities. Document this clearly.

### Smart Contract Architecture:
- Property Token Contract (ERC-3643) (T-REX Protocol)
- Dividend Contract
- Property Registry Contract
- Identity Registry (required for ERC-3643 compliance)
- Compliance Contract (transfer rules, investor limits)
- Marketplace/DEX Contract (for secondary trading)
- Treasury/Escrow Contract (holds funds before distribution)

 ### Backend ↔ Blockchain Sync Strategy
On-chain events are used to update the backend database.
#### Critical scenario for sync of on-chain with db:

- Investor buys tokens directly via MetaMask (bypassing backend)
- Blockchain reorg or failed transaction
- Backend crashes during dividend distribution

##### Recommendation: Add Event Listener Service:

- Implement a separate service that listens to on-chain events
- Use a reliable event listener service (e.g., Etherscan API, Infura)
- Implement retry logic for failed events
- Use a queue to process events in order
- Implement a fallback mechanism for failed events

#### BlockchainModule:
- EventListenerService (watches Transfer, DividendPaid events)
- SyncService (reconciles DB with blockchain state)
- TransactionQueueService (retry failed txns)

### Dividend Distribution Trigger
"Admin triggers dividend" - but what's the actual flow?

#### Missing details:

- Who calculates net distributable income (rent - expenses)?
- How are property management fees deducted?
- What if rent is late or partial?
- What is the snapshot timing (when is token holder list frozen)?

#### Recommendation: Document Dividend Workflow:
1. Rent collected in Stripe → webhook → RentModule
2. RentModule records payment in DB
3. Admin reviews monthly income statement
4. Admin triggers "Prepare Dividend" → calculates net income
5. Backend converts FIAT → USDC via Circle
6. Backend calls DividendContract.distribute(propertyId, amount)
7. Smart contract snapshots token holders
8. Smart contract distributes proportionally
9. EventListener updates DB with distribution records

###  Marketplace Logic Ambiguity
For MVP, we should implement "Marketplace logic" in smart contracts.

Two approaches:
- Order Book DEX: Buyers/sellers post orders, backend matches
- AMM Pool: Liquidity pools per property token (like Uniswap)

***Recommendation***: For MVP, start with simple order book:

- Backend stores buy/sell orders in PostgreSQL
- Backend matches orders off-chain
- Backend executes transfer on-chain
- Later: migrate to full on-chain DEX

### Property Manager vs Admin Overlap
Current flow: "Property Manager creates → Admin approves → Admin tokenizes"

#### Unclear:
- Can Property Manager be the LLC/SPV owner?
- What prevents Property Manager from listing fake properties?
- Who verifies property ownership documents?

#### Recommendation: Add verification workflow:
1. Property Manager uploads: deed, appraisal, insurance
2. Backend stores in S3/Firebase
3. Admin reviews documents
4. Admin marks "Verified"
5. Only then can Admin tokenize

### Tenant Payment Link Security
"Property Manager generates payment link" - potential issues:
- What if link is shared publicly?
- How to prevent double-payment?
- Link expiration?

#### Recommendation: Add to RentModule:
- One-time use links (expire after payment)
- Tenant email verification before payment
- Link tied to specific lease agreement
- Payment amount pre-filled (not editable)

### Critical Missing Pieces
1. Compliance Module Details
ERC-3643 requires:

- Identity Registry: Maps wallet → verified identity
- Compliance Rules: Transfer restrictions (accredited investor only, max holders, lockup periods)

#### Recommendation: Add ComplianceModule:
- IdentityService (links KYC status to wallet)
- TransferValidator (checks rules before allowing transfer)
- ReportingService (generates regulatory reports)

2. Gas Fee Management
Who pays gas for:
- Token minting?
- Dividend distribution (expensive if many holders)?
- Investor trades?

#### Recommendation:
- Admin/Platform pays for minting & dividends (operational cost)
- Investors pay for trades (standard)
- Consider meta-transactions (Biconomy/Gelato) for better UX

3. Property Lifecycle States
Missing state machine for properties:

#### Recommendation:
***States:***
- DRAFT (Property Manager creating)
- PENDING_APPROVAL (Admin reviewing)
- APPROVED (Ready for tokenization)
- TOKENIZED (Active, can receive investments)
- FUNDED (Minimum investment reached)
- ACTIVE (Generating rent)
- LIQUIDATING (Property being sold)
- CLOSED (Tokens burned, investors paid out)

4. Error Handling for FIAT→USDC Conversion
What if:
- Circle API is down?
- Conversion fails mid-transaction?
- Exchange rate slippage?

#### Recommendation: Add PaymentQueueService:

- Retry logic with exponential backoff
- Manual admin intervention for failed conversions
- Notification system for stuck payments

### Documentation Structure Recommendation
Given this complexity, here's how we should structure the docs: 
- user-story.md       → User journeys for each role
- tech-stack.md       → What you already have (stack decisions)
- infra.md            → High-level architecture diagram + flow
- db.md               → PostgreSQL schema + relationships
- server.md           → Backend modules + API endpoints
- client.md           → Frontend pages + components
- resource.md         → Smart contracts + blockchain interactions
- data-flow.md        → Sequence diagrams for key flows
- ci-cd-pipelines.md  → Deployment + testing strategy


####
# Definitive Recommendations
####

1. Token Standard: ERC-3643 (T-REX Protocol)
Why ERC-3643 is the RIGHT choice for you:
#### Your Requirements:
- ✅ Securities compliance (property tokens ARE securities)
- ✅ KYC/AML integration (you're using Sumsub/Persona)
- ✅ Transfer restrictions (accredited investors, geographic limits)
- ✅ Regulatory reporting capabilities
- ✅ Long-term viability (not experimental)
#### ERC-3643 Advantages:
- Built-in compliance layer: Identity Registry + Compliance Rules are part of the standard
- Battle-tested: Used by Tokeny (creators), backed by real security token issuances
- Regulatory-friendly: Designed specifically for securities, accepted by regulators
- Flexible: Can add/modify compliance rules without redeploying tokens
- Interoperable: Works with compliant exchanges and custody solutions

#### Trade-offs (manageable):
- ❌ More complex than ERC-20 (but you NEED this complexity for compliance)
- ❌ Limited DEX support (but you're building your own marketplace anyway)
- ❌ Higher gas costs (but Polygon/Base makes this negligible)

#### Implementation Strategy:
##### Per-Property Token Approach:
- Each property = separate ERC-3643 token contract
- Shared Identity Registry (one for all properties)
- Shared Compliance Contract (reusable rules)
- Property Registry maps propertyId → token address

*** Verdict: Use ERC-3643. It's purpose-built for your use case.***

2. Marketplace: Hybrid Order Book (Off-chain matching, On-chain settlement)
#### Your Constraints:
- Need compliance checks on every transfer (ERC-3643 enforces this)
- Want fast UX (off-chain matching is instant)
- Need audit trail (on-chain settlement provides this)
- MVP first, scale later

Recommended Architecture:
- Order Book DEX: Buyers/sellers post orders, backend matches
- AMM Pool: Liquidity pools per property token (like Uniswap)

#### Phase 1 (MVP): Off-chain Order Book
┌─────────────────────────────────────────┐
│ Frontend: Investor posts buy/sell order │
│ Backend: Stores order in PostgreSQL     │
│ Backend: Matches orders (price-time)    │
│ Backend: Validates compliance rules     │
│ Backend: Executes transfer on-chain     │
│ Smart Contract: Enforces ERC-3643 rules │
└─────────────────────────────────────────┘

##### Pros:
- ✅ Fast matching (no gas for posting orders)
- ✅ Complex order types (limit, market, stop-loss)
- ✅ Easy to modify matching logic
- ✅ Lower gas costs for users
- ✅ Can batch multiple trades

##### Cons:
- ❌ Backend is trusted party (but you're already KYC provider)
- ❌ Not fully decentralized (acceptable for securities)

#### Phase 2 (Future): Hybrid with On-chain Order Book
- Keep off-chain matching for speed
- Add on-chain order book for transparency
- Users choose: fast (off-chain) vs trustless (on-chain)

##### Why NOT pure AMM (Uniswap-style):
AMM Problems for Securities:

- ❌ Compliance nightmare: Can't enforce KYC on liquidity providers
- ❌ Price discovery issues: Low liquidity = high slippage for real estate
- ❌ Impermanent loss: Doesn't make sense for property tokens
- ❌ Regulatory risk: AMMs are under scrutiny for securities

##### AMMs work for:
- Fungible tokens (ETH, USDC)
- High liquidity assets
- Non-securities

##### Your tokens are:
- Securities (regulated)
- Illiquid (real estate is not traded frequently)
- Need compliance checks

##### Verdict: Start with off-chain order book. It's standard for security token exchanges (tZERO, Securitize).

3. Order Book Implementation Details
##### Database Schema (PostgreSQL):
orders:
  - id
  - property_token_address
  - user_id (investor)
  - order_type (BUY/SELL)
  - order_kind (MARKET/LIMIT)
  - quantity (tokens)
  - price_per_token (USDC)
  - status (OPEN/FILLED/CANCELLED/EXPIRED)
  - created_at
  - expires_at

trades:
  - id
  - order_id (buyer)
  - order_id (seller)
  - property_token_address
  - quantity
  - price_per_token
  - total_amount (USDC)
  - tx_hash (blockchain confirmation)
  - status (PENDING/CONFIRMED/FAILED)
  - executed_at

##### Matching Logic:
1. Investor posts SELL order: 100 tokens @ $10.50
2. Backend stores in orders table
3. Another investor posts BUY order: 50 tokens @ $10.50
4. Backend matching engine finds match
5. Backend validates:
   - Buyer has sufficient USDC
   - Seller has sufficient tokens
   - Both pass compliance checks (KYC, accredited status)
6. Backend calls smart contract: transferFrom(seller, buyer, 50)
7. Smart contract enforces ERC-3643 rules
8. Backend records trade in trades table
9. Backend updates order statuses

##### Smart Contract Interface:
```
// MarketplaceContract.sol
  function executeTrade(
      address propertyToken,
      address seller,
      address buyer,
      uint256 amount,
      uint256 pricePerToken
  ) external onlyBackend {
      // Transfer tokens (ERC-3643 compliance checked automatically)
      IERC3643(propertyToken).transferFrom(seller, buyer, amount);

      // Transfer USDC payment
      USDC.transferFrom(buyer, seller, amount * pricePerToken);

      emit TradeExecuted(propertyToken, seller, buyer, amount, pricePerToken);
  }
```
4. Final Architecture Summary

Smart Contracts (Polygon/Base):
├── Identity Registry (shared)
├── Compliance Contract (shared)
├── Property Registry (maps properties)
├── Property Token Contracts (ERC-3643, one per property)
├── Dividend Contract (distributes rent income)
├── Marketplace Contract (executes trades)
└── Treasury Contract (holds USDC for dividends)

Backend (NestJS):
├── AuthModule
├── KYCModule (Sumsub → Identity Registry)
├── PropertyModule (lifecycle states)
├── TokenModule (minting, burning)
├── MarketplaceModule (order book, matching)
├── DividendModule (distribution workflow)
├── RentModule (tenant payments)
├── PaymentsModule (Stripe, Circle)
├── BlockchainModule (Web3, event listeners)
└── ComplianceModule (transfer validation)

Frontend (Next.js):
├── Investor Dashboard (portfolio, marketplace)
├── Property Manager Dashboard (listings, rent links)
├── Admin Dashboard (approvals, tokenization, dividends)
└── Tenant Portal (rent payments)

5. My Definitive Recommendations:
  1. Token Standard: ERC-3643 ✅
    - Non-negotiable for securities compliance
    - Future-proof for regulatory changes
    - Industry standard for tokenized real estate

  2. Marketplace: Off-chain Order Book ✅
    - Fast, compliant, scalable
    - Standard approach for security tokens
    - Can add on-chain components later
  3. Property Lifecycle: Approved ✅
    - DRAFT → PENDING_APPROVAL → APPROVED → TOKENIZED → FUNDED → ACTIVE → LIQUIDATING → CLOSED
  4. Blockchain: Polygon (recommended) or Base
    - Polygon: More mature, better tooling
    - Base: Coinbase-backed, growing ecosystem
    - Both: Low fees, EVM-compatible