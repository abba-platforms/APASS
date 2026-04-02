# CHANGELOG

All notable changes to the APASS protocol are documented in this file.

This project follows semantic versioning:

MAJOR.MINOR.PATCH

---

## [v1.0.3] - 2026-04-03

### Added
- Added APASS_MOBILITY.md: Institutional framework extending APASS across e-hailing (Yango, Bolt, Uber) and alternative accommodation platforms (Airbnb, short-term rentals), including system architecture, economic model, quantitative analysis, token dynamics, network effects, compliance posture, and deployment strategy.

### Updated
- Updated README to include "Mobility & Alternative Accommodation" section with backlink to APASS_MOBILITY.md.

---

## [v1.0.3] - 2026-03-27

### Added

- `ADD_MERCHANTS.md`

  Added a comprehensive merchant onboarding specification in the repository root defining the v1 operational framework for onboarding tourism operators into the APASS ecosystem.

  The document includes:

  - Merchant data model and structured record format
  - Required and optional merchant fields
  - Merchant and transaction ID standards
  - Merchant status lifecycle definitions
  - Discount policy structure and requirements
  - Multi-property merchant support
  - Wallet address handling guidelines
  - Storage options (JSON and database evolution path)
  - Recommended repository file structure
  - Merchant onboarding workflow for initial participants
  - Transaction logging and verification standards
  - APASS Points calculation logic
  - Support and dispute handling framework
  - Suggested backend API endpoints for future implementation
  - Security and data handling practices
  - Public vs internal data separation model
  - Success criteria for first merchant onboarding
  - Defined upgrade path toward scalable infrastructure

  This addition establishes a standardized and operationally ready approach for onboarding APASS merchants and executing real-world transactions.

---

## [v1.0.3] - 2026-03-27

### Added

- `ADD_APASS.md`

  Added a beginner-friendly wallet onboarding guide in the repository root explaining how to add APASS to BEP-20 compatible wallets, including MetaMask, Trust Wallet, and Binance Web3 Wallet.

  The document covers:

  - Wallet setup and network selection (BNB Smart Chain / BEP-20)
  - Adding APASS using the official contract address
  - Receiving APASS tokens using a wallet address
  - Explanation of wallet addresses for non-technical users
  - Guidance on zero balances before receipt and token logo visibility
  - Reminder that BNB is required for transaction fees
  - Security best practices for safe wallet usage

  This addition improves user onboarding and makes APASS more accessible to participants with no prior crypto experience.

---

## [v1.0.3] - 2026-03-23 - Added APASS Rewards & Points System Documentation

### Added
- Introduced `APASS_POINTS_REWARDS.md` in the repository root
- Comprehensive documentation of the APASS Rewards & Points system
- Detailed explanation of:
  - APASS Points (non-transferable engagement layer)
  - APASS Token (transferable on-chain value layer)
  - Points-to-token discount mechanism
  - Merchant incentive model (discounts for token holders)
  - Traveler and token holder benefits
  - Protocol controls and economic safeguards
  - Institutional and regulatory positioning

### Updated
- README updated with a concise summary of the APASS Rewards & Points system
- Added direct link to full documentation for improved accessibility and clarity

### Notes
- No changes to smart contracts or protocol logic
- Documentation update only

---

## [v1.0.3] - 2026-03-23

### Added

- docs/INVESTOR_PITCH.md

  Introduced a full institutional-grade investor pitch for Africa Tourism Pass (APASS), covering:

  - Executive Summary and Problem Definition
  - APASS Protocol Architecture and Functional Layers
  - Market Opportunity including tourism volume and long-term growth projections
  - Token Utility and ecosystem participation model
  - Revenue & Value Capture including token acquisition via CEX and platform
  - Business Model outlining infrastructure-aligned monetization pathways
  - Strategic Integration including stablecoins (USDT, NADD, ZARU, cNGN)
  - Go-To-Market Strategy with zero-cost merchant onboarding and platform deployment
  - Initial Market Focus with Namibia as the launch market
  - Use of Proceeds aligned to ecosystem scaling
  - Team & Execution including planned CEO and full operational structure
  - Execution Readiness with deployed contracts and onboarding framework
  - Investment Thesis and macro positioning
  - Appendix outlining basis of market data and projections

  The document is structured for institutional use and supports investor engagement, partnerships, and ecosystem expansion.

---

## [v1.0.3] - 2026-03-21

### Added

- Added MERCHANT_ONBOARDING.md to the root directory

  Introduces an institutional-grade framework for onboarding tourism merchants onto the Africa Tourism Pass (APASS) network, covering registration, verification, participation setup, activation, transaction verification, rewards, compliance, and operational structure.

  The document also includes ASCII diagrams illustrating onboarding flow, transaction verification, and ecosystem relationships.

---

## [v1.0.3] - 2026-03-21

### Updated

- Updated README.md to include "Additional Protocol Documents" section

  Introduces structured references to APASS_MODEL.md and APASS_NADD.md located in the root directory, providing clear visibility into the operational and integration layers of the APASS protocol.

  The update includes:

  - APASS_MODEL.md, defining the operational model of the protocol, including traveler interaction flows, merchant participation, verification mechanisms, reward issuance logic, and the incentive feedback loop.

  - APASS_NADD.md, defining the integration model between APASS and the Namibia Digital Dollar (NADD), establishing a dual-layer architecture with APASS as a non-custodial verification and incentive layer, and NADD as a regulated settlement layer representing the Namibian Dollar (NAD) on a 1:1 basis onchain.
  
  - The section also reinforces that APASS remains payment-agnostic, supporting multiple settlement methods including credit cards, mobile payments, bank transfers, and cash, while positioning NADD as an integrated settlement option within Namibia.

---

## [v1.0.2] - 2026-03-21

### Added

- Added APASS_NADD.md to the root directory

  Introduces the integration model between Africa Tourism Pass (APASS) and the Namibia Digital Dollar (NADD), defining a dual-layer architecture where APASS operates as a non-custodial verification and incentive protocol, and NADD functions as a regulated digital settlement layer representing the Namibian Dollar (NAD) on a 1:1 basis onchain.

  The document establishes Namibia as the initial deployment environment and outlines system roles, integration architecture, operational flow, economic structure, compliance separation, and deployment strategy.

  It further clarifies that APASS remains payment-agnostic and does not depend on NADD as an exclusive settlement mechanism. Tourism transactions may be settled through multiple payment methods, including credit cards, mobile payments, bank transfers, and cash, with NADD serving as an integrated settlement option within Namibia.

---

## [1.0.1] - 2026-03-21 - Model Definition Update

### Model and System Definition

- Added APASS_MODEL.md defining the tourism interaction model and system-level operational flow
- Documented end-to-end traveler lifecycle including token acquisition, participation, verification, rewards, and reuse
- Introduced merchant interaction model covering onboarding, verification, and ecosystem participation
- Defined non-custodial settlement structure operating alongside conventional payment systems
- Established economic feedback loop linking tourism activity with token access and sustained participation
- Added full system architecture diagram illustrating interaction between Router, Merchant Registry, Rewards Ledger, Treasury, Token Layer, and Burn Controller
- Defined data and analytics layer for measurable tourism activity and ecosystem insights
- Documented risk management and control mechanisms including role-based access and verification safeguards
- Clarified system boundaries and interoperability with external systems including wallets, exchanges, and POS integrations

---

## [1.0.0] - 2026-03-20 - Initial Protocol Release

### Protocol

- Deployed the APASS protocol on BNB Smart Chain (BSC Mainnet)
- Implemented modular smart contract architecture compiled from APASSProtocol.sol
- Established role-based governance through AccessManager
- Activated Treasury for controlled token custody and distribution
- Enabled verification-driven transaction flow via Router
- Implemented RewardsLedger for reward accounting
- Integrated BurnController for supply adjustment mechanisms
- Deployed MerchantRegistry for participant onboarding
- Enabled Redemption contract for token redemption flows
- Activated DisputeRegistry for dispute handling and resolution

### Token System

- Introduced APASS as the native protocol token (BEP-20 / ERC-20 compatible)
- Enforced controlled supply model through treasury-governed distribution
- Implemented reward issuance and burn-linked supply mechanics

### Security

- Completed internal OpenZeppelin-style audit
- No critical, high, or medium vulnerabilities identified
- Audit score: 9.2 / 10
- Enforced role-based access control across all privileged operations
- Established deterministic treasury and distribution logic
- Implemented verification-gated reward issuance

### Contracts

- Published full contract system via:
  - contracts/APASSProtocol.sol.md
- Documented complete contract registry and deployment mapping:
  - docs/CONTRACTS.md
- Verified implementation contract on BscScan
- Deployed upgradeable contract architecture under governance control

### Documentation

- Added WHITEPAPER.md (institutional protocol specification)
- Added ECONOMIC_ANALYSIS.md (macro and sector analysis)
- Added AUDIT.md (internal audit documentation)
- Added README.md (protocol overview and system entry point)

- Added docs/ARCHITECTURE.md (system design)
- Added docs/CONTRACTS.md (contract registry)
- Added docs/SECURITY.md (security framework)
- Added docs/GOVERNANCE.md (role and control structure)
- Added docs/DEPLOYMENT.md (deployment procedures)
- Added docs/INTEGRATION.md (integration framework and system interaction model)

### Integration

- Defined APASS integration framework for merchants, platforms, payment systems, governments, and exchanges
- Introduced structured tree-based integration flow for end-to-end system interaction
- Established Router-based smart contract interaction model for integrations
- Documented merchant onboarding and verification workflows
- Clarified integration boundaries and non-custodial operational model

### Repository

- Established institutional-grade repository structure
- Integrated full documentation stack and contract references
- Prepared repository for public access, audit, and integration
- Enabled version-controlled protocol lifecycle management

---

## Versioning Policy

MAJOR

Incremented for changes that affect protocol architecture, governance model, or economic structure.

MINOR

Incremented for backward-compatible feature additions and enhancements.

PATCH

Incremented for bug fixes, optimizations, and non-breaking updates.

---

## Notes

Version 1.0.0 represents the baseline production release of the APASS protocol, including full deployment, documentation, contract system availability, and integration framework.

All subsequent updates will build upon this version.
