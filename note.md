token-prop/
├── apps/
│   ├── client/                 # Next.js Frontend
│   │   ├── src/
│   │   │   ├── app/           # App Router pages
│   │   │   ├── components/    # React components
│   │   │   ├── lib/           # Utilities
│   │   │   ├── services/      # API & Web3 services
│   │   │   └── types/         # TypeScript types
│   │   ├── public/
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── server/                # NestJS Backend
│   │   ├── src/
│   │   │   ├── modules/       # Feature modules
│   │   │   ├── common/        # Shared utilities
│   │   │   ├── config/        # Configuration
│   │   │   └── prisma/        # Prisma client
│   │   ├── prisma/
│   │   │   ├── schema.prisma
│   │   │   └── migrations/
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── contracts/             # Smart Contracts
│       ├── contracts/
│       │   ├── PropertyToken.sol
│       │   ├── DividendDistributor.sol
│       │   ├── PropertyRegistry.sol
│       │   └── Escrow.sol
│       ├── scripts/
│       ├── test/
│       ├── hardhat.config.ts
│       └── package.json
│
├── packages/
│   ├── types/                 # Shared TypeScript types
│   ├── config/                # Shared configuration
│   └── utils/                 # Shared utilities
│
├── docs/
│   ├── ARCHITECTURE.md        # System architecture
│   ├── API.md                 # API documentation
│   ├── CONTRACTS.md           # Smart contract specs
│   └── DEPLOYMENT.md          # Deployment guide
│
├── .github/
│   └── workflows/             # CI/CD pipelines
│
├── package.json               # Root workspace config
├── turbo.json                 # Turborepo config
└── README.md


File management
Firebase 
Pheonix