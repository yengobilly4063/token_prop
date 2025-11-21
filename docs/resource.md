# Smart Contracts & Blockchain Resources

## Overview
This document describes all smart contracts deployed on Polygon/Base, their interfaces, and blockchain interactions.

## Contract Architecture

```
Identity Registry (Shared)
  │
  ├─> Compliance Contract (Shared)
  │     │
  │     ├─> Property Token 1 (ERC-3643)
  │     ├─> Property Token 2 (ERC-3643)
  │     └─> Property Token 3 (ERC-3643)
  │
  ├─> Property Registry
  ├─> Dividend Contract
  ├─> Marketplace Contract
  └─> Treasury Contract
```

## Core Contracts

### 1. Identity Registry (ERC-3643 Requirement)

**Purpose:** Maps wallet addresses to verified identities

**Key Functions:**
```solidity
function registerIdentity(
    address _userAddress,
    bytes32 _identityHash,
    uint16 _country
) external onlyAgent;

function updateIdentity(
    address _userAddress,
    bytes32 _identityHash
) external onlyAgent;

function deleteIdentity(address _userAddress) external onlyAgent;

function isVerified(address _userAddress) external view returns (bool);

function getCountry(address _userAddress) external view returns (uint16);
```

**Events:**
```solidity
event IdentityRegistered(address indexed userAddress, bytes32 identityHash);
event IdentityUpdated(address indexed userAddress, bytes32 identityHash);
event IdentityRemoved(address indexed userAddress);
```

### 2. Compliance Contract (ERC-3643 Requirement)

**Purpose:** Enforces transfer restrictions and compliance rules

**Key Functions:**
```solidity
function canTransfer(
    address _from,
    address _to,
    uint256 _value
) external view returns (bool);

function setMaxHolders(uint256 _max) external onlyOwner;

function setCountryRestrictions(
    uint16[] calldata _countries,
    bool _restricted
) external onlyOwner;

function setAccreditedOnly(bool _required) external onlyOwner;
```

**Events:**
```solidity
event ComplianceRuleUpdated(string ruleName, bytes32 ruleHash);
```

### 3. Property Token Contract (ERC-3643)

**Purpose:** Represents fractional ownership of a property

**Key Functions:**
```solidity
// Standard ERC-20 functions
function transfer(address to, uint256 amount) external returns (bool);
function transferFrom(address from, address to, uint256 amount) external returns (bool);
function balanceOf(address account) external view returns (uint256);
function totalSupply() external view returns (uint256);

// ERC-3643 specific
function mint(address to, uint256 amount) external onlyAgent;
function burn(address from, uint256 amount) external onlyAgent;
function pause() external onlyAgent;
function unpause() external onlyAgent;

// Custom functions
function getPropertyId() external view returns (uint256);
function getPropertyMetadata() external view returns (string memory);
```

**Events:**
```solidity
event Transfer(address indexed from, address indexed to, uint256 value);
event Approval(address indexed owner, address indexed spender, uint256 value);
event Minted(address indexed to, uint256 amount);
event Burned(address indexed from, uint256 amount);
```

### 4. Property Registry Contract

**Purpose:** Maps property IDs to token contract addresses

**Key Functions:**
```solidity
function registerProperty(
    uint256 _propertyId,
    address _tokenAddress,
    string calldata _metadataURI
) external onlyAdmin;

function getTokenAddress(uint256 _propertyId) external view returns (address);

function getPropertyMetadata(uint256 _propertyId) external view returns (string memory);

function updateValuation(
    uint256 _propertyId,
    uint256 _newValuation
) external onlyAdmin;
```

**Events:**
```solidity
event PropertyRegistered(uint256 indexed propertyId, address tokenAddress);
event ValuationUpdated(uint256 indexed propertyId, uint256 newValuation);
```

### 5. Dividend Distribution Contract

**Purpose:** Distributes rental income to token holders

**Key Functions:**
```solidity
function distribute(
    address _tokenAddress,
    uint256 _amount
) external onlyAdmin;

function claimDividend(address _tokenAddress) external;

function getDividendBalance(
    address _tokenAddress,
    address _holder
) external view returns (uint256);

function getDistributionHistory(
    address _tokenAddress
) external view returns (Distribution[] memory);
```

**Events:**
```solidity
event DividendDistributed(
    address indexed tokenAddress,
    uint256 amount,
    uint256 timestamp
);
event DividendClaimed(
    address indexed tokenAddress,
    address indexed holder,
    uint256 amount
);
```

### 6. Marketplace Contract

**Purpose:** Executes trades with compliance checks

**Key Functions:**
```solidity
function executeTrade(
    address _tokenAddress,
    address _seller,
    address _buyer,
    uint256 _amount,
    uint256 _pricePerToken
) external onlyBackend;

function getTradeHistory(
    address _tokenAddress
) external view returns (Trade[] memory);
```

**Events:**
```solidity
event TradeExecuted(
    address indexed tokenAddress,
    address indexed seller,
    address indexed buyer,
    uint256 amount,
    uint256 pricePerToken,
    uint256 timestamp
);
```

### 7. Treasury Contract

**Purpose:** Holds platform reserves and manages escrow

**Key Functions:**
```solidity
function deposit(uint256 _amount) external;

function withdraw(uint256 _amount, address _to) external onlyAdmin;

function getBalance() external view returns (uint256);

function createEscrow(
    uint256 _propertyId,
    uint256 _amount
) external onlyAdmin;

function releaseEscrow(uint256 _escrowId) external onlyAdmin;
```

**Events:**
```solidity
event Deposited(address indexed from, uint256 amount);
event Withdrawn(address indexed to, uint256 amount);
event EscrowCreated(uint256 indexed escrowId, uint256 amount);
event EscrowReleased(uint256 indexed escrowId);
```

## Contract Deployment

### Deployment Order
```
1. Deploy Identity Registry
2. Deploy Compliance Contract
3. Deploy Property Registry
4. Deploy Dividend Contract
5. Deploy Marketplace Contract
6. Deploy Treasury Contract
7. For each property:
   a. Deploy Property Token (ERC-3643)
   b. Register in Property Registry
   c. Link to Identity Registry
   d. Link to Compliance Contract
```

### Hardhat Deployment Script
```typescript
// scripts/deploy.ts
import { ethers } from 'hardhat';

async function main() {
  // Deploy Identity Registry
  const IdentityRegistry = await ethers.getContractFactory('IdentityRegistry');
  const identityRegistry = await IdentityRegistry.deploy();
  await identityRegistry.deployed();
  console.log('IdentityRegistry deployed to:', identityRegistry.address);

  // Deploy Compliance Contract
  const Compliance = await ethers.getContractFactory('Compliance');
  const compliance = await Compliance.deploy(identityRegistry.address);
  await compliance.deployed();
  console.log('Compliance deployed to:', compliance.address);

  // Deploy Property Registry
  const PropertyRegistry = await ethers.getContractFactory('PropertyRegistry');
  const propertyRegistry = await PropertyRegistry.deploy();
  await propertyRegistry.deployed();
  console.log('PropertyRegistry deployed to:', propertyRegistry.address);

  // Deploy Dividend Contract
  const Dividend = await ethers.getContractFactory('DividendDistribution');
  const dividend = await Dividend.deploy();
  await dividend.deployed();
  console.log('Dividend deployed to:', dividend.address);

  // Deploy Marketplace
  const Marketplace = await ethers.getContractFactory('Marketplace');
  const marketplace = await Marketplace.deploy(compliance.address);
  await marketplace.deployed();
  console.log('Marketplace deployed to:', marketplace.address);

  // Deploy Treasury
  const Treasury = await ethers.getContractFactory('Treasury');
  const treasury = await Treasury.deploy();
  await treasury.deployed();
  console.log('Treasury deployed to:', treasury.address);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

## Contract Verification

```bash
# Verify on Polygonscan
npx hardhat verify --network polygon <CONTRACT_ADDRESS> <CONSTRUCTOR_ARGS>

# Example
npx hardhat verify --network polygon 0x123... 0xIdentityRegistryAddress
```

## Gas Optimization

**Strategies:**
- Use `uint256` instead of smaller uints (unless packing)
- Batch operations when possible
- Use events instead of storage for historical data
- Implement EIP-2612 (permit) for gasless approvals
- Use Merkle trees for large airdrops

## Security Considerations

**Audits:**
- Third-party audit by Certik, OpenZeppelin, or Trail of Bits
- Bug bounty program on Immunefi

**Access Control:**
- Multi-sig wallet for admin operations (Gnosis Safe)
- Time-locks for critical functions
- Role-based access control (RBAC)

**Testing:**
- 100% test coverage
- Fuzz testing with Echidna
- Formal verification for critical contracts

## Upgradeability

**Proxy Pattern:**
```solidity
// Use OpenZeppelin's TransparentUpgradeableProxy
import "@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";

// Deploy proxy pointing to implementation
const proxy = await upgrades.deployProxy(PropertyToken, [args], {
  initializer: 'initialize',
});
```

## Monitoring

**Events to Monitor:**
- All `Transfer` events
- `DividendDistributed` events
- `TradeExecuted` events
- `IdentityRegistered` events
- Failed transactions

**Alerts:**
- Large transfers (> $100k)
- Failed dividend distributions
- Unusual trading volume
- Contract paused

## Contract Addresses (Example)

**Polygon Mainnet:**
```
Identity Registry:    0x1234567890123456789012345678901234567890
Compliance Contract:  0x2345678901234567890123456789012345678901
Property Registry:    0x3456789012345678901234567890123456789012
Dividend Contract:    0x4567890123456789012345678901234567890123
Marketplace Contract: 0x5678901234567890123456789012345678901234
Treasury Contract:    0x6789012345678901234567890123456789012345
```

**Testnet (Mumbai):**
```
Identity Registry:    0xabcdef...
Compliance Contract:  0xbcdef0...
...
```
