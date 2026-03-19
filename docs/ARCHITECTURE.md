# APASS Protocol Architecture

Version: 1.0.0  
Status: Official  
Network: BNB Smart Chain (BEP-20 / EVM Compatible)

---

# 1. Overview

The Africa Tourism Pass (APASS) protocol is a modular, upgradeable smart contract system designed to support verified tourism participation, reward distribution, and redemption across a unified digital infrastructure.

The architecture separates responsibilities across independent contracts to ensure:

- Security through isolation of critical functions  
- Scalability through modular design  
- Auditability through deterministic execution paths  
- Governance through role-based access control  

The protocol operates entirely on-chain, with all critical economic actions governed by smart contracts.

---

# 2. Architectural Principles

The APASS protocol is built on the following core principles:

Separation of Concerns  
Each contract is responsible for a distinct operational function, reducing systemic risk and simplifying audits.

Deterministic Execution  
All state changes are governed by explicit contract logic, ensuring predictable outcomes.

Treasury Isolation  
Token inventory is managed through a dedicated treasury contract and cannot be arbitrarily transferred.

Verification-Based Incentives  
Rewards are only issued after validated tourism activity.

Role-Based Governance  
Administrative permissions are enforced through the Access Manager contract.

Upgradeable Infrastructure  
The system uses UUPS upgradeability to allow controlled evolution of the protocol.

---

# 3. High-Level Architecture

```text
                           +----------------------+
                           |   Access Manager     |
                           | (Role Governance)    |
                           +----------+-----------+
                                      |
           +--------------------------+--------------------------+
           |                                                     |
+----------v-----------+                           +-------------v------------+
|      Treasury        |                           |        APASS Token       |
|  Token Inventory     |                           |  ERC20 / BEP-20 Token    |
|  Distribution Logic  |                           |  Protocol Minting        |
+----------+-----------+                           +-------------+------------+
           |                                                     |
           v                                                     v
+----------+-----------+                           +-------------+------------+
|     Rewards Ledger   |                           |    Burn Controller       |
|  Reward Accounting   |                           |   Supply Management      |
+----------+-----------+                           +-------------+------------+
           |
           v
+----------+-----------+
|  Verification Router |
|  Policy Enforcement  |
+----------+-----------+
           |
           v
+----------+-----------+
|   Merchant Registry  |
|  Verified Operators  |
+----------+-----------+
           |
           v
+----------+-----------+
|    Redemption        |
| Tourism Settlement   |
+----------+-----------+
           |
           v
+----------+-----------+
|   Dispute Registry   |
| Claim Resolution     |
+----------------------+
```
---

# 4. Core Contracts

## 4.1 APASS Token

The APASS token is a BEP-20 native, ERC-20 compatible digital asset deployed on BNB Smart Chain.

Responsibilities:

- Token issuance via protocol-controlled minting  
- Standard ERC-20 transfer functionality  
- Integration with treasury and protocol modules  

Minting is restricted to authorized roles and cannot be executed arbitrarily.

---

## 4.2 Treasury

The Treasury contract manages all protocol token inventory.

Responsibilities:

- Maintain token reserves  
- Allocate tokens to defined pools  
- Distribute tokens through authorized modules  

Key Properties:

- Tokens cannot be freely transferred out  
- All movements must pass through protocol logic  
- Pools are tagged (e.g., Rewards Pool, Burn Compensation Pool)

---

## 4.3 Access Manager

The Access Manager enforces role-based governance across the protocol.

Responsibilities:

- Assign and revoke roles  
- Restrict access to sensitive functions  
- Control upgrade permissions  

---

## 4.4 Rewards Ledger

The Rewards Ledger records reward entitlements generated through verified tourism activity.

Responsibilities:

- Track user reward balances  
- Interface with Treasury for distribution  
- Maintain accounting integrity  

---

## 4.5 Verification Router

The Verification Router validates tourism interactions before rewards are issued.

Responsibilities:

- Enforce policy rules  
- Validate transaction parameters  
- Trigger reward allocation  

---

## 4.6 Merchant Registry

The Merchant Registry maintains verified tourism operators.

Responsibilities:

- Register tourism providers  
- Validate merchant eligibility  

---

## 4.7 Burn Controller

The Burn Controller manages token supply adjustments.

Responsibilities:

- Execute token burns  
- Maintain supply balance  

---

## 4.8 Redemption System

The Redemption contract enables conversion of APASS tokens into tourism services.

Responsibilities:

- Process redemption requests  
- Trigger settlement flows  

---

## 4.9 Dispute Registry

The Dispute Registry provides a structured resolution mechanism.

Responsibilities:

- Record disputes  
- Track claims  

---

# 5. Interaction Flow

```text
Traveler
   |
   v
Tourism Operator (Merchant)
   |
   v
Verification Router
   |
   v
Rewards Ledger
   |
   v
Treasury
   |
   v
Traveler Wallet
```

The interaction flow represents the end-to-end lifecycle of a verified tourism transaction within the APASS ecosystem.

A traveler engages with a registered tourism operator (merchant). The transaction is submitted to the Verification Router, where protocol policies and validation rules are enforced.

Upon successful verification, the Rewards Ledger records the corresponding reward entitlement. The Treasury then releases the allocated tokens based on the validated interaction.

Finally, the APASS tokens are transferred to the traveler’s wallet, completing the incentive distribution cycle.

This flow ensures that all rewards are strictly tied to verified tourism activity and that token distribution remains controlled, auditable, and policy-compliant.

---

# 6. Treasury Pool Model

The treasury maintains structured pools:

- Rewards Pool  
- Burn Compensation Pool  
- Unallocated Inventory  

Each pool is tagged and managed independently.

Token flows must reference a valid pool identifier before execution.

---

# 7. Upgradeability (UUPS Model)

The APASS protocol uses UUPS (Universal Upgradeable Proxy Standard).

Key Characteristics:

- Logic contracts can be upgraded  
- Storage remains persistent  
- Upgrade authority restricted via Access Manager  

---

# 8. Security Architecture

The protocol incorporates the following protections:

- Reentrancy protection  
- Role-based access control  
- Treasury isolation  
- Deterministic validation  
- Modular isolation  

---

# 9. Trust Assumptions and Boundaries

The protocol assumes:

- Governance roles are controlled by trusted entities  
- Smart contracts execute deterministically as deployed  
- External integrations (CEX, APIs, merchants) operate independently  

Trust boundaries exist between:

- On-chain protocol logic  
- Off-chain merchant systems  
- External exchange infrastructure  

---

# 10. Failure Modes and Safeguards

Potential failure scenarios include:

- Invalid verification inputs  
- Unauthorized access attempts  
- Merchant misconfiguration  
- Reward abuse attempts  

Safeguards include:

- Role restrictions  
- Policy validation rules  
- Treasury isolation  
- Transaction constraints  

---

# 11. Event Logging and Observability

All critical protocol actions emit on-chain events.

These include:

- Reward issuance  
- Treasury allocations  
- Merchant registration  
- Redemption execution  
- Governance actions  

This enables:

- On-chain monitoring  
- Analytics integration  
- Audit traceability  

---

# 12. Gas and Scalability Considerations

The protocol is optimized for:

- Minimal storage writes  
- Efficient contract interactions  
- Modular execution paths  

Scalability is supported through:

- Independent module execution  
- Off-chain indexing (optional)  
- Batched operations (future support)  

---

# 13. Integration Interfaces

The APASS protocol can integrate with:

- Centralized exchanges (CEX)  
- Wallet providers  
- Tourism platforms  
- Payment gateways  
- Analytics systems  

Integration occurs through:

- Standard ERC-20 interfaces  
- Contract function calls  
- Event-based data extraction  

---

# 14. Deployment Configuration (Mainnet)

Network: BNB Smart Chain  
Chain ID: 56  

All contracts are deployed and verified on BscScan.

Refer to:

/deployments/mainnet.json

---

# 15. Design Summary

The APASS architecture is designed to function as a programmable tourism infrastructure layer.

Key characteristics:

- Modular and upgradeable  
- Verification-driven reward system  
- Treasury-controlled token economy  
- Role-based governance  
- Fully on-chain execution  

This architecture enables APASS to scale across multiple tourism markets while maintaining security, transparency, and operational integrity.
