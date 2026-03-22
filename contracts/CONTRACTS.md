# APASS Contracts Registry

Version: 2.0.0  
Status: Official  
Network: BNB Smart Chain (BSC Mainnet)  

---

# Table of Contents

- [1. Purpose](#1-purpose)
- [2. Architecture Overview](#2-architecture-overview)
- [3. Deployment Model](#3-deployment-model)
- [4. Repository Structure](#4-repository-structure)
- [5. Mainnet Contract Registry](#5-mainnet-contract-registry)
  - [5.1 Governance Layer](#51-governance-layer)
    - [AccessManager](#accessmanager)
  - [5.2 Treasury Layer](#52-treasury-layer)
    - [Treasury (Proxy)](#treasury-proxy)
  - [5.3 Token Layer](#53-token-layer)
    - [APASS Token](#apass-token)
  - [5.4 Verification Layer](#54-verification-layer)
    - [APASSVerificationRouter](#apassverificationrouter)
  - [5.5 Merchant Participation Layer](#55-merchant-participation-layer)
    - [MerchantRegistry](#merchantregistry)
  - [5.6 Rewards and Accounting Layer](#56-rewards-and-accounting-layer)
    - [RewardsLedger](#rewardsledger)
  - [5.7 Supply Adjustment Layer](#57-supply-adjustment-layer)
    - [BurnController](#burncontroller)
  - [5.8 Redemption and Settlement Layer](#58-redemption-and-settlement-layer)
    - [Redemption](#redemption)
  - [5.9 Dispute and Resolution Layer](#59-dispute-and-resolution-layer)
    - [DisputeRegistry](#disputeregistry)
- [6. Role Configuration](#6-role-configuration)
- [7. Upgradeability Model](#7-upgradeability-model)
- [8. Verification and Transparency](#8-verification-and-transparency)
- [9. Economic Flow Mapping](#9-economic-flow-mapping)
- [10. Repository Integrity Requirements](#10-repository-integrity-requirements)
- [11. Maintenance](#11-maintenance)

---

# 1. Purpose

This document provides the authoritative registry of all smart contracts composing the Africa Tourism Pass (APASS) protocol.

It establishes a clear and verifiable mapping between:

- Deployed on-chain contract addresses  
- Verified contract implementations  
- Repository source code  
- Functional roles within the protocol architecture  

This document is intended for:

- Developers and integrators  
- Security auditors and reviewers  
- Centralized exchanges (CEX) and market makers  
- Institutional partners and policymakers  

---

# 2. Architecture Overview

The APASS protocol is implemented using a modular smart contract architecture.

All contracts are compiled from a unified Solidity source file:

/contracts/APASSProtocol.sol

However, each component is deployed as an independent contract, enabling:

- Separation of concerns  
- Independent interaction between modules  
- Clear audit boundaries  
- Scalable protocol design  

The system consists of the following layers:

- Governance Layer (AccessManager)  
- Treasury Layer  
- Token Layer  
- Verification Layer  
- Rewards and Accounting Layer  
- Merchant Participation Layer  
- Redemption and Settlement Layer  
- Dispute and Resolution Layer  
- Supply Adjustment Layer  

---

# 3. Deployment Model

APASS uses a modular deployment model in which multiple contracts are deployed independently from a unified Solidity source file.

Each contract operates as a distinct on-chain component with clearly defined responsibilities and controlled interaction surfaces.

This model ensures:

- Strong separation between governance, treasury, and execution logic  
- Reduced systemic risk  
- Improved upgrade and maintenance flexibility  
- Clear traceability for audits and institutional review  

---

# 4. Repository Structure

All smart contract logic is contained within:

/contracts/APASSProtocol.sol  

This file is the canonical source of all deployed contracts.

A documentation reference is also maintained:

/docs/APASSProtocol.sol.md  

This file provides a consolidated representation of the full contract system for audit and review purposes.

---

# 5. Mainnet Contract Registry

## 5.1 Governance Layer

### AccessManager <a id="accessmanager"></a>

Address:  
0xa4Dd345Af15eB45F0c0efe32c5AE969C901d5f04  

Function:  
Central governance and role-based access control contract.

Key Responsibilities:  
- Manages all protocol roles and permissions  
- Controls administrative authority across contracts  
- Enforces access restrictions for sensitive operations  

---

## 5.2 Treasury Layer

### Treasury (Proxy) <a id="treasury-proxy"></a>

Address:  
0xd1049ca00F5F5E05939d39e7909101793901C318  

Function:  
Custody and controlled distribution of APASS token inventory.

Key Properties:  
- Central reserve of protocol tokens  
- Tagged pool system (Rewards Pool, Burn Compensation Pool)  
- Distribution restricted to authorized roles  
- No arbitrary withdrawals  

---

## 5.3 Token Layer

### APASS Token <a id="apass-token"></a>

Address:  
0x8f5c7779D860558AD236dC45C247c55b5D7BB3D5  

Standard:  
BEP-20 native, ERC-20 compatible  

Function:  
Native economic unit of the APASS protocol.

Key Properties:  
- Maximum supply: 100,000,000,000 APASS  
- Initial minted supply: 1,000,000,000 APASS  
- Minting controlled via protocol-level authorization  
- Supports rewards, redemption, and external trading  

---

## 5.4 Verification Layer

### APASSVerificationRouter <a id="apassverificationrouter"></a>

Address:  
0x03fEb9637Ff65fcA7708c97Cdb4737a09C45e8cf  

Function:  
Validates tourism interactions before rewards are issued.

Key Controls:  
- Qualification balance thresholds  
- Merchant and holder transaction limits  
- Time-window enforcement rules  
- Discount bounds  
- Verification delay constraints  

---

## 5.5 Merchant Participation Layer

### MerchantRegistry <a id="merchantregistry"></a>

Address:  
0xdB6227623Ebd35ef9c0C81F7Ad279457FC2c290a  

Function:  
Registers and manages eligible tourism merchants.

Key Properties:  
- Defines participation eligibility  
- Integrates with verification layer  
- Controls merchant-level access  

---

## 5.6 Rewards and Accounting Layer

### RewardsLedger <a id="rewardsledger"></a>

Address:  
0xc16FF4E7072E8C34A43e74D141FA8255eC75309e  

Function:  
Maintains accounting of rewards generated from verified tourism activity.

Key Properties:  
- Records reward entitlements  
- Separates accounting from distribution  
- Works with treasury-controlled releases  

---

## 5.7 Supply Adjustment Layer

### BurnController <a id="burncontroller"></a>

Address:  
0x93e15Db33f5EC95eE74f5b6B9632adE2945dF83d  

Function:  
Executes controlled token burn operations.

Key Properties:  
- Permissioned burn execution  
- Supports supply discipline  
- Integrated with economic policy  

---

## 5.8 Redemption and Settlement Layer

### Redemption <a id="redemption"></a>

Address:  
0x699A433210DCBBC4B287E2b3829EC584ECe9DDf1  

Function:  
Handles redemption of APASS tokens for tourism services.

Key Properties:  
- Enables economic loop closure  
- Integrates with treasury and verification systems  
- Controlled settlement execution  

---

## 5.9 Dispute and Resolution Layer

### DisputeRegistry <a id="disputeregistry"></a>

Address:  
0xc725d7C9c068CEF9E33F46b34356e480d792a107  

Function:  
Manages disputes related to tourism transactions and protocol activity.

Key Properties:  
- Structured dispute tracking  
- Transparent claims recording  
- Supports resolution workflows  

---

# 6. Role Configuration

The protocol operates under a structured role-based governance model.

Key roles include:

- ADMIN  
- GOVERNANCE  
- UPGRADER  
- PAUSER  
- TOKEN ADMIN  
- MERCHANT ADMIN  
- BURN ADMIN  
- REWARDS ADMIN  
- TREASURY ADMIN  
- REDEMPTION ADMIN  
- VERIFIER ADMIN  
- DISPUTE ADMIN  

Role assignments are enforced through the AccessManager contract.

---

# 7. Upgradeability Model

The APASS protocol uses a hybrid upgradeability approach.

Upgradeable components:
- Treasury (proxy-based architecture where applicable)  

Non-upgradeable components:
- Token  
- Router  
- Registry modules  
- Controllers  

All upgrade actions must be:

- Authorized through governance roles  
- Executed on-chain  
- Fully traceable  

---

# 8. Verification and Transparency

All contracts are expected to be:

- Verified on BscScan  
- Bytecode-matched with repository source  
- Publicly accessible  

This ensures:

- Auditability  
- Transparency  
- Institutional trust  
- Exchange readiness  

---

# 9. Economic Flow Mapping

The APASS contract system enforces a closed-loop economic model:

1. Tourism activity is verified via Router  
2. Rewards are recorded in RewardsLedger  
3. Tokens are released from Treasury  
4. Tokens are used or traded externally  
5. Tokens are redeemed for tourism services  
6. BurnController adjusts supply where applicable  

This ensures that token circulation is directly tied to verified economic participation.

---

# 10. Repository Integrity Requirements

To maintain institutional standards:

- Full contract source must be provided  
- No truncated or partial files  
- SPDX license identifiers must be present  
- Repository code must match deployed contracts  

This is required for:

- Security audits  
- Exchange listings  
- Institutional adoption  

---

# 11. Maintenance

This document must be updated upon:

- New contract deployments  
- Contract upgrades  
- Address changes  
- Governance updates  
- Verification status changes  

Machine-readable deployment data should be maintained in:

/deployments/mainnet.json  

This document remains the canonical human-readable reference.

---
