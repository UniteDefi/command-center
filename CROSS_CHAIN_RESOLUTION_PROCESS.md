# UniteDefi Cross-Chain Resolution Process Documentation

## Table of Contents
1. [Overview](#overview)
2. [Architecture Components](#architecture-components)
3. [Detailed Flow: Auction Creation to Fund Settlement](#detailed-flow-auction-creation-to-fund-settlement)
4. [Smart Contract Architecture](#smart-contract-architecture)
5. [Resolver Service Architecture](#resolver-service-architecture)
6. [HTLC Implementation Details](#htlc-implementation-details)
7. [Security Considerations](#security-considerations)
8. [Edge Cases and Failure Modes](#edge-cases-and-failure-modes)

## Overview

UniteDefi implements a decentralized cross-chain swap protocol using Dutch auctions and Hash Time-Locked Contracts (HTLCs). The system enables trustless asset exchanges across different blockchain networks through a competitive resolver marketplace.

### Core Principles
- **Dutch Auction Pricing**: Prices start high and decrease linearly over time
- **Competitive Resolution**: Multiple resolvers compete to fill orders at optimal prices
- **Atomic Swaps**: HTLC ensures both parties receive assets or both get refunded
- **Decentralized**: No central authority controls the swap process

## Architecture Components

### 1. Smart Contracts Layer

#### SimpleDutchAuction.sol
- Standalone Dutch auction implementation
- Linear price decrease from `startPrice` to `endPrice` over `duration`
- No external dependencies
- Functions:
  - `createAuction()`: Initialize new auction
  - `getCurrentPrice()`: Calculate current price based on elapsed time
  - `settleAuction()`: Accept bid and transfer assets
  - `isActive()`: Check if auction is still open

#### UniteResolver.sol
- Extends 1inch Fusion+ Resolver base
- Integrates Dutch auction with cross-chain escrow system
- Handles multi-chain order settlement
- Key features:
  - Order validation and parsing
  - Cross-chain escrow deployment
  - Secret management for HTLC

#### Escrow Contracts (from 1inch cross-chain-swap)
- **BaseEscrow**: Common HTLC logic
- **EscrowSrc**: Source chain escrow holding seller's tokens
- **EscrowDst**: Destination chain escrow holding buyer's tokens
- **EscrowFactory**: Deterministic escrow deployment using CREATE2

### 2. Off-Chain Services

#### Orchestrator Service
- Manages lifecycle of seller and resolver services
- Coordinates multi-chain monitoring
- Handles service failures and restarts

#### Seller Service
- Creates test auctions periodically
- Simulates real user behavior
- Configurable auction parameters

#### Resolver Services
- **FastResolver**: Minimal delay strategy
- **PatientResolver**: Waits for 50% price drop
- **BalancedResolver**: 500ms delay strategy
- **RandomResolver**: Unpredictable for testing

## Detailed Flow: Auction Creation to Fund Settlement

### Phase 1: Auction Creation

1. **User Initiates Swap**
   ```
   User calls createAuction() with:
   - tokenToSell: Address of token being sold
   - amountToSell: Quantity of tokens
   - startPrice: Initial high price (e.g., 110% of market)
   - endPrice: Final low price (e.g., 90% of market)
   - duration: Time for price decrease (e.g., 300 seconds)
   ```

2. **Auction Deployment**
   ```
   SimpleDutchAuction contract:
   - Validates parameters
   - Transfers tokens from seller (or holds approval)
   - Sets auction start time
   - Emits AuctionCreated event with all parameters
   ```

3. **Event Propagation**
   ```
   Event includes:
   - auctionId: Unique identifier
   - seller: Address of the seller
   - All auction parameters
   - Block timestamp
   ```

### Phase 2: Resolution Competition

1. **Resolver Monitoring**
   ```
   Each resolver service:
   - Monitors AuctionCreated events via WebSocket/RPC
   - Filters for profitable opportunities
   - Tracks current price using getCurrentPrice()
   ```

2. **Price Evaluation**
   ```
   Current Price = startPrice - ((startPrice - endPrice) * timeElapsed / duration)
   
   Resolver checks:
   - Is current price below their max threshold?
   - Do they have sufficient balance?
   - Is expected profit > gas costs?
   ```

3. **Strategy Execution**
   ```
   Based on resolver type:
   - Fast: Immediate execution if profitable
   - Patient: Wait until price drops to 50% of range
   - Balanced: 500ms delay to avoid gas wars
   - Random: Unpredictable delay (0-2000ms)
   ```

4. **Settlement Transaction**
   ```
   Resolver calls settleAuction():
   - Sends exact current price in ETH/tokens
   - Transaction must arrive before another resolver
   - First valid transaction wins
   ```

### Phase 3: Cross-Chain Settlement (for cross-chain orders)

1. **Order Creation with HTLC Parameters**
   ```
   Cross-chain order includes:
   - Standard auction parameters
   - hashlock: keccak256(secret)
   - srcChainId & dstChainId
   - srcEscrowAddress & dstEscrowAddress (pre-computed)
   - Various timeout parameters
   ```

2. **Source Chain Escrow Deployment**
   ```
   When resolver wins auction:
   - EscrowFactory deploys EscrowSrc using CREATE2
   - Seller's tokens locked in escrow
   - Escrow parameters:
     - hashlock for secret verification
     - srcWithdrawal deadline
     - srcCancellation deadline
     - srcSafetyDeposit (0.001 ETH)
   ```

3. **Destination Chain Escrow Deployment**
   ```
   Resolver on destination chain:
   - Deploys EscrowDst with matching hashlock
   - Deposits buyer's tokens
   - Sets complementary timeouts:
     - dstWithdrawal deadline
     - dstCancellation deadline
     - dstSafetyDeposit (0.001 ETH)
   ```

4. **Secret Revelation**
   ```
   User (seller) workflow:
   - Monitors destination chain for escrow deployment
   - Verifies escrow contains correct tokens
   - Reveals secret by calling withdraw() on dst escrow
   - Secret becomes public on-chain
   ```

5. **Completion**
   ```
   Resolver workflow:
   - Monitors for secret revelation
   - Uses revealed secret to withdraw from src escrow
   - Both parties receive their tokens
   - Safety deposits returned
   ```

### Phase 4: Fund Release and Cleanup

1. **Successful Completion**
   ```
   - Seller receives tokens on destination chain
   - Resolver receives tokens on source chain
   - Auction marked as completed
   - Escrows self-destruct (if implemented)
   ```

2. **Timeout Scenarios**
   ```
   If secret not revealed before deadline:
   - Resolver cancels on destination chain
   - Seller cancels on source chain
   - Both get refunds + safety deposits
   ```

## Smart Contract Architecture

### Contract Interactions
```
User → SimpleDutchAuction → AuctionCreated Event
                ↓
         Resolver Services
                ↓
         settleAuction()
                ↓
    (For cross-chain orders)
                ↓
         EscrowFactory
           ↙        ↘
     EscrowSrc    EscrowDst
```

### State Management
- **Auction State**: active/completed/expired
- **Escrow State**: deployed/funded/withdrawn/cancelled
- **Price State**: Calculated dynamically based on time

### Gas Optimizations
- Minimal storage updates
- Efficient price calculation
- Batch operations where possible
- CREATE2 for predictable addresses

## Resolver Service Architecture

### Service Components

1. **Event Monitoring**
   ```typescript
   - WebSocket connections to multiple chains
   - Event filtering and parsing
   - Automatic reconnection on failure
   - Event deduplication
   ```

2. **Price Oracle Integration**
   ```typescript
   - Real-time price feeds
   - Profitability calculations
   - Gas price estimation
   - Slippage protection
   ```

3. **Transaction Management**
   ```typescript
   - Nonce management
   - Gas optimization
   - Transaction retry logic
   - MEV protection strategies
   ```

4. **Balance Management**
   ```typescript
   - Multi-chain balance tracking
   - Automatic rebalancing
   - Reserve maintenance
   - Profit tracking
   ```

## HTLC Implementation Details

### Security Properties

1. **Atomicity**
   - Both swaps complete or both fail
   - No partial execution possible
   - Guaranteed by hashlock mechanism

2. **Time Bounds**
   - Strict timeout ordering: srcWithdrawal > dstWithdrawal > dstCancellation > srcCancellation
   - Prevents indefinite lock of funds
   - Allows recovery in all scenarios

3. **Safety Deposits**
   - 0.001 ETH on each chain
   - Incentivizes proper completion
   - Covers gas for cancellation

### HTLC Flow Diagram
```
Time →

T0: Order created with hashlock = hash(secret)
T1: Resolver fills on source chain → EscrowSrc deployed
T2: Resolver deposits on destination → EscrowDst deployed
T3: User reveals secret on destination chain
T4: Resolver uses secret on source chain
T5: Both parties have their tokens

Timeout scenarios:
- Before T3: User doesn't reveal → Both cancel
- Between T3-T4: Resolver doesn't claim → User gets both sides
```

### Critical Timeout Relationships
```
srcWithdrawal    = longestTimeout      (e.g., T + 24 hours)
dstWithdrawal    = srcWithdrawal - Δ₁  (e.g., T + 23 hours)
dstCancellation  = dstWithdrawal - Δ₂  (e.g., T + 20 hours)
srcCancellation  = dstCancellation - Δ₃ (e.g., T + 16 hours)

Where Δ₁, Δ₂, Δ₃ ensure sufficient time gaps
```

## Security Considerations

### Attack Vectors and Mitigations

1. **Front-running**
   - Mitigation: Commit-reveal schemes
   - Flashloan protection
   - MEV-resistant ordering

2. **Griefing Attacks**
   - Safety deposits discourage abandonment
   - Timeout mechanisms ensure fund recovery
   - Reputation systems for repeat offenders

3. **Price Manipulation**
   - Multiple price oracles
   - Sanity checks on prices
   - Maximum slippage limits

4. **Chain Reorganization**
   - Confirmation requirements
   - Cross-chain verification
   - Fallback mechanisms

### Best Practices

1. **For Users**
   - Verify escrow deployment before revealing secret
   - Monitor both chains during swap
   - Keep sufficient gas on both chains

2. **For Resolvers**
   - Implement robust monitoring
   - Maintain adequate reserves
   - Use secure key management
   - Implement circuit breakers

## Edge Cases and Failure Modes

### Handled Scenarios

1. **Network Congestion**
   - Extended timeout periods
   - Gas price escalation
   - Priority fee mechanisms

2. **Resolver Failure**
   - Multiple competing resolvers
   - Automatic price reduction attracts new resolvers
   - User can always cancel after timeout

3. **Chain Halts**
   - Independent timeout per chain
   - Recovery procedures documented
   - Emergency pause mechanisms

4. **Price Feed Failures**
   - Fallback price sources
   - Manual override capabilities
   - Conservative default prices

### Unhandled Edge Cases

1. **Extreme Gas Prices**
   - May make small swaps uneconomical
   - Requires manual intervention

2. **Smart Contract Bugs**
   - Formal verification recommended
   - Comprehensive audit coverage
   - Bug bounty program

3. **Regulatory Changes**
   - May affect cross-chain operations
   - Requires legal review per jurisdiction

## Future Enhancements

1. **Multi-hop Swaps**
   - A → B → C chain routes
   - Optimized pathfinding
   - Liquidity aggregation

2. **Advanced Auction Types**
   - Batch auctions
   - Sealed bid auctions
   - Dynamic pricing curves

3. **Enhanced Privacy**
   - Zero-knowledge proofs for orders
   - Private resolver networks
   - Stealth addresses

4. **Layer 2 Integration**
   - Optimistic rollup support
   - ZK rollup compatibility
   - State channel settlements

## Conclusion

The UniteDefi cross-chain resolution process provides a robust, decentralized mechanism for trustless asset swaps across blockchain networks. By combining Dutch auctions with HTLC atomic swaps and competitive resolution, the protocol achieves efficient price discovery while maintaining security and decentralization. The modular architecture allows for easy extension to new chains and enhancement with additional features.