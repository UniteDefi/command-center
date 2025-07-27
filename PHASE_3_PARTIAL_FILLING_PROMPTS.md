# Phase 3: Partial Filling Feature Implementation Prompts

## General Instructions for All Chains
- Continue working in your git worktree at `resolver-branches/[chain-name]`
- Your worktree has all the code from Phase 1 and Phase 2
- Modify existing contracts and services to support partial fills
- Implement atomic partial filling with proper accounting
- NO shortcuts - maintain security and atomicity
- Test extensively with various partial fill scenarios
- If blockers arise, update `BLOCKERS_[CHAIN_NAME].md` in your worktree

## Partial Filling Architecture

With the relayer-resolver architecture, partial filling introduces new complexities:

### Core Changes
1. **Order Splitting**: Allow multiple resolvers to fill portions of one order
2. **Concurrent Escrows**: Multiple escrows can exist for one auction
3. **Relayer Coordination**: Relayer must track multiple partial fills
4. **Safety Deposits**: Each partial fill requires proportional safety deposits
5. **Secret Management**: Same secret works across all partial escrows

### Modified Flow for Partial Fills
1. User creates order for 100 USDT → 100 DAI
2. Relayer posts auction with partial fill enabled
3. Resolver A fills 30% (30 USDT → 30 DAI)
   - Creates escrows with 0.0003 ETH safety deposits
   - Notifies relayer of partial fill
4. Resolver B fills 70% (70 USDT → 70 DAI)
   - Creates separate escrows with 0.0007 ETH deposits
   - Notifies relayer of partial fill
5. Relayer coordinates:
   - Moves user funds to multiple escrows
   - Tracks total filled amount
   - Reveals secret to all escrows after confirmations
6. Resolvers claim their portions using the same secret

### Key Requirements
1. **Atomic Partial Fills**: Each fill is independent and atomic
2. **Price Consistency**: All resolvers get the same price
3. **Fill Tracking**: Accurate tracking across multiple escrows
4. **No Over-Filling**: Total fills cannot exceed order amount
5. **Proportional Deposits**: Safety deposits match fill percentage

---

## EVM-Compatible Chains

### Monad Partial Filling
```
You are implementing partial filling support for Monad <> Base Sepolia.

Context:
- You have working relayer and resolver services from Phase 2
- Monad is fully EVM-compatible
- Must maintain the security model with multiple escrows
- Backward compatibility with full fills

Your tasks:
1. Continue in your worktree (branch: `monad-integration`)
2. Modify smart contracts:
   - Update SimpleDutchAuction.sol:
     * Add partial fill support flag
     * Track total and filled amounts
     * Allow multiple resolver fills at same price
     * Emit PartialFill events
   - Update EscrowFactory:
     * Support multiple escrows per auction
     * Track escrow-to-auction mapping
     * Proportional safety deposits
3. Contract structure updates:
   ```solidity
   struct Auction {
     uint256 totalAmount;
     uint256 filledAmount;
     bool allowPartialFills;
     uint256 currentPrice;
     mapping(address => uint256) resolverFills;
     address[] escrowAddresses;
   }
   ```
4. Update Relayer Service:
   - POST /swap modifications:
     * Accept partialFillEnabled flag
     * Set minimum fill amount
   - Handle multiple POST /escrow-created calls:
     * Track partial fills
     * Validate total doesn't exceed order
     * Move proportional user funds
   - Secret reveal to multiple escrows:
     * Reveal to all partial escrows
     * Handle failed reveals gracefully
5. Update Resolver Service:
   - Implement fill amount strategy:
     * Calculate optimal fill amount
     * Check remaining available amount
     * Proportional safety deposits
   - Partial fill execution:
     * Create escrow for partial amount
     * Notify relayer of partial fill
     * Monitor other resolvers' fills
6. Safety deposit calculations:
   - Base: 0.001 ETH for full fill
   - Partial: (fillAmount / totalAmount) * 0.001 ETH
   - Minimum: 0.0001 ETH per fill
7. Testing requirements:
   - Multiple resolvers race condition
   - Exact amount matching (99.9% + 0.1%)
   - Safety deposit recovery for each resolver
   - Gas optimization for small fills

Requirements:
- No over-filling protection
- Same price for all partial fills
- Independent escrow management
- Atomic partial operations
- Proportional safety deposits

Commits:
- `feat: Add partial fill support to auction`
- `feat: Update escrow factory for multiple escrows`
- `feat: Implement relayer partial fill coordination`
- `feat: Add resolver partial fill strategy`
- `feat: Implement proportional safety deposits`
- `test: Add partial fill test scenarios`
```

### Etherlink Partial Filling
```
You are implementing partial filling support for Etherlink <> Base Sepolia.

Context:
- Building on Phase 2 services
- Consider Etherlink's L2 characteristics
- Optimize for L2 fee structure

Your tasks:
1. Continue in your worktree (branch: `etherlink-integration`)
2. Smart contract modifications:
   - Implement efficient storage for partial fills
   - Consider L2 calldata costs
   - Optimize events for L2
3. Partial fill strategy:
   - Batch multiple small fills
   - Implement fill aggregation
   - Minimize state updates
4. Service updates:
   - L2-optimized fill calculation
   - Batch transaction submission
   - Monitor L1 data availability
5. Testing focus:
   - High-frequency small fills
   - L2-specific edge cases
   - Data availability delays

L2-specific requirements:
- Optimize calldata usage
- Batch where possible
- Consider L1 finality for fills
```

### Tron Partial Filling
```
You are implementing partial filling support for Tron <> Base Sepolia.

Context:
- Tron's energy/bandwidth model
- TVM considerations
- Resource optimization crucial

Your tasks:
1. Continue in your worktree (branch: `tron-integration`)
2. Contract modifications:
   - Energy-efficient partial fill tracking
   - Optimize storage patterns
   - Bandwidth-conscious events
3. Resource management:
   - Calculate energy per partial fill
   - Implement fill size optimization
   - Prevent resource exhaustion
4. Service updates:
   - Resource-aware fill sizing
   - Energy price consideration
   - Bandwidth batching
5. Tron-specific testing:
   - Energy limits per fill
   - Bandwidth optimization
   - Multi-resolver resource sharing

Requirements:
- Never exceed energy limits
- Optimize bandwidth usage
- Fair resource distribution
```

---

## Move-Based Chains

### Aptos Partial Filling
```
You are implementing partial filling support for Aptos <> Base Sepolia.

Context:
- Move's resource model
- Aptos parallel execution benefits partial fills
- Building on relayer-resolver architecture from Phase 2

Your tasks:
1. Continue in your worktree (branch: `aptos-integration`)
2. Move module updates for partial fills:
   ```move
   struct Auction has key, store {
     total_amount: u64,
     filled_amount: u64,
     allow_partial_fills: bool,
     current_price: u64,
     resolver_fills: Table<address, u64>,
     escrow_addresses: vector<address>,
   }
   
   struct PartialEscrow has key {
     auction_id: address,
     resolver: address,
     fill_amount: u64,
     safety_deposit: u64,
     // ... other HTLC fields
   }
   ```
3. Implement Move-specific fill coordination:
   - Atomic partial fill commits
   - Resource-based escrow tracking
   - Parallel-safe auction updates
   - Table for efficient lookups
4. Update Relayer Service for Aptos:
   - Handle multiple escrow creation events
   - Move proportional funds to each escrow
   - Batch secret reveals to all escrows
   - Track fills via Move events
5. Update Resolver Service:
   - Calculate fill amounts in APT terms
   - Create PartialEscrow resources
   - Proportional safety deposits in APT
   - Monitor other resolvers via events
6. Aptos-specific optimizations:
   - Leverage parallel execution for concurrent fills
   - Use resource accounts for escrow addresses
   - Optimize Table access patterns
   - Handle resource conflicts gracefully
7. Safety deposit calculations:
   - Base: 0.001 APT for full fill
   - Partial: (fillAmount / totalAmount) * 0.001 APT
   - Minimum: 0.0001 APT per fill
8. Testing requirements:
   - Parallel resolver competition
   - Resource contention scenarios
   - Exact fill matching
   - Move abort recovery

Aptos advantages:
- Parallel execution enables true concurrent fills
- Resource model prevents double-spending naturally
- Tables provide efficient fill tracking
- Deterministic execution helps coordination

Requirements:
- Atomic partial fill operations
- No over-filling via Move checks
- Proportional safety deposits
- Event-based coordination
- Resource cleanup after completion

Commits:
- `feat: Add partial fill to Move modules`
- `feat: Implement PartialEscrow resource`
- `feat: Update relayer for Aptos partials`
- `feat: Add parallel fill optimization`
- `test: Add Move partial fill tests`
```

### Sui Partial Filling
```
You are implementing partial filling support for Sui <> Base Sepolia.

Context:
- Sui's object model
- Shared objects for coordination
- Building on Phase 2

Your tasks:
1. Continue in your worktree (branch: `sui-integration`)
2. Sui Move updates:
   ```move
   struct Auction has key {
     id: UID,
     total_amount: u64,
     filled_amount: u64,
     fills: VecMap<address, u64>,
   }
   ```
3. Object-based filling:
   - Shared auction object
   - Owned fill receipts
   - Programmable transactions
4. Sui-specific patterns:
   - Use shared objects wisely
   - Implement fill receipts as owned objects
   - Batch operations in PTBs
5. Service updates:
   - Object caching for fills
   - PTB construction for atomicity
   - Handle object versioning
6. Testing focus:
   - Concurrent shared object access
   - Object contention scenarios
   - PTB gas optimization

Leverage Sui's strengths:
- Owned objects for receipts
- PTBs for complex flows
- Object composition
```

---

## Cosmos Ecosystem

### Cosmos Partial Filling
```
You are implementing partial filling support for Cosmos <> Base Sepolia.

Context:
- CosmWasm contracts
- Building on Phase 2
- Consider IBC future

Your tasks:
1. Continue in your worktree (branch: `cosmos-integration`)
2. CosmWasm updates:
   ```rust
   pub struct Auction {
     pub total_amount: Uint128,
     pub filled_amount: Uint128,
     pub fills: HashMap<Addr, Uint128>,
   }
   ```
3. Implement fill mechanics:
   - State management for partial fills
   - Event emission for tracking
   - Query optimization
4. Cosmos patterns:
   - Efficient state storage
   - Submessage coordination
   - Reply handling for fills
5. Service modifications:
   - Multi-message transactions
   - State query optimization
   - Event streaming updates
6. Testing requirements:
   - Concurrent fill simulation
   - State consistency checks
   - Gas profiling

Consider IBC:
- Future cross-chain fill aggregation
- IBC packet optimization
- Relayer coordination
```

---

## Other Platforms

### Near Partial Filling
```
You are implementing partial filling support for Near <> Base Sepolia.

Context:
- Near's async model
- Storage staking per fill
- Building on Phase 2

Your tasks:
1. Continue in your worktree (branch: `near-integration`)
2. Contract updates:
   ```rust
   pub struct Auction {
     pub total_amount: Balance,
     pub filled_amount: Balance,
     pub fills: UnorderedMap<AccountId, Balance>,
   }
   ```
3. Async fill coordination:
   - Promise-based filling
   - Callback aggregation
   - Storage management
4. Near-specific challenges:
   - Storage cost per resolver
   - Async fill confirmation
   - Promise chain management
5. Service updates:
   - Storage deposit handling
   - Promise tracking for fills
   - Callback monitoring
6. Testing focus:
   - Storage stake edge cases
   - Promise failure handling
   - Concurrent async fills

Handle Near's model:
- Pre-pay storage for fills
- Track promise chains
- Handle async failures
```

### Stellar Partial Filling
```
You are implementing partial filling support for Stellar <> Base Sepolia.

Context:
- Soroban contracts
- Stellar's operation model
- Building on Phase 2

Your tasks:
1. Continue in your worktree (branch: `stellar-integration`)
2. Soroban updates:
   - Implement fill tracking
   - Use Stellar's native features
   - Optimize for operations
3. Stellar-specific approach:
   - Leverage multi-op transactions
   - Use memo for fill coordination
   - Path payments for efficiency
4. Contract modifications:
   - Storage-efficient fill tracking
   - Event emission patterns
   - Query optimization
5. Service updates:
   - Operation batching
   - Horizon monitoring
   - Path finding integration
6. Testing requirements:
   - Multi-operation limits
   - Path payment scenarios
   - Memo field utilization

Stellar advantages:
- Native operation batching
- Built-in path payments
- Efficient multi-party txns
```

### Bitcoin Partial Filling
```
You are implementing partial filling support for Bitcoin <> Base Sepolia.

Context:
- Bitcoin Script limitations
- UTXO model constraints
- Creative solutions needed

Your tasks:
1. Continue in your worktree (branch: `bitcoin-integration`)
2. Bitcoin Script approach:
   - Multi-input HTLC design
   - Partial UTXO spending
   - Merkle tree coordination
3. Implementation strategy:
   - Create multiple HTLCs for one order
   - Use OP_RETURN for metadata
   - Coordinate via Base Sepolia
4. UTXO management:
   - Split large UTXOs
   - Track partial spends
   - Optimize UTXO set
5. Service modifications:
   - UTXO pool management
   - Partial spend tracking
   - Fee optimization for small fills
6. Testing critical:
   - UTXO fragmentation
   - Fee economics
   - Confirmation timing

Bitcoin constraints:
- No native partial filling
- UTXO model limitations
- Creative coordination needed
```

### TON Partial Filling
```
You are implementing partial filling support for TON <> Base Sepolia.

Context:
- TON's async messages
- Sharding implications
- Building on Phase 2

Your tasks:
1. Continue in your worktree (branch: `ton-integration`)
2. Smart contract updates:
   - Message-based fill tracking
   - Shard-aware design
   - Async fill aggregation
3. TON-specific patterns:
   - Use internal messages for fills
   - Handle cross-shard fills
   - Implement fill collection
4. Sharding considerations:
   - Distribute fills across shards
   - Handle shard split/merge
   - Optimize message routing
5. Service updates:
   - Message tracking system
   - Shard monitoring
   - Fill aggregation logic
6. Testing complexity:
   - Cross-shard scenarios
   - Message ordering
   - Async timeout handling

TON challenges:
- Async fill coordination
- Shard boundaries
- Message complexity
```

### ICP Partial Filling
```
You are implementing partial filling support for ICP <> Base Sepolia.

Context:
- Canister architecture
- ICP's unique features
- Building on Phase 2

Your tasks:
1. Continue in your worktree (branch: `icp-integration`)
2. Canister updates:
   - Implement fill tracking
   - Use stable memory
   - Inter-canister coordination
3. ICP-specific features:
   - Threshold signatures for coordination
   - HTTP outcalls for monitoring
   - Timer-based aggregation
4. Architecture changes:
   - Fill aggregator canister
   - State certification
   - Atomic multi-canister updates
5. Service modifications:
   - Certified queries for fills
   - Timer-based settlement
   - Cross-canister atomicity
6. Testing requirements:
   - Inter-canister scenarios
   - Upgrade safety
   - Timer reliability

Leverage ICP:
- Certified state
- HTTP outcalls
- Threshold signatures
```

---

## Partial Filling Standards

### Security Requirements
1. **Atomicity**: Each partial fill must be atomic
2. **No Double Spending**: Prevent over-filling
3. **Price Consistency**: Same price for all partials
4. **Timeout Safety**: Handle timeouts with partials
5. **State Consistency**: Cross-chain state sync

### Implementation Standards
1. **Fill Tracking**: Efficient storage patterns
2. **Event Design**: Clear partial fill events
3. **Gas Optimization**: Efficient for small fills
4. **Concurrency**: Handle simultaneous fills
5. **Recovery**: Failed fill recovery mechanism

### Testing Requirements
1. **Concurrent Fills**: Multiple resolvers
2. **Edge Cases**: Dust amounts, rounding
3. **Failure Modes**: Partial timeout scenarios
4. **Gas Analysis**: Cost per fill size
5. **Performance**: Throughput testing

## Commit Guidelines for Phase 3

- `feat: Add partial fill support to contracts`
- `feat: Implement fill tracking mechanism`
- `feat: Add multi-resolver coordination`
- `feat: Update services for partial fills`
- `test: Add partial fill test suite`
- `fix: Handle [edge case] in partial fills`
- `perf: Optimize gas for small fills`
- `docs: Document partial fill architecture`

## Expected Deliverables

Each chain should update:
```
contracts/
├── [Updated contracts with partial fill support]
├── interfaces/IPartialFillable.sol (if applicable)
└── libraries/PartialFillLib.sol (if applicable)

src/services/chains/[chain]/
├── [Updated services for partial fills]
├── PartialFillCoordinator.ts
└── __tests__/partialFill.test.ts

docs/
└── [chain]_partial_fill_design.md
```

## Success Criteria

1. **Functional**: Multiple resolvers can fill one order
2. **Secure**: No double-spending or price manipulation
3. **Efficient**: Gas costs reasonable for small fills
4. **Reliable**: Handles all edge cases gracefully
5. **Tested**: Comprehensive test coverage

Remember: NO COMPROMISES on security or atomicity!