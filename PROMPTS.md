# UniteDefi - Detailed Claude Code Session Prompts

## Overview

Copy-paste these prompts to each Claude Code session. Each session works on a specific repository with detailed technical requirements.

**CRITICAL**: Make small, frequent commits (every 30-45 minutes). Never commit large files or build artifacts.

---

## Session 1: Protocol Development (`unite-defi-protocol`)

### Phase 1 - Day 1 Morning Prompt

```
You are building the core protocol for UniteDefi, a cross-chain swap platform using 1inch Fusion+ dutch auction model. This is a hackathon sprint with 8 days to build a production-ready demo.

SETUP TASKS:
1. Fork the 1inch cross-chain resolver: https://github.com/1inch/cross-chain-resolver-example
2. Rename to unite-defi-protocol
3. Update package.json to @unite-defi/protocol
4. Setup the following structure:
```

src/
├── chains/ # Chain abstractions
├── auction/ # Dutch auction logic  
 ├── resolvers/ # Resolver implementations
├── contracts/ # HTLC contracts
└── api/ # REST API endpoints

```

IMMEDIATE GOALS:
- Basic dutch auction with linear price decrease
- Support for 4 EVM chains: Ethereum, Polygon, BSC, Arbitrum
- Simple resolver that can participate in auctions
- Safety deposits with minimal amounts (0.01 ETH equivalent)

TECHNICAL REQUIREMENTS:
- TypeScript with strict mode
- Small commits every 30-45 minutes
- Focus on getting basic functionality working first
- Use existing 1inch contracts as foundation but customize for our needs

COMMIT STRATEGY:
- "init: fork 1inch resolver as unite-defi-protocol"
- "feat: enhanced project structure for multi-chain"
- "feat: base chain abstraction layer"
- "feat: basic dutch auction price curve logic"
- Continue with small, descriptive commits

Start with forking the repository and setting up the basic structure. Make your first commit, then ask me what to work on next.
```

### Phase 1 - Day 1 Afternoon Prompt

```
Continue building the UniteDefi protocol. You should now have the basic structure set up.

CURRENT FOCUS: Dutch Auction Implementation

TASKS:
1. Create src/auction/dutchAuction.ts with:
   - Linear price decrease over time (start 5% above market, decrease to market over 30 seconds)
   - Configurable auction duration and starting premium
   - getCurrentPrice(startTime, startPrice, endPrice) function
   - Auction state management

2. Enhance HTLC contracts in contracts/:
   - Modify EscrowSrc/EscrowDst for multi-chain support
   - Add safety deposit mechanisms (0.01 ETH equivalent)
   - Add partial fill support using Merkle trees

3. Add EVM chain configurations:
   - src/chains/ethereum.ts - Ethereum mainnet config
   - src/chains/polygon.ts - Polygon with gas optimizations
   - src/chains/bsc.ts - BSC configuration
   - src/chains/arbitrum.ts - Arbitrum L2 configuration

Each chain should extend a base class with:
- Chain ID, RPC endpoints, native token
- Gas estimation methods
- Transaction signing logic
- Contract deployment addresses

COMMIT PATTERN:
- "feat: basic dutch auction price curve logic"
- "feat: enhanced HTLC contracts for multi-chain"
- "feat: Ethereum and Polygon chain support"
- "feat: BSC and Arbitrum chain support"

Make sure each commit is focused on one specific feature. Test locally that auction price calculation works correctly.
```

### Phase 1 - Day 2 Morning Prompt

```
Continue UniteDefi protocol development. Focus on resolver implementation and cross-chain coordination.

CURRENT FOCUS: Resolver Service Implementation

TASKS:
1. Create src/resolvers/basicResolver.ts:
   - Participate in dutch auctions by monitoring price decreases
   - Calculate profit margins (must be >1% to participate)
   - Handle auction bidding logic
   - Manage safety deposits across chains

2. Implement cross-chain transaction coordination in src/services/:
   - crossChainCoordinator.ts - Manages escrow deployment across chains
   - secretManager.ts - Handles secret generation and sharing for HTLC
   - transactionTracker.ts - Tracks transaction status across all chains

3. Add partial fill support:
   - Implement Merkle tree structure for order splitting
   - Generate N+1 secrets for N partial fills
   - Add logic to prevent secret exposure compromising remaining fills

TECHNICAL DETAILS:
- Use viem for EVM interactions
- Implement retry logic for failed transactions
- Add proper error handling with detailed logging
- Create interfaces for easy mocking in tests

COMMIT PATTERN:
- "feat: basic resolver with auction participation"
- "feat: cross-chain transaction coordination"
- "feat: partial fill support with Merkle proofs"

Focus on getting the resolver to successfully participate in a basic auction first, then add complexity.
```

### Phase 1 - Day 2 Afternoon Prompt

```
Final protocol foundation tasks. Focus on API layer and testing.

CURRENT FOCUS: REST API & WebSocket Implementation

TASKS:
1. Create Express.js server in src/api/:
   - server.ts - Main Express app setup
   - routes/swap.ts - Swap endpoints
   - routes/status.ts - Transaction status endpoints
   - middleware/validation.ts - Input validation with Joi

2. API Endpoints to implement:
```

POST /api/quote - Get swap quote with auction details
POST /api/swap - Execute swap and start auction
GET /api/status/:txId - Get transaction status
GET /api/chains - List supported chains and tokens
GET /api/auctions/active - List active auctions

```

3. WebSocket implementation:
- Add Socket.io for real-time auction updates
- Broadcast price changes every second during auctions
- Send transaction status updates
- Handle client connection management

4. Testing:
- Write unit tests for auction logic (small focused tests)
- Integration tests for 4 EVM chains
- Mock tests for resolver logic

COMMIT PATTERN:
- "feat: REST API with core swap endpoints"
- "feat: WebSocket for real-time auction updates"
- "test: auction logic unit tests"
- "test: EVM chains integration tests"

By end of Day 2, you should have a working basic dutch auction that can handle swaps across 4 EVM chains.
```

---

## Session 2: Web Application (`unite-defi-webapp`)

### Phase 1 - Day 1 Morning Prompt

````
You are building the web application for UniteDefi, a cross-chain swap platform. This needs to be a full-featured, 1inch-style interface that looks production-ready.

SETUP TASKS:
1. Create Next.js 14 app with TypeScript:
   ```bash
   npx create-next-app@latest unite-defi-webapp --typescript --tailwind --eslint --app
   cd unite-defi-webapp
````

2. Install required dependencies:

   ```bash
   npm install wagmi viem @rainbow-me/rainbowkit @tanstack/react-query
   npm install lucide-react @radix-ui/react-dialog @radix-ui/react-dropdown-menu
   npm install class-variance-authority clsx tailwind-merge
   ```

3. Setup project structure:
   ```
   src/
   ├── components/
   │   ├── ui/          # shadcn/ui components
   │   ├── swap/        # Swap-specific components
   │   └── layout/      # Layout components
   ├── hooks/           # Custom React hooks
   ├── lib/            # Utilities and configurations
   └── types/          # TypeScript type definitions
   ```

IMMEDIATE GOALS:

- Web3 integration with 4 EVM chains (Ethereum, Polygon, BSC, Arbitrum)
- Token selector with search functionality
- Chain selector with network switching
- Basic swap card layout

COMMIT PATTERN:

- "init: Next.js 14 app with TypeScript and Tailwind"
- "feat: Web3 integration with 4 EVM chains"
- "feat: token selector with search functionality"
- "feat: chain selector with network switching"

Start with the Next.js setup and basic Web3 configuration. Make small commits for each feature.

```

### Phase 1 - Day 1 Afternoon Prompt

```

Continue building the UniteDefi web app. Focus on core swap interface components.

CURRENT FOCUS: Swap Interface Implementation

TASKS:

1. Create components/swap/SwapCard.tsx:

   - Main swap interface with token inputs
   - Amount input with balance display
   - Swap direction toggle button
   - Clean, 1inch-style design with proper spacing

2. Create components/swap/TokenSelector.tsx:

   - Searchable token list with token logos
   - Token balance display
   - Popular tokens section
   - Custom token import functionality

3. Create components/swap/ChainSelector.tsx:

   - Dropdown with all 4 supported chains
   - Chain logos and names
   - Network switching functionality
   - Current chain indicator

4. Add Dutch Auction Price Display:
   - Real-time price updates during auction
   - Countdown timer showing auction progress
   - Current best bid display
   - Price impact calculation

DESIGN REQUIREMENTS:

- Use Tailwind CSS for styling
- Implement proper dark/light theme support
- Mobile-responsive design
- Smooth animations and transitions
- Match 1inch visual quality

COMMIT PATTERN:

- "feat: main swap card with amount inputs"
- "feat: token selector with search functionality"
- "feat: chain selector with network switching"
- "feat: dutch auction price display with countdown"

Focus on making the interface look professional and polished. Each component should be in its own commit.

```

### Phase 1 - Day 2 Prompt

```

Complete the web app foundation with advanced trading features and transaction flow.

CURRENT FOCUS: Advanced Features & Transaction Flow

TASKS:

1. Create components/swap/TransactionSettings.tsx:

   - Slippage tolerance controls (0.1%, 0.5%, 1%, custom)
   - Transaction deadline settings
   - Advanced settings modal
   - Gas price controls

2. Create components/swap/PriceImpact.tsx:

   - Price impact calculation and display
   - Route visualization showing swap path
   - Fee breakdown (gas, protocol fees)
   - Minimum received amount

3. Implement transaction flow:

   - components/swap/ConfirmationModal.tsx
   - Transaction progress with steps
   - Success/error states with proper messaging
   - Transaction hash display and explorer links

4. Add components/TransactionHistory.tsx:

   - List of past transactions with status
   - Transaction details modal
   - Filter by chain/status
   - Export functionality

5. Responsive design optimization:
   - Mobile-first approach
   - Touch-friendly interactions
   - Proper viewport handling
   - Test on different screen sizes

TECHNICAL INTEGRATION:

- Connect to protocol API endpoints
- Implement real-time WebSocket updates
- Add proper error handling
- Loading states for all async operations

COMMIT PATTERN:

- "feat: transaction settings (slippage, deadline)"
- "feat: price impact and route visualization"
- "feat: transaction confirmation and progress"
- "feat: transaction history with details"
- "feat: mobile responsive design"

By end of Day 2, you should have a complete, professional-looking swap interface.

```

---

## Session 3: Extension Development (`unite-defi-extension`)

### Phase 3 - Day 5 Prompt

```

You are building a Chrome extension wallet for UniteDefi that provides a chain-abstracted experience. Users see one USD balance and the extension handles cross-chain complexities automatically.

SETUP TASKS:

1. Create Chrome extension project:

   ```bash
   mkdir unite-defi-extension && cd unite-defi-extension
   npm init -y
   npm install vite @types/chrome typescript react @types/react react-dom @types/react-dom
   npm install @unite-defi/sdk wagmi viem
   ```

2. Create manifest.json for Manifest V3:

   ```json
   {
     "manifest_version": 3,
     "name": "UniteDefi Wallet",
     "version": "1.0.0",
     "permissions": ["storage", "tabs", "notifications"],
     "host_permissions": ["<all_urls>"],
     "background": { "service_worker": "background.js" },
     "content_scripts": [{ "matches": ["<all_urls>"], "js": ["content.js"] }],
     "action": { "default_popup": "popup.html" }
   }
   ```

3. Project structure:
   ```
   src/
   ├── popup/           # Main popup interface
   ├── background/      # Service worker
   ├── content/         # Content scripts
   ├── components/      # Shared React components
   ├── hooks/          # Custom hooks
   └── utils/          # Utility functions
   ```

CORE FEATURES TO BUILD:

1. Multi-chain key management (secure storage)
2. Chain-abstracted balance view (single USD amount)
3. Automatic chain detection for transactions
4. Auto-bridging logic when user lacks funds on target chain

COMMIT PATTERN:

- "init: Chrome extension Manifest V3 setup"
- "feat: extension architecture components"
- "feat: multi-chain key management system"
- "feat: chain-abstracted USD balance display"

Start with basic extension setup and secure key management.

```

### Phase 3 - Day 6 Prompt

```

Continue building the UniteDefi extension. Focus on the chain-abstracted user experience.

CURRENT FOCUS: Smart Transaction Flow

TASKS:

1. Implement src/popup/BalanceView.tsx:

   - Display single USD balance across all 8 chains
   - Real-time balance aggregation
   - Quick access to individual chain balances (expandable)
   - Portfolio performance tracking

2. Create src/popup/TransactionFlow.tsx:

   - Detect when user wants to transact on specific chain
   - Show gas requirements per chain
   - Display "insufficient funds" with auto-bridge suggestion
   - "Bridge & Execute" flow for seamless UX

3. Add src/background/AutoBridging.ts:

   - Calculate optimal route for gas + token swap
   - Find cheapest path to get required funds on target chain
   - Handle multi-step transactions (bridge then execute)
   - Queue management for pending operations

4. Create src/popup/SwapInterface.tsx:

   - Mini swap interface within popup
   - Connect to protocol service APIs
   - Quick swap for common pairs
   - Integration with main web app

5. Add notification system:
   - Chrome notifications for transaction status
   - Success/failure notifications
   - Auction updates for active swaps

TECHNICAL REQUIREMENTS:

- Secure communication between components
- Proper error handling and recovery
- Background sync for balance updates
- Offline mode considerations

COMMIT PATTERN:

- "feat: automatic chain detection for transactions"
- "feat: auto-bridging logic for seamless UX"
- "feat: integrated swap interface in extension"
- "feat: transaction notifications and history"

The extension should feel magical - users never worry about which chain they're on.

```

---

## Session 4: SDK Development (`unite-defi-sdk`)

### Phase 4 - Day 7 Prompt

```

You are building the TypeScript SDK for UniteDefi that allows developers to integrate cross-chain swapping into their applications.

SETUP TASKS:

1. Create TypeScript SDK project:

   ```bash
   mkdir unite-defi-sdk && cd unite-defi-sdk
   npm init -y
   npm install typescript @types/node viem @aptos-labs/ts-sdk stellar-sdk
   npm install -D @types/jest jest ts-jest
   ```

2. Project structure:

   ```
   src/
   ├── core/            # Core SDK functionality
   ├── chains/          # Chain-specific implementations
   ├── types/           # TypeScript type definitions
   ├── utils/           # Utility functions
   └── examples/        # Usage examples
   ```

3. Package.json configuration:
   ```json
   {
     "name": "@unite-defi/sdk",
     "main": "dist/index.js",
     "types": "dist/index.d.ts",
     "files": ["dist/"]
   }
   ```

CORE SDK FEATURES:

1. UniteDefiClient class with all swap functionality
2. Chain abstraction layer
3. Transaction status tracking
4. Type-safe API for all supported chains
5. Built-in error handling and retry logic

SDK API DESIGN:

```typescript
import { UniteDefiClient } from "@unite-defi/sdk";

const client = new UniteDefiClient({
  apiUrl: "https://api.unitedefi.com",
  chains: ["ethereum", "polygon", "sui", "stellar"],
});

// Get quote
const quote = await client.getQuote({
  fromChain: "ethereum",
  toChain: "polygon",
  fromToken: "USDC",
  toToken: "USDC",
  amount: "1000",
});

// Execute swap
const swap = await client.executeSwap(quote, {
  signer: wallet,
});
```

COMMIT PATTERN:

- "feat: TypeScript SDK core implementation"
- "feat: chain abstraction layer for SDK"
- "feat: transaction tracking and status"
- "feat: comprehensive type definitions"

Build a clean, intuitive API that abstracts away cross-chain complexity.

```

---

## Session 5: Documentation & Testing (`unite-defi-docs`)

### Phase 4 - Day 7-8 Prompt

```

You are responsible for comprehensive testing and documentation for UniteDefi. This is critical for the hackathon demo and future adoption.

SETUP TASKS:

1. Create documentation site:

   ```bash
   mkdir unite-defi-docs && cd unite-defi-docs
   npx create-docusaurus@latest . --typescript
   ```

2. Create testing repositories for E2E tests:
   ```bash
   mkdir unite-defi-testing && cd unite-defi-testing
   npm install playwright @playwright/test cypress jest
   ```

DOCUMENTATION TASKS:

1. API Documentation:

   - Complete OpenAPI 3.0 specification
   - Interactive API explorer
   - Code examples for all endpoints
   - Authentication and rate limiting docs

2. Integration Guides:

   - Step-by-step integration for each supported chain
   - SDK usage examples with real code
   - Troubleshooting common issues
   - Best practices for production use

3. User Guides:
   - How to use the web app
   - Extension installation and setup
   - Understanding dutch auctions
   - Security best practices

TESTING TASKS:

1. End-to-End Testing:

   - Test complete swap flow for all 8 chains
   - Test extension functionality across different dApps
   - Performance testing under load
   - Mobile responsiveness testing

2. Security Testing:

   - Contract security audit checklist
   - Extension security review
   - API security testing
   - Key management security testing

3. Demo Preparation:
   - Create demo scripts for hackathon presentation
   - Video recording setup and scripts
   - Live demo environments
   - Backup plans for demo failures

COMMIT PATTERN:

- "docs: comprehensive API documentation"
- "docs: integration guides for all chains"
- "test: comprehensive E2E testing suite"
- "test: security testing and audit"
- "docs: demo preparation and scripts"

Create documentation that developers will actually want to use, and tests that catch real issues.

````

---

## Common Commands & Debugging

### Git Commands for Small Commits
```bash
# Stage and commit frequently
git add src/specific-file.ts
git commit -m "feat: specific feature implementation"
git push origin main

# Check what you're about to commit
git diff --cached
git status

# Never commit these
echo "node_modules/" >> .gitignore
echo "dist/" >> .gitignore
echo ".env" >> .gitignore
echo "*.log" >> .gitignore
````

### Testing Commands

```bash
# Run tests before committing
npm test
npm run lint
npm run build

# Integration testing
npm run test:integration
npm run test:e2e
```

### Debugging Common Issues

```bash
# Check if services are running
curl http://localhost:3000/api/health
netstat -an | grep 3000

# View logs
tail -f logs/app.log
docker logs container-name

# Check blockchain connections
npx hardhat console --network polygon
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' https://polygon-rpc.com
```

Remember: **Small commits every 30-45 minutes**. If a commit is getting large, break it into smaller pieces!
