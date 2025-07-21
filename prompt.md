⏺ Tokamak Network Staking System Summary

  A guide for building a Tokamak DAO governance staking dApp on Sepolia testnet.

  ### Core Features

  - Walelt Connect / Disconnect
  - Users stake TON tokens (18 decimals) to support DAO committee candidates
  - Staked tokens are converted and recorded as WTON (27 decimals)
  - Single transaction staking via approveAndCall
  - Addresses are formatted like `0x1234...5678`
  - Use React, Tailwind CSS, Typescript
  - The token quantity is expressed to four decimal places, with the remainder discarded (e.g., 2.0001)
  - Efficient data fetching with React Query
  - Single candidate query (minimize RPC calls)
  - Write environ variables(contract addresses, rpc url, ...) in .env file and reference it
  - Optimize RPC requests through batching or multicall

  ### Key Contracts

  - DAOCommittee: Manages candidate registry
  - Candidate: Stores staking data per candidate
  - TON/WTON: Governance tokens

  ### Implementation Flow

  1. Query candidate list (candidates(index))
  2. Check candidate info (memo(), totalStaked())
  3. Display candidate list with following columns:
    - Candidate Address
    - Name
    - Contract Address
    - My Staked (user's staked amount)
    - Total Staked (total amount staked for the candidate)
    - Stake button with input field
  4. Implement Stake button, Verify TON balance and stake (approveAndCall)

  Other details (ABIs, addresses, type definitions) can be found in @llm/tokamak-network.md
  RPC_URL=https://sepolia.drpc.org

  ### Precautions
  1. Example of postcss.config.js
  ```javascript
  export default {
    plugins: {
        '@tailwindcss/postcss': {},
        autoprefixer: {},
    },
  }
  ```

  2. Be sure to use the type keyword when importing types