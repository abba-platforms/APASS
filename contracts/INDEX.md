# APASS Smart Contracts

The APASS protocol is implemented using a modular smart contract system compiled from a unified Solidity source file.

All core contract logic is defined within a single canonical implementation:

- [APASSProtocol.sol.md](./APASSProtocol.sol.md) — consolidated contract source and full system logic

Multiple contracts are deployed independently on-chain from this unified source, forming the APASS protocol architecture. A complete registry of all deployed contracts and their addresses is provided here:

- [CONTRACTS.md](./CONTRACTS.md) — official contract registry and mainnet deployment mapping

This structure ensures consistency across contract interfaces, simplifies auditability, and maintains a clear alignment between source code, deployed contracts, and protocol documentation.
