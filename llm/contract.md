

## Contracts

### Addresses
- **DAOCommittee Contract** : 0xA2101482b28E3D99ff6ced517bA41EFf4971a386
- **TON Contract** : 0xa30fe40285b8f5c0457dbc3b7c8a280373c40044
- **WTON Contract** : 0x79e0d92670106c85e9067b56b8f674340dca0bbd
- **DepositManager Contract** : 0x90ffcc7F168DceDBEF1Cb6c6eB00cA73F922956F

### Tokens Decimals

- TON 18
- WTON 27

### TON

The TON interface defines essential interactions with the TON token contract. It includes balance retrieval and a combined approval + execution function (approveAndCall) to enable efficient token operations with other smart contracts.

```solidity
/// @notice Interface for interacting with the TON token contract.
interface TON {
    /// @notice Returns the TON token balance of a specific account.
    /// @param account The address of the account to query.
    /// @return balance The amount of TON tokens held by the given account.
    function balanceOf(address account) external view returns (uint256);

    /// @notice Approves a spender to transfer TON tokens and immediately executes a function on the spender's contract.
    /// @dev This enables atomic interactions such as staking or depositing in a single transaction.
    /// @param spender The address of the contract that will receive the approval and be called.
    /// @param amount The number of tokens to approve for the spender.
    /// @param data Additional call data to pass to the spender's contract (e.g. encoded function selector and parameters).
    /// @return success A boolean indicating whether the operation succeeded.
    function approveAndCall(
        address spender,
        uint256 amount,
        bytes memory data
    ) external returns (bool);
}
```

### DAOCommittee

The DAOCommittee contract is responsible for managing and tracking candidates within a decentralized autonomous organization (DAO). It maintains a registry of candidates, allowing external systems to query candidate addresses, retrieve their detailed metadata.

```solidity
/// @notice This contract manages a list of DAO committee candidates and provides functions to retrieve candidate metadata.
contract DAOCommittee {
    struct CandidateInfo {
        address candidateContract;    // Address of the candidate's contract
        uint256 indexMembers;         // Index position in the members array
        uint128 memberJoinedTime;     // Timestamp when candidate became a committee member
        uint128 rewardPeriod;         // Current reward period for the candidate
        uint128 claimedTimestamp;     // Last timestamp when rewards were claimed
    }

    /// @notice Returns the total number of registered candidates.
    /// @dev Useful for iterating through all candidates using the `candidates` function.
    /// @return length The number of registered candidates.
    function candidatesLength() external view returns (uint256);
    
    /// @notice Returns the address of a candidate at a specific index.
    /// @dev The index must be less than the value returned by `candidatesLength()`.
    /// @param index The index of the candidate in the candidate list.
    /// @return candidateAddress The address of the candidate at the given index.
    function candidates(uint256 index) external view returns (address);
    
    /// @notice Returns the information of a given candidate.
    /// @dev This is a view function that retrieves stored candidate data.
    /// @param candidateAddress The address of the candidate whose information is being queried.
    /// @return candidateInfo A struct containing:
    ///   - candidateContract: The address of the candidate's smart contract.
    ///   - indexMembers: The index of the candidate in the members array.
    ///   - memberJoinedTime: The timestamp when the candidate joined as a committee member.
    ///   - rewardPeriod: The current reward period associated with the candidate.
    ///   - claimedTimestamp: The last time the candidate claimed their reward.
    function candidateInfos(address candidateAddress) external view returns (CandidateInfo memory);
}
```

### Candidate

The Candidate contract provides methods to retrieve the candidate’s descriptive memo, the total amount of WTON tokens staked for the candidate, and the amount staked by individual users.

```solidity
/// @notice Standard contract for candidate contracts used in the DAOCommittee system.
contract Candidate {
    /// @notice Returns a descriptive name or memo associated with the candidate.
    /// @return memo A human-readable string representing the candidate's name or description.
    function memo() external view returns (string memory);
    
    /// @notice Returns the total amount of WTON tokens staked for this candidate by all users.
    /// @return totalStakedAmount The total staked WTON amount.
    function totalStaked() external view returns (uint256);
    
    /// @notice Returns the amount of WTON tokens staked by a specific account for this candidate.
    /// @param account The address of the user whose stake is being queried.
    /// @return stakedAmount The amount of WTON tokens the given account has staked.
    function stakedOf(address account) external view returns (uint256);
}
```

### DepositManager

The DepositManager is an administrative smart contract that allows users to request withdrawals of TON from a specific Layer2.
Withdrawal requests can only be processed after a certain delay, defined by delayBlocks, and the process consists of two steps:

1. `requestWithdrawal` – submits a withdrawal request.
2. `processRequest` – processes the request after the required delay period has passed.

```solidity
/// @notice Handles delayed withdrawal requests for Layer2 systems
/// @dev Users must first request withdrawal, then process it after a delay period
contract DepositManager {

    /// @notice Request a withdrawal from a specific Layer2 instance
    /// @param layer2 The address of the Layer2 contract from which to withdraw
    /// @param amount The amount of WTON to withdraw
    /// @return success True if the request was recorded successfully
    function requestWithdrawal(address layer2, uint256 amount) external returns (bool);

    /// @notice Process a previously requested withdrawal
    /// @param layer2 The address of the Layer2 contract associated with the request
    /// @param receiveTON If true, receive TON; if false, receive in WTON or other form
    /// @return success True if the withdrawal was processed successfully
    function processRequest(address layer2, bool receiveTON) external returns (bool);

    /// @notice Get the delay (in blocks) required before processing a withdrawal
    /// @param layer2 The address of the Layer2 contract
    /// @return delayBlocks The number of blocks to wait before withdrawal is allowed
    function getDelayBlocks(address layer2) external view returns (uint256);

    /// @notice Represents a pending withdrawal request
    /// @dev Stores the block number after which withdrawal can be processed
    struct WithdrawalReqeust {
        uint128 withdrawableBlockNumber; ///< Block number after which withdrawal is allowed
        uint128 amount;                  ///< Amount to withdraw
        bool processed;                  ///< Whether this request has already been processed
    }

    /// @notice Returns the required delay in blocks before a withdrawal can be processed
    /// @param layer2 The address of the Layer2 contract
    /// @return delayBlocks The number of blocks to wait before processing is allowed
    function getDelayBlocks(address layer2) external view returns (uint256);

    /// @notice Returns the current withdrawal request index for a user on a specific Layer2
    /// @param layer2 The Layer2 contract address
    /// @param account The user's address
    /// @return index The current index of withdrawal requests
    function withdrawalRequestIndex(address layer2, address account) external view returns (uint256 index) {
        return _withdrawalRequestIndex[layer2][account];
    }

    /// @notice Retrieves details of a specific withdrawal request
    /// @param layer2 The Layer2 contract address
    /// @param account The user's address
    /// @param index The index of the withdrawal request
    /// @return withdrawableBlockNumber The block number after which withdrawal is allowed
    /// @return amount The requested withdrawal amount
    /// @return processed Whether the request has already been processed
    function withdrawalRequest(address layer2, address account, uint256 index)
        external
        view
        returns (
            uint128 withdrawableBlockNumber,
            uint128 amount,
            bool processed
        )
}
```

### Usage

### Stake TON
1. Obtain the token amount the user wants to stake (e.g. via frontend input)
2. Get the `WTON` token contract address from the application config or registry(ex, .env)
3. Call `TON.approveAndCall()` from the user’s wallet, with:
	- spender: `WTON` token address
	- amount: Number of `TON` tokens to stake
	- data: ABI-encoded payload: abi.encode(`depositManager`, `candidateContract`)
        - depositManager: address of `depositManager` contract
        - candidateContract: address of candidate's `candidateContract`

### Unstake TON
1. Get the token amount to unstake (from user input on frontend)
2. Call `DepositManager.requestWithdrawal(layer2, amount)`

### Withdraw TON
1. Get current withdrawal request by calling `DepositManager.withdrawalRequest(layer2, account)`
2. Call `DepositManager.withdrawalRequest(layer2, account, index)` to get the current withdrawal request
3. A withdrawal request is eligible for processing if:
   - `withdrawalRequest.processed == false`
   - `withdrawalRequest.withdrawableBlockNumber < block.number`
4. If both conditions are met, call `DepositManager.processRequest(layer2, true)`
   - The second parameter `receiveTON` is always set to `true`

### Querying Candidates
1. Get total number of candidates: `IDAOCommittee.candidatesLength()`
2. Iterate through candidates: `IDAOCommittee.candidates(index)` for each index
3. Get candidate details: `IDAOCommittee.candidateInfos(candidateAddress)`
4. Get candidate name: `ICandidate.memo()` on the candidate contract