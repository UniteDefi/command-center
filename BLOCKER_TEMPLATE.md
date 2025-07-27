# Blocker Documentation Template

When you encounter a blocker during implementation, create a file named `BLOCKERS_[CHAIN_NAME].md` in the resolver directory root with the following format:

---

# [CHAIN_NAME] Implementation Blockers

## Blocker Summary
**Date**: [YYYY-MM-DD HH:MM UTC]  
**Phase**: [1/2/3]  
**Severity**: [Critical/High/Medium]  
**Status**: [Active/Resolved]

## Blocker Details

### Issue Description
[Provide a clear, concise description of what is blocking progress]

### Technical Context
```
[Include relevant code snippets, error messages, or configuration that demonstrates the issue]
```

### What Was Attempted
1. [First approach tried]
   - Result: [What happened]
   - Why it failed: [Technical reason]

2. [Second approach tried]
   - Result: [What happened]
   - Why it failed: [Technical reason]

### Root Cause Analysis
[Explain the fundamental reason why this is blocking progress. Be specific about technical limitations]

### Required Information or Decisions

**Option 1**: [Describe first potential solution]
- Pros: [List advantages]
- Cons: [List disadvantages]
- Implementation effort: [High/Medium/Low]

**Option 2**: [Describe second potential solution]
- Pros: [List advantages]
- Cons: [List disadvantages]
- Implementation effort: [High/Medium/Low]

**Recommended Option**: [Your recommendation and why]

### Questions for the Team

1. [Specific question that needs answering]
2. [Another question if applicable]
3. [Any additional clarifications needed]

### Dependencies
- [List any external dependencies this blocker has]
- [E.g., waiting for testnet faucet, RPC endpoint, documentation]

### Workaround (if any)
[Describe any temporary workaround that could unblock progress, even if not ideal]

### Next Steps
Once this blocker is resolved, the implementation will:
1. [First thing to do after unblocking]
2. [Second step]
3. [Continue with normal flow]

---

## Additional Blockers (if multiple)

[Repeat the above format for each additional blocker]

---

## Resolution Log

### [YYYY-MM-DD HH:MM UTC] - Blocker #1 Resolved
**Resolution**: [How it was resolved]
**Implemented Solution**: [Which option was chosen]
**Commit**: [Commit hash that implements the resolution]

---

# Example Usage

## Example 1: Bitcoin Script Limitation

### Issue Description
Bitcoin Script cannot access external state or make HTTP calls, preventing direct verification of Base Sepolia escrow deployment.

### Technical Context
```bash
# Bitcoin Script is limited to:
OP_CHECKLOCKTIMEVERIFY  # Time locks
OP_HASH160              # Hash verification
# But cannot:
# - Call external APIs
# - Access other chain state
# - Store complex state
```

### What Was Attempted
1. Tried to encode Base Sepolia state in Bitcoin transaction
   - Result: Data size exceeds OP_RETURN limit (80 bytes)
   - Why it failed: Bitcoin has strict data limits

2. Attempted to use Bitcoin full node with custom indexer
   - Result: Would require custom infrastructure
   - Why it failed: Goes against decentralized architecture

### Required Information or Decisions

**Option 1**: Implement coordinator service with proof system
- Pros: Maintains security through cryptographic proofs
- Cons: Requires additional infrastructure
- Implementation effort: High

**Option 2**: Use existing bridge protocols (e.g., tBTC approach)
- Pros: Battle-tested, existing infrastructure
- Cons: May not fit exact use case
- Implementation effort: Medium

**Question**: Are we open to requiring a coordinator service for Bitcoin, or must it be fully trustless?

---

## Example 2: Aptos Move Resource Access

### Issue Description
Cannot directly query Base Sepolia events from within Aptos Move modules, blocking cross-chain verification.

### Technical Context
```move
// Move modules cannot:
module htlc::escrow {
    // ❌ Cannot make HTTP calls
    // ❌ Cannot access external chain state
    // ❌ Cannot import web3 libraries
}
```

### Required Information or Decisions

**Option 1**: Off-chain service monitors both chains and triggers Move functions
- Pros: Clean separation of concerns
- Cons: Requires trusted service
- Implementation effort: Medium

**Option 2**: Use Aptos Oracle framework (if available)
- Pros: More decentralized
- Cons: May not exist yet
- Implementation effort: Unknown

**Question**: Is an off-chain monitoring service acceptable for MVP, or do we need fully on-chain verification?

---

Remember: Be specific, provide evidence, and always suggest potential solutions!