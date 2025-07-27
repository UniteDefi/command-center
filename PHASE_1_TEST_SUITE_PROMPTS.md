# Phase 1: Test Suite Implementation Prompts

## General Instructions

- Create a git worktree to avoid conflicts with other sessions
- Work in the `resolver-branches/[chain-name]` directory
- Use the following setup commands:
  ```bash
  # From the project root (NOT from resolver/)
  mkdir -p resolver-branches
  cd resolver
  git worktree add ../resolver-branches/[chain-name] -b [chain-name]-integration
  cd ../resolver-branches/[chain-name]
  ```
- All work happens in your worktree directory
- Commit frequently with descriptive messages for each small feature
- DO NOT mock any implementations or take shortcuts or simulations of the logic
- DO NOT assume functionality that doesn't exist
- If you encounter blockers, document them in `BLOCKERS_[CHAIN_NAME].md` in your worktree
- Test must demonstrate the complete HTLC flow between your chain and Base Sepolia
- For understanding the logic, read `../../command-center/CROSS_CHAIN_RESOLUTION_PROCESS.md`
- For understanding the compatiblity of the chain you are working on, read `../../command-center/CHAIN_COMPATIBILITY_ANALYSIS.md`
- For reference code on how the logic is implemented for EVM <> EVM, check out `../../cross-chain-resolver-example/contracts` and the test suite at `../../cross-chain-resolver-example/tests`

## Cross-Chain Trade Flow Overview

The UniteDefi protocol implements a gasless, user-friendly cross-chain swap mechanism:

1. **User Pre-Approval**: Users pre-approve the escrow factory for spending their tokens (one-time setup)
2. **Gasless Trade Initiation**: User signs a gasless message with trade details and hash(secret)
3. **Relayer Service**: Posts the trade on-chain as a Dutch auction (no escrow created yet)
4. **Resolver Competition**: Resolvers monitor price drops and compete to fill orders
5. **Escrow Creation**: Winning resolver creates escrows with safety deposits on BOTH chains
6. **Asset Settlement**: Relayer moves user funds to escrow, resolver deposits on destination
7. **Secret Reveal**: After confirmations, relayer reveals secret to unlock funds
8. **Completion**: Resolver reads revealed secret and claims funds on source chain

### Detailed Flow Example
User wants to swap 50 USDT (Base Sepolia) → 50 DAI (Arbitrum Sepolia):
1. User signs gasless request with trade details + hash(secret)
2. Relayer posts Dutch auction on-chain
3. Resolver creates escrows with 0.001 ETH safety deposits on both chains
4. Relayer moves user's 50 USDT to source escrow
5. Resolver deposits 50 DAI to destination escrow
6. After confirmations, relayer reveals secret on destination chain
7. Resolver sees secret and withdraws on source chain

## Base Test Flow Requirements

Each test suite must demonstrate:

1. **Setup Phase**:
   - Deploy escrow factory contracts on both chains
   - Deploy Dutch auction contract
   - Set up user token approvals

2. **Trade Initiation**:
   - Simulate relayer posting Dutch auction (without escrow creation)
   - Verify auction parameters and price decay

3. **Resolver Competition**:
   - Simulate resolver monitoring price drops
   - Winner creates escrows with safety deposits on BOTH chains
   - Verify exclusive lock (no other resolver can fill)

4. **Settlement Phase**:
   - Simulate relayer moving user funds to source escrow
   - Resolver deposits agreed tokens to destination escrow
   - Wait for block confirmations

5. **Secret Reveal**:
   - Relayer reveals secret on destination chain
   - Verify resolver can read revealed secret
   - Resolver withdraws on source chain using secret

6. **Edge Cases**:
   - Test resolver timeout (other resolvers can claim safety deposits)
   - Test cancellation flows
   - Test partial confirmations

---

## EVM-Compatible Chains

### Monad Integration

```
You are implementing the UniteDefi cross-chain HTLC test suite for Monad <> Base Sepolia.

Context:
- Monad is fully EVM-compatible
- Use existing Solidity contracts from the resolver directory
- The test should use Foundry (forge test)
- Follow the exact trade flow described above

Your tasks:
1. Set up worktree as instructed above with branch: `monad-integration`
2. Add Monad testnet configuration to foundry.toml
3. Create test file: test/crosschain/MonadBaseHTLC.t.sol
4. Deploy contracts:
   - EscrowFactory on both Monad and Base Sepolia
   - SimpleDutchAuction contract
   - Test tokens (or use existing ones)
5. Implement the complete flow test:
   - User approves EscrowFactory for token spending
   - Simulate relayer posting auction (hash of secret, trade params)
   - Resolver monitors and creates escrows with 0.001 ETH deposits
   - Simulate relayer API moving user funds to escrow
   - Resolver deposits tokens on destination
   - Relayer reveals secret after confirmations
   - Resolver claims funds using revealed secret
6. Test edge cases:
   - Resolver timeout (other resolvers claim safety deposits)
   - Cancellation before secret reveal
   - Insufficient confirmations
7. Mock the relayer service API calls in tests
8. Document RPC endpoints and faucets used

Requirements:
- No mocking of on-chain behavior - use actual testnet deployments
- Mock only the relayer service API interactions
- Verify safety deposit mechanics work correctly
- Test exclusive resolver lock after escrow creation
- Proper event emission verification
- Gas usage tracking
- Clear test names describing each scenario

Critical: The resolver who creates escrows becomes exclusively responsible for completing the trade. Test this lock mechanism thoroughly.

If you encounter any issues with Monad testnet (RPC, faucets, etc.), document in BLOCKERS_MONAD.md
```

### Etherlink Integration

```
You are implementing the UniteDefi cross-chain HTLC test suite for Etherlink <> Base Sepolia.

Context:
- Etherlink is Tezos' EVM-compatible L2
- Use existing Solidity contracts from the resolver directory
- The test should use Foundry (forge test)

Your tasks:
1. Set up worktree as instructed above with branch: `etherlink-integration`
2. Add Etherlink testnet (Ghostnet) configuration to foundry.toml
3. Create test file: test/crosschain/EtherlinkBaseHTLC.t.sol
4. Deploy SimpleDutchAuction and Escrow contracts to both chains
5. Implement full HTLC flow test with proper assertions
6. Test both success and timeout scenarios
7. Use forge script to deploy contracts if needed
8. Document RPC endpoints and faucets used

Requirements:
- No mocking - use actual testnet deployments
- Verify Etherlink's block time behavior
- Test with realistic timeout values
- Include gas cost comparisons

If you encounter any issues with Etherlink testnet, document in BLOCKERS_ETHERLINK.md
```

### Tron Integration

```
You are implementing the UniteDefi cross-chain HTLC test suite for Tron <> Base Sepolia.

Context:
- Tron uses TVM (modified EVM)
- May need slight contract modifications
- Use TronBox or Tron-IDE for deployment
- The test should demonstrate cross-chain HTLC

Your tasks:
1. Set up worktree as instructed above with branch: `tron-integration`
2. Set up Tron testnet (Shasta) configuration
3. Port contracts to be TVM-compatible if needed
4. Create test file: test/crosschain/TronBaseHTLC.test.js
5. Deploy contracts using TronWeb
6. Implement full HTLC flow test
7. Handle Tron's energy/bandwidth model
8. Test both success and timeout scenarios

Requirements:
- Account for Tron's resource model (Energy, Bandwidth)
- Handle TRX vs SUN unit conversions properly
- Verify events using Tron event server
- No shortcuts - implement complete flow

If TVM incompatibilities arise, document in BLOCKERS_TRON.md
```

---

## Move-Based Chains

### Aptos Integration

```
You are implementing the UniteDefi cross-chain HTLC test suite for Aptos <> Base Sepolia.

Context:
- Aptos uses Move language
- You need to port the HTLC logic to Move modules
- Use Aptos CLI and Move Prover for testing
- Must implement the exact trade flow with Aptos adaptations

Your tasks:
1. Set up worktree as instructed above with branch: `aptos-integration`
2. Create Move project structure under test/aptos/
3. Implement Move modules following the trade flow:
   - DutchAuction module (no escrow creation on auction start)
   - EscrowFactory module (handles token approvals)
   - HTLC Escrow module (with safety deposits)
   - CrossChain event emission
4. Adapt the flow for Move:
   - User pre-approves EscrowFactory resource
   - Auction posted without escrow creation
   - Resolver creates escrow resources with safety deposits
   - Implement exclusive resolver lock mechanism
   - Safety deposit in APT (Aptos native token)
5. Write Move tests demonstrating:
   - Token approval to factory
   - Auction creation with hash(secret)
   - Resolver competition and escrow creation
   - Exclusive lock after escrow creation
   - Relayer-initiated fund movement
   - Secret reveal and withdrawal
   - Timeout scenarios with safety deposit claims
6. Create TypeScript integration test:
   - Deploy Move modules to Aptos testnet
   - Deploy Solidity contracts to Base Sepolia
   - Simulate complete cross-chain flow
   - Mock relayer API interactions
   - Monitor events on both chains
7. Handle Move-specific challenges:
   - Resource account for deterministic addresses
   - Coin vs Token standards
   - Event streaming differences

Requirements:
- Implement safety deposit mechanism (0.001 APT)
- Ensure exclusive resolver lock works correctly
- Use SHA3-256 for hash compatibility
- Handle Move abort codes properly
- No mocking of on-chain behavior
- Mock only relayer service calls

Critical: Once a resolver creates escrows on both chains, they have exclusive rights to complete the trade. Other resolvers can only intervene if timeout occurs.

If Move limitations block implementation, document in BLOCKERS_APTOS.md
```

### Sui Integration

```
You are implementing the UniteDefi cross-chain HTLC test suite for Sui <> Base Sepolia.

Context:
- Sui uses object-centric Move variant
- Shared objects for escrow state
- Different from Aptos Move
- Use Sui CLI and Sui TypeScript SDK

Your tasks:
1. Set up worktree as instructed above with branch: `sui-integration`
2. Create Sui project structure under test/sui/
3. Implement Sui Move modules:
   - DutchAuction as shared object
   - HTLC Escrow using shared objects
   - Event emission for cross-chain
4. Write Sui Move tests:
   - Test auction price calculation
   - Test HTLC state transitions
   - Test timeout conditions
5. Create TypeScript integration test:
   - Deploy packages to Sui testnet
   - Deploy Solidity contracts to Base Sepolia
   - Execute cross-chain swap using both SDKs
   - Verify object mutations
6. Use programmable transactions for atomicity

Requirements:
- Leverage Sui's object model properly
- Use Clock object for time-based logic
- Handle gas coins and object ownership
- No shortcuts in state management

If Sui's model creates blockers, document in BLOCKERS_SUI.md
```

---

## Cosmos Ecosystem

### Cosmos Integration

```
You are implementing the UniteDefi cross-chain HTLC test suite for Cosmos <> Base Sepolia.

Context:
- Use CosmWasm for smart contracts
- Rust-based contract development
- Can leverage IBC for future enhancement
- Use wasmd testnet

Your tasks:
1. Set up worktree as instructed above with branch: `cosmos-integration`
2. Create CosmWasm project under test/cosmwasm/
3. Implement Rust contracts:
   - dutch_auction contract
   - htlc_escrow contract
   - Cross-chain event emission
4. Write Rust unit tests:
   - Test auction mechanics
   - Test HTLC with proper hash validation
   - Test timeout logic
5. Create integration test:
   - Deploy to wasmd testnet
   - Deploy Solidity to Base Sepolia
   - Execute cross-chain swap
   - Use CosmJS for interaction
6. Implement instantiate2 for deterministic addresses

Requirements:
- Follow CosmWasm best practices
- Proper error handling with contract errors
- Use native Cosmos SDK features where applicable
- Test both native tokens and CW20

If CosmWasm limitations arise, document in BLOCKERS_COSMOS.md
```

---

## Other Platforms

### Near Integration

```
You are implementing the UniteDefi cross-chain HTLC test suite for Near <> Base Sepolia.

Context:
- Near uses Rust with near-sdk-rs
- Asynchronous contract calls
- Storage staking requirements
- Use Near testnet

Your tasks:
1. Set up worktree as instructed above with branch: `near-integration`
2. Create Near project under test/near/
3. Implement Near contracts:
   - dutch_auction.rs
   - htlc_escrow.rs
   - Handle async cross-contract calls
4. Write Near simulation tests:
   - Test auction flow
   - Test HTLC with callbacks
   - Test storage management
5. Create integration test:
   - Deploy to Near testnet
   - Deploy Solidity to Base Sepolia
   - Handle Near's async nature in cross-chain flow
   - Use near-api-js
6. Implement deterministic account creation

Requirements:
- Handle storage staking properly
- Deal with async callback patterns
- No synchronous assumptions
- Test promise resolution chains

If Near's async model blocks HTLC atomicity, document in BLOCKERS_NEAR.md
```

### Stellar Integration

```
You are implementing the UniteDefi cross-chain HTLC test suite for Stellar <> Base Sepolia.

Context:
- Use Soroban smart contracts (Rust)
- Stellar has native multi-sig and time bounds
- Different asset model than EVM
- Use Stellar testnet

Your tasks:
1. Set up worktree as instructed above with branch: `stellar-integration`
2. Create Soroban project under test/stellar/
3. Implement Soroban contracts:
   - Dutch auction contract
   - HTLC escrow contract
   - Asset bridging logic
4. Write Soroban tests:
   - Test with Stellar assets
   - Test time bounds feature
   - Test multi-sig escrow
5. Create integration test:
   - Deploy to Stellar testnet
   - Deploy Solidity to Base Sepolia
   - Bridge XLM or custom asset
   - Use Stellar SDK
6. Handle Stellar's account model

Requirements:
- Leverage Stellar's native features
- Handle trustlines for custom assets
- Proper sequence number management
- Test operation batching

If Soroban limitations arise, document in BLOCKERS_STELLAR.md
```

### Bitcoin Integration

```
You are implementing the UniteDefi cross-chain HTLC test suite for Bitcoin <> Base Sepolia.

Context:
- Bitcoin has limited scripting (no smart contracts)
- Use Bitcoin Script for HTLC
- Requires creative approach for the trade flow
- Use Bitcoin testnet3

Your tasks:
1. Set up worktree as instructed above with branch: `bitcoin-integration`
2. Create Bitcoin integration under test/bitcoin/
3. Adapt the trade flow for Bitcoin's limitations:
   - No auction contract - track auctions off-chain
   - No token approvals - use UTXO model
   - Safety deposits as separate UTXOs
   - Relayer service coordinates Bitcoin side
4. Implement Bitcoin Script approach:
   - Create HTLC with OP_CHECKLOCKTIMEVERIFY and OP_HASH160
   - Safety deposit UTXO locked to resolver
   - Main swap UTXO with HTLC conditions
   - P2SH addresses for all scripts
5. Modified flow for Bitcoin:
   - User signs PSBT authorizing UTXO spend
   - Relayer posts auction data to Base Sepolia
   - Resolver creates Bitcoin HTLC + safety deposit
   - Resolver creates Base Sepolia escrow
   - Relayer broadcasts user's signed transaction
   - Secret reveal happens on Base Sepolia first
   - Resolver claims Bitcoin using revealed secret
6. Create Node.js test suite:
   - Use bitcoinjs-lib for script creation
   - Implement UTXO tracking
   - Test signature flows
   - Mock relayer coordination
7. Handle Bitcoin-specific challenges:
   - UTXO selection and change handling
   - Fee estimation for different scenarios
   - Transaction malleability concerns
   - Block-based timeouts vs timestamp

Requirements:
- Implement real Bitcoin Script (no shortcuts)
- Safety deposit mechanism using separate UTXO
- Exclusive resolver lock via script conditions
- Proper fee calculation for all transactions
- Test timeout scenarios in blocks
- Mock only relayer API, not Bitcoin behavior

Critical: The resolver who posts the Bitcoin HTLC with safety deposit has exclusive rights. Design scripts to enforce this.

This is complex - if fundamental issues arise, document in BLOCKERS_BITCOIN.md
```

### TON Integration

```
You are implementing the UniteDefi cross-chain HTLC test suite for TON <> Base Sepolia.

Context:
- TON uses FunC/Tact languages
- Asynchronous message passing
- Complex sharding model
- Use TON testnet

Your tasks:
1. Set up worktree as instructed above with branch: `ton-integration`
2. Create TON project under test/ton/
3. Implement in FunC or Tact:
   - Dutch auction contract
   - HTLC escrow contract
   - Message routing logic
4. Write TON tests:
   - Test inter-contract messages
   - Test sharding scenarios
   - Test async state updates
5. Create integration test:
   - Deploy to TON testnet
   - Deploy Solidity to Base Sepolia
   - Handle async message complexity
   - Use TON SDK
6. Implement workchain considerations

Requirements:
- Handle TON's async message model
- Proper gas management
- Test message ordering
- No sync assumptions

If TON's model prevents atomic HTLC, document in BLOCKERS_TON.md
```

### ICP Integration

```
You are implementing the UniteDefi cross-chain HTLC test suite for ICP <> Base Sepolia.

Context:
- ICP uses canisters (Rust/Motoko)
- Async inter-canister calls
- Unique capabilities (HTTP outcalls, threshold sigs)
- Use ICP testnet

Your tasks:
1. Set up worktree as instructed above with branch: `icp-integration`
2. Create ICP project under test/icp/
3. Implement canisters in Rust:
   - Dutch auction canister
   - HTLC escrow canister
   - Cross-chain monitoring canister
4. Write canister tests:
   - Test update calls
   - Test query calls
   - Test inter-canister calls
5. Create integration test:
   - Deploy to ICP testnet
   - Deploy Solidity to Base Sepolia
   - Use HTTP outcalls for monitoring
   - Demonstrate cross-chain flow
6. Leverage ICP's unique features

Requirements:
- Handle cycles management
- Proper async/await patterns
- Test upgrade scenarios
- Use certified variables

If ICP's model creates issues, document in BLOCKERS_ICP.md
```

---

## Testing Standards

All test suites must include:

1. **Setup Documentation**: Clear instructions for running tests
2. **Network Configuration**: RPC endpoints, chain IDs, faucets
3. **Gas Analysis**: Track and report gas costs
4. **Event Verification**: Confirm all events are emitted correctly
5. **Error Scenarios**: Test all failure modes
6. **Timing Analysis**: Measure block times and timeout safety

## Commit Guidelines

Make frequent, atomic commits:

- `feat: Add [chain] testnet configuration`
- `feat: Implement Dutch auction contract for [chain]`
- `feat: Add HTLC escrow logic`
- `feat: Create cross-chain test setup`
- `test: Add success scenario test`
- `test: Add timeout scenario test`
- `fix: Handle [specific issue]`
- `docs: Add setup instructions`

## Output Structure

Each chain integration should produce:

```
test/
├── crosschain/
│   └── [Chain]BaseHTLC.t.sol (or appropriate extension)
├── [chain]/
│   ├── contracts/ (if non-EVM)
│   ├── scripts/
│   └── README.md
└── reports/
    └── [chain]_gas_analysis.md
```
