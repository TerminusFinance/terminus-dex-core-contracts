# Terminus Decentralized Exchange
[![TON](https://img.shields.io/badge/based%20on-TON-blue)](https://ton.org/)
[![License](https://img.shields.io/badge/license-GPL--3.0-brightgreen)](https://opensource.org/licenses/GPL-3.0)
The license for Terminus Decentralized Exchange is the GNU General Public License v3.0 (GPL-3.0), see [LICENSE](LICENSE).

Core contracts for the Terminus DEX protocol.

# TON Router Smart Contract

The Router contract facilitates token swaps, liquidity provision, and pool management on the TON blockchain. Below are the supported actions with their opcodes and parameters.

## Actions

### Initialization (`data`)
- **Purpose**: Initializes the router's storage with initial configuration data.
- **Parameters**:
  - `isLocked` (boolean): Lock status of the contract (true = locked, false = unlocked).
  - `adminAddress` (Address): Address of the contract administrator.
  - `LPWalletCode` (Cell): Code of the LP wallet smart contract.
  - `poolCode` (Cell): Code of the pool smart contract.
  - `LPAccountCode` (Cell): Code of the LP account smart contract.
- **Usage**: Used during contract deployment to set up the initial state (`storage`).
- **Result**: Returns a `Cell` containing the serialized initial data.

### Set Fees (`setFees`)
- **Opcode**: `0x355423e5`
- **Purpose**: Updates pool fees (LP, protocol, and referral) and the protocol fee recipient address for a specific token pair.
- **Parameters**:
  - `jetton0Address` (Address): Wallet address of the first token in the pool.
  - `jetton1Address` (Address): Wallet address of the second token in the pool.
  - `newLPFee` (uint8): New LP fee value (8-bit integer).
  - `newProtocolFee` (uint8): New protocol fee value (8-bit integer).
  - `newRefFee` (uint8): New referral fee value (8-bit integer).
  - `newProtocolFeeAddress` (Address): New address to receive protocol fees.
- **Result**: Sends a message to the router to update the fee parameters for the specified pool.
- **Restrictions**: Can only be called by the admin (`admin_address`).

### Initiate Code Upgrade (`initCodeUpgrade`)
- **Opcode**: `0xdf1e233d`
- **Purpose**: Proposes a new version of the router contract code for an upgrade.
- **Parameters**:
  - `newCode` (Cell): The new contract code to be applied.
- **Result**: Initiates the code upgrade process, which must be finalized with `finalizeUpgrades`.
- **Restrictions**: Can only be called by the admin (`admin_address`).

### Initiate Admin Upgrade (`initAdminUpgrade`)
- **Opcode**: `0x2fb94384`
- **Purpose**: Proposes a new admin address for the router contract.
- **Parameters**:
  - `newAdmin` (Address): Address of the new administrator.
- **Result**: Initiates the admin change process, which must be finalized with `finalizeUpgrades`.
- **Restrictions**: Can only be called by the current admin (`admin_address`).

### Cancel Code Upgrade (`cancelCodeUpgrade`)
- **Opcode**: `0x357ccc67`
- **Purpose**: Cancels a previously proposed code upgrade.
- **Parameters**: None.
- **Result**: Resets the code upgrade process, discarding the proposed new code.
- **Restrictions**: Can only be called by the admin (`admin_address`).

### Cancel Admin Upgrade (`cancelAdminUpgrade`)
- **Opcode**: `0xa4ed9981`
- **Purpose**: Cancels a previously proposed admin change.
- **Parameters**: None.
- **Result**: Resets the admin upgrade process, keeping the current admin address.
- **Restrictions**: Can only be called by the current admin (`admin_address`).

### Finalize Upgrades (`finalizeUpgrades`)
- **Opcode**: `0x6378509f`
- **Purpose**: Completes the upgrade process for code and/or admin address changes.
- **Parameters**: None.
- **Result**: Applies the proposed code and/or admin address updates to the contract state.
- **Restrictions**: Can only be called by the admin (`admin_address`) after initiating an upgrade.

### Reset Gas (`resetGas`)
- **Opcode**: `0x42a0fb43`
- **Purpose**: Resets accumulated gas (TON) on the contract, refunding excess to the admin.
- **Parameters**: None.
- **Result**: Sends any excess TON accumulated by the contract to the admin address.
- **Restrictions**: Can only be called by the admin (`admin_address`).

### Lock Contract (`lock`)
- **Opcode**: `0x878f9b0e`
- **Purpose**: Locks the router, preventing swap and liquidity provision operations.
- **Parameters**: None.
- **Result**: Sets the `is_locked` flag in the contract state to `true`.
- **Restrictions**: Can only be called by the admin (`admin_address`).

### Unlock Contract (`unlock`)
- **Opcode**: `0x6ae4b0ef`
- **Purpose**: Unlocks the router, allowing swap and liquidity provision operations.
- **Parameters**: None.
- **Result**: Sets the `is_locked` flag in the contract state to `false`.
- **Restrictions**: Can only be called by the admin (`admin_address`).

### Collect Fees (`collectFees`)
- **Opcode**: `0x1fcb7d3d`
- **Purpose**: Requests the collection of accumulated fees from a specified pool.
- **Parameters**:
  - `jetton0Address` (Address): Wallet address of the first token in the pool.
  - `jetton1Address` (Address): Wallet address of the second token in the pool.
- **Result**: Sends a message to the pool to distribute accumulated fees (e.g., to the protocol and caller).
- **Restrictions**: Can be called by any user, provided the pool supports fee collection.

### Pay To (`payTo`)
- **Opcode**: `0xf93bb43f`
- **Purpose**: Facilitates token payouts from a pool to a user (e.g., after swaps, liquidity withdrawal, or refunds).
- **Parameters**:
  - `owner` (Address): Address of the recipient of the tokens.
  - `tokenAAmount` (BN): Amount of the first token to transfer (in nanotons).
  - `walletTokenAAddress` (Address): Wallet address of the first token.
  - `tokenBAmount` (BN): Amount of the second token to transfer (in nanotons).
  - `walletTokenBAddress` (Address): Wallet address of the second token.
- **Result**: Sends one or two messages to the respective token wallets to transfer the specified amounts to the `owner`.
- **Restrictions**: Can only be called by a valid pool contract.

### Swap Tokens (`swap`)
- **Opcode**: `0x7362d09c` (with payload opcode `0x25938561`)
- **Purpose**: Initiates a token swap operation through a pool.
- **Parameters**:
  - `jettonAmount` (BN): Amount of tokens to swap (in nanotons).
  - `fromAddress` (Address): Address of the sender of the tokens.
  - `walletTokenBAddress` (Address): Wallet address of the target token (to swap into).
  - `toAddress` (Address): Address to receive the swapped tokens.
  - `expectedOutput` (BN): Minimum expected output amount of the target token (in nanotons).
  - `refAddress` (Address, optional): Referral address for referral fees (if applicable).
- **Result**: Forwards the swap request to the appropriate pool, including token amounts and conditions.
- **Restrictions**: Router must be unlocked (`is_locked = false`).

### Provide Liquidity (`provideLiquidity`)
- **Opcode**: `0x7362d09c` (with payload opcode `0xfcf9e58f`)
- **Purpose**: Initiates the addition of liquidity to a pool using one token of the pair.
- **Parameters**:
  - `jettonAmount` (BN): Amount of tokens to provide (in nanotons).
  - `fromAddress` (Address): Address of the sender of the tokens.
  - `walletTokenBAddress` (Address): Wallet address of the second token in the pool pair.
  - `minLPOut` (BN): Minimum expected amount of LP tokens to receive (in nanotons).
- **Result**: Forwards the liquidity provision request to the appropriate pool with specified parameters.
- **Restrictions**: Router must be unlocked (`is_locked = false`).

### Get Pool Address (`getPoolAddress`)
- **Opcode**: `0xd1db969b`
- **Purpose**: Retrieves the address of a pool for a given token pair (getter method).
- **Parameters**:
  - `walletTokenAAddress` (Address): Wallet address of the first token.
  - `walletTokenBAddress` (Address): Wallet address of the second token.
- **Result**: Returns the pool address via an external message or internal processing (handled by `handle_getter_messages`).
- **Restrictions**: Can be called by any user.

## Usage Example
Here’s an example of how to use the `swap` function in TypeScript:

```typescript
import { beginCell, Address, BN } from "ton-core";
import { swap } from "./router";

const swapMsg = swap({
  jettonAmount: new BN(1000000000), // 1 TON in nanotons
  fromAddress: Address.parse("EQ...sender..."),
  walletTokenBAddress: Address.parse("EQ...tokenB..."),
  toAddress: Address.parse("EQ...recipient..."),
  expectedOutput: new BN(500000000), // 0.5 TON in nanotons
  refAddress: Address.parse("EQ...referral..."),
});

// Send `swapMsg` to the router contract via a TON client
