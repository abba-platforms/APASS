# Africa Tourism Pass (APASS)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE) ![Built with Solidity](https://img.shields.io/badge/built%20with-Solidity-363636) ![ERC-20 Compliant](https://img.shields.io/badge/ERC--20-Compliant-brightgreen) [![Security Policy](https://img.shields.io/badge/Security-Policy-blue.svg)](SECURITY.md)
![Creator](https://img.shields.io/badge/Creator-Simon%20Kapenda-lightgrey.svg)

## Reimagining Tourism Across Africa

### Discover Africa. Earn Rewards. Unlock Experiences.

By [Simon Kapenda](https://linkedin.com/in/simonkapenda).     
**Date:** March 20, 2026      
**Protocol Network:** BNB Smart Chain  
**Repository Status:** Active Infrastructure

------------------------------------------------------------

# Executive Summary      

Africa Tourism Pass (APASS) is a blockchain-native digital infrastructure designed to support the modernization and integration of Africa’s tourism economy. It introduces a unified verification, rewards, and settlement framework capable of connecting tourism operators, travelers, and services across national boundaries. At the core of the system is the APASS token, which functions as the native economic unit embedded within the protocol architecture.

APASS token enables holders to access discounts from participating tourism merchants while earning rewards from verified tourism activity. Travelers continue to transact with merchants using conventional payment methods such as credit cards, mobile payments, bank transfers, or cash, while the APASS token operates as a digital verification and incentive layer integrated with these transactions.

When an APASS token holder interacts with a participating merchant, a QR scan or point-of-sale verification records the interaction on-chain, creating a transparent and verifiable record of tourism activity. Each verified interaction generates reward points and triggers a controlled token burn, linking real-world participation with disciplined token supply management.

Accumulated reward points may be applied when acquiring additional APASS tokens, reinforcing continued engagement within the ecosystem and strengthening network effects across tourism operators and travelers.

By combining tourism incentives, blockchain-based verification, and a structured rewards system, APASS is designed to increase tourism participation, support operator revenue growth, and enable more coordinated cross-border travel across Africa.

---

# Rewards & Points

APASS introduces a structured Rewards & Points system designed to incentivize verified tourism activity across its ecosystem.

Travelers earn APASS Points through participation, which can be used to access discounted APASS tokens, while participating tourism operators can offer pricing incentives to token holders, creating stronger customer acquisition and retention.

Full details:  
https://github.com/abba-platforms/APASS/blob/main/APASS_POINTS_REWARDS.md

---

# Protocol Status

- Network: BNB Smart Chain (Mainnet)
- Status: Deployed and Operational
- Contract Verification: Completed (BscScan)
- Architecture: Modular, role-governed smart contract system
- Upgradeability: Controlled via governance roles

---

# System Overview

APASS operates as a verification-driven economic layer.

Tourism transactions continue to occur using conventional payment systems, while APASS overlays a blockchain-based verification and incentive mechanism that records participation, distributes rewards, and enforces economic policy.

The system is designed to:

- Enable verifiable tourism activity across jurisdictions
- Improve coordination between tourism operators
- Increase repeat participation and traveler engagement
- Introduce measurable, on-chain economic activity
- Support long-term growth of tourism contribution to GDP

---

# System Flow

```text
Traveler
   │
   ▼
Merchant Interaction
   │
   ▼
Verification (QR / POS)
   │
   ▼
Router (On-Chain Entry)
   │
   ├───────────────┬───────────────┬───────────────┐
   ▼               ▼               ▼               ▼
Rewards Ledger   Treasury     Burn Controller   Registry
   │               │               │               │
   ▼               ▼               ▼               ▼
Rewards Issued   Token Flow    Supply Adjusted   Activity Logged
   │
   ▼
User Outcome
   ├── Reward Points
   ├── APASS Tokens
   └── Verified Tourism Record
```

---

# How It Works

1. A traveler interacts with a participating tourism merchant  
2. The interaction is verified via QR scan or point-of-sale integration  
3. The transaction is recorded on-chain  
4. Rewards are generated and accounted for  
5. A controlled token burn is triggered  
6. Reward points accumulate for future participation  

This creates a closed-loop system where:

- Participation drives rewards
- Rewards drive engagement
- Engagement drives ecosystem growth

---

# Architecture

APASS is implemented using a modular smart contract system.

All contracts are compiled from a unified source file:

/contracts/APASSProtocol.sol

Each component is deployed independently, forming a structured protocol composed of:

- Governance Layer (AccessManager)
- Treasury Layer
- Token Layer
- Verification Layer (Router)
- Rewards and Accounting Layer
- Merchant Registry
- Redemption Layer
- Dispute Resolution Layer
- Supply Adjustment Layer

Full contract registry and deployment mapping:

https://github.com/abba-platforms/APASS/blob/main/docs/CONTRACTS.md

Contract architecture reference:

https://github.com/abba-platforms/APASS/blob/main/docs/ARCHITECTURE.md

---

# Smart Contracts

The APASS smart contract system is fully deployed on BNB Smart Chain (BSC Mainnet).

All contracts are derived from:

https://github.com/abba-platforms/APASS/blob/main/contracts/APASSProtocol.sol.md

Full registry:

https://github.com/abba-platforms/APASS/blob/main/docs/CONTRACTS.md

---

# Core Contract Addresses (BSC Mainnet)

All contracts are deployed on BNB Smart Chain (BSC Mainnet) and are publicly verifiable on BscScan.

---

## Token Layer

- APASS Token (Main Contract)       
  Symbol: APASS  
  Standard: BEP-20 (ERC-20 compatible)  
  Decimals: 18  
  Max Supply: 100,000,000,000  
  Address: 0x8f5c7779D860558AD236dC45C247c55b5D7BB3D5  
  BscScan: https://bscscan.com/token/0x8f5c7779D860558AD236dC45C247c55b5D7BB3D5  

- APASS Token Implementation (Verified)  
  Address: 0x1233b2D383Aee736C562392aC8e919873ad9dC6b  
  BscScan: https://bscscan.com/address/0x1233b2D383Aee736C562392aC8e919873ad9dC6b  

---

## Governance Layer

- AccessManager  
  Address: 0xa4Dd345Af15eB45F0c0efe32c5AE969C901d5f04  
  BscScan: https://bscscan.com/address/0xa4Dd345Af15eB45F0c0efe32c5AE969C901d5f04  

---

## Treasury Layer

- Treasury (Proxy)  
  Address: 0xd1049ca00F5F5E05939d39e7909101793901C318  
  BscScan: https://bscscan.com/address/0xd1049ca00F5F5E05939d39e7909101793901C318  

---

## Verification Layer

- APASSVerificationRouter  
  Address: 0x03fEb9637Ff65fcA7708c97Cdb4737a09C45e8cf  
  BscScan: https://bscscan.com/address/0x03fEb9637Ff65fcA7708c97Cdb4737a09C45e8cf  

---

## Merchant Participation Layer

- MerchantRegistry  
  Address: 0xdB6227623Ebd35ef9c0C81F7Ad279457FC2c290a  
  BscScan: https://bscscan.com/address/0xdB6227623Ebd35ef9c0C81F7Ad279457FC2c290a  

---

## Rewards and Accounting Layer

- RewardsLedger  
  Address: 0xc16FF4E7072E8C34A43e74D141FA8255eC75309e  
  BscScan: https://bscscan.com/address/0xc16FF4E7072E8C34A43e74D141FA8255eC75309e  

---

## Supply Adjustment Layer

- BurnController  
  Address: 0x93e15Db33f5EC95eE74f5b6B9632adE2945dF83d  
  BscScan: https://bscscan.com/address/0x93e15Db33f5EC95eE74f5b6B9632adE2945dF83d  

---

## Redemption and Settlement Layer

- Redemption  
  Address: 0x699A433210DCBBC4B287E2b3829EC584ECe9DDf1  
  BscScan: https://bscscan.com/address/0x699A433210DCBBC4B287E2b3829EC584ECe9DDf1  

---

## Dispute and Resolution Layer

- DisputeRegistry  
  Address: 0xc725d7C9c068CEF9E33F46b34356e480d792a107  
  BscScan: https://bscscan.com/address/0xc725d7C9c068CEF9E33F46b34356e480d792a107  

---  

# Economic Model

APASS is designed as a participation-driven economic system.

Core principles:

- Rewards are issued only upon verified activity
- Token supply is influenced by controlled burn mechanisms
- Treasury governs distribution flows
- Incentives are aligned with real-world tourism activity

APASS is structured for both on-chain utility and secondary market trading on centralized exchanges (CEX), supporting liquidity formation and market-based price discovery.

Detailed analysis:

https://github.com/abba-platforms/APASS/blob/main/ECONOMIC_ANALYSIS.md

---

## Mobility & Alternative Accommodation

APASS extends beyond traditional accommodation to include e-hailing services (Yango, Bolt, Uber) and alternative stay platforms such as Airbnb and short-term rentals. Without altering existing payment or booking systems, APASS recognizes verified travel activity across mobility and decentralized accommodation, enabling a unified incentive layer across the full tourism journey. For a detailed framework, see: https://github.com/abba-platforms/APASS/blob/main/APASS_MOBILITY.md

---

# Scope

APASS is not a payment processor and does not replace existing financial systems.  
It functions as a verification and incentive layer operating alongside existing payment infrastructure.

---

# Integration

APASS is designed for integration with:

- Tourism operators (hotels, lodges, tour providers)
- Payment platforms and POS systems
- Government tourism boards
- Travel aggregators and booking systems

Integration is facilitated through on-chain verification flows and modular contract interfaces.

---

# Governance

APASS operates under a structured role-based governance model enforced through the AccessManager contract.

Governance documentation:

https://github.com/abba-platforms/APASS/blob/main/docs/GOVERNANCE.md

---

# Security

The protocol is designed with layered security controls.

Security documentation:

https://github.com/abba-platforms/APASS/blob/main/docs/SECURITY.md

---

# Deployment

Deployment documentation:

https://github.com/abba-platforms/APASS/blob/main/docs/DEPLOYMENT.md

---

# Documentation

The APASS repository provides comprehensive technical, economic, and operational documentation:

- Whitepaper: https://github.com/abba-platforms/APASS/blob/main/WHITEPAPER.md
- Economic Analysis: https://github.com/abba-platforms/APASS/blob/main/ECONOMIC_ANALYSIS.md
- Tokenomics: https://github.com/abba-platforms/APASS/blob/main/docs/TOKENOMICS.md
- Architecture: https://github.com/abba-platforms/APASS/blob/main/docs/ARCHITECTURE.md
- Governance: https://github.com/abba-platforms/APASS/blob/main/docs/GOVERNANCE.md
- Security: https://github.com/abba-platforms/APASS/blob/main/docs/SECURITY.md
- Deployment: https://github.com/abba-platforms/APASS/blob/main/docs/DEPLOYMENT.md
- Integration: https://github.com/abba-platforms/APASS/blob/main/docs/INTEGRATION.md

### Models and Integrations

- APASS Model: https://github.com/abba-platforms/APASS/blob/main/APASS_MODEL.md
- APASS + NADD Integration: https://github.com/abba-platforms/APASS/blob/main/APASS_NADD.md

### Operations

- Merchant Onboarding: https://github.com/abba-platforms/APASS/blob/main/MERCHANT_ONBOARDING.md

### Smart Contracts

- Contracts Overview: https://github.com/abba-platforms/APASS/blob/main/docs/CONTRACTS.md
- Contracts Source: https://github.com/abba-platforms/APASS/blob/main/contracts/APASSProtocol.sol.md  

---

# Repository Structure

```text
/contracts
/docs
/APASS_POINTS_REWARDS.md
/APASS_MODEL.md
/APASS_NADD.md
/AUDIT.md
/CHANGELOG.md
/ECONOMIC_ANALYSIS.md
/LICENSE.md
/MERCHANT_ONBOARDING.md
/README.md
/WHITEPAPER.md
```
---

# Positioning

APASS is not a standalone application.

It is a protocol-level infrastructure designed to:

- Integrate fragmented tourism systems
- Introduce verifiable economic coordination
- Enable cross-border participation
- Establish a standardized incentive layer for tourism

---

# Additional Protocol Documents

The APASS repository includes supplementary institutional documents that define the operational and integration layers of the protocol beyond the core whitepaper.

## APASS Operational Model

- [APASS_MODEL.md](./APASS_MODEL.md)

This document provides a structured definition of how the APASS protocol operates in real-world tourism environments. It outlines:

- Traveler interaction lifecycle
- Merchant participation model
- Verification mechanisms (QR, POS, mobile)
- Reward issuance logic
- Incentive feedback loop driving continued engagement

It serves as the practical execution layer of the APASS protocol.

---

## APASS–NADD Integration Model

- [APASS_NADD.md](./APASS_NADD.md)

This document defines the integration between APASS and the Namibia Digital Dollar (NADD), establishing a dual-layer architecture:

- APASS as a non-custodial verification and incentive layer
- NADD as a regulated digital settlement layer representing the Namibian Dollar (NAD) on a 1:1 basis onchain

The document includes:

- System roles and responsibilities
- Integration architecture and operational flow
- Economic and settlement model
- Compliance and custody separation
- Deployment strategy with Namibia as the initial market

It further clarifies that APASS remains payment-agnostic, supporting multiple forms of payment including credit cards, mobile payments, bank transfers, and cash, with NADD serving as an integrated settlement option within Namibia.

---

## Merchant Onboarding

A structured framework outlining how tourism merchants—including hotels, lodges, tour operators, and tourism service providers—can join the APASS network at no cost, integrate verification mechanisms, and participate in the incentive-driven tourism ecosystem.

Documentation: https://github.com/abba-platforms/APASS/blob/main/MERCHANT_ONBOARDING.md

---

# Founder

Simon Kapenda  
Founder, Africa Tourism Pass (APASS), creator of [Namibia Digital Dollar (NADD)](https://nadd.io), [SACE Index (SACE)](https://sacex.io), [Naira Index (NXY)](https://github.com/abba-platforms/NXY), [Kenya 30 Index (KEN30)](https://ken30.com), [Namibia 30 Index (NAX30)](https://nax30.io), [CillarCoin (CILLAR)](https://cillar.io), [US Health Index (USHX)](https://ushindex.com), and [Abba App (Abba)](https://abbapp.com).

---

# License

This project is licensed under the MIT License.

Copyright (c) 2026 Simon Kapenda, Abba Industries Inc.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
