# UniteDefi Integration Prompt Usage Guide

This guide explains how to use the three-phase prompts to implement UniteDefi for a new blockchain.

## Overview

The integration process has three phases:
1. **Phase 1**: Test Suite Implementation - Build the core HTLC logic and tests
2. **Phase 2**: Node.js Services - Convert tests into production services (Relayer + Resolver)
3. **Phase 3**: Partial Filling - Add support for multiple resolvers filling one order

## Before You Start

### Required Reading
Each Claude session should have access to:
- `CROSS_CHAIN_RESOLUTION_PROCESS.md` - Explains the protocol flow
- `CHAIN_COMPATIBILITY_ANALYSIS.md` - Chain-specific considerations
- `BLOCKER_TEMPLATE.md` - How to document blockers

### Git Worktree Setup
Each chain uses a separate git worktree to avoid conflicts:
```bash
# From project root
mkdir -p resolver-branches
cd resolver
git worktree add ../resolver-branches/[chain-name] -b [chain-name]-integration
```

## Phase 1: Test Suite Implementation

### When to Use
- Starting a new chain integration from scratch
- Building the foundational HTLC logic
- Creating test suites that demonstrate the complete flow

### How to Start a Fresh Claude Session

1. **Open the Phase 1 prompt file**: `PHASE_1_TEST_SUITE_PROMPTS.md`
2. **Find your chain's section** (e.g., "Monad Integration", "Aptos Integration")
3. **Copy the ENTIRE prompt** for your chain, including:
   - The context section
   - All numbered tasks
   - Requirements
   - The closing line about blockers

4. **Start your prompt with**:
```
I'm implementing UniteDefi cross-chain swaps. Please read these documents first:
- command-center/CROSS_CHAIN_RESOLUTION_PROCESS.md
- command-center/CHAIN_COMPATIBILITY_ANALYSIS.md

Then implement the following:

[PASTE THE ENTIRE CHAIN-SPECIFIC PROMPT FROM PHASE_1_TEST_SUITE_PROMPTS.md]
```

### What Phase 1 Delivers
- Smart contracts (or equivalent) for your chain
- Test suite demonstrating the complete HTLC flow
- Integration with Base Sepolia
- Documentation of any blockers

## Phase 2: Node.js Services Implementation

### When to Use
- After completing Phase 1 successfully
- Ready to build production services
- Converting test logic into real services

### How to Start Phase 2

1. **Continue in the same worktree** from Phase 1
2. **Open**: `PHASE_2_SERVICES_PROMPTS.md`
3. **Find your chain's section**
4. **Start your prompt with**:
```
I've completed Phase 1 of the UniteDefi implementation for [CHAIN].
The test suite is working and demonstrates the HTLC flow.

Now I need to implement Phase 2 - the production services:

[PASTE THE ENTIRE CHAIN-SPECIFIC PROMPT FROM PHASE_2_SERVICES_PROMPTS.md]
```

### What Phase 2 Delivers
- Relayer Service (Express API) with all endpoints
- Resolver Service for your chain
- Integration with the orchestrator
- Production-ready error handling

## Phase 3: Partial Filling Implementation

### When to Use
- After completing Phase 2 successfully
- Services are working for single fills
- Ready to add multi-resolver support

### How to Start Phase 3

1. **Continue in the same worktree**
2. **Open**: `PHASE_3_PARTIAL_FILLING_PROMPTS.md`
3. **Find your chain's section**
4. **Start your prompt with**:
```
I've completed Phase 1 and 2 of the UniteDefi implementation for [CHAIN].
The services are working for full order fills.

Now I need to implement Phase 3 - partial filling support:

[PASTE THE ENTIRE CHAIN-SPECIFIC PROMPT FROM PHASE_3_PARTIAL_FILLING_PROMPTS.md]
```

### What Phase 3 Delivers
- Updated contracts supporting partial fills
- Multi-escrow coordination
- Proportional safety deposits
- Updated services for partial fill logic

## Important Notes

### For Fresh Claude Sessions
Always provide context about:
1. Which phase you're working on
2. What's already been completed
3. The chain you're implementing
4. Any specific blockers from previous work

### For Resuming Work
If resuming in a new Claude session:
```
I'm continuing the UniteDefi [CHAIN] implementation.
I'm currently on Phase [N] and have completed [describe what's done].
My worktree is at: resolver-branches/[chain-name]

[Continue with specific questions or next steps]
```

### Handling Blockers
If you encounter blockers:
1. Create `BLOCKERS_[CHAIN_NAME].md` using the template
2. Document the specific issue
3. Stop and ask for clarification
4. Do NOT mock or simulate the problematic functionality

## Quick Reference

### File Structure After All Phases
```
resolver-branches/[chain-name]/
├── contracts/           # Smart contracts
├── test/               # Test suites
├── src/
│   ├── services/
│   │   ├── relayer/    # Relayer API service
│   │   ├── resolver/   # Resolver service
│   │   └── chains/[chain]/  # Chain-specific code
├── BLOCKERS_[CHAIN].md # If any blockers
└── README.md           # Setup instructions
```

### Commit Pattern
- Phase 1: `feat: Add [chain] testnet configuration`
- Phase 2: `feat: Implement [chain] relayer service`  
- Phase 3: `feat: Add partial fill support to [chain]`

## Best Practices

1. **Never skip phases** - Each builds on the previous
2. **Test thoroughly** - Use real testnets, no mocking
3. **Document everything** - Especially RPC endpoints and faucets
4. **Commit frequently** - Small, atomic commits
5. **Ask when stuck** - Create blocker documentation

## Troubleshooting

### "I don't see my chain in the prompts"
- Check if it's listed under a category (EVM, Move, Cosmos, etc.)
- Some chains may need custom adaptation

### "The worktree command fails"
- Ensure you're in the resolver directory
- Check if the branch already exists
- Try: `git worktree list` to see existing worktrees

### "I'm getting merge conflicts"
- You're working in the wrong directory
- Always work in `resolver-branches/[chain-name]`
- Never work directly in the `resolver/` directory