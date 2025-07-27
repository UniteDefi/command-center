# Phase 1 Adaptation Prompt for Existing Implementations

Use this prompt if you've already started Phase 1 with the old flow and need to adapt to the new relayer-resolver architecture.

## For Monad and Etherlink (or any chain with existing Phase 1 work)

```
You've already implemented Phase 1 for [CHAIN_NAME] using an older version of the UniteDefi flow.
My current implementation needs to be adapted to the new architecture.

Current worktree location: resolver-branches/[chain-name]

The NEW architecture has these key differences:
1. Users pre-approve tokens to EscrowFactory (gasless after that)
2. Relayer service posts auctions WITHOUT creating escrows
3. Resolvers create escrows with safety deposits on BOTH chains
4. The resolver who creates escrows has exclusive rights to complete
5. Relayer moves user funds only AFTER resolver commits
6. Relayer reveals secret after confirmations

Please help me adapt my existing implementation:

1. Review my current contracts/tests and identify what needs changing
2. Update the auction contract to NOT create escrows on auction creation
3. Ensure escrow creation happens separately by resolvers
4. Add safety deposit mechanism (0.001 ETH) to escrows
5. Implement exclusive resolver lock after escrow creation
6. Update tests to reflect the new flow:
   - User approval to factory
   - Relayer posting auction
   - Resolver creating escrows with deposits
   - Relayer moving funds
   - Secret reveal sequence
7. Ensure the test demonstrates the complete new flow

Key files to check and update:
- SimpleDutchAuction.sol (should not create escrows)
- EscrowFactory.sol (should handle user approvals)
- Test files (should follow new flow)

Do NOT delete existing work - adapt it to the new flow.
Make commits for each significant change with clear messages.
```

## How to Use This Adaptation Prompt

1. **Navigate to your existing worktree**:

   ```bash
   cd resolver-branches/monad  # or etherlink
   ```

2. **Start a new Claude session with**:

   ```
   I need to adapt my existing UniteDefi implementation. Please read:
   - command-center/CROSS_CHAIN_RESOLUTION_PROCESS.md (for new flow)
   - command-center/CHAIN_COMPATIBILITY_ANALYSIS.md

   Then help me adapt using this prompt:

   [PASTE THE ADAPTATION PROMPT ABOVE]
   ```

3. **What to Expect**:
   - Claude will review your existing code
   - Identify specific changes needed
   - Help you refactor without losing work
   - Update tests to match new flow
   - Ensure safety deposits work correctly

## Key Changes to Look For

### Before (Old Flow)

- Auction creation might create escrows
- Direct resolver-to-user interaction
- No safety deposits
- No exclusive locks

### After (New Flow)

- Auction creation is just data posting
- Relayer coordinates everything
- Safety deposits required (0.001 ETH)
- Exclusive resolver locks
- Gasless user experience

## Verification Checklist

After adaptation, verify:

- [ ] Users only approve tokens (no other on-chain actions)
- [ ] Auctions created without escrows
- [ ] Resolvers create escrows with safety deposits
- [ ] Only the first resolver can complete the trade
- [ ] Relayer moves funds after resolver commits
- [ ] Secret reveal happens through relayer
- [ ] Tests demonstrate the complete new flow

## Common Issues During Adaptation

1. **Escrow creation in auction contract**

   - Remove any escrow creation from auction logic
   - Move it to a separate resolver action

2. **Missing safety deposits**

   - Add payable to escrow creation
   - Require 0.001 ETH deposit
   - Return deposit on successful completion

3. **No exclusive lock mechanism**

   - Add mapping to track which resolver owns an auction
   - Prevent other resolvers from filling after escrows created

4. **Direct user fund transfers**
   - Change to relayer-initiated transfers
   - User funds move only after resolver commits
