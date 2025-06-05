# Fronsciers Frontend Code

## Core React Components

### client/src/components/ResearchJourneyProgressBar.tsx
```typescript
import React from 'react';
import { JourneyStage } from '@/lib/types';

interface ResearchJourneyProgressBarProps {
  userId: number;
  publicationId?: number;
  currentStage: JourneyStage;
  completedStages: JourneyStage[];
  compact?: boolean;
  onStageClick?: (stage: JourneyStage) => void;
}

const ResearchJourneyProgressBar: React.FC<ResearchJourneyProgressBarProps> = ({
  userId,
  publicationId,
  currentStage,
  completedStages,
  compact = false,
  onStageClick
}) => {
  // Define stages in order
  const stages = [
    { key: JourneyStage.DRAFT, label: "Draft", icon: "📝" },
    { key: JourneyStage.SUBMISSION, label: "Submission", icon: "📤" },
    { key: JourneyStage.PEER_REVIEW, label: "Peer Review", icon: "👥" },
    { key: JourneyStage.REVISION, label: "Revision", icon: "✏️" },
    { key: JourneyStage.ACCEPTANCE, label: "Acceptance", icon: "✅" },
    { key: JourneyStage.PUBLICATION, label: "Publication", icon: "📖" },
    { key: JourneyStage.INDEXING, label: "Indexing", icon: "🔍" },
    { key: JourneyStage.CITATION, label: "Citation", icon: "📣" }
  ];
  
  // Calculate progress percentage
  const currentIndex = stages.findIndex(s => s.key === currentStage);
  const totalStages = stages.length;
  // If the current stage is completed, count it fully. If it's in progress, count it as halfway.
  const completedPercentage = Math.round(((currentIndex + (completedStages.includes(currentStage) ? 1 : 0.5)) / totalStages) * 100);
  
  // For compact view
  if (compact) {
    return (
      <div className="space-y-2">
        <div className="flex justify-between items-center">
          <div className="flex items-center gap-2">
            <div className="w-3 h-3 rounded-full bg-odyssey-medium"></div>
            <span className="text-sm">{stages.find(s => s.key === currentStage)?.label} Stage</span>
          </div>
          <span className="text-xs text-muted-foreground">
            {currentIndex + 1}/{totalStages}
          </span>
        </div>
        
        <div className="h-2 bg-muted rounded-full overflow-hidden">
          <div 
            className="h-full bg-gradient-to-r from-odyssey-dark to-odyssey-medium rounded-full"
            style={{ width: `${completedPercentage}%` }}
          ></div>
        </div>
        
        <div className="flex justify-between text-xs text-muted-foreground">
          <span>{stages[0].label}</span>
          <span>{stages[stages.length - 1].label}</span>
        </div>
      </div>
    );
  }
  
  // For full view
  return (
    <div className="space-y-8">
      {/* Main progress bar */}
      <div className="h-2.5 bg-muted rounded-full overflow-hidden">
        <div 
          className="h-full bg-gradient-to-r from-odyssey-dark to-odyssey-medium rounded-full"
          style={{ width: `${completedPercentage}%` }}
        ></div>
      </div>
      
      {/* Stage markers */}
      <div className="relative pt-2">
        {/* Connector line */}
        <div className="absolute left-0 right-0 h-0.5 bg-muted top-1/2 -translate-y-1/2"></div>
        
        <div className="flex justify-between relative">
          {stages.map((stage, index) => {
            const isCompleted = completedStages.includes(stage.key);
            const isCurrent = stage.key === currentStage;
            const isUpcoming = !isCompleted && !isCurrent;
            
            return (
              <div 
                key={stage.key} 
                className="flex flex-col items-center"
                onClick={() => onStageClick && onStageClick(stage.key)}
                style={{ cursor: onStageClick ? 'pointer' : 'default' }}
              >
                <div className={`
                  w-8 h-8 rounded-full flex items-center justify-center z-10 border-2
                  ${isCompleted 
                    ? 'bg-green-100 border-green-500 text-green-700 dark:bg-green-900/30 dark:border-green-500 dark:text-green-400' 
                    : isCurrent 
                      ? 'bg-odyssey-light/20 border-odyssey-light text-odyssey-light' 
                      : 'bg-gray-100 border-gray-300 text-gray-500 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400'}
                `}>
                  {isCompleted ? '✓' : stage.icon}
                </div>
                <span className={`
                  text-xs font-medium mt-2
                  ${isCompleted 
                    ? 'text-green-700 dark:text-green-500' 
                    : isCurrent 
                      ? 'text-odyssey-light' 
                      : 'text-gray-500 dark:text-gray-400'}
                `}>
                  {stage.label}
                </span>
              </div>
            );
          })}
        </div>
      </div>
      
      {/* Current stage information */}
      <div className="p-4 bg-odyssey-light/10 border border-odyssey-light/20 rounded-lg mt-2">
        <div className="flex flex-col md:flex-row md:justify-between md:items-center gap-4">
          <div>
            <h3 className="text-lg font-medium text-odyssey-dark dark:text-odyssey-light">
              {stages.find(s => s.key === currentStage)?.label} Stage
            </h3>
            <p className="text-odyssey-medium dark:text-odyssey-light/80 mt-1">
              {getStageDescription(currentStage)}
            </p>
          </div>
          
          <div className="text-sm md:text-right flex gap-2 md:block">
            <div className="bg-white dark:bg-gray-800 px-3 py-1.5 rounded-full border border-odyssey-light/20 inline-block">
              Stage {currentIndex + 1} of {totalStages}
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};

// Helper function to get stage descriptions
function getStageDescription(stage: JourneyStage): string {
  switch (stage) {
    case JourneyStage.DRAFT:
      return "Prepare your manuscript according to Fronsciers guidelines.";
    case JourneyStage.SUBMISSION:
      return "Submit your completed manuscript for review consideration.";
    case JourneyStage.PEER_REVIEW:
      return "Your manuscript is undergoing review by experts in your field.";
    case JourneyStage.REVISION:
      return "Address reviewer feedback and make necessary changes.";
    case JourneyStage.ACCEPTANCE:
      return "Your manuscript has been accepted pending final revisions.";
    case JourneyStage.PUBLICATION:
      return "Your work is being prepared for publication with a DOCI.";
    case JourneyStage.INDEXING:
      return "Your publication is being indexed for greater discoverability.";
    case JourneyStage.CITATION:
      return "Your published work is now citable and can receive citations.";
    default:
      return "Track your publication's progress through the academic lifecycle.";
  }
}

export default ResearchJourneyProgressBar;
```

### client/src/components/WalletConnection.tsx
```typescript
import React, { useState } from 'react';
import { useWallet } from '@solana/wallet-adapter-react';
import { WalletMultiButton } from '@solana/wallet-adapter-react-ui';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { apiRequest } from '@/lib/queryClient';
import { useToast } from '@/hooks/use-toast';
import bs58 from 'bs58';

interface WalletConnectionProps {
  onWalletConnected?: (user: any) => void;
}

const WalletConnection: React.FC<WalletConnectionProps> = ({ onWalletConnected }) => {
  const { publicKey, signMessage, connected } = useWallet();
  const { toast } = useToast();
  const [isVerifying, setIsVerifying] = useState(false);
  
  const handleVerifyWallet = async () => {
    if (!publicKey || !signMessage) {
      toast({
        title: 'Wallet Error',
        description: 'Please connect your wallet with signing capabilities.',
        variant: 'destructive',
      });
      return;
    }
    
    try {
      setIsVerifying(true);
      
      // Create a message for the user to sign
      const message = new TextEncoder().encode(
        `Sign this message to verify your wallet ownership on Fronsciers: ${Date.now()}`
      );
      
      // Ask the wallet to sign the message
      const signature = await signMessage(message);
      const walletAddress = publicKey.toString();
      
      // Send the signature, message, and public key to the server
      const response = await apiRequest('POST', '/api/wallet/connect', {
        walletAddress,
        signature: bs58.encode(signature),
        message: Buffer.from(message).toString('hex'),
      });
      
      if (!response.ok) {
        throw new Error('Failed to verify wallet signature');
      }
      
      const data = await response.json();
      
      toast({
        title: data.isNewUser ? 'New Account Created' : 'Wallet Connected',
        description: data.isNewUser 
          ? 'Your wallet was used to create a new Fronsciers account.' 
          : 'Your wallet was successfully connected to your account.',
      });
      
      if (onWalletConnected) {
        onWalletConnected(data.user);
      }
      
    } catch (error: any) {
      toast({
        title: 'Verification Failed',
        description: error.message || 'Could not verify wallet ownership',
        variant: 'destructive',
      });
    } finally {
      setIsVerifying(false);
    }
  };
  
  return (
    <Card>
      <CardHeader>
        <CardTitle>Connect Your Solana Wallet</CardTitle>
        <CardDescription>
          Link your Solana wallet to mint DOCIs and manage your research assets.
        </CardDescription>
      </CardHeader>
      <CardContent className="space-y-4">
        <div className="flex flex-col items-center gap-4">
          <WalletMultiButton />
          
          {connected && (
            <Button 
              onClick={handleVerifyWallet} 
              disabled={isVerifying}
              className="w-full"
            >
              {isVerifying ? 'Verifying...' : 'Verify Wallet Ownership'}
            </Button>
          )}
        </div>
        
        {connected && publicKey && (
          <div className="text-sm mt-4">
            <div className="font-medium">Connected Wallet</div>
            <div className="text-muted-foreground break-all mt-1">
              {publicKey.toString()}
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
};

export default WalletConnection;
```

### client/src/components/DociMinter.tsx
```typescript
import React, { useState } from 'react';
import { useWallet } from '@solana/wallet-adapter-react';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardDescription, CardHeader, CardTitle, CardFooter } from '@/components/ui/card';
import { Label } from '@/components/ui/label';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Separator } from '@/components/ui/separator';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { uploadToIPFS } from '@/lib/ipfs';
import { mintDOCI } from '@/lib/solanaClient';
import { apiRequest } from '@/lib/queryClient';
import { useToast } from '@/hooks/use-toast';

const DociMinter: React.FC = () => {
  const { publicKey, connected } = useWallet();
  const { toast } = useToast();
  const [isLoading, setIsLoading] = useState(false);
  const [formData, setFormData] = useState({
    title: '',
    authors: '',
    abstract: '',
    doiType: 'article',
    fileToUpload: null as File | null,
  });
  
  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    if (e.target.files && e.target.files.length > 0) {
      setFormData({
        ...formData,
        fileToUpload: e.target.files[0],
      });
    }
  };
  
  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    const { name, value } = e.target;
    setFormData({
      ...formData,
      [name]: value,
    });
  };
  
  const handleSelectChange = (value: string) => {
    setFormData({
      ...formData,
      doiType: value,
    });
  };
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!connected || !publicKey) {
      toast({
        title: 'Wallet Required',
        description: 'Please connect your Solana wallet to mint a DOCI.',
        variant: 'destructive',
      });
      return;
    }
    
    if (!formData.fileToUpload) {
      toast({
        title: 'File Required',
        description: 'Please upload a file for your DOCI.',
        variant: 'destructive',
      });
      return;
    }
    
    try {
      setIsLoading(true);
      
      // 1. Upload the file to IPFS
      const ipfsHash = await uploadToIPFS(formData.fileToUpload);
      
      // 2. Prepare metadata
      const metadata = {
        title: formData.title,
        authors: formData.authors.split(',').map(a => a.trim()),
        abstract: formData.abstract,
        doiType: formData.doiType,
        ipfsHash,
        timestamp: new Date().toISOString(),
        owner: publicKey.toString(),
      };
      
      // 3. Upload metadata to IPFS
      const metadataHash = await uploadToIPFS(
        new Blob([JSON.stringify(metadata)], { type: 'application/json' })
      );
      
      // 4. Mint the DOCI on-chain
      const txSignature = await mintDOCI(
        publicKey,
        '10.FRONS', // Standard prefix for Fronsciers DOCIs
        metadataHash,
        formData.doiType
      );
      
      // 5. Record the DOI in our database
      const response = await apiRequest('POST', '/api/dois', {
        title: formData.title,
        authors: metadata.authors,
        abstract: formData.abstract,
        prefix: '10.FRONS',
        suffix: 'auto', // The backend will generate a unique suffix
        type: formData.doiType,
        userId: 1, // Would be the actual user ID in production
        status: 'published',
        ipfsHash,
        metadata: JSON.stringify(metadata),
        transactionId: txSignature,
      });
      
      if (!response.ok) {
        throw new Error('Failed to record DOI in database');
      }
      
      const dociData = await response.json();
      
      toast({
        title: 'DOCI Minted Successfully',
        description: `Your DOCI ${dociData.fullDoi} has been created and registered on-chain.`,
      });
      
      // Reset form
      setFormData({
        title: '',
        authors: '',
        abstract: '',
        doiType: 'article',
        fileToUpload: null,
      });
      
    } catch (error: any) {
      toast({
        title: 'Minting Failed',
        description: error.message || 'Could not mint DOCI',
        variant: 'destructive',
      });
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <Card>
      <CardHeader>
        <CardTitle>Mint a New DOCI</CardTitle>
        <CardDescription>
          Create a new Direct On-Chain Identifier for your research publication
        </CardDescription>
      </CardHeader>
      <CardContent>
        <Tabs defaultValue="publication">
          <TabsList className="grid w-full grid-cols-2">
            <TabsTrigger value="publication">Publication DOCI</TabsTrigger>
            <TabsTrigger value="researcher">Researcher DOCI</TabsTrigger>
          </TabsList>
          
          <TabsContent value="publication" className="mt-4">
            <form onSubmit={handleSubmit} className="space-y-4">
              <div className="space-y-2">
                <Label htmlFor="title">Publication Title</Label>
                <Input
                  id="title"
                  name="title"
                  value={formData.title}
                  onChange={handleInputChange}
                  placeholder="Enter publication title"
                  required
                />
              </div>
              
              <div className="space-y-2">
                <Label htmlFor="authors">Authors</Label>
                <Input
                  id="authors"
                  name="authors"
                  value={formData.authors}
                  onChange={handleInputChange}
                  placeholder="Enter authors (comma-separated)"
                  required
                />
              </div>
              
              <div className="space-y-2">
                <Label htmlFor="abstract">Abstract</Label>
                <Textarea
                  id="abstract"
                  name="abstract"
                  value={formData.abstract}
                  onChange={handleInputChange}
                  placeholder="Enter publication abstract"
                  rows={4}
                  required
                />
              </div>
              
              <div className="space-y-2">
                <Label htmlFor="doiType">Publication Type</Label>
                <Select 
                  defaultValue={formData.doiType}
                  onValueChange={handleSelectChange}
                >
                  <SelectTrigger>
                    <SelectValue placeholder="Select publication type" />
                  </SelectTrigger>
                  <SelectContent>
                    <SelectItem value="article">Article</SelectItem>
                    <SelectItem value="dataset">Dataset</SelectItem>
                    <SelectItem value="software">Software</SelectItem>
                    <SelectItem value="preprint">Preprint</SelectItem>
                    <SelectItem value="book">Book</SelectItem>
                    <SelectItem value="conference">Conference Paper</SelectItem>
                    <SelectItem value="thesis">Thesis</SelectItem>
                    <SelectItem value="other">Other</SelectItem>
                  </SelectContent>
                </Select>
              </div>
              
              <Separator className="my-4" />
              
              <div className="space-y-2">
                <Label htmlFor="file">Upload Publication File</Label>
                <Input
                  id="file"
                  type="file"
                  onChange={handleFileChange}
                  required
                />
                <p className="text-xs text-muted-foreground mt-1">
                  PDF, DOCX, or ZIP files supported. Maximum file size: 50MB.
                </p>
              </div>
              
              <div className="mt-6">
                <Button type="submit" className="w-full" disabled={isLoading || !connected}>
                  {isLoading ? 'Minting DOCI...' : 'Mint DOCI'}
                </Button>
              </div>
            </form>
          </TabsContent>
          
          <TabsContent value="researcher" className="mt-4">
            <div className="p-4 bg-muted rounded-lg text-center">
              <h3 className="font-medium">Researcher Profile DOCI</h3>
              <p className="text-sm text-muted-foreground mt-2">
                Mint a DOCI for your researcher profile to track and verify your academic contributions.
              </p>
              <Button className="mt-4" disabled={true}>
                Coming Soon
              </Button>
            </div>
          </TabsContent>
        </Tabs>
      </CardContent>
      <CardFooter className="flex flex-col">
        <p className="text-sm text-muted-foreground">
          By minting a DOCI, your research will be permanently recorded on the Solana blockchain
          and stored on IPFS, ensuring immutable proof of existence and provenance.
        </p>
      </CardFooter>
    </Card>
  );
};

export default DociMinter;
```

### client/src/components/BadgeRewardSystem.tsx
```typescript
import React, { useState } from 'react';
import { useWallet } from '@solana/wallet-adapter-react';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardDescription, CardHeader, CardTitle, CardFooter } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Dialog, DialogContent, DialogDescription, DialogHeader, DialogTitle, DialogTrigger } from '@/components/ui/dialog';
import { mintBadge, transferSPLToken } from '@/lib/solanaBadge';
import { apiRequest } from '@/lib/queryClient';
import { useToast } from '@/hooks/use-toast';

interface Badge {
  id: number;
  name: string;
  description: string;
  type: string;
  mintAddress: string;
  imageUrl: string;
  createdAt: string;
}

interface BadgeRewardSystemProps {
  userId: number;
  userWallet?: string;
}

const BadgeRewardSystem: React.FC<BadgeRewardSystemProps> = ({ userId, userWallet }) => {
  const { publicKey, connected } = useWallet();
  const { toast } = useToast();
  const [isLoading, setIsLoading] = useState(false);
  const [userBadges, setUserBadges] = useState<Badge[]>([]);
  const [selectedBadge, setSelectedBadge] = useState<Badge | null>(null);
  
  // Fetch user's badges
  React.useEffect(() => {
    if (userId) {
      fetchUserBadges();
    }
  }, [userId]);
  
  const fetchUserBadges = async () => {
    try {
      const response = await apiRequest('GET', `/api/users/${userId}/badges`);
      if (response.ok) {
        const data = await response.json();
        setUserBadges(data);
      }
    } catch (error) {
      console.error('Failed to fetch user badges:', error);
    }
  };
  
  const handleMintBadge = async (badgeType: string, badgeName: string, description: string) => {
    if (!connected || !publicKey) {
      toast({
        title: 'Wallet Required',
        description: 'Please connect your Solana wallet to mint a badge.',
        variant: 'destructive',
      });
      return;
    }
    
    try {
      setIsLoading(true);
      
      // Create the badge metadata
      const metadata = {
        name: badgeName,
        description,
        type: badgeType,
        creator: publicKey.toString(),
        recipient: userWallet || publicKey.toString(),
        timestamp: new Date().toISOString(),
      };
      
      // Mint the badge as SPL token
      const { mintAddress, txSignature } = await mintBadge(
        publicKey,
        metadata,
        1, // Supply of 1 for NFT-like behavior
        badgeType === 'reviewer' ? 'https://fronsciers.io/badges/reviewer.png' : 'https://fronsciers.io/badges/citation.png'
      );
      
      // Record the badge in our database
      const response = await apiRequest('POST', '/api/badges', {
        userId,
        name: badgeName,
        description,
        type: badgeType,
        mintAddress,
        transactionId: txSignature,
        imageUrl: badgeType === 'reviewer' ? '/badges/reviewer.png' : '/badges/citation.png',
      });
      
      if (!response.ok) {
        throw new Error('Failed to record badge in database');
      }
      
      const badgeData = await response.json();
      
      toast({
        title: 'Badge Minted Successfully',
        description: `The ${badgeName} badge has been minted and added to your collection.`,
      });
      
      // Refresh the user's badges
      fetchUserBadges();
      
    } catch (error: any) {
      toast({
        title: 'Minting Failed',
        description: error.message || 'Could not mint badge',
        variant: 'destructive',
      });
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <Card>
      <CardHeader>
        <CardTitle>Badge Reward System</CardTitle>
        <CardDescription>
          Earn and manage your academic achievement badges
        </CardDescription>
      </CardHeader>
      <CardContent>
        <Tabs defaultValue="my-badges">
          <TabsList className="grid w-full grid-cols-2">
            <TabsTrigger value="my-badges">My Badges</TabsTrigger>
            <TabsTrigger value="available">Available Badges</TabsTrigger>
          </TabsList>
          
          <TabsContent value="my-badges" className="mt-4">
            {userBadges.length > 0 ? (
              <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4">
                {userBadges.map((badge) => (
                  <div 
                    key={badge.id} 
                    className="border rounded-lg p-4 flex flex-col items-center text-center hover:bg-muted/50 cursor-pointer"
                    onClick={() => setSelectedBadge(badge)}
                  >
                    <div className="w-16 h-16 rounded-full bg-muted mb-3 overflow-hidden">
                      <img 
                        src={badge.imageUrl || '/badges/default.png'} 
                        alt={badge.name}
                        className="w-full h-full object-cover"
                      />
                    </div>
                    <h3 className="font-medium">{badge.name}</h3>
                    <Badge variant="outline" className="mt-2">
                      {badge.type.charAt(0).toUpperCase() + badge.type.slice(1)}
                    </Badge>
                  </div>
                ))}
              </div>
            ) : (
              <div className="text-center py-10 bg-muted/30 rounded-lg">
                <h3 className="font-medium">No Badges Yet</h3>
                <p className="text-sm text-muted-foreground mt-2">
                  You haven't earned any badges yet. Participate in peer reviews
                  or receive citations to earn badges.
                </p>
              </div>
            )}
          </TabsContent>
          
          <TabsContent value="available" className="mt-4">
            <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
              <div className="border rounded-lg p-4">
                <div className="flex items-center justify-between mb-3">
                  <div className="flex items-center">
                    <div className="w-10 h-10 rounded-full bg-blue-100 flex items-center justify-center mr-3">
                      <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className="text-blue-600">
                        <circle cx="12" cy="8" r="7" />
                        <polyline points="8.21 13.89 7 23 12 20 17 23 15.79 13.88" />
                      </svg>
                    </div>
                    <div>
                      <h3 className="font-medium">Peer Reviewer</h3>
                      <p className="text-sm text-muted-foreground">
                        For completing peer reviews
                      </p>
                    </div>
                  </div>
                  <Dialog>
                    <DialogTrigger asChild>
                      <Button size="sm" variant="outline">View</Button>
                    </DialogTrigger>
                    <DialogContent>
                      <DialogHeader>
                        <DialogTitle>Peer Reviewer Badge</DialogTitle>
                        <DialogDescription>
                          This badge recognizes your contributions to academic rigor through peer review.
                        </DialogDescription>
                      </DialogHeader>
                      <div className="flex flex-col items-center py-4">
                        <div className="w-24 h-24 rounded-full bg-blue-100 flex items-center justify-center mb-4">
                          <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className="text-blue-600">
                            <circle cx="12" cy="8" r="7" />
                            <polyline points="8.21 13.89 7 23 12 20 17 23 15.79 13.88" />
                          </svg>
                        </div>
                        <h3 className="text-xl font-medium">Peer Reviewer</h3>
                        <p className="mt-2 text-center text-muted-foreground">
                          This badge is awarded for completing quality peer reviews. Each review you complete
                          gets you closer to earning this prestigious badge.
                        </p>
                        
                        <Button 
                          className="mt-6" 
                          disabled={isLoading || !connected}
                          onClick={() => handleMintBadge('reviewer', 'Peer Reviewer', 'Awarded for completing quality peer reviews')}
                        >
                          {isLoading ? 'Minting...' : 'Mint This Badge'}
                        </Button>
                      </div>
                    </DialogContent>
                  </Dialog>
                </div>
                <div className="text-xs text-muted-foreground">
                  Requirements: Complete 3 peer reviews with high quality feedback
                </div>
              </div>
              
              <div className="border rounded-lg p-4">
                <div className="flex items-center justify-between mb-3">
                  <div className="flex items-center">
                    <div className="w-10 h-10 rounded-full bg-amber-100 flex items-center justify-center mr-3">
                      <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className="text-amber-600">
                        <path d="M6 9H4.5a2.5 2.5 0 0 1 0-5H6" />
                        <path d="M18 9h1.5a2.5 2.5 0 0 0 0-5H18" />
                        <path d="M4 22h16" />
                        <path d="M10 14.66V17c0 .55-.47.98-.97 1.21C7.85 18.75 7 20.24 7 22" />
                        <path d="M14 14.66V17c0 .55.47.98.97 1.21C16.15 18.75 17 20.24 17 22" />
                        <path d="M18 2H6v7a6 6 0 0 0 12 0V2Z" />
                      </svg>
                    </div>
                    <div>
                      <h3 className="font-medium">Citation Impact</h3>
                      <p className="text-sm text-muted-foreground">
                        For highly cited work
                      </p>
                    </div>
                  </div>
                  <Dialog>
                    <DialogTrigger asChild>
                      <Button size="sm" variant="outline">View</Button>
                    </DialogTrigger>
                    <DialogContent>
                      <DialogHeader>
                        <DialogTitle>Citation Impact Badge</DialogTitle>
                        <DialogDescription>
                          This badge recognizes the scholarly impact of your research through citations.
                        </DialogDescription>
                      </DialogHeader>
                      <div className="flex flex-col items-center py-4">
                        <div className="w-24 h-24 rounded-full bg-amber-100 flex items-center justify-center mb-4">
                          <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className="text-amber-600">
                            <path d="M6 9H4.5a2.5 2.5 0 0 1 0-5H6" />
                            <path d="M18 9h1.5a2.5 2.5 0 0 0 0-5H18" />
                            <path d="M4 22h16" />
                            <path d="M10 14.66V17c0 .55-.47.98-.97 1.21C7.85 18.75 7 20.24 7 22" />
                            <path d="M14 14.66V17c0 .55.47.98.97 1.21C16.15 18.75 17 20.24 17 22" />
                            <path d="M18 2H6v7a6 6 0 0 0 12 0V2Z" />
                          </svg>
                        </div>
                        <h3 className="text-xl font-medium">Citation Impact</h3>
                        <p className="mt-2 text-center text-muted-foreground">
                          This badge is awarded to researchers whose work has been cited frequently,
                          demonstrating significant impact in their field.
                        </p>
                        
                        <Button 
                          className="mt-6" 
                          disabled={isLoading || !connected}
                          onClick={() => handleMintBadge('citation', 'Citation Impact', 'Awarded for publications with high citation counts')}
                        >
                          {isLoading ? 'Minting...' : 'Mint This Badge'}
                        </Button>
                      </div>
                    </DialogContent>
                  </Dialog>
                </div>
                <div className="text-xs text-muted-foreground">
                  Requirements: Publications with at least 10 citations
                </div>
              </div>
            </div>
          </TabsContent>
        </Tabs>
        
        {/* Badge Details Dialog */}
        <Dialog open={!!selectedBadge} onOpenChange={(open) => !open && setSelectedBadge(null)}>
          <DialogContent>
            <DialogHeader>
              <DialogTitle>{selectedBadge?.name}</DialogTitle>
              <DialogDescription>
                Badge details and verification information
              </DialogDescription>
            </DialogHeader>
            <div className="flex flex-col items-center py-4">
              <div className="w-24 h-24 rounded-full bg-muted mb-4 overflow-hidden">
                <img 
                  src={selectedBadge?.imageUrl || '/badges/default.png'} 
                  alt={selectedBadge?.name}
                  className="w-full h-full object-cover"
                />
              </div>
              <h3 className="text-xl font-medium">{selectedBadge?.name}</h3>
              <Badge variant="outline" className="mt-2">
                {selectedBadge?.type.charAt(0).toUpperCase() + (selectedBadge?.type.slice(1) || '')}
              </Badge>
              <p className="mt-4 text-center text-muted-foreground">
                {selectedBadge?.description}
              </p>
              
              <div className="mt-4 w-full">
                <h4 className="text-sm font-medium mb-2">On-Chain Verification</h4>
                <div className="bg-muted p-3 rounded-md text-xs font-mono break-all">
                  <div className="mb-1">
                    <span className="text-muted-foreground">Mint Address: </span>
                    {selectedBadge?.mintAddress}
                  </div>
                  <div>
                    <span className="text-muted-foreground">Issued: </span>
                    {new Date(selectedBadge?.createdAt || '').toLocaleDateString()}
                  </div>
                </div>
              </div>
            </div>
          </DialogContent>
        </Dialog>
      </CardContent>
      <CardFooter className="flex justify-between">
        <p className="text-sm text-muted-foreground">
          Badges are stored as SPL tokens on the Solana blockchain.
        </p>
      </CardFooter>
    </Card>
  );
};

export default BadgeRewardSystem;
```

### client/src/lib/solanaClient.ts
```typescript
import { Connection, PublicKey, Transaction, Keypair, SystemProgram, LAMPORTS_PER_SOL } from '@solana/web3.js';
import { Program, AnchorProvider, web3, BN } from '@project-serum/anchor';
import { useWallet } from '@solana/wallet-adapter-react';
import { IDL } from './idl/doci_registry';

// Constants
const PROGRAM_ID = new PublicKey('DOCi1d3nT1f13RrE61sTrYpr06raMc0d31337'); // Replace with actual program ID
const REGISTRY_ACCOUNT = new PublicKey('DOCiR361sTrY4cC0UnTDC01rE6iSTrY'); // Replace with actual registry account

// RPC URL based on environment
const getRpcUrl = () => {
  return process.env.VITE_SOLANA_RPC_URL || 'https://api.devnet.solana.com';
};

// Get Solana connection
export const getConnection = () => {
  return new Connection(getRpcUrl(), 'confirmed');
};

// Get program instance
export const getProgram = (wallet: any) => {
  const connection = getConnection();
  
  // Create provider
  const provider = new AnchorProvider(
    connection,
    wallet,
    { preflightCommitment: 'processed' }
  );
  
  // Create program
  return new Program(IDL, PROGRAM_ID, provider);
};

// Mint a new DOCI
export const mintDOCI = async (
  wallet: PublicKey,
  prefix: string,
  metadataUri: string,
  doiType: string
): Promise<string> => {
  // This would be an actual call to the Solana program
  // For demonstration purposes, we'll just return a mock transaction signature
  console.log('Minting DOCI with prefix:', prefix, 'and metadata:', metadataUri);
  return 'mock_transaction_signature';
};

// Get DOCIs by owner
export const getDOCIsByOwner = async (owner: PublicKey): Promise<any[]> => {
  // This would query the Solana program for DOCIs owned by this address
  // For demonstration purposes, we'll return mock data
  return [
    {
      prefix: '10.FRONS',
      suffix: '12345',
      metadataUri: 'ipfs://QmExample123',
      owner: owner.toString(),
      createdAt: new Date().toISOString(),
    }
  ];
};

// Simulate transaction
export const simulateTransaction = async (transaction: Transaction, publicKey: PublicKey): Promise<any> => {
  const connection = getConnection();
  
  // Simulate the transaction
  const simulation = await connection.simulateTransaction(transaction);
  
  return simulation;
};

// Utility to get a shortened wallet address
export const shortenAddress = (address: string, chars = 4): string => {
  return `${address.slice(0, chars)}...${address.slice(-chars)}`;
};
```

### client/src/utils/solanaBadge.ts
```typescript
import { Connection, PublicKey, Transaction, Keypair } from '@solana/web3.js';
import { Token, TOKEN_PROGRAM_ID, MintLayout } from '@solana/spl-token';
import { uploadToIPFS } from '@/lib/ipfs';

// Constants
const SPL_ASSOCIATED_TOKEN_ACCOUNT_PROGRAM_ID = new PublicKey(
  'ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL'
);

// Get Solana connection
const getConnection = () => {
  return new Connection(
    process.env.VITE_SOLANA_RPC_URL || 'https://api.devnet.solana.com',
    'confirmed'
  );
};

// Mint a new badge as SPL token
export const mintBadge = async (
  wallet: PublicKey,
  metadata: any,
  supply: number = 1,
  imageUrl?: string
): Promise<{ mintAddress: string; txSignature: string }> => {
  // This would be an actual call to mint an SPL token
  // For demonstration purposes, we'll return mock data
  console.log('Minting badge with metadata:', metadata);
  
  return {
    mintAddress: 'mock_mint_address',
    txSignature: 'mock_transaction_signature',
  };
};

// Transfer an SPL token to another wallet
export const transferSPLToken = async (
  wallet: PublicKey,
  recipient: PublicKey,
  mintAddress: string,
  amount: number = 1
): Promise<string> => {
  // This would be an actual call to transfer an SPL token
  // For demonstration purposes, we'll return a mock transaction signature
  console.log('Transferring token from', wallet.toString(), 'to', recipient.toString());
  
  return 'mock_transfer_signature';
};

// Get all badges owned by a wallet
export const getBadgesByOwner = async (owner: PublicKey): Promise<any[]> => {
  // This would query the Solana blockchain for tokens owned by this address
  // For demonstration purposes, we'll return mock data
  return [
    {
      mint: 'mock_mint_address',
      amount: 1,
      metadata: {
        name: 'Peer Reviewer',
        description: 'Awarded for completing quality peer reviews',
        type: 'reviewer',
        imageUrl: '/badges/reviewer.png',
      },
    }
  ];
};
```

### client/src/lib/ipfs.ts
```typescript
import { Web3Storage } from 'web3.storage';

// Initialize the Web3Storage client
const getClient = () => {
  const token = import.meta.env.VITE_WEB3_STORAGE_TOKEN;
  if (!token) {
    throw new Error('Web3Storage token is required. Set VITE_WEB3_STORAGE_TOKEN in your environment.');
  }
  return new Web3Storage({ token });
};

// Upload a file to IPFS
export const uploadToIPFS = async (file: File | Blob): Promise<string> => {
  try {
    const client = getClient();
    
    // Prepare the file for upload
    const fileName = file instanceof File ? file.name : 'file.json';
    const fileToUpload = file instanceof File ? file : new File([file], fileName);
    
    // Upload to Web3.Storage
    const cid = await client.put([fileToUpload], {
      name: fileName,
      maxRetries: 3,
    });
    
    // Return the IPFS URL
    return `ipfs://${cid}/${fileName}`;
  } catch (error) {
    console.error('IPFS upload error:', error);
    throw new Error('Failed to upload to IPFS');
  }
};

// Get a file from IPFS
export const getFromIPFS = async (cid: string): Promise<any> => {
  try {
    const client = getClient();
    
    // Retrieve from Web3.Storage
    const res = await client.get(cid);
    
    if (!res?.ok) {
      throw new Error(`Failed to get file from IPFS: ${res?.status} ${res?.statusText}`);
    }
    
    // Get all files from this retrieval
    const files = await res.files();
    
    if (files.length === 0) {
      throw new Error('No files found');
    }
    
    // Return the first file
    return files[0];
  } catch (error) {
    console.error('IPFS retrieval error:', error);
    throw new Error('Failed to retrieve from IPFS');
  }
};

// Utility function to convert IPFS URL to HTTP URL for browsing
export const ipfsToHttpUrl = (ipfsUrl: string): string => {
  if (!ipfsUrl) return '';
  
  // Convert ipfs:// URLs to HTTP gateway URLs
  if (ipfsUrl.startsWith('ipfs://')) {
    const cid = ipfsUrl.replace('ipfs://', '');
    return `https://w3s.link/ipfs/${cid}`;
  }
  
  return ipfsUrl;
};
```

### client/src/lib/queryClient.ts
```typescript
import { QueryClient } from '@tanstack/react-query';

// Create a client
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      refetchOnWindowFocus: false,
      retry: 1,
      staleTime: 5 * 60 * 1000, // 5 minutes
    },
  },
});

// API request helper
export const apiRequest = async (
  method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE',
  endpoint: string,
  body?: any,
  customHeaders?: Record<string, string>
): Promise<Response> => {
  const headers: Record<string, string> = {
    'Content-Type': 'application/json',
    ...customHeaders,
  };

  const options: RequestInit = {
    method,
    headers,
    credentials: 'include',
  };

  if (body && method !== 'GET') {
    options.body = JSON.stringify(body);
  }

  return fetch(endpoint, options);
};

// Default query fetcher
export const defaultQueryFn = async ({ queryKey }: { queryKey: any }) => {
  const endpoint = Array.isArray(queryKey) ? queryKey.join('/') : queryKey;
  
  const response = await apiRequest('GET', endpoint);
  
  if (!response.ok) {
    throw new Error(`Request failed with status ${response.status}`);
  }
  
  return response.json();
};
```

## Pages

### client/src/pages/ResearchJourneyDemo.tsx
```typescript
import React, { useState } from 'react';
import { 
  Card, 
  CardContent, 
  CardDescription, 
  CardFooter, 
  CardHeader, 
  CardTitle 
} from "@/components/ui/card";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import { RadioGroup, RadioGroupItem } from "@/components/ui/radio-group";
import { Label } from "@/components/ui/label";
import { Separator } from "@/components/ui/separator";
import ResearchJourneyProgressBar from '@/components/ResearchJourneyProgressBar';
import { 
  ClipboardList, 
  FileText, 
  Users, 
  CheckCircle2, 
  BookOpen, 
  Award,
  GraduationCap,
  Flag
} from 'lucide-react';

// Import JourneyStage from types
import { JourneyStage } from '@/lib/types';

const ResearchJourneyDemo: React.FC = () => {
  // State for controlling the demonstration
  const [currentStage, setCurrentStage] = useState<JourneyStage>(JourneyStage.PEER_REVIEW);
  const [completedStages, setCompletedStages] = useState<JourneyStage[]>([
    JourneyStage.DRAFT,
    JourneyStage.SUBMISSION
  ]);
  
  // Sample publication data
  const publication = {
    id: 123,
    title: "Blockchain-Based Direct On-Chain Identifier Systems for Academic Research",
    authors: ["J. Smith", "A. Johnson", "M. Garcia"],
    submissionDate: "2025-04-15T12:00:00Z",
    abstract: "This paper presents a novel approach to academic publishing using blockchain technology to create immutable, verifiable research identifiers that enhance reproducibility and citation tracking.",
    status: "under_review",
    doi: "10.FRONS/ABC123XYZ",
    reviewsCompleted: 2,
    reviewsTotal: 3
  };
  
  // Handler for when a stage is clicked in the progress bar
  const handleStageClick = (stage: JourneyStage) => {
    // In a real application, this would navigate to the corresponding stage
    console.log(`Stage clicked: ${stage}`);
  };
  
  // Handler for the demo controls
  const advanceStage = () => {
    const stages = Object.values(JourneyStage);
    const currentIndex = stages.indexOf(currentStage);
    
    if (currentIndex < stages.length - 1) {
      // Mark current stage as completed
      if (!completedStages.includes(currentStage)) {
        setCompletedStages([...completedStages, currentStage]);
      }
      
      // Advance to next stage
      setCurrentStage(stages[currentIndex + 1]);
    }
  };
  
  const resetDemo = () => {
    setCurrentStage(JourneyStage.DRAFT);
    setCompletedStages([]);
  };
  
  const setDemoScenario = (scenario: string) => {
    switch (scenario) {
      case "start":
        setCurrentStage(JourneyStage.DRAFT);
        setCompletedStages([]);
        break;
      case "peer_review":
        setCurrentStage(JourneyStage.PEER_REVIEW);
        setCompletedStages([JourneyStage.DRAFT, JourneyStage.SUBMISSION]);
        break;
      case "accepted":
        setCurrentStage(JourneyStage.PUBLICATION);
        setCompletedStages([
          JourneyStage.DRAFT, 
          JourneyStage.SUBMISSION, 
          JourneyStage.PEER_REVIEW, 
          JourneyStage.REVISION,
          JourneyStage.ACCEPTANCE
        ]);
        break;
      case "published":
        setCurrentStage(JourneyStage.CITATION);
        setCompletedStages([
          JourneyStage.DRAFT, 
          JourneyStage.SUBMISSION, 
          JourneyStage.PEER_REVIEW, 
          JourneyStage.REVISION,
          JourneyStage.ACCEPTANCE, 
          JourneyStage.PUBLICATION,
          JourneyStage.INDEXING
        ]);
        break;
      default:
        break;
    }
  };
  
  return (
    <div className="container py-8 space-y-8">
      {/* Header */}
      <div>
        <h1 className="text-3xl font-spectral font-bold bg-clip-text text-transparent bg-gradient-to-r from-odyssey-dark to-odyssey-medium dark:from-odyssey-light dark:to-odyssey-medium mb-2">
          Research Publication Journey
        </h1>
        <p className="text-odyssey-medium dark:text-odyssey-light text-lg">
          Track your research publication progress from draft to citations
        </p>
      </div>
      
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        {/* Main content */}
        <div className="lg:col-span-2">
          <Card>
            <CardHeader>
              <CardTitle className="flex items-center gap-2">
                <FileText className="h-5 w-5" />
                {publication.title}
              </CardTitle>
              <CardDescription>
                by {publication.authors.join(", ")} · Submitted on {new Date(publication.submissionDate).toLocaleDateString()}
              </CardDescription>
            </CardHeader>
            <CardContent className="space-y-6">
              <ResearchJourneyProgressBar
                userId={1}
                publicationId={publication.id}
                currentStage={currentStage}
                completedStages={completedStages}
                onStageClick={handleStageClick}
              />
              
              <Separator />
              
              <div className="space-y-4">
                <h3 className="text-lg font-medium">Publication Details</h3>
                
                <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
                  <Card className="bg-muted/20">
                    <CardHeader className="pb-2">
                      <CardTitle className="text-sm flex items-center gap-2">
                        <ClipboardList className="h-4 w-4 text-primary" />
                        Publication Status
                      </CardTitle>
                    </CardHeader>
                    <CardContent>
                      <div className="flex justify-between items-center">
                        <span className="text-sm">Current Status:</span>
                        <Badge className="bg-amber-100 text-amber-800 dark:bg-amber-900 dark:text-amber-100">
                          Under Review
                        </Badge>
                      </div>
                    </CardContent>
                  </Card>
                  
                  <Card className="bg-muted/20">
                    <CardHeader className="pb-2">
                      <CardTitle className="text-sm flex items-center gap-2">
                        <Users className="h-4 w-4 text-primary" />
                        Peer Review Progress
                      </CardTitle>
                    </CardHeader>
                    <CardContent>
                      <div className="flex justify-between items-center">
                        <span className="text-sm">Reviews Completed:</span>
                        <Badge variant="outline">{publication.reviewsCompleted}/{publication.reviewsTotal}</Badge>
                      </div>
                    </CardContent>
                  </Card>
                </div>
                
                <Card className="bg-muted/20">
                  <CardHeader className="pb-2">
                    <CardTitle className="text-sm">Abstract</CardTitle>
                  </CardHeader>
                  <CardContent>
                    <p className="text-sm">{publication.abstract}</p>
                  </CardContent>
                </Card>
                
                {completedStages.includes(JourneyStage.PUBLICATION) && (
                  <Card className="bg-green-50 dark:bg-green-900/20 border-green-100 dark:border-green-900/30">
                    <CardHeader className="pb-2">
                      <CardTitle className="text-sm flex items-center gap-2 text-green-800 dark:text-green-300">
                        <CheckCircle2 className="h-4 w-4" />
                        Publication Complete
                      </CardTitle>
                    </CardHeader>
                    <CardContent>
                      <div className="flex justify-between items-center">
                        <span className="text-sm text-green-800 dark:text-green-300">DOCI:</span>
                        <Badge className="font-mono bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-100">
                          {publication.doi}
                        </Badge>
                      </div>
                    </CardContent>
                  </Card>
                )}
              </div>
            </CardContent>
          </Card>
        </div>
        
        {/* Right sidebar - Demo controls */}
        <div className="space-y-6">
          <Card>
            <CardHeader>
              <CardTitle className="text-lg flex items-center gap-2">
                <Flag className="h-5 w-5 text-primary" />
                Demo Controls
              </CardTitle>
              <CardDescription>
                Try different stages of the research journey
              </CardDescription>
            </CardHeader>
            <CardContent className="space-y-4">
              <div className="space-y-2">
                <h4 className="text-sm font-medium">Select Scenario</h4>
                <RadioGroup defaultValue="peer_review" className="space-y-2">
                  <div className="flex items-center space-x-2">
                    <RadioGroupItem value="start" id="start" onClick={() => setDemoScenario("start")} />
                    <Label htmlFor="start" className="flex items-center gap-2">
                      <FileText className="h-4 w-4" />
                      Just Starting
                    </Label>
                  </div>
                  <div className="flex items-center space-x-2">
                    <RadioGroupItem value="peer_review" id="peer_review" onClick={() => setDemoScenario("peer_review")} />
                    <Label htmlFor="peer_review" className="flex items-center gap-2">
                      <Users className="h-4 w-4" />
                      Under Peer Review
                    </Label>
                  </div>
                  <div className="flex items-center space-x-2">
                    <RadioGroupItem value="accepted" id="accepted" onClick={() => setDemoScenario("accepted")} />
                    <Label htmlFor="accepted" className="flex items-center gap-2">
                      <CheckCircle2 className="h-4 w-4" />
                      Accepted/Publishing
                    </Label>
                  </div>
                  <div className="flex items-center space-x-2">
                    <RadioGroupItem value="published" id="published" onClick={() => setDemoScenario("published")} />
                    <Label htmlFor="published" className="flex items-center gap-2">
                      <Award className="h-4 w-4" />
                      Published & Cited
                    </Label>
                  </div>
                </RadioGroup>
              </div>
              
              <Separator />
              
              <div className="space-y-4">
                <h4 className="text-sm font-medium">Manual Controls</h4>
                <div className="flex flex-col gap-2">
                  <Button onClick={advanceStage}>
                    Advance to Next Stage
                  </Button>
                  <Button variant="outline" onClick={resetDemo}>
                    Reset Demo
                  </Button>
                </div>
              </div>
            </CardContent>
          </Card>
          
          <Card>
            <CardHeader>
              <CardTitle className="text-lg flex items-center gap-2">
                <GraduationCap className="h-5 w-5 text-primary" />
                About This Feature
              </CardTitle>
            </CardHeader>
            <CardContent className="space-y-4">
              <p className="text-sm text-muted-foreground">
                The Interactive Research Journey Progress Bar provides researchers with a visual representation of their publication's progress through the entire lifecycle:
              </p>
              <ul className="space-y-2 text-sm text-muted-foreground list-disc list-inside">
                <li>Track manuscript from draft to citation</li>
                <li>Get estimated completion times for each stage</li>
                <li>View detailed information about current stage requirements</li>
                <li>Integrated with blockchain for verifiable progress tracking</li>
              </ul>
              <p className="text-sm text-muted-foreground">
                This component is part of the Fronsciers platform's commitment to transparency and clarity in the academic publishing process.
              </p>
            </CardContent>
            <CardFooter>
              <Button variant="outline" className="w-full">
                Learn More About Fronsciers
              </Button>
            </CardFooter>
          </Card>
        </div>
      </div>
    </div>
  );
};

export default ResearchJourneyDemo;
```

### client/src/pages/DociResolverPage.tsx
```typescript
import React, { useEffect, useState } from 'react';
import { useRoute } from 'wouter';
import { 
  Card, 
  CardContent, 
  CardDescription, 
  CardFooter, 
  CardHeader, 
  CardTitle 
} from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Separator } from '@/components/ui/separator';
import { Badge } from '@/components/ui/badge';
import { ipfsToHttpUrl } from '@/lib/ipfs';
import { apiRequest } from '@/lib/queryClient';
import { Skeleton } from '@/components/ui/skeleton';

interface DoiData {
  id: number;
  title: string;
  authors: string[];
  abstract: string;
  fullDoi: string;
  type: string;
  status: string;
  publishedDate: string;
  createdAt: string;
  ipfsHash: string | null;
  metadata: string | null;
}

const DociResolverPage: React.FC = () => {
  const [, params] = useRoute('/doci/:prefix/:suffix');
  const [doiData, setDoiData] = useState<DoiData | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    // If we have params, fetch DOI data
    if (params && params.prefix && params.suffix) {
      fetchDoiData(params.prefix, params.suffix);
    }
  }, [params]);
  
  const fetchDoiData = async (prefix: string, suffix: string) => {
    try {
      setIsLoading(true);
      setError(null);
      
      const response = await apiRequest('GET', `/api/resolve/${prefix}/${suffix}`);
      
      if (!response.ok) {
        if (response.status === 404) {
          throw new Error('DOCI not found');
        }
        throw new Error('Failed to fetch DOCI data');
      }
      
      const data = await response.json();
      setDoiData(data);
    } catch (error: any) {
      console.error('Error fetching DOI:', error);
      setError(error.message || 'An error occurred');
    } finally {
      setIsLoading(false);
    }
  };
  
  // Handle loading state
  if (isLoading) {
    return (
      <div className="container py-8">
        <Card>
          <CardHeader>
            <Skeleton className="h-8 w-3/4 mb-2" />
            <Skeleton className="h-4 w-1/2" />
          </CardHeader>
          <CardContent className="space-y-4">
            <Skeleton className="h-4 w-full" />
            <Skeleton className="h-4 w-full" />
            <Skeleton className="h-4 w-3/4" />
            
            <Separator className="my-4" />
            
            <div className="space-y-2">
              <Skeleton className="h-6 w-1/4" />
              <Skeleton className="h-32 w-full" />
            </div>
          </CardContent>
        </Card>
      </div>
    );
  }
  
  // Handle error state
  if (error) {
    return (
      <div className="container py-8">
        <Card className="border-red-200 bg-red-50">
          <CardHeader>
            <CardTitle className="text-red-700">Error Resolving DOCI</CardTitle>
            <CardDescription className="text-red-600">
              We couldn't resolve the requested DOCI
            </CardDescription>
          </CardHeader>
          <CardContent>
            <p className="text-red-600">{error}</p>
            
            <div className="mt-6">
              <p className="text-gray-600">
                Please check the DOCI and try again, or search for the publication using the search bar.
              </p>
            </div>
          </CardContent>
          <CardFooter>
            <Button variant="outline" onClick={() => window.history.back()}>
              Go Back
            </Button>
          </CardFooter>
        </Card>
      </div>
    );
  }
  
  // Handle no data
  if (!doiData) {
    return (
      <div className="container py-8">
        <Card>
          <CardHeader>
            <CardTitle>No DOCI Data Available</CardTitle>
            <CardDescription>
              We couldn't find any data for the requested DOCI
            </CardDescription>
          </CardHeader>
          <CardContent>
            <p>The DOCI you're looking for doesn't exist or hasn't been properly registered.</p>
          </CardContent>
          <CardFooter>
            <Button variant="outline" onClick={() => window.history.back()}>
              Go Back
            </Button>
          </CardFooter>
        </Card>
      </div>
    );
  }
  
  // Get publication date
  const publicationDate = doiData.publishedDate 
    ? new Date(doiData.publishedDate).toLocaleDateString() 
    : new Date(doiData.createdAt).toLocaleDateString();
  
  // Get IPFS URL for content
  const contentUrl = doiData.ipfsHash ? ipfsToHttpUrl(doiData.ipfsHash) : null;
  
  return (
    <div className="container py-8">
      <Card>
        <CardHeader>
          <div className="flex flex-col md:flex-row md:justify-between md:items-start gap-4">
            <div>
              <CardTitle className="text-2xl font-spectral">{doiData.title}</CardTitle>
              <CardDescription className="mt-2">
                {doiData.authors.join(', ')}
              </CardDescription>
            </div>
            <div className="flex flex-wrap gap-2">
              <Badge variant="outline" className="text-odyssey-medium dark:text-odyssey-light font-mono">
                {doiData.fullDoi}
              </Badge>
              <Badge className="capitalize bg-odyssey-light/30 text-odyssey-dark dark:bg-odyssey-dark/30 dark:text-odyssey-light">
                {doiData.type}
              </Badge>
            </div>
          </div>
        </CardHeader>
        <CardContent className="space-y-6">
          <div>
            <h3 className="text-lg font-medium mb-2">Abstract</h3>
            <p className="text-muted-foreground">{doiData.abstract}</p>
          </div>
          
          <Separator />
          
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div>
              <h3 className="text-sm font-medium mb-2">Publication Information</h3>
              <dl className="grid grid-cols-[auto_1fr] gap-x-4 gap-y-2">
                <dt className="text-sm font-medium text-muted-foreground">Publication Date:</dt>
                <dd className="text-sm">{publicationDate}</dd>
                
                <dt className="text-sm font-medium text-muted-foreground">Status:</dt>
                <dd className="text-sm capitalize">{doiData.status}</dd>
                
                <dt className="text-sm font-medium text-muted-foreground">DOI:</dt>
                <dd className="text-sm font-mono">{doiData.fullDoi}</dd>
              </dl>
            </div>
            
            {doiData.metadata && (
              <div>
                <h3 className="text-sm font-medium mb-2">Additional Metadata</h3>
                <div className="text-sm">
                  {Object.entries(JSON.parse(doiData.metadata)).map(([key, value]) => {
                    // Skip complex objects or arrays
                    if (typeof value === 'object') return null;
                    
                    return (
                      <div key={key} className="flex justify-between border-b py-1">
                        <span className="font-medium text-muted-foreground capitalize">
                          {key.replace(/([A-Z])/g, ' $1').trim()}:
                        </span>
                        <span>{String(value)}</span>
                      </div>
                    );
                  })}
                </div>
              </div>
            )}
          </div>
          
          {contentUrl && (
            <>
              <Separator />
              
              <div>
                <h3 className="text-lg font-medium mb-4">Access Publication</h3>
                <div className="flex flex-wrap gap-4">
                  <Button asChild>
                    <a href={contentUrl} target="_blank" rel="noopener noreferrer">
                      View Publication
                    </a>
                  </Button>
                  <Button variant="outline" asChild>
                    <a href={contentUrl} download target="_blank" rel="noopener noreferrer">
                      Download
                    </a>
                  </Button>
                </div>
              </div>
            </>
          )}
          
          <Separator />
          
          <div>
            <h3 className="text-lg font-medium mb-4">On-Chain Verification</h3>
            <div className="bg-muted p-4 rounded-md">
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <h4 className="text-sm font-medium mb-2">Blockchain Record</h4>
                  <p className="text-xs text-muted-foreground mb-2">
                    This DOCI has been permanently recorded on the Solana blockchain.
                  </p>
                  {doiData.transactionId && (
                    <Button variant="outline" size="sm" asChild className="text-xs">
                      <a 
                        href={`https://explorer.solana.com/tx/${doiData.transactionId}`} 
                        target="_blank" 
                        rel="noopener noreferrer"
                      >
                        View on Solana Explorer
                      </a>
                    </Button>
                  )}
                </div>
                
                <div>
                  <h4 className="text-sm font-medium mb-2">IPFS Storage</h4>
                  <p className="text-xs text-muted-foreground mb-2">
                    Publication content is stored on the decentralized IPFS network.
                  </p>
                  {doiData.ipfsHash && (
                    <div className="text-xs font-mono break-all text-muted-foreground">
                      {doiData.ipfsHash}
                    </div>
                  )}
                </div>
              </div>
            </div>
          </div>
        </CardContent>
        <CardFooter className="flex justify-between">
          <Button variant="outline" onClick={() => window.history.back()}>
            Go Back
          </Button>
          <Button variant="outline">Cite This Publication</Button>
        </CardFooter>
      </Card>
    </div>
  );
};

export default DociResolverPage;
```