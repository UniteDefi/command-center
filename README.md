# UniteDefi 🌐

> **Cross-Chain DeFi Swaps Made Simple** - A chain-abstracted wallet and swap platform enabling seamless cross-chain transactions through intent-based architecture.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Built with Next.js](https://img.shields.io/badge/Built%20with-Next.js-black)](https://nextjs.org/)
[![Powered by 1inch](https://img.shields.io/badge/Powered%20by-1inch-blue)](https://1inch.io/)

## 🏆 Hackathon Project

UniteDefi is an 8-day hackathon sprint project that reimagines cross-chain DeFi interactions. We're building a chain-abstracted wallet that makes swapping tokens across different blockchains as simple as a single transaction.

### 🎯 Key Features

- **Chain Abstraction**: Users don't need to worry about which chain their assets are on
- **Intent-Based Architecture**: Express what you want, not how to do it
- **Cross-Chain Swaps**: Seamless token swaps across Ethereum, Base, Arbitrum, and more
- **Dutch Auction Mechanism**: Fair and efficient price discovery for cross-chain trades
- **Solver Network**: Decentralized network of solvers competing to fulfill user intents

## 🏗️ Architecture Overview

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   UniteDefi UI  │────▶│  Intent Engine  │────▶│     Solver      │
│    (Next.js)    │     │   (Resolver)    │     │    Network      │
└─────────────────┘     └─────────────────┘     └─────────────────┘
         │                       │                        │
         │                       │                        │
         ▼                       ▼                        ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   User Wallet   │     │  Smart Contracts│     │   Cross-Chain   │
│   (MetaMask)    │     │   (Solidity)    │     │    Bridges      │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## 🚀 Quick Start

### Prerequisites

- Node.js v18+ and Yarn
- [Foundry](https://book.getfoundry.sh/getting-started/installation) for smart contract development
- MetaMask or compatible Web3 wallet

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/unite-defi.git
   cd unite-defi
   ```

2. **Install dependencies**
   ```bash
   # Install resolver dependencies
   cd resolver
   yarn install
   
   # Install UI dependencies
   cd ../swap-ui
   yarn install
   ```

3. **Set up environment variables**
   ```bash
   # In resolver/
   cp .env.example .env
   
   # In swap-ui/
   cp .env.example .env
   ```

4. **Start development servers**
   ```bash
   # Terminal 1: Start resolver
   cd resolver
   yarn dev
   
   # Terminal 2: Start UI
   cd swap-ui
   yarn dev
   ```

5. **Open the app**
   Navigate to [http://localhost:3000](http://localhost:3000)

## 📁 Project Structure

```
unite-defi/
├── resolver/           # Cross-chain resolver (1inch-based)
│   ├── contracts/     # Smart contracts
│   ├── src/          # TypeScript resolver logic
│   └── tests/        # Contract tests
├── swap-ui/          # Next.js frontend
│   ├── app/          # Next.js app directory
│   ├── components/   # React components
│   └── lib/          # Utilities and hooks
├── sdk/              # TypeScript SDK (in development)
└── docs/             # Documentation
```

## 🛠️ Technology Stack

- **Frontend**: Next.js 15, React 19, TypeScript, Tailwind CSS
- **Web3**: Wagmi, Viem, RainbowKit
- **Smart Contracts**: Solidity, Foundry
- **Resolver**: Node.js, Express, 1inch Fusion SDK
- **Supported Chains**: Ethereum, Base, Arbitrum, Optimism, Polygon

## 📚 Documentation

- [Technical Architecture](docs/ARCHITECTURE.md)
- [API Reference](docs/API.md)
- [Smart Contract Documentation](docs/CONTRACTS.md)
- [Troubleshooting Guide](docs/TROUBLESHOOTING.md)
- [Contributing Guidelines](docs/CONTRIBUTING.md)

## 🧪 Testing

```bash
# Run contract tests
cd resolver
forge test

# Run UI tests
cd swap-ui
yarn test
```

## 🚢 Deployment

See our [deployment guide](docs/DEPLOYMENT.md) for detailed instructions on deploying to production.

## 🤝 Contributing

We welcome contributions! Please see our [contributing guidelines](docs/CONTRIBUTING.md) for details.

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built on top of [1inch Cross-Chain Resolver](https://github.com/1inch/cross-chain-resolver)
- Inspired by the vision of chain abstraction and intent-based architectures
- Thanks to all hackathon mentors and sponsors

## 📞 Contact

- **Team**: UniteDefi Team
- **Email**: team@unitedefi.xyz
- **Discord**: [Join our server](https://discord.gg/unitedefi)

---

**Built with ❤️ during the hackathon**