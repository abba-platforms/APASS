# APASS Contracts Registry

Version: 1.0.0  
Status: Official  
Network: BNB Smart Chain (BSC Mainnet)  

---

# 1. Purpose

This document provides the authoritative registry of all smart contracts composing the Africa Tourism Pass (APASS) protocol.

It establishes a clear mapping between:

- Deployed on-chain contracts  
- Verified source code  
- Repository structure  
- Functional roles within the protocol architecture  

This document is intended for:

- Developers and protocol integrators  
- Security auditors and reviewers  
- Centralized exchanges (CEX) and market makers  
- Institutional partners and policymakers  

---

# 2. System Overview

The APASS protocol is a modular smart contract system implemented through a consolidated architecture.

All core protocol components are defined within a unified contract source and deployed as coordinated modules.

The system includes the following functional layers:

- Protocol Layer (APASSProtocol)  
- Verification Layer (APASSVerificationRouter)  
- Treasury Layer (Treasury)  
- Governance and Access Control Layer  
- Rewards and Accounting Layer  
- Merchant Participation Layer  
- Redemption and Settlement Layer  
- Dispute and Resolution Layer  
- Supply Adjustment Layer  

The APASS token exists as the native economic unit of the system and is implemented as part of the protocol architecture.

---

# 3. Contract Architecture Model

The APASS protocol is implemented using a consolidated smart contract architecture.

All contract logic is defined within a single Solidity source file:

/contracts/APASSProtocol.sol

This file contains multiple logically separated contracts forming the full protocol system, including:

- APASSProtocol (core coordination and governance layer)  
- APASSVerificationRouter (verification and policy enforcement)  
- Treasury (custody and controlled distribution layer)  
- AccessManager (role-based access control)  
- RewardsLedger (reward accounting)  
- MerchantRegistry (merchant eligibility management)  
- BurnController (supply adjustment mechanisms)  
- Redemption (tourism settlement layer)  
- DisputeRegistry (claims and dispute handling)  

The APASS token is implemented as part of the protocol system and is not maintained as a standalone contract file within the repository.

Token issuance is controlled through protocol-level functions and treasury-governed distribution mechanisms.

This architecture ensures:

- Centralized control of token issuance within protocol governance  
- Reduced contract surface area  
- Simplified audit traceability  
- Strong alignment between economic logic and protocol execution  

---

# 4. Repository Structure

All smart contract logic is maintained under:

/contracts/APASSProtocol.sol

This file is the canonical source of all deployed contract logic.

A documentation reference is also provided:

/docs/APASSProtocol.sol.md

This document contains a consolidated and formatted representation of the full contract system for audit and review purposes.

---

# 5. Mainnet Contract Registry

## 5.1 APASS Token

Token Address:  
0x8f5c7779D860558AD236dC45C247c55b5D7BB3D5  

Standard:  
BEP-20 native, ERC-20 compatible  

Implementation:  
Protocol-integrated (not a standalone repository contract file)  

Function:  
The APASS token serves as the native economic unit of the protocol, enabling reward distribution, redemption, and external trading.

Key Properties:  
- Maximum supply: 100,000,000,000 APASS  
- Initial minted supply: 1,000,000,000 APASS  
- Minting controlled via protocol-level functions (protocolMint)  
- No unrestricted public mint interface  

Repository Reference:  
/contracts/APASSProtocol.sol  

---

## 5.2 APASSProtocol (Proxy)

Contract Name:  
APASSProtocol  

Proxy Address:  
0xa4Dd345Af15eB45F0c0efe32c5AE969C901d5f04  

Implementation Address:  
0x1233b2D383Aee736C562392aC8e919873ad9dC6b  

Type:  
Upgradeable (UUPS Proxy)  

Function:  
Serves as the central coordination and execution layer of the APASS protocol.

Key Responsibilities:  
- Orchestrates protocol operations  
- Enforces governance and role control  
- Coordinates treasury and reward flows  
- Enables controlled upgradeability  

Repository Reference:  
/contracts/APASSProtocol.sol  

---

## 5.3 APASSVerificationRouter

Contract Name:  
APASSVerificationRouter  

Address:  
0x03fEb9637Ff65fcA7708c97Cdb4737a09C45e8cf  

Upgradeability:  
Non-upgradeable  

Function:  
Validates tourism interactions prior to reward issuance.

Key Controls:  
- Qualification balance enforcement  
- Merchant and holder limits  
- Short-window and pair-window controls  
- Discount bounds  
- Verification delay constraints  

Repository Reference:  
/contracts/APASSProtocol.sol  

---

## 5.4 Treasury

Contract Name:  
Treasury  

Address:  
0xd1049ca00F5F5E05939d39e7909101793901C318  

Upgradeability:  
Non-upgradeable  

Function:  
Custody and controlled distribution of APASS token inventory.

Key Properties:  
- Central token reserve  
- Tagged pool system (Rewards, Burn Compensation)  
- Restricted distribution via authorized roles  
- No arbitrary withdrawals  

Repository Reference:  
/contracts/APASSProtocol.sol  

---

## 5.5 AccessManager

Contract Name:  
AccessManager  

Type:  
Role-based access control  

Function:  
Enforces permissions and governance controls across the protocol.

Key Responsibilities:  
- Role assignment and revocation  
- Administrative control enforcement  
- Protection of sensitive operations  

Repository Reference:  
/contracts/APASSProtocol.sol  

---

## 5.6 RewardsLedger

Contract Name:  
RewardsLedger  

Function:  
Maintains accounting of reward entitlements from verified tourism activity.

Key Properties:  
- Tracks reward allocations  
- Separates accounting from distribution  
- Works with treasury-controlled releases  

Repository Reference:  
/contracts/APASSProtocol.sol  

---

## 5.7 MerchantRegistry

Contract Name:  
MerchantRegistry  

Function:  
Registers and manages eligible tourism operators.

Key Properties:  
- Defines valid merchants  
- Enforces participation eligibility  
- Integrates with verification logic  

Repository Reference:  
/contracts/APASSProtocol.sol  

---

## 5.8 BurnController

Contract Name:  
BurnController  

Function:  
Executes controlled token burn operations and supply adjustments.

Key Properties:  
- Permissioned burn execution  
- Supports economic balancing  
- Linked to treasury accounting  

Repository Reference:  
/contracts/APASSProtocol.sol  

---

## 5.9 Redemption

Contract Name:  
Redemption  

Function:  
Handles redemption of APASS tokens for tourism services.

Key Properties:  
- Enables economic loop closure  
- Integrates with treasury and verification layers  
- Controlled settlement execution  

Repository Reference:  
/contracts/APASSProtocol.sol  

---

## 5.10 DisputeRegistry

Contract Name:  
DisputeRegistry  

Function:  
Manages disputes related to tourism transactions and protocol activity.

Key Properties:  
- Structured dispute tracking  
- Transparent claims recording  
- Supports resolution workflows  

Repository Reference:  
/contracts/APASSProtocol.sol  

---

# 6. Governance and Roles

The APASS system operates under role-based access control.

Core roles include:

- DEFAULT_ADMIN_ROLE  
- TREASURY_AUTHORIZED_ROLE  
- DISTRIBUTOR_ROLE  
- BURN_ADMIN_ROLE  

Governance structure is defined in:

/docs/GOVERNANCE.md  

---

# 7. Upgradeability Model

The APASS protocol uses a hybrid upgradeability model.

Non-upgradeable components:
- APASSVerificationRouter  
- Treasury  

Upgradeable component:
- APASSProtocol (UUPS Proxy)  

Token logic is protocol-integrated and governed through the system architecture.

All upgrades are:

- Governance-controlled  
- Executed on-chain  
- Fully traceable  

---

# 8. Verification and Transparency

All contracts must be:

- Verified on BscScan  
- Matched with repository source  
- Bytecode-consistent  

This ensures:

- Public transparency  
- Auditability  
- Exchange readiness  
- Institutional trust  

---

# 9. Economic Flow Mapping

The APASS contract system enforces a closed-loop economic model:

1. Tourism activity is verified  
2. Rewards are recorded in RewardsLedger  
3. Tokens are released from Treasury  
4. Tokens are utilized or traded  
5. Tokens are redeemed for services  
6. Optional burn mechanisms adjust supply  

This ensures that token circulation is directly tied to verified economic participation.

---

# 10. Repository Integrity Requirements

To maintain institutional standards:

- Full contract source must be provided  
- No truncated files  
- SPDX license identifiers required  
- Deployed contracts must match repository code  

This is required for:

- Audit readiness  
- Exchange due diligence  
- Institutional adoption  

---

# 11. Maintenance

This document must be updated upon:

- New deployments  
- Contract upgrades  
- Address changes  
- Governance updates  
- Verification changes  

Machine-readable deployment data:

/deployments/mainnet.json  

This document remains the canonical human-readable reference.

---
