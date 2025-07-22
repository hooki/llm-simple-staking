# Frontend Development Guide

## Tech Stack
- **React + TypeScript** (strict mode)
- **Wagmi v2** + **Viem** for blockchain interaction
- **TanStack Query** for data fetching
- **Effect** for functional programming & error handling
- **Tailwind CSS** for styling
- **Vite** + **Biome** for build/linting
- **Bun** package manager

## Project Structure

- `index.html`: Entry point (required for Vite)
- `.env`: Environment variables (must prefix with `VITE_`)
- `vite.config.ts`: Vite config
- `biome.json`: Lint/format config
- `src/main.ts`: App entry point
- `src/config.ts`: Environment manager
- `src/contracts.ts`: Smart contract ABIs
- `src/blockchain.ts`: Ethereum-related logic
- `src/balance.ts`: Token balance utils
- `src/ui.ts`: UI components

## Development Rules

### Core Principles
- **Type-safe**: TypeScript strict mode, `.tsx` for React components
- **Concise**: Minimal, focused implementations
- **Effect-based**: Use Effect for structured error handling
- **Query optimization**: `refetchInterval: 5000` for live data

### Wagmi Integration
- `useReadContract` for view/pure functions
- `useWriteContract` for payable/nonpayable functions
- No BigInt in React Query keys (convert to string)
- Wrap app with `WagmiProvider` + `QueryClientProvider`

### ABI Imports
```ts
import { abi } from '../llm/abis/[ContractName].json'
```
Always extract `.abi` property from JSON files.

## UI Standards

### Formatting
- **Addresses**: `0x1234...5678` (truncated)
- **Tokens**: Max 2 decimals, e.g., `123.45 TON`
- **Balance**: `Balance: 123.45 TOKEN`

### Components
- **WalletConnect**: "Connect Wallet" or truncated address
- **StakingForm**: Input + balance + action buttons
- **CandidateInfo**: Display candidate data and stats

## Blockchain Operations

### Candidate Data Retrieval
1. `candidatesLength()` → get total count
2. `candidates(index)` → get candidate addresses
3. `candidateInfos(address)` → get metadata
4. `memo()` on candidate contract → get name
5. `totalStaked()` → total WTON staked (decimals: 27)
6. `stakedOf(user)` → user's WTON stake (decimals: 27)

### Staking Flow
**Stake TON:**
1. Check `TON.balanceOf(address)`
2. Validate input ≤ balance
3. Execute:
```ts
// data = abi.encode(DEPOSIT_MANAGER_ADDRESS, CANDIDATE_ADDRESS)
TON.approveAndCall(wtonAddress, amount, data)
```

**Unstake TON:**
1. Check `Candidate.stakedOf(address)` (WTON decimals: 27)
2. Validate input ≤ staked amount
3. Execute:
```ts
depositManager.requestWithdrawal(candidateContract, amount)
```

**Withdraw TON:**
1. Get index: `DepositManager.withdrawalRequestIndex(layer2, account)`
2. Get data: `DepositManager.withdrawalRequest(layer2, account, index)`
3. Enable withdraw when:
   - `processed == false`
   - `withdrawableBlockNumber < block.number` (use `useBlockNumber()`)
4. Execute withdrawal transaction

## Configuration Notes
- Environment variables must use `VITE_` prefix
- PostCSS config for Tailwind:
```js
export default {
  plugins: {
    '@tailwindcss/postcss': {},
    autoprefixer: {},
  },
}
```
- Use `type` imports for TypeScript types
- No API key wallet SDKs (avoid WalletConnect with keys)