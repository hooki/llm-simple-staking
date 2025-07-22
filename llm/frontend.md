## Tech Stack

- **Framework**: Effect (functional programming)
- **Blockchain**: Wagmi
- **Package Manager**: Bun
- **Code Quality**: Biome.js
- **Build Tool**: Vite
- **Styling**: Tailwind CSS

---

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

---

## Development Principles

- Code must be **concise** and **type-safe** using **TypeScript**
- Use **Effect** for structured and typed error handling
- Use **environment variables** for configuration (`VITE_` prefix)
- Use `useQuery` to minimize redundant RPC calls
  - For live data, set `refetchInterval = 5000` (5 seconds)
- Use `wagmi` React hooks:
  - `useReadContract` for `view` or `pure` functions
  - `useWriteContract` for `nonpayable` or `payable` functions
- Do **not** use wallet SDKs that require API keys (e.g., WalletConnect)
- Do **not** use `BigInt` in `queryKey` of `useQuery`

### ABI Import Guidelines
- Import contract ABIs from `.llm/abis`
- When importing from JSON files, **always extract the `.abi` property**
  - `import { abi } from './MyContract.json'` (if `abi` exists)

### App Providers
- Wrap the entire app with:
  - `WagmiProvider`
  - `QueryClientProvider`
- All components using Wagmi hooks must be under `WagmiProvider`

### TypeScript Rules
- Use `.tsx` for all React components
- Enable and follow **TypeScript strict mode**

---

## UI/UX Principles

- **Address**: Display as `0x1234...5678`
- **Token Amount**: Show up to 2 decimals; truncate beyond
- **Long Strings**: Truncate for readability
- Use **consistent formatting** across all displays

---

## Components

### 🗳️ Candidate List Retrieval (`wagmi` + `useReadContract`)

1. Call `candidatesLength` to get the total number of candidates.
   - Signature: `function candidatesLength() external view returns (uint256)`
   - Use `useReadContract`
2. Loop from `0` to `candidateLength - 1` and call `candidates(index)` to get each candidate's address.
   - Signature: `function candidates(uint256 index) external view returns (address)`
3. For each candidate address, call `candidateInfos(candidateAddress)` to get metadata.
   - Returns:
     - `candidateContract`: address
     - `indexMembers`: uint256
     - `memberJoinedTime`: uint256
     - `rewardPeriod`: uint256
     - `claimedTimestamp`: uint256
4. Call `memo()` on `candidateContract` to get candidate name
5. Call `totalStaked()` on `candidateContract` to get total staked TON
6. Call `stakedOf(userAddress)` on `candidateContract` to get user's staked TON

### 📥 Stake Input Form

- Fields: Number input, balance display, stake button
- Disable stake if input is invalid or exceeds balance

### 🔌 Wallet Connect Button

- Disconnected: Show “Connect Wallet”
- Connected: Show shortened address (e.g., `0x1234...abcd`)
- On click: Connect wallet or open selector

### Token Balance Display

- Format: `Balance: 123.45 TOKEN`
- Precision: Up to 2 decimal places

---

## Feature Logic

### Wallet Connection

1. Check wallet availability
2. Request account access
3. Ensure connected to Ethereum Mainnet
4. Update UI on connect

### TON Staking

1. User inputs staking amount
2. Check balance via `TON.balanceOf(address)`
3. If input > balance → disable stake button
4. On stake:
```ts
  // spender: WTON contract address
	// amount: number of TON tokens
	// data: abi.encode(DEPOSIT_MANAGER, CANDIDATE_CONTRACT)
  TON.approveAndCall(spender, amount, data)
```

### TON Unstaking

1. User inputs unstaking amount
2. Check staked amount of WTON(stakedAmount) via `Candidate.stakedOf(address)`
3. If input > stakedAmount → disable unstake button
4. On unstake:
```ts
depositManager.requestWithdrawal(candidaetContract, amount);
```

### TON Withdrawal
1. Check the current user's withdrawal request information  
2. If there is a withdrawal request that is eligible for processing, enable the withdraw button
   - `withdrawalRequest.processed == false`
   - `withdrawalRequest.withdrawableBlockNumber < block.number`
   - In frontend `block.number` can be obtained using Wagmi's `useBlockNumber()`
3. When the withdraw button is clicked, send the withdrawal transaction 

## Precautions
- Tailwind CSS setup
```
export default {
  plugins: {
    '@tailwindcss/postcss': {},
    autoprefixer: {},
  },
}
```
- Always use `type` keyword for importing TypeScript types
- Ensure `@tailwindcss/postcss` is installed