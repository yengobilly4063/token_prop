# Frontend Client (Next.js)

## Overview
The frontend is built with Next.js 14+ using the App Router, providing server-side rendering, optimized performance, and a modern React-based UI.

## Project Structure

```
app/
├── (public)/
│   ├── page.tsx                    # Landing page
│   ├── marketplace/
│   │   └── page.tsx                # Browse properties
│   ├── login/page.tsx
│   └── register/page.tsx
├── (investor)/
│   ├── dashboard/page.tsx          # Portfolio overview
│   ├── properties/
│   │   └── [id]/page.tsx           # Property details
│   ├── marketplace/page.tsx        # Buy/Sell tokens
│   └── transactions/page.tsx
├── (property-manager)/
│   ├── dashboard/page.tsx
│   ├── properties/
│   │   ├── new/page.tsx
│   │   └── [id]/edit/page.tsx
│   └── rent-links/page.tsx
├── (admin)/
│   ├── dashboard/page.tsx
│   ├── properties/
│   │   └── pending/page.tsx        # Approval queue
│   ├── tokenization/page.tsx
│   ├── dividends/page.tsx
│   └── users/page.tsx
├── (tenant)/
│   ├── pay/[linkId]/page.tsx       # Rent payment
│   └── history/page.tsx
├── api/                            # API routes
└── layout.tsx                      # Root layout

components/
├── ui/                             # shadcn/ui components
│   ├── button.tsx
│   ├── card.tsx
│   ├── dialog.tsx
│   └── ...
├── property/
│   ├── PropertyCard.tsx
│   ├── PropertyDetails.tsx
│   └── PropertyForm.tsx
├── wallet/
│   ├── WalletConnect.tsx
│   └── WalletBalance.tsx
├── marketplace/
│   ├── OrderBook.tsx
│   ├── OrderForm.tsx
│   └── TradeHistory.tsx
└── layout/
    ├── Header.tsx
    ├── Sidebar.tsx
    └── Footer.tsx

lib/
├── api.ts                          # API client
├── wagmi.ts                        # Web3 config
└── utils.ts

hooks/
├── useProperty.ts
├── useMarketplace.ts
└── useWallet.ts
```

## Pages by Role

### Public Pages

#### Landing Page (`/`)
- Hero section with value proposition
- Featured properties
- How it works section
- Call-to-action buttons

#### Marketplace (`/marketplace`)
- Browse all tokenized properties
- Filter by property type, location, ROI
- Property cards with key metrics
- Search functionality

#### Login (`/login`)
- Email/password login
- Optional Web3 wallet login
- Forgot password link

#### Register (`/register`)
- User registration form
- Role selection (Investor, Property Manager, Tenant)
- Email verification

### Investor Dashboard

#### Portfolio Overview (`/investor/dashboard`)
- Total portfolio value
- Token holdings by property
- Recent dividends received
- Performance charts

#### Property Details (`/investor/properties/[id]`)
- Property information
- Token price and supply
- Historical performance
- Dividend history
- Buy tokens button

#### Marketplace (`/investor/marketplace`)
- Order book for each property
- Create buy/sell orders
- Recent trades
- Price charts

#### Transactions (`/investor/transactions`)
- All transactions (purchases, sales, dividends)
- Filter by type and date
- Export to CSV

### Property Manager Dashboard

#### Dashboard (`/property-manager/dashboard`)
- Properties overview
- Pending approvals status
- Recent rent payments
- Investor statistics

#### Create Property (`/property-manager/properties/new`)
- Property information form
- Document upload (deed, appraisal, insurance)
- Financial details
- Submit for approval

#### Edit Property (`/property-manager/properties/[id]/edit`)
- Edit property details (before tokenization)
- Update documents
- View approval status

#### Rent Links (`/property-manager/rent-links`)
- Generate payment links for tenants
- View active links
- Revoke links
- Payment history

### Admin Dashboard

#### Dashboard (`/admin/dashboard`)
- System overview
- Pending approvals count
- Total properties tokenized
- Platform metrics

#### Pending Properties (`/admin/properties/pending`)
- List of properties awaiting approval
- Review documents
- Approve or reject

#### Tokenization (`/admin/tokenization`)
- Approved properties ready for tokenization
- Deploy ERC-3643 contracts
- View tokenization status

#### Dividends (`/admin/dividends`)
- Properties with rent collected
- Calculate net income
- Trigger dividend distribution
- Distribution history

#### Users (`/admin/users`)
- User management
- KYC status overview
- Role assignments

### Tenant Portal

#### Rent Payment (`/tenant/pay/[linkId]`)
- Property details
- Payment amount
- Stripe checkout
- Payment confirmation

#### Payment History (`/tenant/history`)
- All rent payments
- Receipts
- Upcoming payments

## Key Components

### PropertyCard
```typescript
interface PropertyCardProps {
  property: Property;
  showActions?: boolean;
}

export function PropertyCard({ property, showActions }: PropertyCardProps) {
  return (
    <Card>
      <CardHeader>
        <Image src={property.imageUrls[0]} alt={property.name} />
        <CardTitle>{property.name}</CardTitle>
      </CardHeader>
      <CardContent>
        <p>{property.city}, {property.state}</p>
        <p>Token Price: ${property.tokenPrice}</p>
        <p>Annual Return: {property.annualReturn}%</p>
      </CardContent>
      {showActions && (
        <CardFooter>
          <Button>View Details</Button>
        </CardFooter>
      )}
    </Card>
  );
}
```

### WalletConnect
```typescript
import { ConnectButton } from '@rainbow-me/rainbowkit';

export function WalletConnect() {
  return (
    <ConnectButton
      accountStatus="address"
      chainStatus="icon"
      showBalance={true}
    />
  );
}
```

### OrderForm
```typescript
interface OrderFormProps {
  tokenId: string;
  orderType: 'BUY' | 'SELL';
}

export function OrderForm({ tokenId, orderType }: OrderFormProps) {
  const [quantity, setQuantity] = useState(0);
  const [price, setPrice] = useState(0);

  const handleSubmit = async () => {
    await createOrder({ tokenId, orderType, quantity, price });
  };

  return (
    <form onSubmit={handleSubmit}>
      <Input
        type="number"
        placeholder="Quantity"
        value={quantity}
        onChange={(e) => setQuantity(Number(e.target.value))}
      />
      <Input
        type="number"
        placeholder="Price per token"
        value={price}
        onChange={(e) => setPrice(Number(e.target.value))}
      />
      <Button type="submit">
        {orderType === 'BUY' ? 'Buy' : 'Sell'} Tokens
      </Button>
    </form>
  );
}
```

## State Management

### React Query for Server State
```typescript
// hooks/useProperty.ts
import { useQuery } from '@tanstack/react-query';

export function useProperty(id: string) {
  return useQuery({
    queryKey: ['property', id],
    queryFn: () => fetchProperty(id),
  });
}

export function useProperties() {
  return useQuery({
    queryKey: ['properties'],
    queryFn: fetchProperties,
  });
}
```

### Context for Global State
```typescript
// contexts/AuthContext.tsx
const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
}
```

## Web3 Integration

### Wagmi Configuration
```typescript
// lib/wagmi.ts
import { configureChains, createConfig } from 'wagmi';
import { polygon } from 'wagmi/chains';
import { alchemyProvider } from 'wagmi/providers/alchemy';
import { publicProvider } from 'wagmi/providers/public';

const { chains, publicClient } = configureChains(
  [polygon],
  [
    alchemyProvider({ apiKey: process.env.NEXT_PUBLIC_ALCHEMY_API_KEY! }),
    publicProvider(),
  ]
);

export const config = createConfig({
  autoConnect: true,
  publicClient,
});

export { chains };
```

### Contract Interactions
```typescript
// hooks/useTokenPurchase.ts
import { useContractWrite, usePrepareContractWrite } from 'wagmi';

export function useTokenPurchase(tokenAddress: string, amount: number) {
  const { config } = usePrepareContractWrite({
    address: tokenAddress,
    abi: ERC3643_ABI,
    functionName: 'transfer',
    args: [userAddress, amount],
  });

  return useContractWrite(config);
}
```

## Styling

### TailwindCSS + shadcn/ui
```typescript
// Example component with Tailwind
export function PropertyCard() {
  return (
    <div className="rounded-lg border bg-card text-card-foreground shadow-sm">
      <div className="flex flex-col space-y-1.5 p-6">
        <h3 className="text-2xl font-semibold leading-none tracking-tight">
          Property Name
        </h3>
      </div>
      <div className="p-6 pt-0">
        <p className="text-sm text-muted-foreground">
          Property description
        </p>
      </div>
    </div>
  );
}
```

## API Client

```typescript
// lib/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL || 'http://localhost:4000/api/v1',
  withCredentials: true, // For session cookies
});

// Request interceptor
api.interceptors.request.use((config) => {
  // Add any headers
  return config;
});

// Response interceptor
api.interceptors.response.use(
  (response) => response.data,
  (error) => {
    // Handle errors globally
    if (error.response?.status === 401) {
      // Redirect to login
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

## Environment Variables

```env
NEXT_PUBLIC_API_URL=https://api.tokenprop.com/api/v1
NEXT_PUBLIC_ALCHEMY_API_KEY=...
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=...
```

## Running the Frontend

**Development:**
```bash
npm run dev
```

**Build:**
```bash
npm run build
```

**Production:**
```bash
npm run start
```

**Lint:**
```bash
npm run lint
```
