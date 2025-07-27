# Phase 2: Node.js Services Implementation Prompts

## General Instructions for All Chains

- Continue working in your git worktree at `resolver-branches/[chain-name]`
- Your worktree already has all the resolver code from Phase 1
- Convert test suite logic into production-ready Node.js services
- Implement both seller service and resolver service
- NO mocking - use real chain interactions
- Services must handle network failures gracefully
- Implement proper logging and monitoring
- If blockers arise, update your `BLOCKERS_[CHAIN_NAME].md` in your worktree

## Service Architecture Overview

The UniteDefi protocol requires three main services:

### 1. **Relayer Service (Express API)**
   - Receives gasless user requests (trade details + hash(secret))
   - Posts Dutch auctions on-chain
   - Monitors escrow creation by resolvers
   - Moves user funds to escrow after resolver commits
   - Reveals secrets after confirmations
   - Handles user authentication and request validation

### 2. **Resolver Service**
   - Monitors Dutch auction price drops
   - Competes to fill orders at profitable prices
   - Creates escrows with safety deposits on BOTH chains
   - Deposits tokens on destination chain
   - Monitors for secret reveal to claim source tokens
   - Manages timeout scenarios

### 3. **Shared Components**
   - Chain clients for multi-chain interaction
   - Event monitoring and filtering
   - Transaction management and retry logic
   - Safety deposit tracking
   - Confirmation monitoring

## Service Requirements

Each implementation must include:

1. **Relayer API Endpoints**:
   - `POST /swap` - Receive user's gasless swap request
   - `POST /escrow-created` - Resolver notifies escrow creation
   - `POST /tokens-deposited` - Resolver confirms token deposit
   - `GET /swap-status/:id` - Check swap status
   - Internal: Move funds and reveal secrets

2. **Resolver Service Features**:
   - Dutch auction monitoring
   - Profitability calculation
   - Escrow creation with safety deposits
   - Exclusive lock enforcement
   - Secret monitoring and claiming
   - Timeout handling for safety deposits

3. **Error Handling**:
   - Retry logic with exponential backoff
   - Circuit breakers for RPC failures
   - Graceful degradation
   - Safety deposit recovery mechanisms

---

## EVM-Compatible Chains

### Monad Services

```
You are converting the Monad <> Base Sepolia test suite into production Node.js services.

Context:
- You have completed the test suite in Phase 1
- Monad is fully EVM-compatible
- Use ethers.js or viem for chain interaction
- Implement the three-service architecture (Relayer, Resolver, Shared)

Your tasks:
1. Continue in your worktree (branch: `monad-integration`)
2. Create service structure:
   - src/services/chains/monad/
   - src/services/relayer/
   - src/services/resolver/
3. Implement Relayer Service (Express API):
   - POST /swap endpoint:
     * Validate user request (trade params + hash(secret))
     * Post Dutch auction on-chain
     * Return swap ID
   - POST /escrow-created endpoint:
     * Verify resolver created escrows on both chains
     * Move user funds to source escrow
     * Update swap status
   - POST /tokens-deposited endpoint:
     * Verify resolver deposited on destination
     * Wait for confirmations
     * Reveal secret on destination chain
   - GET /swap-status/:id endpoint
4. Implement Monad Resolver Service:
   - Monitor Dutch auction events
   - Calculate profitability (price vs gas costs)
   - Create escrows with 0.001 ETH safety deposits
   - Call relayer API after escrow creation
   - Deposit tokens on destination
   - Monitor for secret reveal
   - Claim tokens using revealed secret
   - Handle timeout scenarios
5. Implement safety deposit mechanics:
   - Track deposit amounts
   - Monitor timeout windows
   - Implement claim logic for abandoned swaps
6. Add monitoring and reliability:
   - Prometheus metrics
   - Health checks for both services
   - Circuit breakers for RPC failures
   - Transaction retry logic

Requirements:
- Relayer must verify on-chain state before actions
- Resolver must handle exclusive lock properly
- Safety deposits must be recoverable
- Handle concurrent swaps without conflicts
- Test resolver competition scenarios

Commit pattern:
- `feat: Add Monad relayer service base`
- `feat: Implement swap endpoints`
- `feat: Add Monad resolver service`
- `feat: Implement safety deposit logic`
- `feat: Add monitoring and metrics`
```

### Etherlink Services

```
You are converting the Etherlink <> Base Sepolia test suite into production Node.js services.

Context:
- You have completed the test suite in Phase 1
- Etherlink is Tezos' EVM L2
- Consider Etherlink's specific characteristics (block time, finality)
- Use ethers.js or viem

Your tasks:
1. Continue in your worktree (branch: `etherlink-integration`)
2. Create service structure:
   - src/services/chains/etherlink/
   - Implement EtherlinkChainService
3. Implement EtherlinkSellerService:
   - Handle Etherlink's block time
   - Adjust timeout strategies
   - Implement proper nonce management
4. Implement EtherlinkResolverService:
   - Consider L2 <> L1 messaging delays
   - Optimize for Etherlink's fee structure
   - Handle reorgs if applicable
5. Add Etherlink-specific features:
   - Block confirmation strategies
   - Fee optimization
   - Batch transaction support if available
6. Integration and monitoring:
   - Custom metrics for L2 behavior
   - Alert on unusual block times
   - Track L2-specific errors

Requirements:
- Account for L2 characteristics
- Implement circuit breaker for RPC issues
- Handle nonce gaps
- Test with concurrent operations

Commit atomically for each component.
```

### Tron Services

```
You are converting the Tron <> Base Sepolia test suite into production Node.js services.

Context:
- You have completed the test suite in Phase 1
- Tron uses TVM with energy/bandwidth model
- Use TronWeb for interaction
- Handle resource management

Your tasks:
1. Continue in your worktree (branch: `tron-integration`)
2. Create service structure:
   - src/services/chains/tron/
   - Implement TronChainService
3. Implement TronSellerService:
   - Energy/bandwidth calculation
   - TRX balance management
   - Handle resource delegation
   - Implement Tron event monitoring
4. Implement TronResolverService:
   - Resource estimation before execution
   - Optimize energy usage
   - Handle TronWeb specific errors
   - Implement Tron transaction monitoring
5. Tron-specific features:
   - Resource management service
   - Energy price monitoring
   - Bandwidth optimization
   - Handle frozen TRX if needed
6. Integration requirements:
   - Convert between TRX and SUN
   - Handle Tron's 1-second blocks
   - Monitor resource consumption

Requirements:
- Never run out of energy/bandwidth
- Handle Tron's unique error codes
- Test resource edge cases
- Implement resource alerts

Make focused commits for each resource-related feature.
```

---

## Move-Based Chains

### Aptos Services

```
You are converting the Aptos <> Base Sepolia test suite into production Node.js services.

Context:
- You have completed Move modules in Phase 1
- Use Aptos SDK for TypeScript
- Handle Move's resource model
- Adapt the three-service architecture for Move

Your tasks:
1. Continue in your worktree (branch: `aptos-integration`)
2. Create service structure:
   - src/services/chains/aptos/
   - src/services/relayer/
   - src/services/resolver/
3. Implement Relayer Service for Aptos:
   - POST /swap endpoint:
     * Validate request and hash(secret)
     * Call Move function to create auction
     * Store swap metadata
   - POST /escrow-created endpoint:
     * Verify escrow resources created
     * Execute Move function to transfer user funds
     * Handle resource account permissions
   - POST /tokens-deposited endpoint:
     * Verify via Aptos events
     * Wait for confirmations
     * Call Move function to reveal secret
   - Aptos-specific adaptations:
     * Use resource accounts for deterministic addresses
     * Handle Coin vs Token standards
     * Manage sequence numbers
4. Implement Aptos Resolver Service:
   - Monitor Move events for auctions
   - Calculate profitability in APT
   - Create escrow resources with safety deposits
   - Handle exclusive lock via Move logic
   - Call relayer after escrow creation
   - Monitor for secret reveal in events
   - Execute withdrawal Move function
5. Handle Move-specific challenges:
   - Resource existence checks
   - Abort code handling
   - Event indexing differences
   - Gas estimation in gas units
6. Cross-chain coordination:
   - Aptos event to Base Sepolia mapping
   - Handle block time differences
   - Coordinate confirmations

Requirements:
- Safety deposits in APT (0.001 APT)
- Handle Move's resource model properly
- Implement proper error recovery
- Test with Aptos's parallel execution
- Monitor resource account balances

Commit pattern:
- `feat: Add Aptos relayer service`
- `feat: Implement Move function calls`
- `feat: Add Aptos resolver service`
- `feat: Handle resource accounts`
- `feat: Add cross-chain coordination`
```

### Sui Services

```
You are converting the Sui <> Base Sepolia test suite into production Node.js services.

Context:
- You have completed Sui Move modules in Phase 1
- Use Sui TypeScript SDK
- Handle object-based model
- Manage shared object contention

Your tasks:
1. Continue in your worktree (branch: `sui-integration`)
2. Create service structure:
   - src/services/chains/sui/
   - Implement SuiChainService
3. Implement SuiSellerService:
   - Shared object management
   - Programmable transaction building
   - Event subscription via websocket
   - Object version tracking
4. Implement SuiResolverService:
   - Object discovery and caching
   - Transaction block construction
   - Gas coin management
   - Concurrent transaction handling
5. Sui-specific challenges:
   - Handle object equivocation
   - Manage gas coin splitting/merging
   - Optimize for object access patterns
   - Cache object references
6. Integration features:
   - Object state synchronization
   - Event ordering guarantees
   - Failed transaction recovery

Requirements:
- Never lose track of objects
- Handle shared object congestion
- Implement object caching strategy
- Test with high concurrency

Make atomic commits for object management features.
```

---

## Cosmos Ecosystem

### Cosmos Services

```
You are converting the Cosmos <> Base Sepolia test suite into production Node.js services.

Context:
- You have completed CosmWasm contracts in Phase 1
- Use CosmJS for interaction
- Consider IBC possibilities
- Handle Cosmos-specific features

Your tasks:
1. Continue in your worktree (branch: `cosmos-integration`)
2. Create service structure:
   - src/services/chains/cosmos/
   - Implement CosmosChainService
3. Implement CosmosSellerService:
   - CosmWasm contract execution
   - Cosmos event monitoring
   - Handle contract migrations
   - Implement proper signing
4. Implement CosmosResolverService:
   - Query optimization
   - Gas estimation for Cosmos
   - Handle chain upgrades
   - Multi-message transactions
5. Cosmos-specific features:
   - Amino/Proto encoding handling
   - Account sequence management
   - Fee grant support if applicable
   - Memo field utilization
6. Future IBC readiness:
   - Structure for IBC messages
   - Relayer integration points
   - Channel management prep

Requirements:
- Handle chain upgrades gracefully
- Implement proper fee strategies
- Test with multiple message types
- Account for block time variability

Commit each Cosmos feature separately.
```

---

## Other Platforms

### Near Services

```
You are converting the Near <> Base Sepolia test suite into production Node.js services.

Context:
- You have completed Near contracts in Phase 1
- Use near-api-js
- Handle async nature
- Manage storage staking

Your tasks:
1. Continue in your worktree (branch: `near-integration`)
2. Create service structure:
   - src/services/chains/near/
   - Implement NearChainService
3. Implement NearSellerService:
   - Async contract calls
   - Promise resolution tracking
   - Storage management
   - Access key rotation
4. Implement NearResolverService:
   - Handle callback patterns
   - Implement retry for failed promises
   - Gas estimation with safety margin
   - Storage deposit management
5. Near-specific challenges:
   - Promise ID tracking
   - Async state management
   - Storage cost prediction
   - Handle view vs change methods
6. Integration requirements:
   - Cross-chain state correlation
   - Handle Near's finality
   - Implement proper logging for async flows

Requirements:
- Never lose track of promises
- Handle storage stake properly
- Test with promise chains
- Implement callback monitoring

Make commits for each async pattern implementation.
```

### Stellar Services

```
You are converting the Stellar <> Base Sepolia test suite into production Node.js services.

Context:
- You have completed Soroban contracts in Phase 1
- Use Stellar SDK
- Handle Stellar's asset model
- Manage sequence numbers

Your tasks:
1. Continue in your worktree (branch: `stellar-integration`)
2. Create service structure:
   - src/services/chains/stellar/
   - Implement StellarChainService
3. Implement StellarSellerService:
   - Soroban contract interaction
   - Asset trustline management
   - Transaction building with memos
   - Horizon API integration
4. Implement StellarResolverService:
   - Operation batching
   - Fee bumping strategies
   - Sequence number management
   - Path payment evaluation
5. Stellar-specific features:
   - Multi-operation transactions
   - Sponsor reserves if needed
   - Handle ledger close times
   - Asset issuer coordination
6. Integration features:
   - Bridge asset management
   - Cross-chain asset mapping
   - Handle Stellar's 5-second ledgers

Requirements:
- Never break sequence numbers
- Handle trustline limits
- Test operation batching limits
- Implement fee bump transactions

Commit each Stellar feature atomically.
```

### Bitcoin Services

```
You are converting the Bitcoin <> Base Sepolia test suite into production Node.js services.

Context:
- You have completed Bitcoin Script HTLC in Phase 1
- Use bitcoinjs-lib and Bitcoin RPC
- Handle UTXO management
- Coordinate with Base Sepolia

Your tasks:
1. Continue in your worktree (branch: `bitcoin-integration`)
2. Create service structure:
   - src/services/chains/bitcoin/
   - Implement BitcoinChainService
3. Implement BitcoinSellerService:
   - UTXO management service
   - P2SH address generation
   - Transaction construction
   - Block height monitoring
4. Implement BitcoinResolverService:
   - UTXO discovery
   - Fee estimation (sat/vByte)
   - Transaction broadcasting
   - Confirmation tracking
5. Bitcoin-specific features:
   - Implement UTXO selection algorithms
   - Handle transaction malleability
   - RBF (Replace-By-Fee) support
   - CPFP (Child-Pays-For-Parent) if needed
6. Cross-chain coordination:
   - Block height correlation
   - Timeout calculation in blocks
   - Handle reorgs

Requirements:
- Never create dust outputs
- Implement proper fee estimation
- Handle mempool congestion
- Test timeout scenarios thoroughly

Make focused commits for each UTXO feature.
```

### TON Services

```
You are converting the TON <> Base Sepolia test suite into production Node.js services.

Context:
- You have completed TON contracts in Phase 1
- Use TON SDK
- Handle async messages
- Manage sharding complexity

Your tasks:
1. Continue in your worktree (branch: `ton-integration`)
2. Create service structure:
   - src/services/chains/ton/
   - Implement TONChainService
3. Implement TONSellerService:
   - Message construction
   - Workchain management
   - Event monitoring across shards
   - BOC (Bag of Cells) handling
4. Implement TONResolverService:
   - Message routing logic
   - Gas management
   - Handle message bouncing
   - Shard-aware monitoring
5. TON-specific challenges:
   - Async message tracking
   - Handle split/merge events
   - Message ordering issues
   - Gas computation complexity
6. Integration requirements:
   - Cross-shard coordination
   - Message delivery confirmation
   - Handle TON's unique errors

Requirements:
- Track all async messages
- Handle shard dynamics
- Implement message retry logic
- Test with high message volume

Commit each messaging feature separately.
```

### ICP Services

```
You are converting the ICP <> Base Sepolia test suite into production Node.js services.

Context:
- You have completed ICP canisters in Phase 1
- Use agent-js
- Leverage ICP's unique features
- Handle cycles management

Your tasks:
1. Continue in your worktree (branch: `icp-integration`)
2. Create service structure:
   - src/services/chains/icp/
   - Implement ICPChainService
3. Implement ICPSellerService:
   - Canister call management
   - Cycles monitoring
   - HTTP outcall orchestration
   - State certification checks
4. Implement ICPResolverService:
   - Update vs query optimization
   - Cycles top-up automation
   - Inter-canister call tracking
   - Certified variable validation
5. ICP-specific features:
   - HTTP outcalls for Base monitoring
   - Threshold signatures if applicable
   - Canister upgrade handling
   - Cycles wallet integration
6. Advanced features:
   - Use ICP as oracle for other chains
   - Implement certified cross-chain state
   - Leverage ICP's timers

Requirements:
- Never run out of cycles
- Handle canister upgrades
- Implement proper certification checks
- Test HTTP outcall reliability

Make atomic commits for each ICP feature.
```

---

## Service Standards

All services must implement:

### 1. Configuration Management

```typescript
- Environment variables for all settings
- Separate configs for testnet/mainnet
- Hot-reload for non-critical configs
- Secrets management (never in code)
```

### 2. Error Handling

```typescript
- Exponential backoff with jitter
- Circuit breakers for external calls
- Graceful degradation
- Structured error logging
```

### 3. Monitoring & Metrics

```typescript
- Prometheus metrics for all operations
- Health check endpoints
- Performance tracking
- Alert thresholds
```

### 4. Testing Requirements

```typescript
- Unit tests for business logic
- Integration tests with testnets
- Load testing for concurrent operations
- Failure injection testing
```

### 5. Documentation

```typescript
- API documentation
- Configuration guide
- Operational runbook
- Troubleshooting guide
```

## Commit Guidelines for Phase 2

- `feat: Implement [chain] chain service base`
- `feat: Add [chain] seller service`
- `feat: Add [chain] resolver service`
- `feat: Implement [chain] event monitoring`
- `feat: Add retry logic for [chain]`
- `feat: Implement metrics for [chain]`
- `test: Add unit tests for [chain] service`
- `docs: Add [chain] service documentation`
- `fix: Handle [specific error] in [chain]`
- `perf: Optimize [chain] query performance`

## Expected Deliverables

Each chain should produce:

```
src/services/chains/[chain]/
├── index.ts
├── [Chain]ChainService.ts
├── [Chain]SellerService.ts
├── [Chain]ResolverService.ts
├── config.ts
├── types.ts
├── utils/
├── __tests__/
└── README.md
```
