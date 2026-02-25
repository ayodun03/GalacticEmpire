# GalacticEmpire (GLXE) - Starfleet Credits Token Contract

A Clarity smart contract implementing a fungible token with time-based mechanics using a solar rotation system. Built for the Stacks blockchain.

## Overview

GalacticEmpire is a fungible token contract themed around space exploration and empire management. It features a unique "solar rotation" time-tracking mechanism that governs token transfers, staking (cryopod storage), and governance (stellar conquests).

**Token Details:**
- **Name:** GalacticEmpire
- **Symbol:** GLXE
- **Max Supply:** 77,000,000,000 tokens

## Core Concepts

### Solar Rotation System

The contract uses a custom time-tracking mechanism called "solar rotations" instead of block heights. Each rotation represents a unit of time that must be manually advanced using the `advance-solar-rotation` function.

### Key Features

1. **Hyperspace Transfers** - Token transfers with cooldown periods
2. **Cryopod Storage** - Time-locked token staking mechanism
3. **Stellar Conquests** - Governance proposals with time-based voting periods

## Functions

### Time Management

#### `advance-solar-rotation`
```clarity
(advance-solar-rotation)
```
Increments the current solar rotation by 1. Anyone can call this function.

**Returns:** The new solar rotation count

#### `get-solar-rotation`
```clarity
(get-solar-rotation)
```
Returns the current solar rotation count (read-only).

---

### Token Transfers

#### `hyperspace-transfer`
```clarity
(hyperspace-transfer (amount uint) (destination principal))
```
Transfer tokens with a warp cooldown mechanism.

**Parameters:**
- `amount` - Number of tokens to transfer
- `destination` - Recipient's principal address

**Constraints:**
- Minimum 10 solar rotations must pass between transfers from the same sender
- Standard token transfer rules apply

**Error Codes:**
- `u102` - Hyperdrive cooldown not met (less than 10 rotations since last transfer)

---

### Cryopod Storage (Staking)

#### `enter-cryopod-storage`
```clarity
(enter-cryopod-storage (credit-amount uint) (hibernate-duration uint))
```
Lock tokens for a specified number of solar rotations.

**Parameters:**
- `credit-amount` - Number of tokens to lock
- `hibernate-duration` - Number of solar rotations to lock tokens

**How it works:**
1. Tokens are transferred to the contract
2. An awaken rotation is calculated: `current-rotation + hibernate-duration`
3. Tokens remain locked until the awaken rotation is reached

#### `awaken-from-cryopod`
```clarity
(awaken-from-cryopod)
```
Withdraw tokens from cryopod storage after the hibernate period ends.

**Constraints:**
- Current solar rotation must be >= awaken rotation
- Only the original depositor can withdraw

**Error Codes:**
- `u111` - No cryopod storage found for sender
- `u112` - Awaken rotation not yet reached

---

### Stellar Conquests (Governance)

#### `propose-stellar-conquest`
```clarity
(propose-stellar-conquest (conquest-directive (string-utf8 200)) (preparation-period uint))
```
Create a new governance proposal (conquest).

**Parameters:**
- `conquest-directive` - Description of the proposal (max 200 characters)
- `preparation-period` - Number of solar rotations before the conquest launches

**Returns:** The conquest ID

**How it works:**
1. Creates a new conquest with a unique ID
2. Sets the launch rotation: `current-rotation + preparation-period`
3. Stores proposal details in the stellar-conquests map

#### `join-stellar-conquest`
```clarity
(join-stellar-conquest (conquest-id uint))
```
Vote on or join an active conquest.

**Parameters:**
- `conquest-id` - ID of the conquest to join

**Constraints:**
- Current rotation must be less than the conquest's launch rotation
- Conquest must exist

**Error Codes:**
- `u113` - Conquest not found
- `u114` - Conquest launch rotation has passed

---

## Data Structures

### Warp Last Rotation Map
Tracks the last solar rotation when each principal performed a hyperspace transfer.

```clarity
{last-warp-rotation: uint}
```

### Cryopod Storage Map
Stores information about locked tokens for each principal.

```clarity
{
  credit-amount: uint,
  hibernate-rotation: uint,
  awaken-rotation: uint
}
```

### Stellar Conquests Map
Stores governance proposal information.

```clarity
{
  fleet-admiral: principal,
  conquest-directive: (string-utf8 200),
  allies: uint,
  opposition: uint,
  is-active: bool,
  strategy-rotation: uint,
  launch-rotation: uint
}
```

---

## Error Codes

| Code | Constant | Description |
|------|----------|-------------|
| u100 | ERR-COMMANDER-ONLY | Restricted to supreme commander |
| u101 | ERR-FUEL-DEPLETED | Insufficient tokens |
| u102 | ERR-HYPERDRIVE-COOLDOWN | Transfer cooldown not met (< 10 rotations) |
| u111 | - | No cryopod storage found |
| u112 | - | Cryopod awaken rotation not reached |
| u113 | - | Conquest not found |
| u114 | - | Conquest launch rotation has passed |

---

## Usage Examples

### Basic Token Transfer with Cooldown

```clarity
;; First transfer
(contract-call? .galacticempire hyperspace-transfer u1000 'ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM)

;; Advance time
(contract-call? .galacticempire advance-solar-rotation) ;; Repeat 10+ times

;; Second transfer (now allowed)
(contract-call? .galacticempire hyperspace-transfer u500 'ST2CY5V39NHDPWSXMW9QDT3HC3GD6Q6XX4CFRK9AG)
```

### Staking Tokens (Cryopod)

```clarity
;; Lock 5000 tokens for 20 solar rotations
(contract-call? .galacticempire enter-cryopod-storage u5000 u20)

;; Advance time by 20+ rotations
(contract-call? .galacticempire advance-solar-rotation) ;; Repeat 20+ times

;; Withdraw tokens
(contract-call? .galacticempire awaken-from-cryopod)
```

### Creating a Governance Proposal

```clarity
;; Create a conquest with 15 rotation preparation period
(contract-call? .galacticempire propose-stellar-conquest u"Expand to Alpha Centauri" u15)

;; Join the conquest (within 15 rotations)
(contract-call? .galacticempire join-stellar-conquest u0)
```

---

## Design Considerations

### Solar Rotation vs Block Height

This contract uses a manual solar rotation counter instead of Stacks block heights. This design choice has implications:

**Advantages:**
- More predictable timing for testing and development
- Can be advanced independently of blockchain time
- Useful for demo and simulation environments

**Disadvantages:**
- Requires manual advancement (not automatic)
- Dependent on someone calling `advance-solar-rotation`
- Could be manipulated if not properly managed in production

### Production Recommendations

For a production deployment, consider:

1. **Automated Rotation Advancement:** Implement an off-chain service to regularly call `advance-solar-rotation`
2. **Access Control:** Add restrictions on who can advance rotations
3. **Rate Limiting:** Prevent rapid rotation advancement to maintain time integrity
4. **Alternative:** Replace solar rotations with `block-height` for automatic time progression

---

## Security Notes

- The contract has a `SUPREME-COMMANDER` constant (tx-sender) that is defined but not actively used
- No minting or burning functions are implemented
- Token transfers are subject to cooldown periods, which could affect liquidity
- The manual time system requires trust in rotation advancement

---

## License

This contract is provided as-is for educational and development purposes.

---

## Contributing

When modifying this contract, ensure:
- All rotation-based logic remains consistent
- Error handling is preserved
- Map updates are atomic and complete
- Test all time-dependent functions across multiple rotations