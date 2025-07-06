# VeriHire - Privacy-First Recruitment with World ID & ZK Proofs

VeriHire transforms recruitment by combining **World ID verification**, **zero-knowledge proofs**, and **soul-bound NFT credentials** to create a privacy-preserving, sybil-proof hiring platform. Built for the World ecosystem with native World Chain integration.

## 🌟 Key Features (Implemented)

### ✅ World ID Integration

- **Biometric Verification**: Sybil-proof identity verification using World ID
- **World Mini App**: Native integration with World App ecosystem
- **Human-Verified Network**: Built exclusively for verified humans

### ✅ Zero-Knowledge Credentials

- **Privacy-Preserving**: Employment verification without exposing raw documents
- **Selective Disclosure**: Share only necessary credential information
- **Magic Link Verification**: Secure employer verification system

### ✅ Soul-Bound NFT Credentials

- **World Chain Native**: Deployed on World Chain (Chain ID: 480)
- **Non-Transferable**: ERC-5192 soul-bound tokens for permanent credentials
- **Trust Score Badges**: Dynamic metadata with color-coded trust levels
- **Gas Subsidies**: Reduced costs for World ID verified users

## 🏗️ Architecture

```
VeriHire/
├── frontend(new)/          # World Mini App (Primary)
│   ├── World ID integration
│   ├── Mini App components
│   └── World Chain transactions
├── frontend/               # Web Dashboard (Legacy)
│   └── Recruiter interface
├── contracts/              # Smart Contracts
│   ├── TrustMatchNFT.sol  # Soul-bound NFT contract
│   └── Deployment scripts
└── Trust_Match_PRD.md     # Product Requirements
```

## 🚀 Quick Start

### Prerequisites

- Node.js 18+
- World App (for testing Mini App)
- Alchemy API key for World Chain
- Supabase account

### 1. Clone & Install

```bash
git clone <repository-url>
cd VeriHire

# Install World Mini App dependencies
cd frontend(new)
npm install
```

### 2. Environment Setup

Create `frontend(new)/.env.local`:

```bash
# World Chain Configuration (Chain ID: 480)
WORLD_CHAIN_RPC_URL=https://worldchain-mainnet.g.alchemy.com/v2/YOUR_ALCHEMY_API_KEY
TRUST_MATCH_NFT_CONTRACT_ADDRESS=0x... # Your deployed contract
DEPLOYER_PRIVATE_KEY=0x... # Private key with minting permissions

# World ID Configuration
NEXT_PUBLIC_WORLD_APP_ID=app_staging_test
WORLD_ACTION_ID=trust-match-verification

# Supabase Configuration
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key

# AI Integration
ASI_API_KEY=your_asi_api_key
```

### 3. Deploy Smart Contract

```bash
cd contracts

# Deploy to World Chain
forge create --rpc-url https://worldchain-mainnet.g.alchemy.com/v2/YOUR_API_KEY \
  --private-key YOUR_PRIVATE_KEY \
  --chain-id 480 \
  src/TrustMatchNFT.sol:TrustMatchNFT \
  --constructor-args "Trust Match Badges" "TMB" "https://trustmatch.app/metadata/"
```

### 4. Database Setup

Run in Supabase SQL editor:

```sql
-- Create NFT credentials table
CREATE TABLE IF NOT EXISTS nft_credentials (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    candidate_id TEXT NOT NULL UNIQUE,
    token_id TEXT NOT NULL,
    contract_address TEXT NOT NULL,
    wallet_address TEXT NOT NULL,
    trust_score INTEGER NOT NULL,
    verification_count INTEGER NOT NULL DEFAULT 0,
    transaction_hash TEXT NOT NULL,
    credential_hash TEXT NOT NULL,
    token_uri TEXT,
    minted_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable RLS and create policies
ALTER TABLE nft_credentials ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Public read access" ON nft_credentials FOR SELECT USING (true);
```

### 5. Run Development Server

```bash
cd frontend(new)
npm run dev
```

### 6. Test in World App

1. Use ngrok to expose local server: `ngrok http 3000`
2. Configure Mini App URL in [World Developer Portal](https://developer.world.org)
3. Test in World App on mobile device

## 🔧 Core Functionality

### World ID Verification Flow

```typescript
// Automatic verification in World Mini App
const { isConnected, isWorldIdVerified } = useWorldMiniApp();

// For World Mini Apps, users are automatically verified
if (isConnected && isWorldIdVerified) {
  // Proceed with credential creation
  proceedToCredentialFlow();
}
```

### Employment Verification (Magic Links)

```typescript
// Send verification request to employer
const verificationResult = await sendMagicLink({
  employerEmail: "manager@company.com",
  candidateInfo: {
    name: "John Doe",
    position: "Software Engineer",
    company: "Tech Corp",
    duration: "2020-2023",
  },
});

// Employer verifies through secure link
// ZK proof generated without exposing raw employment data
```

### Soul-Bound NFT Minting

```typescript
// Mint NFT credential with trust score
const nftResult = await mintTrustCredential({
  walletAddress: "0x...",
  trustScore: 85, // AI-calculated score
  verificationCount: 3,
  credentialHash: "0x...", // ZK proof hash
});

// NFT Features:
// - Non-transferable (ERC-5192)
// - Dynamic metadata based on trust score
// - Color-coded badges (Platinum/Gold/Silver/Bronze)
// - Permanent employment credentials
```

## 🎨 Trust Score System

### Badge Levels

- 🏆 **90-100**: Platinum Badge (Exceptional verification)
- 🥇 **75-89**: Gold Badge (Strong verification)
- 🥈 **60-74**: Silver Badge (Good verification)
- 🥉 **40-59**: Bronze Badge (Basic verification)
- ⚪ **0-39**: Unverified (Insufficient verification)

### AI-Powered Scoring (ASI Integration)

- Employment verification success rate
- Role consistency and career progression
- Skill alignment with claimed experience
- Timeline consistency across employers
- Portfolio quality assessment

## 🌍 World Chain Integration

### Why World Chain?

- **Human-Verified Network**: Built for World ID users
- **Gas Subsidies**: Reduced transaction costs for verified users
- **Native Integration**: Seamless World App experience
- **Sybil Resistance**: One credential per verified human

### Network Details

- **Chain ID**: 480
- **Native Currency**: ETH
- **Block Explorer**: https://worldscan.org
- **RPC Endpoint**: Alchemy World Chain API

## 🔐 Privacy & Security

### Zero-Knowledge Implementation

- **Selective Disclosure**: Share only necessary information
- **Commitment Schemes**: Pedersen commitments for privacy
- **Proof Verification**: Cryptographic proof validation
- **No Raw Data Storage**: Only hashes and proofs stored

### Security Features

- **World ID Verification**: Biometric sybil protection
- **Magic Link Security**: JWT tokens with expiration
- **Soul-Bound Tokens**: Non-transferable credentials
- **Access Controls**: Role-based permissions

## 📱 World Mini App Features

### Native World App Integration

```typescript
// World App specific functionality
const { sendTransaction, share, copyToClipboard } = useWorldMiniApp();

// Native transaction support
await sendTransaction({
  to: contractAddress,
  value: "0",
  data: mintCalldata,
});

// Share credentials
await share({
  title: "My Trust Credential",
  url: `https://worldscan.org/token/${tokenId}`,
});
```

### Safe Area Handling

```tsx
// Automatic adjustment for World App UI
<WorldAppLayout>
  <div className="safe-area-content">
    {/* Content automatically adjusts for World App interface */}
  </div>
</WorldAppLayout>
```

## 🧪 Testing

### Local Development

```bash
# Run tests
cd contracts
forge test

# Test World Mini App
cd frontend(new)
npm run dev
# Use ngrok for World App testing
```

### World App Testing

1. Deploy to staging environment
2. Configure in World Developer Portal
3. Test full flow in World App
4. Verify NFT minting on World Chain

## 📊 Current Implementation Status

### ✅ Completed Features

- [x] World ID integration with biometric verification
- [x] World Mini App with native World App experience
- [x] Magic link employment verification system
- [x] Zero-knowledge credential generation
- [x] Soul-bound NFT minting on World Chain
- [x] AI-powered trust score calculation (ASI)
- [x] Dynamic NFT metadata with trust badges
- [x] Database integration with audit trails
- [x] World Chain transaction support

### 🚧 In Development

- [ ] Recruiter marketplace with candidate discovery
- [ ] The Graph Protocol integration for indexing
- [ ] Advanced ZK proof systems
- [ ] Cross-chain credential bridging
- [ ] Enterprise ATS integrations

## 🛠️ Technical Stack

### Frontend (World Mini App)

- **Framework**: Next.js 15 with React 18
- **Styling**: Tailwind CSS with World App theming
- **World Integration**: MiniKit SDK
- **State Management**: React hooks with context

### Backend

- **API**: Next.js API routes
- **Database**: Supabase PostgreSQL
- **Authentication**: World ID + JWT tokens
- **Email**: Resend for magic links

### Blockchain

- **Network**: World Chain (Chain ID: 480)
- **Contracts**: Solidity with Foundry
- **Standards**: ERC-5192 (Soul-bound tokens)
- **Gas**: Subsidized for World ID users

### AI & Privacy

- **AI Scoring**: ASI Alliance integration
- **ZK Proofs**: Custom implementation with Poseidon hashing
- **Privacy**: Zero-knowledge credential system

## 📚 Documentation

- [World Mini App Setup](<frontend(new)/WORLD_MINI_APP_README.md>)
- [World Chain Deployment](frontend/WORLD_CHAIN_DEPLOYMENT.md)
- [NFT Setup Guide](frontend/NFT_SETUP_GUIDE.md)
- [Product Requirements](Trust_Match_PRD.md)

## 🤝 Contributing

1. Fork the repository
2. Create feature branch: `git checkout -b feature/amazing-feature`
3. Commit changes: `git commit -m 'Add amazing feature'`
4. Push to branch: `git push origin feature/amazing-feature`
5. Open Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🌟 Acknowledgments

- **World Foundation** for World ID and World Chain infrastructure
- **ASI Alliance** for AI-powered trust scoring
- **Supabase** for database and authentication services
- **The Graph Protocol** for decentralized indexing (planned)

---

**VeriHire**: Transforming recruitment with privacy-first, human-verified credentials on World Chain! 🌍✨

_Built for the World ecosystem - Where verified humans build the future of work._
