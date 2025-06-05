# Fronsciers Architecture Code Documentation

This document provides details about the architecture and code implementation of the Fronsciers platform.

## Directory Structure

```
/
├── client/                # Frontend React application
│   ├── src/
│   │   ├── components/    # Reusable UI components
│   │   │   ├── DociCopyButton.tsx           # Button for copying DOCIs
│   │   │   ├── DociExplainer.tsx            # Component explaining DOCI usage
│   │   │   ├── ResearcherDociCard.tsx       # Researcher profile card with DOCI
│   │   │   └── [other UI components]
│   │   ├── pages/                           # Page components
│   │   │   ├── Dashboard.tsx                # Main dashboard
│   │   │   ├── DociResolverPage.tsx         # DOCI resolver implementation
│   │   │   ├── UserPortfolio.tsx            # Researcher profile page
│   │   │   └── [other page components]
│   │   ├── layouts/       # Layout components
│   │   ├── data/          # Sample data
│   │   │   ├── sampleArticles.ts            # Sample publications with DOCIs
│   │   │   └── sampleResearchers.ts         # Sample researcher profiles with DOCIs
│   │   ├── lib/           # Utility functions and configurations
│   │   └── App.tsx        # Main application component
├── server/                # Backend Express application
│   ├── routes/            # API route definitions
│   │   ├── doci-resolver-routes.ts          # DOCI resolver API endpoints
│   │   └── [other route files]
│   ├── services/          # Business logic services
│   ├── db.ts              # Database connection setup
│   ├── storage.ts         # Data storage interface implementation
│   └── index.ts           # Express server setup
├── shared/                # Shared code between client and server
│   └── schema.ts          # Database schema and type definitions
└── solana-program/        # Solana blockchain smart contracts
    └── [program files]    # Anchor framework program files
```

## Key Components Implementation

### 1. DOCI Resolver System

The DOCI resolver is the core feature that allows direct access to content via DOCIs.

#### Backend Implementation (server/routes/doci-resolver-routes.ts)

```typescript
// DOCI resolver endpoint that handles both researcher and publication DOCIs
router.get('/api/resolve/:prefix/:suffix', async (req: Request, res: Response) => {
  try {
    const { prefix, suffix } = req.params;
    const fullDoci = `${prefix}/${suffix}`;
    
    // Check if this is a researcher DOCI (has the R- pattern in suffix)
    if (suffix.startsWith('R-')) {
      // Look up researcher in sample data
      const researcher = sampleResearchers.find(r => r.doci === fullDoci);
      
      if (researcher) {
        // Record resolution for analytics
        await storage.recordDoiResolution({
          doiId: researcher.id,
          userAgent: req.headers['user-agent'] || null,
          ipAddress: req.ip || null
        });
        
        return res.json({
          exists: true,
          type: 'researcher',
          data: researcher
        });
      }
    } else {
      // Look up publication in sample data
      const publication = sampleArticles.find(p => p.doci === fullDoci);
      
      if (publication) {
        // Record resolution for analytics
        await storage.recordDoiResolution({
          doiId: publication.id,
          userAgent: req.headers['user-agent'] || null,
          ipAddress: req.ip || null
        });
        
        return res.json({
          exists: true,
          type: 'publication',
          data: publication
        });
      }
    }
    
    // If we get here, the DOCI was not found
    return res.status(404).json({
      exists: false,
      message: `DOCI ${fullDoci} not found`
    });
  } catch (error) {
    console.error('Error resolving DOCI:', error);
    return res.status(500).json({
      exists: false,
      message: 'An error occurred while resolving the DOCI'
    });
  }
});
```

#### Frontend Implementation (client/src/pages/DociResolverPage.tsx)

```typescript
export default function DociResolverPage() {
  const { prefix, suffix } = useParams();
  const { toast } = useToast();
  const [copied, setCopied] = useState(false);
  
  const fullDoci = `${prefix}/${suffix}`;
  
  // Fetch DOCI data from our API
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

  // Render different views based on the DOCI type (researcher or publication)
  if (dociData?.type === "researcher") {
    return <ResearcherProfileView data={dociData.data} doci={fullDoci} />;
  }

  if (dociData?.type === "publication") {
    return <PublicationView data={dociData.data} doci={fullDoci} />;
  }
  
  // Default fallback or error states
  // ...
}
```

### 2. Database Schema

The database schema defines the structure of our data (shared/schema.ts):

```typescript
// User model
export const users = pgTable("users", {
  id: serial("id").primaryKey(),
  username: text("username").notNull().unique(),
  email: text("email"),
  name: text("name"),
  institution: text("institution"),
  role: text("role"),
  walletAddress: text("wallet_address").unique(),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull(),
});

// Researcher DOCI model
export const researcherDocis = pgTable("researcher_docis", {
  id: serial("id").primaryKey(),
  userId: integer("user_id").references(() => users.id).notNull(),
  prefix: text("prefix").notNull(),
  suffix: text("suffix").notNull(),
  fullDoci: text("full_doci").notNull().unique(),
  onChainAddress: text("on_chain_address"),
  transactionId: text("transaction_id"),
  status: text("status", { enum: ["pending", "active", "revoked"] }).default("active").notNull(),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull(),
});

// Publication DOI/DOCI model
export const dois = pgTable("dois", {
  id: serial("id").primaryKey(),
  userId: integer("user_id").references(() => users.id).notNull(),
  prefix: text("prefix").notNull(),
  suffix: text("suffix").notNull(),
  fullDoi: text("full_doi").notNull().unique(),
  title: text("title").notNull(),
  authors: text("authors").notNull(),
  abstract: text("abstract"),
  publicationType: text("publication_type", { 
    enum: Object.values(PublicationType) 
  }).notNull(),
  publicationStatus: text("publication_status", { 
    enum: Object.values(PublicationStatus) 
  }).default("published").notNull(),
  journal: text("journal"),
  year: integer("year"),
  url: text("url"),
  ipfsHash: text("ipfs_hash"),
  onChainAddress: text("on_chain_address"),
  transactionId: text("transaction_id"),
  status: text("status", { enum: ["pending", "active", "revoked"] }).default("active").notNull(),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull(),
});

// DOI Resolution tracking
export const doiResolutions = pgTable("doi_resolutions", {
  id: serial("id").primaryKey(),
  doiId: integer("doi_id").references(() => dois.id),
  userAgent: text("user_agent"),
  ipAddress: text("ip_address"),
  createdAt: timestamp("created_at").defaultNow().notNull(),
});

// Transaction records
export const transactions = pgTable("transactions", {
  id: serial("id").primaryKey(),
  userId: integer("user_id").references(() => users.id).notNull(),
  doiId: integer("doi_id").references(() => dois.id),
  transactionType: text("transaction_type").notNull(),
  amount: numeric("amount"),
  currency: text("currency"),
  walletAddress: text("wallet_address"),
  transactionId: text("transaction_id"),
  status: text("status").notNull(),
  metadata: jsonb("metadata"),
  createdAt: timestamp("created_at").defaultNow().notNull(),
});
```

### 3. DOCI Copy Button Component

This reusable component allows for easy sharing of DOCIs (client/src/components/DociCopyButton.tsx):

```typescript
interface DociCopyButtonProps {
  doci: string;
  variant?: 'default' | 'outline' | 'secondary' | 'ghost' | 'link' | 'destructive';
  size?: 'default' | 'sm' | 'lg' | 'icon';
  className?: string;
  showText?: boolean;
}

/**
 * A reusable button component for copying DOCIs to clipboard
 */
export default function DociCopyButton({
  doci,
  variant = 'outline',
  size = 'sm',
  className = '',
  showText = true
}: DociCopyButtonProps) {
  const { toast } = useToast();
  const [copied, setCopied] = useState(false);

  const copyToClipboard = () => {
    const dociUrl = `${window.location.origin}/doci/${doci}`;
    navigator.clipboard.writeText(dociUrl);
    setCopied(true);
    
    toast({
      title: "DOCI link copied!",
      description: `The direct link to ${doci} has been copied to your clipboard.`
    });
    
    setTimeout(() => setCopied(false), 2000);
  };

  return (
    <Button 
      variant={variant} 
      size={size} 
      onClick={copyToClipboard}
      className={`flex items-center gap-1 ${className}`}
    >
      {copied ? <CheckCircle2 className="h-4 w-4" /> : <Copy className="h-4 w-4" />}
      {showText && (copied ? "Copied" : "Copy DOCI")}
    </Button>
  );
}
```

### 4. DOCI Explainer Component

Explains the DOCI system to users (client/src/components/DociExplainer.tsx):

```typescript
export default function DociExplainer() {
  // Get Rejoel's researcher profile as example
  const researcherProfile = sampleResearchers.find(r => r.name === "Rejoel Mangasa Siagian");
  const researcherPublications = sampleArticles.filter(a => a.authorDoci === researcherProfile?.doci);
  
  return (
    <Card className="bg-white dark:bg-odyssey-dark/30 shadow-lg">
      <CardHeader>
        <CardTitle className="text-2xl font-bold text-gradient">DOCI System Explained</CardTitle>
        <CardDescription>
          Direct On-Chain Identifiers (DOCIs) transform how research is identified, accessed, and shared
        </CardDescription>
      </CardHeader>
      <CardContent>
        <Tabs defaultValue="use-cases" className="w-full">
          <TabsList className="grid grid-cols-3 mb-4">
            <TabsTrigger value="use-cases">Use Cases</TabsTrigger>
            <TabsTrigger value="examples">Live Examples</TabsTrigger>
            <TabsTrigger value="benefits">Benefits</TabsTrigger>
          </TabsList>
          
          <TabsContent value="use-cases">
            {/* Use Cases content */}
          </TabsContent>
          
          <TabsContent value="examples">
            {/* Examples content */}
          </TabsContent>
          
          <TabsContent value="benefits">
            {/* Benefits content */}
          </TabsContent>
        </Tabs>
      </CardContent>
    </Card>
  );
}
```

### 5. Storage Interface Implementation

The storage interface provides a consistent way to access data (server/storage.ts):

```typescript
export interface IStorage {
  // User operations
  getUser(id: number): Promise<User | undefined>;
  getUserByUsername(username: string): Promise<User | undefined>;
  getUserByWalletAddress(walletAddress: string): Promise<User | undefined>;
  createUser(user: InsertUser): Promise<User>;
  
  // Researcher DOCI operations
  getResearcherDoci(id: number): Promise<ResearcherDoci | undefined>;
  getResearcherDociByUserId(userId: number): Promise<ResearcherDoci | undefined>;
  getResearcherDociByFullDoci(fullDoci: string): Promise<ResearcherDoci | undefined>;
  createResearcherDoci(doci: InsertResearcherDoci): Promise<ResearcherDoci>;
  updateResearcherDoci(id: number, doci: Partial<InsertResearcherDoci>): Promise<ResearcherDoci | undefined>;
  getNextResearcherDoiSuffix(): Promise<string>;
  
  // DOI operations
  getDoi(id: number): Promise<Doi | undefined>;
  getDoiByFullDoi(fullDoi: string): Promise<Doi | undefined>;
  getDoisByUser(userId: number): Promise<Doi[]>;
  getRecentDois(limit?: number): Promise<Doi[]>;
  createDoi(doi: InsertDoi): Promise<Doi>;
  updateDoi(id: number, doi: Partial<InsertDoi>): Promise<Doi | undefined>;
  getDoiStats(): Promise<{ totalDois: number, resolutionCount: number, transactionCount: number }>;
  getNextDoiSuffix(): Promise<string>;
  
  // Resolution operations
  recordDoiResolution(resolution: InsertDoiResolution): Promise<DoiResolution>;
  getDoiResolutionCountByDoi(doiId: number): Promise<number>;
  
  // Additional operations...
}

export class DatabaseStorage implements IStorage {
  // Implementation of the interface methods using PostgreSQL and Drizzle ORM
  // ...
}
```

### 6. Solana Blockchain Integration

Example of a Solana blockchain integration for minting DOCIs (solana-program/programs/doci-registry/src/lib.rs):

```rust
#[program]
pub mod doci_registry {
    use super::*;

    // Initialize the DOCI registry
    pub fn initialize(ctx: Context<Initialize>, authority: Pubkey) -> Result<()> {
        let registry = &mut ctx.accounts.registry;
        registry.authority = authority;
        registry.doci_count = 0;
        Ok(())
    }

    // Mint a new DOCI
    pub fn mint_doci(
        ctx: Context<MintDoci>,
        prefix: String,
        suffix: String,
        metadata_uri: String,
    ) -> Result<()> {
        let registry = &mut ctx.accounts.registry;
        let doci = &mut ctx.accounts.doci;
        
        // Set DOCI data
        doci.owner = *ctx.accounts.owner.key;
        doci.prefix = prefix;
        doci.suffix = suffix;
        doci.metadata_uri = metadata_uri;
        doci.created_at = Clock::get()?.unix_timestamp;
        doci.updated_at = Clock::get()?.unix_timestamp;
        doci.active = true;
        
        // Update registry
        registry.doci_count += 1;
        
        emit!(DociMinted {
            owner: *ctx.accounts.owner.key,
            doci_address: *doci.to_account_info().key,
            prefix: doci.prefix.clone(),
            suffix: doci.suffix.clone(),
        });
        
        Ok(())
    }

    // Update DOCI metadata
    pub fn update_doci(
        ctx: Context<UpdateDoci>,
        new_metadata_uri: String,
    ) -> Result<()> {
        let doci = &mut ctx.accounts.doci;
        
        // Only owner can update
        require!(
            *ctx.accounts.owner.key == doci.owner,
            DoiError::Unauthorized
        );
        
        // Update metadata
        doci.metadata_uri = new_metadata_uri;
        doci.updated_at = Clock::get()?.unix_timestamp;
        
        emit!(DociUpdated {
            owner: *ctx.accounts.owner.key,
            doci_address: *doci.to_account_info().key,
            prefix: doci.prefix.clone(),
            suffix: doci.suffix.clone(),
        });
        
        Ok(())
    }
}

#[derive(Accounts)]
pub struct MintDoci<'info> {
    #[account(mut)]
    pub registry: Account<'info, Registry>,
    
    #[account(
        init,
        payer = owner,
        space = 8 + DOCI_SIZE,
        seeds = [b"doci", registry.key().as_ref(), prefix.as_bytes(), suffix.as_bytes()],
        bump
    )]
    pub doci: Account<'info, Doci>,
    
    #[account(mut)]
    pub owner: Signer<'info>,
    
    pub system_program: Program<'info, System>,
}

#[account]
pub struct Doci {
    pub owner: Pubkey,            // 32 bytes
    pub prefix: String,           // 4 + max 20 bytes
    pub suffix: String,           // 4 + max 20 bytes
    pub metadata_uri: String,     // 4 + max 200 bytes
    pub created_at: i64,          // 8 bytes
    pub updated_at: i64,          // 8 bytes
    pub active: bool,             // 1 byte
}
```

## Frontend-Backend Integration

The frontend integrates with the backend through API calls for DOCI resolution and data retrieval. The Solana blockchain is used to store the authoritative record of DOCIs, while IPFS/Arweave is used for storing larger documents and metadata.

### API Integration Example

```typescript
// Client-side query
const { data: dociData, isLoading } = useQuery({
  queryKey: ['/api/resolve', prefix, suffix],
  queryFn: async () => {
    const response = await fetch(`/api/resolve/${prefix}/${suffix}`);
    if (!response.ok) {
      throw new Error('Failed to resolve DOCI');
    }
    return response.json();
  }
});

// Server-side handler
app.get('/api/resolve/:prefix/:suffix', async (req, res) => {
  try {
    const { prefix, suffix } = req.params;
    const fullDoci = `${prefix}/${suffix}`;
    
    // First check database cache
    const existingDoi = await storage.getDoiByFullDoi(fullDoci);
    if (existingDoi) {
      return res.json({
        exists: true,
        type: 'publication',
        data: existingDoi
      });
    }
    
    // Check if researcher DOCI
    const existingResearcher = await storage.getResearcherDociByFullDoci(fullDoci);
    if (existingResearcher) {
      return res.json({
        exists: true,
        type: 'researcher',
        data: existingResearcher
      });
    }
    
    // If not in database, verify on blockchain
    // This would use the Solana Web3.js library to check on-chain
    const onChainData = await verifyDociOnChain(prefix, suffix);
    if (onChainData) {
      // If found on chain but not in database, retrieve IPFS data
      const ipfsData = await retrieveFromIpfs(onChainData.metadata_uri);
      
      // Return data and cache in database for future access
      return res.json({
        exists: true,
        type: ipfsData.type,
        data: ipfsData
      });
    }
    
    return res.status(404).json({
      exists: false,
      message: `DOCI ${fullDoci} not found`
    });
  } catch (error) {
    console.error('Error resolving DOCI:', error);
    return res.status(500).json({
      exists: false,
      message: 'An error occurred while resolving the DOCI'
    });
  }
});
```

This architecture documentation provides a technical overview of how the Fronsciers platform is built, focusing on the DOCI system implementation and the integration between frontend, backend, and blockchain components.