# UniteDefi - Getting Started Guide

## Pre-Sprint Setup (30 minutes)

### 1. GitHub Organization Setup

```bash
# Create GitHub organization: unite-defi
# Create 5 repositories:
# - unite-defi-protocol
# - unite-defi-webapp
# - unite-defi-extension
# - unite-defi-sdk
# - unite-defi-docs
```

### 2. NPM Organization Setup

```bash
# Login to npm
npm login

# Verify @unite-defi scope is available
npm org ls unite-defi
```

### 3. Development Environment

```bash
# Required tools
node --version  # v18+
npm --version   # v9+
git --version   # v2.30+

# Install global tools
npm install -g typescript ts-node nodemon

# Chrome for extension testing
# Metamask extension for Web3 testing
```

### 4. API Keys & Environment Variables

Create `.env` files for each project:

```bash
# Protocol service
ETHEREUM_RPC_URL=https://eth.llamarpc.com
POLYGON_RPC_URL=https://polygon.llamarpc.com
BSC_RPC_URL=https://bsc.llamarpc.com
ARBITRUM_RPC_URL=https://arb1.arbitrum.io/rpc

# Optional: Use Alchemy/Infura for better reliability
ALCHEMY_API_KEY=your_key_here
INFURA_PROJECT_ID=your_id_here

# Database (use local PostgreSQL for hackathon)
DATABASE_URL=postgresql://localhost:5432/unitedefi

# Security
JWT_SECRET=your-super-secret-key
ENCRYPTION_KEY=32-byte-hex-key
```

---

## Phase 1: Foundation Testing (End of Day 2)

### What Should Work:

1. **Protocol Service (`unite-defi-protocol`)**:

   - ✅ Dutch auction price calculation
   - ✅ Basic resolver can participate in auctions
   - ✅ REST API endpoints respond correctly
   - ✅ WebSocket broadcasts auction updates
   - ✅ All 4 EVM chains configured

2. **Web Application (`unite-defi-webapp`)**:
   - ✅ Wallet connection works for all 4 chains
   - ✅ Token selector shows tokens with balances
   - ✅ Swap interface calculates prices correctly
   - ✅ Dutch auction countdown displays properly
   - ✅ Mobile responsive design

### Testing Commands:

```bash
# Terminal 1: Start protocol service
cd unite-defi-protocol
npm run dev

# Terminal 2: Start web app
cd unite-defi-webapp
npm run dev

# Terminal 3: Test API endpoints
cd unite-defi-protocol
npm run test:integration

# Test specific auction functionality
curl -X POST http://localhost:3001/api/quote \
  -H "Content-Type: application/json" \
  -d '{
    "fromChain": "ethereum",
    "toChain": "polygon",
    "fromToken": "USDC",
    "toToken": "USDC",
    "amount": "1000"
  }'
```

### Manual Testing Checklist:

- [ ] Connect MetaMask to web app on all 4 chains
- [ ] Select different tokens and see balances update
- [ ] Start a small test swap and watch auction countdown
- [ ] Switch chains and verify UI updates correctly
- [ ] Test on mobile device or DevTools mobile view

### Common Issues & Fixes:

#### **Issue: "Cannot connect to wallet"**

```bash
# Check if MetaMask is installed and unlocked
# Verify network configurations in Web3 provider
# Check browser console for errors

# Debug Web3 connection:
console.log(window.ethereum?.selectedAddress)
console.log(window.ethereum?.chainId)
```

#### **Issue: "API endpoints returning 500 errors"**

```bash
# Check if protocol service is running
curl http://localhost:3001/health

# Check database connection
npm run db:test

# View server logs
tail -f logs/server.log
```

#### **Issue: "Dutch auction not updating prices"**

```bash
# Check WebSocket connection
# Open browser DevTools -> Network -> WS tab
# Should see constant price updates

# Test auction logic directly
cd unite-defi-protocol
npm run test:auction
```

#### **Issue: "Token balances not loading"**

```bash
# Check RPC endpoints are responding
curl -X POST https://eth.llamarpc.com \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

# Test token contract calls
cd unite-defi-protocol
node scripts/test-token-balances.js
```

---

## Phase 2: Non-EVM Chains Testing (End of Day 4)

### What Should Work:

1. **All 8 Chains Integrated**:

   - ✅ Ethereum, Polygon, BSC, Arbitrum (Phase 1)
   - ✅ Sui, Stellar, Aptos, TON (Phase 2)
   - ✅ Cross-chain swaps between any pair
   - ✅ Chain-specific wallet connections

2. **Advanced Features**:
   - ✅ Multi-chain portfolio view showing USD values
   - ✅ Cross-protocol secret management
   - ✅ Partial fills working correctly

### Testing Commands:

```bash
# Test Sui integration
cd unite-defi-protocol
node scripts/test-sui-connection.js

# Test Stellar integration
node scripts/test-stellar-connection.js

# Test cross-chain swap (Ethereum to Sui)
curl -X POST http://localhost:3001/api/swap \
  -H "Content-Type: application/json" \
  -d '{
    "fromChain": "ethereum",
    "toChain": "sui",
    "fromToken": "USDC",
    "toToken": "USDC",
    "amount": "100",
    "userAddress": "0x..."
  }'
```

### Manual Testing Checklist:

- [ ] Connect to Sui wallet (Sui Wallet browser extension)
- [ ] Connect to Stellar wallet (Freighter extension)
- [ ] Connect to Aptos wallet (Petra extension)
- [ ] Connect to TON wallet (Tonkeeper)
- [ ] Execute test swaps between EVM and non-EVM chains
- [ ] Verify transaction status updates work across all chains
- [ ] Test partial fills with large orders

### Common Issues & Fixes:

#### **Issue: "Sui wallet connection fails"**

```bash
# Install Sui Wallet extension
# Check Sui devnet/testnet configuration
# Verify Sui SDK installation:
npm list @mysten/sui.js
```

#### **Issue: "Cross-chain swap stuck in pending"**

```bash
# Check both chain's transaction status
# Verify secret sharing between escrows
# Check resolver logs:
tail -f logs/resolver.log

# Debug HTLC contract state:
cd unite-defi-protocol
node scripts/debug-htlc-state.js --txId=abc123
```

#### **Issue: "Non-EVM chains showing wrong balances"**

```bash
# Test individual chain connections
node scripts/test-chain-balance.js --chain=sui --address=0x123

# Check API response format differences:
curl http://localhost:3001/api/balance/sui/0x123
curl http://localhost:3001/api/balance/ethereum/0x456
```

---

## Phase 3: Extension Testing (End of Day 6)

### What Should Work:

1. **Chrome Extension**:
   - ✅ Installs and loads without errors
   - ✅ Shows aggregated USD balance across all chains
   - ✅ Auto-detects when user needs different chain
   - ✅ Suggests auto-bridging for insufficient funds
   - ✅ Mini swap interface works within popup

### Testing Commands:

```bash
# Build extension for testing
cd unite-defi-extension
npm run build

# Load unpacked extension in Chrome:
# 1. Open chrome://extensions/
# 2. Enable "Developer mode"
# 3. Click "Load unpacked"
# 4. Select unite-defi-extension/dist folder
```

### Manual Testing Checklist:

- [ ] Extension icon appears in Chrome toolbar
- [ ] Popup opens and shows wallet interface
- [ ] Multi-chain balance aggregation works
- [ ] Can execute swaps directly from extension
- [ ] Auto-bridging logic triggers correctly
- [ ] Notifications work for transaction updates
- [ ] Extension works on different dApp websites

### Common Issues & Fixes:

#### **Issue: "Extension fails to load"**

```bash
# Check manifest.json syntax
cat unite-defi-extension/dist/manifest.json | jq .

# Check Chrome extension console:
# Right-click extension icon -> Inspect popup
# Check for JavaScript errors

# Verify build process:
cd unite-defi-extension
npm run build
ls -la dist/
```

#### **Issue: "Cannot connect to protocol service from extension"**

```bash
# Check CORS settings in protocol service
# Verify Content Security Policy in manifest.json:
"content_security_policy": {
  "extension_pages": "script-src 'self'; object-src 'self'"
}

# Test API calls from extension:
# Open extension popup -> DevTools -> Network tab
```

#### **Issue: "Auto-bridging not triggering"**

```bash
# Check balance detection logic
# Verify gas estimation for target chain
# Test with small amounts first

# Debug auto-bridging logic:
cd unite-defi-extension
node debug/test-auto-bridge.js
```

---

## Phase 4: Final Polish Testing (End of Day 8)

### What Should Work:

1. **Complete System**:
   - ✅ All 8 chains working seamlessly
   - ✅ Advanced dutch auction features
   - ✅ Professional UI with animations
   - ✅ Extension provides magical UX
   - ✅ SDK allows easy integration
   - ✅ Comprehensive documentation

### Performance Testing:

```bash
# Load testing for protocol service
cd unite-defi-protocol
npm install -g autocannon
autocannon -c 10 -d 30 http://localhost:3001/api/quote

# Frontend performance testing
cd unite-defi-webapp
npm run lighthouse

# Extension performance testing
# Monitor memory usage in chrome://extensions/
```

### Security Testing:

```bash
# Run security audit
npm audit --audit-level high

# Check for common vulnerabilities
cd unite-defi-protocol
npm install -g eslint-plugin-security
eslint --ext .ts src/

# Test HTLC contract security
cd unite-defi-protocol/contracts
slither . --solc-remaps @openzeppelin/=node_modules/@openzeppelin/
```

### Demo Preparation Checklist:

- [ ] All services start up cleanly
- [ ] Demo scenarios work reliably
- [ ] Backup demo environment ready
- [ ] Video recording setup tested
- [ ] Presentation slides prepared
- [ ] GitHub repositories are public and well-documented
- [ ] NPM packages published to @unite-defi scope

---

## Emergency Debugging Guide

### Service Won't Start:

```bash
# Check port conflicts
lsof -i :3000
lsof -i :3001

# Check logs for startup errors
tail -f ~/.pm2/logs/unite-defi-protocol-error.log

# Kill and restart everything
pkill -f "node.*unite-defi"
pm2 delete all
pm2 start ecosystem.config.js
```

### Database Issues:

```bash
# Reset database
cd unite-defi-protocol
npm run db:reset
npm run db:migrate
npm run db:seed

# Check database connection
psql -h localhost -U postgres -d unitedefi -c "SELECT NOW();"
```

### Blockchain Connection Issues:

```bash
# Test all RPC endpoints
cd unite-defi-protocol
npm run test:rpc-health

# Switch to backup RPC providers
# Edit .env file with alternative providers
```

### Extension Not Loading:

```bash
# Clear extension data
# Go to chrome://extensions/
# Click "Remove" on UniteDefi extension
# Clear browser cache: Ctrl+Shift+Delete
# Rebuild and reload extension

cd unite-defi-extension
rm -rf dist/
npm run build
# Load unpacked extension again
```

### Performance Issues:

```bash
# Check system resources
top
df -h
free -m

# Optimize database queries
cd unite-defi-protocol
npm run analyze:queries

# Enable query caching
# Add Redis for caching layer
```

---

## Success Metrics Per Phase

### Phase 1 Success:

- [ ] 4 EVM chains working with basic dutch auction
- [ ] Web app can connect wallets and show balances
- [ ] API responds in <200ms
- [ ] 40+ meaningful commits across repositories

### Phase 2 Success:

- [ ] All 8 chains integrated and working
- [ ] Cross-chain swaps execute successfully
- [ ] Portfolio view shows accurate USD values
- [ ] 80+ total commits

### Phase 3 Success:

- [ ] Extension installs and works smoothly
- [ ] Chain abstraction provides magical UX
- [ ] Auto-bridging logic works correctly
- [ ] 120+ total commits

### Phase 4 Success:

- [ ] Production-ready demo quality
- [ ] All advanced features working
- [ ] Comprehensive documentation
- [ ] 200+ total commits
- [ ] Ready for hackathon submission

## Final Deployment Checklist

### Before Demo:

- [ ] All services deployed to production URLs
- [ ] Extension submitted to Chrome Web Store (unlisted)
- [ ] NPM packages published
- [ ] GitHub repositories clean and documented
- [ ] Demo video recorded
- [ ] Backup demo environment ready

### Demo Day:

- [ ] Test complete flow 30 minutes before presentation
- [ ] Have backup laptop ready
- [ ] Mobile hotspot for internet backup
- [ ] Local demo environment as fallback
- [ ] Team roles assigned for presentation

**Remember: Small commits every 30-45 minutes throughout the entire sprint!**
