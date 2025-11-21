# User Stories & Journeys

## Overview
This document describes user journeys for each role in the platform, detailing their goals, actions, and expected outcomes.

---

## User Roles

1. **Admin**: Platform administrator with full system access
2. **Property Manager**: Lists and manages properties
3. **Investor**: Buys, sells, and holds property tokens
4. **Tenant**: Pays rent for properties

---

## 1. Investor Journey

### Story 1: First-Time Investor Registration

**As an** investor  
**I want to** register and complete KYC verification  
**So that** I can invest in tokenized real estate

**Journey:**
1. Visit landing page
2. Click "Get Started" or "Sign Up"
3. Fill registration form (email, password, name)
4. Receive verification email
5. Click email link to verify account
6. Log in to platform
7. Prompted to complete KYC
8. Click "Start KYC Verification"
9. Redirected to Sumsub iframe
10. Upload ID document and selfie
11. Wait for verification (1-24 hours)
12. Receive email: "KYC Approved"
13. Log in and see "Verified" badge

**Acceptance Criteria:**
- User can register with valid email
- Email verification link expires after 24 hours
- KYC status synced to blockchain Identity Registry
- User cannot invest without KYC approval

---

### Story 2: Browsing and Investing in a Property

**As a** verified investor  
**I want to** browse properties and purchase tokens  
**So that** I can earn passive income from real estate

**Journey:**
1. Log in to investor dashboard
2. Navigate to "Marketplace"
3. Browse tokenized properties
4. Filter by location, property type, ROI
5. Click on property card to view details
6. Review property information:
   - Location, size, type
   - Total value and token price
   - Expected annual return
   - Historical performance
   - Documents (deed, appraisal)
7. Click "Buy Tokens"
8. Enter quantity to purchase
9. Review total cost in USDC
10. Click "Connect Wallet" (if not connected)
11. Approve MetaMask connection
12. Click "Approve USDC Spend"
13. Confirm transaction in MetaMask
14. Wait for approval confirmation
15. Click "Confirm Purchase"
16. Transaction submitted to blockchain
17. Wait for confirmation (5-30 seconds)
18. See success message: "Tokens purchased successfully"
19. Tokens appear in portfolio

**Acceptance Criteria:**
- Only KYC-approved investors can buy tokens
- Minimum investment amount enforced
- Compliance checks pass before purchase
- Tokens appear in portfolio immediately after confirmation
- Transaction recorded in transaction history

---

### Story 3: Receiving Dividend Payments

**As an** investor holding property tokens  
**I want to** receive dividend payments automatically  
**So that** I earn passive income from rental properties

**Journey:**
1. Log in to investor dashboard
2. View portfolio with token holdings
3. Admin triggers dividend distribution (monthly/quarterly)
4. Receive email notification: "Dividend distributed"
5. Check wallet - USDC balance increased
6. Navigate to "Dividends" page
7. View dividend history:
   - Date, property, amount, token balance
8. Export dividend report for tax purposes

**Acceptance Criteria:**
- Dividends distributed proportionally to token holdings
- Snapshot taken at specific block number
- Email notification sent to all recipients
- Dividend history visible in dashboard
- Export to CSV available

---

### Story 4: Trading Tokens on Marketplace

**As an** investor  
**I want to** sell my tokens to other investors  
**So that** I can exit my investment or rebalance my portfolio

**Journey:**
1. Log in to investor dashboard
2. Navigate to "Marketplace"
3. Select property token to sell
4. Click "Sell Tokens"
5. Enter quantity and price per token
6. Review order details
7. Click "Post Sell Order"
8. Order appears in order book
9. Wait for buyer to match order
10. Receive notification: "Order matched"
11. Trade executed automatically
12. USDC received in wallet
13. Tokens removed from portfolio
14. Trade appears in transaction history

**Acceptance Criteria:**
- Seller must own tokens to create sell order
- Order book displays all active orders
- Matching engine finds best price
- Compliance checks enforced on buyer
- Trade settled on-chain

---

## 2. Property Manager Journey

### Story 1: Listing a New Property

**As a** property manager  
**I want to** list my property for tokenization  
**So that** I can raise capital and manage it on the platform

**Journey:**
1. Log in to property manager dashboard
2. Click "Add New Property"
3. Fill property information form:
   - Name, description
   - Address, city, state, country
   - Property type (residential, commercial)
   - Purchase price, current valuation
   - Total tokens, token price
   - Monthly rent, expected annual return
   - Management fee percentage
   - Distribution period (monthly/quarterly)
4. Upload property images
5. Upload documents:
   - Property deed
   - Recent appraisal
   - Insurance policy
   - Lease agreement (if applicable)
6. Review all information
7. Click "Save as Draft"
8. Property saved with status: DRAFT
9. Make any edits if needed
10. Click "Submit for Approval"
11. Property status changes to PENDING_APPROVAL
12. Receive confirmation: "Property submitted for review"
13. Wait for admin approval (1-7 days)
14. Receive email: "Property approved" or "Property rejected"

**Acceptance Criteria:**
- All required fields must be filled
- Documents must be uploaded (deed, appraisal, insurance)
- Property saved as DRAFT before submission
- Cannot edit property after submission
- Email notifications sent on status changes

---

### Story 2: Generating Rent Payment Links

**As a** property manager  
**I want to** generate payment links for tenants  
**So that** tenants can pay rent easily

**Journey:**
1. Log in to property manager dashboard
2. Navigate to "Rent Links"
3. Click "Generate Payment Link"
4. Select property
5. Enter rent amount
6. Set expiration date (optional)
7. Click "Generate Link"
8. Copy payment link
9. Send link to tenant via email/SMS
10. Tenant clicks link and pays rent
11. Receive notification: "Rent payment received"
12. View payment in "Payment History"
13. Payment automatically converted to USDC
14. USDC added to property's dividend pool

**Acceptance Criteria:**
- Link is unique and one-time use
- Link expires after set date or after use
- Payment amount is pre-filled
- Stripe handles payment processing
- Payment recorded in database
- Email confirmation sent to tenant and property manager

---

## 3. Admin Journey

### Story 1: Approving Property for Tokenization

**As an** admin  
**I want to** review and approve properties  
**So that** only legitimate properties are tokenized

**Journey:**
1. Log in to admin dashboard
2. See notification: "3 properties pending approval"
3. Navigate to "Pending Properties"
4. View list of properties with status PENDING_APPROVAL
5. Click on property to review
6. Review property details:
   - Basic information
   - Financial details
   - Uploaded documents
7. Download and verify documents:
   - Property deed (ownership proof)
   - Appraisal (valuation verification)
   - Insurance policy
8. Check property manager's credentials
9. Verify property exists (Google Maps, public records)
10. Decision: Approve or Reject
11. If rejecting, enter rejection reason
12. Click "Approve Property" or "Reject Property"
13. Property status updated to APPROVED or DRAFT
14. Property manager notified via email
15. If approved, property appears in "Ready for Tokenization"

**Acceptance Criteria:**
- Admin can view all pending properties
- All documents accessible for review
- Rejection reason required if rejecting
- Property manager notified of decision
- Approved properties ready for tokenization

---

### Story 2: Tokenizing an Approved Property

**As an** admin  
**I want to** tokenize an approved property  
**So that** investors can start buying tokens

**Journey:**
1. Log in to admin dashboard
2. Navigate to "Tokenization"
3. View list of APPROVED properties
4. Click on property to tokenize
5. Review tokenization parameters:
   - Total supply (number of tokens)
   - Token symbol (e.g., PROP001)
   - Initial price per token
6. Click "Deploy Token Contract"
7. Confirm transaction in MetaMask
8. Wait for contract deployment (30-60 seconds)
9. Contract deployed successfully
10. Contract address stored in database
11. Property status updated to TOKENIZED
12. Property appears in marketplace
13. Investors can now purchase tokens
14. Property manager notified: "Property tokenized"

**Acceptance Criteria:**
- Only APPROVED properties can be tokenized
- ERC-3643 contract deployed correctly
- Contract linked to Identity Registry and Compliance Contract
- Contract address stored in database
- Property visible in marketplace
- Property manager and investors notified

---

### Story 3: Distributing Dividends

**As an** admin  
**I want to** distribute rental income to investors  
**So that** investors receive their share of profits

**Journey:**
1. Log in to admin dashboard
2. Navigate to "Dividends"
3. View properties with rent collected
4. Select property for dividend distribution
5. Review income statement:
   - Total rent collected: $10,000
   - Management fees (5%): -$500
   - Platform fees (2%): -$200
   - Net distributable income: $9,300
6. Click "Calculate Dividends"
7. System calculates per-token dividend:
   - 10,000 tokens outstanding
   - $9,300 / 10,000 = $0.93 per token
8. Review distribution details:
   - Number of token holders: 50
   - Total USDC to distribute: $9,300
9. Click "Trigger Distribution"
10. Confirm transaction in MetaMask
11. Smart contract snapshots token holders
12. Smart contract distributes USDC proportionally
13. Wait for confirmation (30-60 seconds)
14. Distribution complete
15. All investors notified via email
16. Distribution recorded in database

**Acceptance Criteria:**
- Only properties with collected rent can distribute
- Fees deducted before distribution
- Snapshot taken at specific block
- Distribution proportional to token holdings
- All investors receive USDC in wallets
- Email notifications sent
- Distribution history recorded

---

## 4. Tenant Journey

### Story 1: Paying Rent via Payment Link

**As a** tenant  
**I want to** pay rent online  
**So that** I can fulfill my lease obligations conveniently

**Journey:**
1. Receive payment link from property manager
2. Click link in email/SMS
3. Redirected to payment page
4. See property details and rent amount
5. Choose payment method:
   - Credit/debit card (Stripe)
   - Bank transfer
6. Enter payment details
7. Review payment summary
8. Click "Pay $1,000"
9. Payment processed by Stripe
10. See confirmation: "Payment successful"
11. Receive email receipt
12. Payment link deactivated (one-time use)
13. Payment converted to USDC by backend
14. USDC added to property's dividend pool

**Acceptance Criteria:**
- Payment link works only once
- Rent amount is pre-filled and not editable
- Stripe handles secure payment processing
- Email receipt sent immediately
- Payment recorded in database
- Property manager notified of payment
- Payment converted to USDC automatically

---

### Story 2: Viewing Payment History

**As a** tenant  
**I want to** view my rent payment history  
**So that** I can track my payments and download receipts

**Journey:**
1. Log in to tenant portal (optional registration)
2. Navigate to "Payment History"
3. View list of all rent payments:
   - Date, property, amount, status
4. Filter by date range or property
5. Click on payment to view details
6. See payment receipt with:
   - Property address
   - Payment date and amount
   - Payment method
   - Transaction ID
7. Click "Download Receipt" (PDF)
8. Receipt downloaded for records

**Acceptance Criteria:**
- All payments visible in history
- Payments sorted by date (newest first)
- Filter and search functionality
- Receipt available for download
- Receipt includes all payment details

---

## User Flow Summary

### Investor Flow
```
Register → KYC → Browse Properties → Buy Tokens → Receive Dividends → Trade Tokens
```

### Property Manager Flow
```
Register → List Property → Upload Documents → Submit for Approval → Generate Rent Links → Collect Rent
```

### Admin Flow
```
Review Properties → Approve → Tokenize → Monitor System → Distribute Dividends
```

### Tenant Flow
```
Receive Payment Link → Pay Rent → View Payment History
```

---

## Key Success Metrics

**For Investors:**
- Time to first investment < 24 hours (including KYC)
- Average ROI visibility
- Dividend payout reliability
- Token liquidity (trading volume)

**For Property Managers:**
- Property approval time < 7 days
- Rent collection rate > 95%
- Investor satisfaction

**For Platform:**
- Total properties tokenized
- Total value locked (TVL)
- Number of active investors
- Trading volume
- Dividend distribution success rate
