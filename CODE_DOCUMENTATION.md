# Fronsciers Code Documentation

## Core Components

### 1. Database Schema (shared/schema.ts)
The backbone of the application is the database schema which defines the structure for:
- DOCIs (Direct On-Chain Identifiers)
- Researcher profiles
- Submissions
- Reviews
- Citations
- Transactions
- Badges

```typescript
// Example schema definition
export const dois = pgTable('dois', {
  id: serial('id').primaryKey(),
  prefix: text('prefix').notNull(),
  suffix: text('suffix').notNull(),
  title: text('title').notNull(),
  authors: text('authors').notNull(),
  abstract: text('abstract'),
  type: text('type').notNull().default('article'),
  status: text('status').notNull().default('published'),
  version: integer('version').notNull().default(1),
  sourceUri: text('source_uri'),
  metadataUri: text('metadata_uri'),
  txHash: text('tx_hash'),
  txTimestamp: timestamp('tx_timestamp'),
  userId: integer('user_id').references(() => users.id),
});
```

### 2. Storage Layer (server/storage.ts)
The DatabaseStorage class provides a complete interface for database operations, implementing the IStorage interface:

```typescript
export class DatabaseStorage implements IStorage {
  async getDoi(id: number): Promise<Doi | undefined> {
    const [doi] = await db.select().from(dois).where(eq(dois.id, id));
    return doi || undefined;
  }
  
  async createDoi(insertDoi: InsertDoi): Promise<Doi> {
    const [doi] = await db.insert(dois).values(insertDoi).returning();
    return doi;
  }
  
  // Additional methods for all entities (researchers, submissions, reviews, etc.)
}
```

### 3. API Routes (server/routes.ts)
The API routes handle HTTP requests and use the storage layer for database operations:

```typescript
export async function registerRoutes(app: Express): Promise<Server> {
  // DOCI resolution endpoint
  app.get("/api/resolve/:prefix/:suffix", async (req: Request, res: Response) => {
    try {
      const { prefix, suffix } = req.params;
      const fullDoi = `${prefix}/${suffix}`;
      
      // First try to resolve as publication DOCI
      const doi = await storage.getDoiByFullDoi(fullDoi);
      if (doi) {
        await storage.recordDoiResolution({
          doiId: doi.id,
          timestamp: new Date(),
          referrer: req.headers.referer || null
        });
        
        return res.json({
          type: "publication",
          exists: true,
          data: doi
        });
      }
      
      // Try to resolve as researcher DOCI
      if (prefix === "10.FRONS-R") {
        const researcher = await storage.getResearcherDociByFullDoci(fullDoi);
        if (researcher) {
          return res.json({
            type: "researcher",
            exists: true,
            data: researcher
          });
        }
      }
      
      throw new Error("DOCI not found");
    } catch (error) {
      console.error("Error resolving DOCI:", error);
      res.status(500).json({
        exists: false,
        message: error.message || "Failed to resolve DOCI"
      });
    }
  });

  // Many more API endpoints for DOCI management, submissions, reviews, etc.
}
```

### 4. Solana Smart Contracts

#### APC Escrow Program (solana-program/apc-escrow/src/lib.rs)
Manages article processing charges through a Solana smart contract:

```rust
#[program]
pub mod apc_escrow {
    use super::*;

    // Initialize an escrow for an Article Processing Charge
    pub fn initialize(
        ctx: Context<Initialize>,
        manuscript_id: u64,
        amount: u64,
        required_approvals: u8,
    ) -> Result<()> {
        let escrow = &mut ctx.accounts.escrow;
        escrow.payer = ctx.accounts.payer.key();
        escrow.manuscript_id = manuscript_id;
        escrow.amount = amount;
        // Additional initialization...
        Ok(())
    }

    // Fund the escrow with tokens
    pub fn fund(ctx: Context<Fund>, amount: u64) -> Result<()> {
        // Fund implementation...
        Ok(())
    }
    
    // Additional methods for approval, release, and refund
}
```

#### Badge Program (solana-program/badge-program/src/lib.rs)
Issues SPL tokens as badges for reviewer contributions:

```rust
#[program]
pub mod badge_program {
    use super::*;

    // Issue a badge to a reviewer
    pub fn issue_badge(
        ctx: Context<IssueBadge>,
        badge_type: BadgeType,
        reviewer_id: String,
    ) -> Result<()> {
        // Badge issuance implementation...
        Ok(())
    }
}
```

### 5. React Frontend Components

#### Researcher Profile (client/src/pages/ResearcherProfile.tsx)
Displays researcher profiles with their academic credentials:

```typescript
const ResearcherProfile = () => {
  const { dociId } = useParams<{ dociId: string }>();
  const { toast } = useToast();
  const [researcher, setResearcher] = useState<any>(null);

  useEffect(() => {
    // Fetch researcher data
    const fetchResearcher = async () => {
      try {
        const response = await apiRequest("GET", `/api/researchers/${dociId}`);
        const data = await response.json();
        setResearcher(data);
      } catch (error) {
        // Error handling...
      }
    };
    
    fetchResearcher();
  }, [dociId]);

  // JSX for rendering the profile with tabs for different sections
  return (
    <div className="container mx-auto py-8 px-4">
      {/* Profile header with avatar and basic info */}
      <Card className="mb-8 overflow-hidden">
        {/* Researcher information */}
      </Card>
      
      {/* Tabs for About, Experience, Publications, etc. */}
      <Tabs value={activeTab} onValueChange={setActiveTab}>
        <TabsList>
          <TabsTrigger value="about">About</TabsTrigger>
          <TabsTrigger value="experience">Experience</TabsTrigger>
          <TabsTrigger value="publications">Publications</TabsTrigger>
          {/* Additional tabs */}
        </TabsList>
        
        {/* Tab content for each section */}
      </Tabs>
    </div>
  );
};
```

#### DOCI Resolver (client/src/pages/DociResolverPage.tsx)
Resolves and displays DOCI content:

```typescript
export default function DociResolverPage() {
  const { prefix, suffix } = useParams();
  const { toast } = useToast();
  const fullDoci = `${prefix}/${suffix}`;
  
  // Fetch DOCI data
  const { data: dociData, isLoading, error } = useQuery({
    queryKey: ['/api/resolve', prefix, suffix],
    queryFn: async () => {
      const response = await fetch(`/api/resolve/${prefix}/${suffix}`);
      if (!response.ok) {
        throw new Error('Failed to resolve DOCI');
      }
      return response.json();
    }
  });

  // Render different views based on DOCI type (publication vs. researcher)
  if (dociData?.type === "researcher") {
    return <ResearcherView data={dociData.data} />;
  } else if (dociData?.type === "publication") {
    return <PublicationView data={dociData.data} />;
  }
  
  // Loading and error states
}
```

#### Browse Research (client/src/pages/BrowseResearch.tsx)
Lists published research with filtering capabilities:

```typescript
export default function BrowseResearch() {
  const [searchQuery, setSearchQuery] = useState("");
  const [typeFilter, setTypeFilter] = useState("all");
  
  // Fetch DOCIs
  const { data: dois, isLoading, error } = useQuery({
    queryKey: ['/api/dois'],
    queryFn: async () => {
      const response = await fetch('/api/dois');
      if (!response.ok) {
        throw new Error('Failed to fetch DOCIs');
      }
      return response.json();
    }
  });

  // Filter DOCIs based on search and type filter
  const filteredDois = useMemo(() => {
    if (!dois) return [];
    
    return dois.filter(doi => {
      // Filter implementation...
    });
  }, [dois, searchQuery, typeFilter]);

  // Render the list of DOCIs with search and filtering UI
}
```

### 6. Solana Integration JavaScript Utilities

#### APC Escrow Utility (client/src/utils/solanaEscrow.ts)
Interacts with the APC escrow smart contract:

```typescript
export function useApcEscrow() {
  const { publicKey, signTransaction } = useWallet();
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // Initialize a new APC escrow account
  const initializeEscrow = async (manuscriptId: number, amount: number, requiredApprovals: number = 3) => {
    if (!publicKey) {
      setError("Wallet not connected");
      return null;
    }

    setLoading(true);
    setError(null);

    try {
      const program = getProgram();
      
      // Convert amount to USDC units (6 decimals)
      const usdcAmount = new BN(amount * 1000000);
      
      // Get the escrow PDA
      const escrowPda = await getEscrowPDA(manuscriptId, publicKey);
      
      // Get the escrow token account PDA
      const escrowTokenAccountPda = await getEscrowTokenAccountPDA(escrowPda);

      // Initialize the escrow on-chain
      const tx = await program.methods
        .initialize(
          new BN(manuscriptId),
          usdcAmount,
          requiredApprovals
        )
        .accounts({
          escrow: escrowPda,
          escrowTokenAccount: escrowTokenAccountPda,
          tokenMint: USDC_MINT,
          payer: publicKey,
          systemProgram: SystemProgram.programId,
          tokenProgram: TOKEN_PROGRAM_ID,
          associatedTokenProgram: ASSOCIATED_TOKEN_PROGRAM_ID,
          rent: web3.SYSVAR_RENT_PUBKEY,
        })
        .rpc();

      return {
        escrowAddress: escrowPda.toString(),
        tokenAccountAddress: escrowTokenAccountPda.toString(),
        transactionSignature: tx,
      };
    } catch (err) {
      // Error handling...
      return null;
    } finally {
      setLoading(false);
    }
  };

  // Additional methods for funding, approving, releasing, and refunding escrow
  return {
    initializeEscrow,
    fundEscrow,
    approveEscrow,
    releaseEscrow,
    refundEscrow,
    loading,
    error
  };
}
```

#### Badge Utility (client/src/utils/solanaBadge.ts)
Manages SPL token badge issuance and verification:

```typescript
export function useBadgeProgram() {
  const { publicKey, signTransaction } = useWallet();

  // Issue a badge to a reviewer
  const issueBadge = async (reviewerId: string, badgeType: BadgeType) => {
    // Implementation for badge issuance
  };

  // Verify a badge
  const verifyBadge = async (badgeId: string) => {
    // Implementation for badge verification
  };

  return {
    issueBadge,
    verifyBadge
  };
}
```

## Key Files Structure Overview

```
fronsciers/
├── client/
│   ├── src/
│   │   ├── components/
│   │   │   ├── WalletConnection.tsx   # Solana wallet integration
│   │   │   ├── SearchBar.tsx          # Global search component
│   │   │   ├── BadgeIssuance.tsx      # Badge issuance interface
│   │   │   └── BlockchainDoiCitation.tsx # Citation tracking
│   │   ├── pages/
│   │   │   ├── ResearcherProfile.tsx  # Researcher profile page
│   │   │   ├── DociResolverPage.tsx   # DOCI resolver
│   │   │   ├── BrowseResearch.tsx     # Research listing page
│   │   │   ├── AuthorDashboard.tsx    # Author dashboard
│   │   │   ├── Subscription.tsx       # Subscription management
│   │   │   └── ApcPayment.tsx         # APC payment interface
│   │   ├── utils/
│   │   │   ├── solanaEscrow.ts        # APC escrow interaction
│   │   │   └── solanaSubscription.ts  # Subscription management
│   │   └── ...
├── server/
│   ├── routes/
│   │   ├── doci-resolver-routes.ts    # DOCI resolution endpoints
│   │   └── researcher-profile-routes.ts # Researcher API endpoints
│   ├── storage.ts                     # Database interface
│   └── routes.ts                      # API route registration
├── shared/
│   └── schema.ts                      # Database schema definition
├── solana-program/
│   ├── apc-escrow/                    # APC escrow smart contract
│   ├── doci-registry/                 # DOCI registration contract
│   └── badge-program/                 # Badge issuance contract
└── migrations/                        # Database migrations
```

## Technical Implementation Details

### 1. DOCI System
The DOCI system is implemented as a composite of on-chain and off-chain components:
- On-chain: Solana program for immutable registration of identifiers
- Off-chain: PostgreSQL database for efficient querying and metadata storage
- Resolution system: Express.js API endpoints that check both on-chain and off-chain data

### 2. Peer Review Mechanism
The peer review process is managed through a multi-signature Solana program:
- Three reviewers must approve a manuscript before funds are released
- Each review is recorded as a transaction on the Solana blockchain
- The APC escrow program manages the financial aspects of the process

### 3. Token Badge System
SPL tokens are used to create badge credentials for reviewers:
- Badges are minted as non-fungible SPL tokens
- Metadata includes reviewer information and achievement level
- Token accounts are associated with reviewer wallets

### 4. Subscription System
The subscription system combines traditional payment processing with blockchain verification:
- Stripe for recurring billing of subscription plans
- On-chain verification of subscription status for premium features
- Tiered access control based on subscription level

### 5. Researcher Profiles
Researcher profiles leverage the DOCI system with a specialized prefix:
- Custom 10.FRONS-R prefix identifies researcher profiles
- Unique suffixes for each researcher
- Profile data stored in PostgreSQL with on-chain verification
- Publications and citations tracked through relationship tables

This documentation provides a comprehensive overview of the Fronsciers codebase and its technical implementation, covering all major components from the database schema to the Solana blockchain integration.