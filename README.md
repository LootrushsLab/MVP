# StrataFi

**Layered DeFi portfolio intelligence on Base.**

StrataFi is a non-custodial dashboard for managing on-chain capital across wallet balances, DeFi positions, and AI-driven yield strategies. Connect a wallet, deploy an ERC-4337 smart account, and use autonomous agents to scan protocols, surface opportunities, and execute user-defined intents — without giving up custody.

> Your capital sits in layers: wallet assets, LP positions, lending markets, and staking. StrataFi unifies them into one intelligent command center.

---

## Table of contents

- [Features](#features)
- [Architecture](#architecture)
- [Tech stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Getting started](#getting-started)
- [Environment variables](#environment-variables)
- [Agent backend](#agent-backend)
- [On-chain contracts](#on-chain-contracts)
- [Project structure](#project-structure)
- [Available scripts](#available-scripts)
- [Mock mode](#mock-mode)
- [Security model](#security-model)
- [Roadmap](#roadmap)
- [License](#license)

---

## Features

### Dashboard

| View | Description |
|------|-------------|
| **Portfolio Overview** | Net worth, allocation charts, performance history, and agent strategy feed |
| **Asset Vault** | Native ETH and ERC-20 balances across EOA + smart account |
| **DeFi Positions** | On-chain discovery and valuation of LP/staked positions (Uniswap V3, Aerodrome, etc.) |
| **Yield Strategies** | AI-recommended upgrades, target APY control, intent registration |
| **Transaction History** | On-chain activity via Basescan API |
| **AI Agent Chat** | Conversational strategist with real-time WebSocket updates |

### Web3 capabilities

- **Wallet connect** — MetaMask, WalletConnect, Coinbase Wallet via [Reown AppKit](https://docs.reown.com)
- **Smart account prediction** — deterministic address from account factory (`getAddress(eoa, 0)`)
- **Session-based agent auth** — 24-hour on-chain sessions via `SessionModule`
- **Intent registry** — register optimization intents on Base Mainnet
- **Multi-source pricing** — Chainlink oracle, CoinGecko, on-chain pool `slot0`

### Supported protocols (Base Mainnet)

Uniswap V3 · Aerodrome (Slipstream CL) · Aave V3 · Compound V3 · Chainlink price feeds

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     StrataFi Dashboard (this repo)           │
│  React · Vite · wagmi · Reown AppKit · TanStack Query        │
└──────────────┬──────────────────────────────┬───────────────┘
               │                              │
               ▼                              ▼
   ┌───────────────────────┐      ┌──────────────────────────┐
   │   Base Mainnet RPC    │      │  StrataFi Agent API      │
   │   (reads + writes)    │      │  (external, :8000)       │
   └───────────────────────┘      └──────────────────────────┘
               │                              │
               ▼                              ▼
   ┌───────────────────────┐      ┌──────────────────────────┐
   │  Smart contracts      │      │  Optimize · Strategies   │
   │  Account Factory      │      │  Discovery · WebSocket   │
   │  Intent Registry      │      │  chat                    │
   │  Session Module       │      └──────────────────────────┘
   └───────────────────────┘
```

This repository is the **frontend only**. Smart contract source lives in `StrataFi-foundry`; the AI agent runs as a separate FastAPI service.

---

## Tech stack

| Category | Tools |
|----------|-------|
| Framework | React 18, TypeScript 5, Vite 5 |
| Styling | Tailwind CSS, shadcn/ui (Radix), Framer Motion |
| Web3 | wagmi 3, viem 2, Reown AppKit |
| Data | TanStack Query, on-chain multicall reads |
| Charts | Recharts |
| Routing | React Router 6 |

---

## Prerequisites

- **Node.js** 18+ (LTS recommended) — [nodejs.org](https://nodejs.org/)
- A **Reown / WalletConnect project ID** for wallet connections
- *(Optional)* StrataFi **agent backend** running locally for AI features
- *(Optional)* **Basescan API key** for enriched transaction history

---

## Getting started

### 1. Clone and install

```powershell
git clone <repository-url>
cd MVP
npm install --ignore-scripts
```

> **Note:** `postinstall` runs Vite and can hang during install. Use `--ignore-scripts` if installation stalls.

### 2. Configure environment

```powershell
copy .env.example .env
```

Edit `.env` and set at minimum `VITE_WALLETCONNECT_PROJECT_ID`.

### 3. Start the dev server

```powershell
npm run dev
```

Open **http://localhost:8080**. The app hot-reloads on file changes.

### 4. *(Optional)* Start the agent backend

For portfolio optimization, yield strategies, and agent chat:

```powershell
# In your StrataFi agent repository
python -m uvicorn main:app --reload --port 8000
```

Ensure `VITE_AGENT_API_URL=http://localhost:8000` in `.env`.

---

## Environment variables

| Variable | Required | Description |
|----------|----------|-------------|
| `VITE_WALLETCONNECT_PROJECT_ID` | Yes* | Reown Cloud project ID — [dashboard.reown.com](https://dashboard.reown.com) |
| `VITE_AGENT_API_URL` | No | Agent API base URL (default: `http://localhost:8000`) |
| `VITE_BASESCAN_API_KEY` | No | Etherscan/Basescan API key for transaction history |

\*Required for mobile wallets and WalletConnect. Desktop MetaMask works without it in many cases.

---

## Agent backend

When connected, the dashboard communicates with the agent API:

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/start-discovery` | POST | Trigger DeFi position scan on wallet connect |
| `/optimize` | POST | Portfolio analysis and upgrade opportunities |
| `/strategies` | GET | Yield strategy recommendations |
| `/history` | GET | Agent activity feed |
| `/chat` | WebSocket | Agent chat and APY target scanning |
| `/sandbox/provision` | POST | Testnet faucet (sandbox mode) |

If the agent is offline, the dashboard still loads — on-chain reads and mock mode continue to work.

---

## On-chain contracts

Deployed on **Base Mainnet** (chain ID `8453`):

| Contract | Address |
|----------|---------|
| Account Factory | `0x52bdc8eb286b279aab769fc3ec433cc8df61ce43` |
| Intent Registry | `0x142f2c70fb19a46b2c71052cf0f5c72210bd737b` |
| Session Module | `0x94044d6cdf49f1278f4a097776f40dd6e5943820` |
| Agent NFT | `0x5eCeC712ab5a1DFfaeab0086f04124574D5913Ec` |

Full addresses and ABIs are in `src/constants/contracts.ts`.

---

## Project structure

```
MVP/
├── public/                  # Static assets
├── src/
│   ├── components/
│   │   ├── dashboard/       # Overview, DeFi, Yield, Chat, etc.
│   │   ├── wallet/          # Landing page, connect modal
│   │   └── ui/              # shadcn/Radix component library
│   ├── contexts/            # Wallet, environment, app state
│   ├── hooks/               # On-chain data, agent API, DeFi discovery
│   ├── constants/           # Contract addresses, ABIs, mock data
│   ├── lib/                 # Gauge discovery utilities
│   ├── pages/               # Route entry points
│   ├── App.tsx              # Root providers and routing
│   ├── wagmi.ts             # Reown AppKit + wagmi config
│   └── main.tsx             # React entry
├── index.html
├── vite.config.ts
├── tailwind.config.ts
└── package.json
```

---

## Available scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start Vite dev server on port 8080 |
| `npm run build` | Production build to `dist/` |
| `npm run build:dev` | Development-mode build |
| `npm run preview` | Preview production build locally |
| `npm run lint` | Run ESLint |

---

## Mock mode

Enable mock mode from the dashboard header to explore the UI with sample portfolio data — no wallet or agent backend required. Preference is stored in `localStorage` under `StrataFi_mock_mode`.

---

## Security model

- **Non-custodial** — StrataFi never stores private keys
- **User-signed transactions** — all on-chain writes require explicit wallet approval
- **Session-scoped agents** — agent operators receive time-limited on-chain sessions only
- **Transparent execution** — all activity settles on Base and is verifiable on Basescan

Use a test wallet with minimal funds when experimenting with session authorization or intent registration.

---

## Roadmap

- [x] Wallet connect + smart-account prediction
- [x] Portfolio overview, DeFi position discovery, yield strategy UI
- [x] Agent optimize/discovery API + Intent Registry integration
- [x] Session module hardening and safer defaults
- [ ] Expanded protocol coverage and live APY / risk sourcing
- [ ] Multi-chain agent operation
- [ ] Production agent backend, monitoring, and contributor documentation

---

## License

MIT — see [LICENSE](LICENSE).
