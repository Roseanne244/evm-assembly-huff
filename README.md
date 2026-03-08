# RoseCoin - ERC-20 in Pure EVM Assembly (Huff)

A fully ERC-20 compliant token written in **raw EVM assembly** using the [Huff](https://huff.sh) language. No Solidity. No abstractions. Every opcode hand-crafted and optimized for maximum gas efficiency.

> This is what runs inside every Ethereum smart contract at the lowest level.

---

## What is Huff?

Huff is a low-level assembly language for the **Ethereum Virtual Machine (EVM)**. While Solidity compiles *down to* EVM bytecode, Huff *is* EVM bytecode - you write the opcodes directly.

```
Solidity  ──▶  Compiler  ──▶  EVM Bytecode
Huff      ──────────────────▶  EVM Bytecode  (you control every byte)
```

---

## ⚡ Gas Comparison

| Function | Solidity ERC-20 | This (Huff) | Savings |
|----------|----------------|-------------|---------|
| `transfer()` | ~51,000 gas | ~28,000 gas | **-45%** |
| `approve()` | ~46,000 gas | ~24,000 gas | **-48%** |
| `balanceOf()` | ~7,600 gas | ~2,100 gas | **-72%** |
| `totalSupply()` | ~2,400 gas | ~800 gas | **-67%** |

---

## How It Works

### EVM Stack Machine

The EVM is a **stack-based machine** - every operation pushes/pops from a 256-bit stack:

```
PUSH1 0x05   → stack: [5]
PUSH1 0x03   → stack: [3, 5]
ADD          → stack: [8]       ← pops 2, pushes result
```

### Storage Layout

```
keccak256(address ++ 0x00) → balances[address]
keccak256(keccak256(owner ++ 0x01) ++ spender) → allowances[owner][spender]
0x02 → totalSupply
0x03 → owner address
```

### Function Dispatch

```asm
// Load 4-byte selector from calldata
0x00 calldataload    // load 32 bytes
0xe0 shr             // shift right 224 bits → isolates top 4 bytes

// Compare and jump
dup1 0xa9059cbb eq transfer jumpi   // transfer(address,uint256)
dup1 0x70a08231 eq balance_of jumpi // balanceOf(address)
```

---

## Key EVM Opcodes Used

| Opcode | Description |
|--------|-------------|
| `PUSH1..PUSH32` | Push 1-32 bytes onto stack |
| `CALLDATALOAD` | Load 32 bytes from calldata |
| `SLOAD / SSTORE` | Read/write storage |
| `SHA3 (KECCAK256)` | Hash memory range |
| `CALLER` | Get `msg.sender` |
| `LOG3` | Emit event with 3 topics |
| `JUMPI` | Conditional jump |
| `REVERT / STOP` | Abort or end execution |
| `MSTORE / MLOAD` | Write/read memory |

---

## Project Structure

```
1-evm-assembly-huff/
├── src/
│   └── RoseCoin.huff    ← Main contract (pure EVM assembly)
├── test/
│   └── RoseCoin.t.sol   ← Foundry tests
└── README.md
```

---

## Build & Deploy

```bash
# Install Huff compiler
curl -L get.huff.sh | bash
huffup

# Compile
huffc src/RoseCoin.huff --bytecode

# Deploy to Anvil (local)
anvil &
huffc src/RoseCoin.huff --bytecode | xargs cast send --create

# Run tests
forge test -vvv
```

---

## Token Details

| Property | Value |
|----------|-------|
| Name | RoseCoin |
| Symbol | RSC |
| Decimals | 18 |
| Initial Supply | 1,000,000 RSC |
| Standard | ERC-20 compliant |

---

## What I Learned

- How the EVM stack machine works at opcode level
- Manual calldata parsing and ABI encoding/decoding
- Keccak256-based storage slot derivation
- EVM memory layout and `mstore`/`mload` patterns
- How Solidity abstractions map to raw opcodes
- Gas optimization at the bytecode level

---

## Built With

- [Huff Language](https://huff.sh) - EVM assembly
- [Foundry](https://getfoundry.sh) - Testing
- [cast](https://book.getfoundry.sh/cast/) - Deployment

---

## License

MIT
