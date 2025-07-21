# Tokamak Network - Smart Contract Reference

## Overview
This document provides a comprehensive reference for the Tokamak Network smart contracts, specifically focusing on the DAO governance and staking system. The contracts enable decentralized governance through candidate selection and TON token staking.

## Core Concepts

### DAO Governance System
The Tokamak Network uses a Decentralized Autonomous Organization (DAO) structure where:
- Users can stake TON tokens to support candidates
- Candidates compete for committee positions based on staked amounts
- Committee members participate in governance decisions
- Rewards are distributed based on participation and staking

### Key Components
1. **DAOCommittee Contract**: Manages the overall committee structure and candidate registry
2. **Candidate Contracts**: Individual contracts for each candidate containing their staking data
3. **TON Token Contract**: The native governance token used for staking and voting(decimals: 18)
4. **WTON Token Contract**: The Wrapped TON token used for staking and voting(decimals: 27)

### Contract Addresses

- **DAOCommittee Contract** : 0xA2101482b28E3D99ff6ced517bA41EFf4971a386
- **TON Contract** : 0xa30fe40285b8f5c0457dbc3b7c8a280373c40044
- **WTON Contract** : 0x79e0d92670106c85e9067b56b8f674340dca0bbd
- **DepositManager Contract** : 0x90ffcc7F168DceDBEF1Cb6c6eB00cA73F922956F
- Record the Contract address in the .env file according to the bundler you are using, and write the code in a way that references the .env file.

## Smart Contract Interfaces

### DAOCommittee Contract

The DAOCommittee contract serves as the central registry for all candidates and manages committee membership.

```solidity
interface IDAOCommittee {
    struct CandidateInfo {
        address candidateContract;    // Address of the candidate's contract
        uint256 indexMembers;         // Index position in the members array
        uint128 memberJoinedTime;     // Timestamp when candidate became a committee member
        uint128 rewardPeriod;         // Current reward period for the candidate
        uint128 claimedTimestamp;     // Last timestamp when rewards were claimed
    }
    
    // Returns the total number of registered candidates
    function candidatesLength() external view returns (uint256);
    
    // Returns the address of a candidate at a specific index
    function candidates(uint256 index) external view returns (address);
    
    // Returns detailed information about a specific candidate
    function candidateInfos(address candidateAddress) external view returns (CandidateInfo memory);
}
```

**Key Methods:**
- `candidatesLength()`: Get the total count of all registered candidates in the system
- `candidates(index)`: Retrieve a candidate's address by their index in the registry (0-based indexing)
- `candidateInfos(address)`: Get comprehensive information about a specific candidate including their contract address, membership status, and reward tracking data

### Candidate Contract

Candidate is CandidateContract. Each candidate has their own contract that manages staking operations and stores candidate-specific information.

```solidity
interface ICandidate {
    // Returns the candidate's descriptive name or memo
    function memo() external view returns (string memory);
    
    // Returns the total amount of WTON tokens staked for this candidate
    function totalStaked() external view returns (uint256);
    
    // Returns the amount of WTON tokens staked by a specific account for this candidate
    function stakedOf(address account) external view returns (uint256);
}
```

**Key Methods:**
- `memo()`: Retrieve the candidate's name or description (used for identification in UIs)
- `totalStaked()`: Get the aggregate amount of all WTON(decimals: 27) tokens staked for this candidate
- `stakedOf(account)`: Query how many WTON(decimals: 27) tokens a specific user has staked for this candidate

### TON Token Contract

The TON token is the native governance token of the Tokamak Network used for staking and voting.

```solidity
interface ITON {
    // Returns the TON token balance of a specific account
    function balanceOf(address account) external view returns (uint256);
    
    // Approves spending and calls a function on the spender contract in a single transaction
    function approveAndCall(
        address spender, 
        uint256 amount, 
        bytes memory data
    ) external returns (bool);
}
```

**Key Methods:**
- `balanceOf(account)`: Check the TON token balance of any address
- `approveAndCall(spender, amount, data)`: A convenience method that combines ERC20 approval with a contract call. This method:
  - Approves the `spender` to use `amount` of caller's TON(decimals: 18) tokens
  - Calls `onApprove(owner, amount, data)` on the spender contract
  - Enables atomic operations like stake-and-delegate in a single transaction

## Usage Patterns

### Staking Flow
1. User checks their TON(decimals: 18) balance using `ITON.balanceOf(userAddress)`
2. User selects a candidate and checks current stakes using `ICandidate.totalStaked()`
3. User stakes tokens using `ITON.approveAndCall()` targeting the candidate contract
  => spender = WTON, amount = Number of Tokens, data = abi.encode(DEPOSIT_MANAGER, CANDIDATE_CONTRACT)
4. The candidate contract updates the stake record and can be queried via `ICandidate.stakedOf(userAddress)`

### Querying Candidates
1. Get total number of candidates: `IDAOCommittee.candidatesLength()`
2. Iterate through candidates: `IDAOCommittee.candidates(index)` for each index
3. Get candidate details: `IDAOCommittee.candidateInfos(candidateAddress)`
4. Get candidate name: `ICandidate.memo()` on the candidate contract

### Monitoring Stakes
- Total stake for a candidate: `ICandidate.totalStaked()`
- User's stake for a candidate: `ICandidate.stakedOf(userAddress)`
- User's TON balance: `ITON.balanceOf(userAddress)`