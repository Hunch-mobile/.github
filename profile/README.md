# Hunch - Organization Documentation

## Overview

Hunch is a mobile prediction market and social trading platform built on Solana. The platform enables users to trade prediction markets, follow successful traders, and leverage copy-trading functionality to replicate strategies from top performers across both native and external markets.

---

## Architecture

### System Components

| Component | Name | Description |
|-----------|------|-------------|
| Frontend | hunch-app | React Native mobile application |
| Backend | hunch-backend | REST API service hosted on Vercel |
| Blockchain | Solana Mainnet | Settlement layer for trades and USDC transfers |

### High-Level Architecture

```
Mobile App (hunch-app)
        |
        v
   REST API (hunch-backend)
        |
        +---> Privy (Authentication)
        +---> Jupiter Prediction (Markets)
        +---> Polymarket (Leaderboard/Data)
        +---> Solana RPC (Blockchain)
        +---> Database (User Data)
```

---

## Frontend: hunch-app

### Technology Stack

| Category | Technology | Version |
|----------|------------|---------|
| Framework | React Native | 0.81.5 |
| Platform | Expo | 54 |
| Routing | expo-router | 6 |
| Styling | NativeWind (Tailwind CSS) | 4 |
| State Management | React Context | - |
| Authentication | Privy | @privy-io/expo |
| Blockchain | Solana Web3.js | - |
| Package Manager | pnpm | 10.11 |

### Directory Structure

```
hunch-app/
├── app/                      # Expo Router screens (file-based routing)
│   ├── _layout.tsx           # Root layout, providers, auth flow
│   ├── (tabs)/               # Tab navigator
│   │   ├── index.tsx         # Home - Markets and events
│   │   ├── social.tsx        # Social feed
│   │   ├── leaderboard.tsx   # Trader rankings
│   │   └── profile.tsx       # User profile and positions
│   ├── onboarding/           # User onboarding flow
│   ├── event/[ticker]/       # Event detail screens
│   ├── market/[ticker]/      # Market trading screens
│   ├── user/                 # User profiles
│   ├── profile/[identifier]/ # Unified profile (internal/external)
│   └── trade/[tradeId]/      # Trade detail
├── components/               # Reusable UI components
│   ├── AddCashSheet.tsx      # Deposit functionality
│   ├── CreditCard.tsx        # Balance and withdrawal
│   ├── MarketTradeSheet.tsx  # Trading interface
│   ├── CopyTradeSheet.tsx    # Copy trading configuration
│   ├── PositionCard.tsx      # Position display
│   └── skeletons/            # Loading states
├── contexts/
│   └── UserContext.tsx       # Global user state
├── hooks/                    # Custom React hooks
├── lib/
│   ├── api.ts                # API client
│   ├── tradeService.ts       # Trade execution
│   ├── types.ts              # TypeScript definitions
│   └── pushNotifications.ts  # Push notification handling
├── constants/                # Theme and configuration
└── assets/                   # Static resources
```

### Core Features

#### Market Trading
- Browse prediction markets across categories (crypto, sports, politics, entertainment)
- Real-time price charts and market data
- Swipe-to-trade interface for quick order execution
- Position management with PnL tracking

#### Social Trading
- Follow traders on the platform
- Curated feeds (For You, Following)
- Trade sharing and social posts
- Top trader discovery

#### Copy Trading
- Automatic trade replication from selected leaders
- Support for both internal (Hunch) and external (Polymarket) traders
- Configurable parameters: amount per trade, maximum allocation
- Key Quorum delegation for secure automated trading

#### Portfolio Management
- Aggregated position view by market and side
- Real-time PnL calculations
- Active and historical position tracking
- USDC balance management

#### Funding
- Deposit via QR code (direct USDC transfer)
- Deposit via Privy (card/Apple Pay)
- Withdrawal to external wallets

### Authentication Flow

1. User initiates OAuth via Privy (Twitter/X, Apple, Google)
2. Privy creates embedded Solana wallet
3. Backend bootstraps and syncs user data
4. Onboarding sequence:
   - Link X/Twitter account
   - Claim username
   - Select interests
   - Follow suggested traders
5. Auth gate enforces routing based on completion state

---

## Backend: hunch-backend

### Technology Stack

| Category | Technology |
|----------|------------|
| Runtime | Node.js |
| Hosting | Vercel (Serverless) |
| API Style | REST |
| Authentication | Privy JWT |
| Database | PostgreSQL (inferred) |

### API Endpoints

#### Authentication

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/auth/bootstrap-oauth-user` | POST | Initialize OAuth user session |

#### Users

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/users/sync` | POST | No | Sync user from Privy |
| `/api/users/:userId` | GET | No | Retrieve user profile |
| `/api/users/push-token` | POST/DELETE | Yes | Manage push notification tokens |
| `/api/users/:userId/preferences` | GET/POST | Yes | User preferences |
| `/api/users/top` | GET | No | Top users by metrics |
| `/api/users/search` | GET | No | Search users |
| `/api/users/username/check` | GET | No | Username availability |
| `/api/users/username/claim` | POST | Yes | Claim username |
| `/api/users/onboarding/progress` | POST | Yes | Update onboarding state |
| `/api/users/delegation-status` | GET | Yes | Copy trading delegation status |

#### Social

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/follow` | POST | Yes | Follow user |
| `/api/follow` | DELETE | Yes | Unfollow user |
| `/api/follow/following` | GET | No | Get following list |
| `/api/follow/followers` | GET | No | Get followers list |

#### Trading

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/trades` | POST | Yes | Record trade |
| `/api/trades` | PATCH | Yes | Update trade quote |
| `/api/trades` | GET | No | Get user trades |
| `/api/trades/:tradeId` | GET | Yes | Get trade details |
| `/api/positions` | GET | No | Get aggregated positions |

#### Copy Trading

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/copy-settings` | POST/GET/DELETE | Yes | Manage internal copy settings |
| `/api/copy-settings/:followerId/:leaderId` | PATCH | Yes | Toggle copy for leader |
| `/api/copy/settings` | GET/POST/DELETE | Yes | Manage external copy settings |
| `/api/copy/all-leaders` | GET | Yes | Get all copy leaders |

#### Feed

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/feed/for-you` | GET | No | Personalized feed |
| `/api/feed/following` | GET | No | Following feed |
| `/api/feed/external-trades` | GET | Yes | External platform trades |
| `/api/feed/top-trader-trades` | GET | No | Top trader activity |

#### Posts

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/posts` | POST | Yes | Create post |
| `/api/posts/:postId` | DELETE | Yes | Delete post |

#### Markets and Events

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/home/feed` | GET | Home feed with events and markets |
| `/api/events/evidence` | GET | Event news and evidence |
| `/api/markets/by-condition-id` | GET | Market lookup |

#### Jupiter Prediction Proxy

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/jupiter-prediction/events` | GET | List prediction events |
| `/api/jupiter-prediction/events/:eventId` | GET | Event details |
| `/api/jupiter-prediction/orders` | POST | Create sponsor-signed order |

#### Polymarket Proxy

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/polymarket/leaderboard` | GET | Trader leaderboard |
| `/api/polymarket/positions` | GET | User positions |
| `/api/polymarket/closed-positions` | GET | Historical positions |
| `/api/polymarket/candlesticks/:conditionId` | GET | Price history |

#### Utilities

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/profiles/:identifier` | GET | Unified profile lookup |
| `/api/send-usdc` | POST | Build USDC transfer transaction |

### Authentication

- **Provider**: Privy
- **Method**: JWT Bearer tokens
- **Flow**: Client obtains token via Privy SDK, includes in `Authorization` header
- **Error Codes**: `MISSING_TOKEN`, `INVALID_TOKEN`, `USER_NOT_FOUND`, `DELEGATION_REQUIRED`

---

## External Integrations

### Privy
- OAuth authentication (Twitter/X, Apple, Google)
- Embedded Solana wallet creation and management
- JWT token issuance and validation
- Transaction sponsorship

### Jupiter Prediction
- Primary source for prediction market events
- Order routing and execution
- Sponsor-signed transaction generation

### Polymarket
- Trader leaderboard data
- Position and trade history
- Candlestick price data
- Copy trading source

### Solana
- USDC settlement
- Transaction signing and confirmation
- Wallet balance queries

### CoinGecko
- SOL/USD price feed

### Expo
- Push notification delivery

---

## Data Models

### User
- Privy-linked identity
- Username (unique, claimed)
- Preferences and interests
- Onboarding state
- Push notification tokens

### Trade
- Market reference
- Side (Yes/No)
- Amount and price
- Transaction signature
- Timestamp

### Position
- Aggregated by market and side
- Entry price (weighted average)
- Current value
- PnL calculation
- Trade count

### Copy Settings
- Leader reference (internal or external)
- Amount per trade
- Maximum total allocation
- Active status

### Post
- Author reference
- Content (text or position share)
- Timestamp
- Engagement metrics

---

## Deployment

### Frontend (hunch-app)
- **Build System**: EAS (Expo Application Services)
- **iOS**: Deployment target 17.5, App Store distribution
- **Android**: Compile SDK 35, Play Store distribution
- **Bundle Identifier**: `run.hunch.app`

### Backend (hunch-backend)
- **Platform**: Vercel (Serverless)
- **Type**: Serverless functions

---

## Security Considerations

### Authentication
- Privy handles OAuth and wallet security
- JWT tokens for API authentication
- No direct credential storage

### Trading
- Sponsor-signed transactions for user convenience
- Key Quorum delegation for copy trading
- Transaction confirmation verification

### Data
- Sensitive operations require authentication
- User data isolated by Privy identity
- Push tokens managed securely

---

## Development

### Prerequisites
- Node.js 18+
- pnpm 10+
- Expo CLI
- iOS Simulator / Android Emulator

### Setup

```bash
# Install dependencies
pnpm install

# Start development server
pnpm start

# Run on iOS
pnpm ios

# Run on Android
pnpm android
```

---

## Contact

For technical questions or access requests, contact the development team.
