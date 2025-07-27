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

I'm implementing UniteDefi cross-chain swaps. Please read these documents first:

- command-center/CROSS_CHAIN_RESOLUTION_PROCESS.md
- command-center/CHAIN_COMPATIBILITY_ANALYSIS.md

Then implement the following:
