# Fronsciers: Decentralized Academic Publishing Platform

## Project Overview

Fronsciers is a cutting-edge decentralized research platform that leverages blockchain technology to revolutionize academic collaboration and research discovery. The platform aims to provide truly immutable, transparent, and interoperable Direct On-Chain Identifiers (DOCIs) on the Solana blockchain, emulating and extending the traditional DOI system capabilities.

**Beta Test          :** https://fronsciers.replit.app/ 

**GitHub Repository  :** [https://github.com/rejoelm/fronsciers](https://github.com/rejoelm/fronsciers)

## Core Features

### 1. DOCI System (Direct On-Chain Identifiers)
- Immutable registration of research artifacts on the Solana blockchain
- Custom namespaces with prefix (10.FRONS) and unique suffixes
- Automatic resolver system that allows direct DOCI navigation in browsers
- Metadata storage with IPFS/Arweave integration for content persistence

### 2. Researcher Profiles
- Researcher-specific DOCIs with 10.FRONS-R prefix (e.g., 10.FRONS-R/R-7A8B9C)
- Comprehensive academic credential management
- Publication tracking and citation metrics
- Professional information with education, experience, and skills

### 3. Blockchain-Powered Peer Review
- Anonymous peer review assignment through smart contracts
- Conflict-of-interest detection
- Three-reviewer consensus requirement
- On-chain review history with immutable timestamps

### 4. Financial Infrastructure
- APC (Article Processing Charge) escrow system secured by Solana smart contracts
- Automatic escrow release upon peer review consensus
- Refund mechanism for rejected submissions
- Tiered subscription plans (Basic: $9.99/mo, Premium: $19.99/mo, Institutional: $49.99/mo)

### 5. SPL Token Badge System
- Merit-based reviewer incentivization
- Tiered badges (Bronze, Silver, Gold, Platinum)
- On-chain validation of reviewer contributions
- Transferable badges as tokenized credentials

## Technology Stack

### Frontend
- **Framework**: React with TypeScript
- **Routing**: Wouter for lightweight routing
- **UI Components**: Shadcn UI with Tailwind CSS
- **State Management**: TanStack Query (React Query)
- **Form Handling**: React Hook Form with Zod validation

### Backend
- **Server**: Express.js with TypeScript
- **Database**: PostgreSQL with Drizzle ORM
- **Authentication**: Passport.js with Web3 wallet integration

### Blockchain Integration
- **Network**: Solana (Devnet/Mainnet)
- **Smart Contracts**: Anchor framework for Solana programs
- **Wallet Integration**: Wallet Adapter for multiple Solana wallets
- **Token Standard**: Solana SPL tokens for badges

### Storage
- **Distributed Storage**: IPFS via Web3.Storage
- **Permanent Storage**: Arweave integration
- **Database**: PostgreSQL for relational data

## Project Structure

```
fronsciers/
├── client/                   # Frontend React application
│   ├── src/                  
│   │   ├── components/       # Reusable UI components
│   │   ├── pages/            # Page components
│   │   ├── hooks/            # Custom React hooks
│   │   ├── lib/              # Utility functions
│   │   └── utils/            # Helper functions
├── server/                   # Backend Express application
│   ├── routes/               # API routes
│   ├── services/             # Business logic
│   └── storage.ts            # Database interface
├── shared/                   # Shared types and schemas
│   └── schema.ts             # Database schema definitions
├── solana-program/           # Solana smart contracts
│   ├── apc-escrow/           # APC escrow program
│   ├── badge-program/        # SPL token badge program
│   └── doci-registry/        # DOCI registration program
└── migrations/               # Database migrations
```

## Smart Contract Details

### APC Escrow Program
Handles the article processing charge escrow system with:
- Manuscript ID tracking
- Required approval thresholds
- Automatic fund release upon consensus
- Refund mechanisms for rejected manuscripts

```rust
// Key functions in the APC escrow program
pub fn initialize(ctx: Context<Initialize>, manuscript_id: u64, amount: u64, required_approvals: u8) -> Result<()>
pub fn fund(ctx: Context<Fund>, amount: u64) -> Result<()>
pub fn approve(ctx: Context<Approve>) -> Result<()>
pub fn release(ctx: Context<Release>) -> Result<()>
pub fn refund(ctx: Context<Refund>) -> Result<()>
```

### DOCI Registry Program
Manages the registration and resolution of Direct On-Chain Identifiers:
- Custom prefix/suffix generation
- Metadata storage and retrieval
- Ownership verification
- Citation tracking

### Badge Issuance Program
Handles the SPL token badge system for reviewer incentives:
- Badge minting based on review quality and quantity
- Tiered badge levels with different requirements
- Transferable badges as tokenized credentials

## Getting Started

### Prerequisites
- Node.js (v18+)
- PostgreSQL database
- Solana CLI tools (for local blockchain development)
- Solana wallet (Phantom, Jupiter, etc.)

### Installation

1. Clone the repository:
```bash
git clone https://github.com/rejoelm/fronsciers.git
cd fronsciers
```

2. Install dependencies:
```bash
npm install
```

3. Set up environment variables:
Create a `.env` file in the root directory with the following variables:
```
DATABASE_URL=postgresql://username:password@localhost:5432/fronsciers
STRIPE_SECRET_KEY=your_stripe_secret_key
VITE_STRIPE_PUBLIC_KEY=your_stripe_public_key
```

4. Initialize the database:
```bash
npm run db:push
```

5. Start the development server:
```bash
npm run dev
```

This will start both the frontend and backend servers concurrently.

6. Access the application:
Open your browser and navigate to `http://localhost:5000`

## Payment Integration

Fronsciers integrates with Stripe for subscription management and USDC for on-chain payments:

### Subscription Plans
- Basic: $9.99/mo or $99.99/yr
- Premium: $19.99/mo or $199.99/yr
- Institutional: $49.99/mo or $499.99/yr

### On-Chain Payments
- DOCI Minting: $50 USDC fee
- APC Payment: Variable based on publication type

## Deployment

The application is configured for deployment on Replit with automatic HTTPS and domain provisioning.

## Development Team

The Fronsciers platform was developed by Rejoel Mangasa Siagian and team from the Faculty of Medicine, Universitas Indonesia.

---

For questions or support, please contact: rejoel.mangasa@ui.ac.id
