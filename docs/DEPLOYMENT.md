# APASS Deployment Guide

Version: 1.0.0  
Status: Mainnet Deployment Complete  
Network: BNB Smart Chain (BSC Mainnet)

---

# 1. Overview

This document defines the deployment process and operational setup of the APASS protocol on BNB Smart Chain.

The deployment follows a structured, multi-contract architecture with controlled initialization, role assignment, treasury configuration, and system wiring.

All deployments are executed using Hardhat with environment-based configuration.

---

# 2. Deployment Environment

Requirements:

- Node.js (v18+ recommended)
- Hardhat
- Ethers.js
- BSC Mainnet RPC endpoint
- Private key with sufficient BNB for gas

Environment Variables:

- PRIVATE_KEY
- BSC_MAINNET_RPC_URL
- BSCSCAN_API_KEY

---

# 3. Core Contracts

The APASS system consists of the following primary contracts:

- APASSToken  
- APASSProtocol  
- APASSVerificationRouter  
- Treasury  
- Access Manager (AccessControl / Roles)

Contracts are deployed as upgradeable (UUPS pattern where applicable).

---

# 4. Deployment Sequence

The deployment must follow this exact order:

1. Deploy Access Manager / Core Protocol (APASSProtocol)
2. Deploy APASSToken
3. Deploy Verification Router
4. Deploy Treasury
5. Link contracts (setRouter, setTreasury, etc.)
6. Assign roles
7. Configure router parameters
8. Tag treasury pools
9. Mint initial supply
10. Wire reward and redemption systems

Deviation from this order may result in failed transactions or inconsistent state.

---

# 5. Contract Wiring

Post-deployment, contracts must be interconnected.

Executed operations:

- setRouter
- setRedemption
- setDisputeRegistry

These establish communication between protocol modules.

All wiring must be verified on-chain.

---

# 6. Role Assignment

The APASS protocol uses role-based access control (RBAC) to enforce permissions across all contracts.

Roles are assigned through AccessControl and restrict execution of sensitive functions.

Key roles include:

DEFAULT_ADMIN_ROLE  
- Grants full administrative control  
- Can assign and revoke other roles  

BURN_ADMIN_ROLE  
- Authorized to execute token burn operations  
- Typically assigned to a secure multisig (SAFE)  

DISTRIBUTOR_ROLE  
- Authorized to distribute tokens from treasury-controlled pools  
- Used for reward and market distribution flows  

TREASURY_AUTHORIZED_ROLE  
- Permits interaction with treasury functions  
- Required for pool allocation and fund movements  

---

Role Assignment Execution:

Roles are granted using:

grantRole(role, address)

All role assignments must be:

- Executed by an authorized admin  
- Verified on-chain after execution  
- Restricted to trusted addresses only  

---

Operational Notes:

- Unauthorized calls to restricted functions will revert  
- Roles should not be assigned to public or unknown addresses  
- Critical roles should be assigned to multisig (SAFE) where possible  
- Role assignments must be audited after deployment  

Improper role configuration will result in failed transactions and restricted system functionality.

---

# 7. Router Configuration

The verification router enforces system-wide policy controls.

Configured parameters include:

- Qualification balance
- Merchant daily limits
- Holder daily limits
- Short window limits
- Pair window limits
- Transaction caps
- Discount bounds
- Verification delay

These parameters define the economic and operational constraints of the protocol.

---

# 8. Treasury Setup

The treasury is the central custody layer.

Actions performed:

- Treasury deployed and initialized
- Authorized modules registered:
  - Redemption
  - Dispute Registry
- Pools defined and tagged:
  - Rewards Pool
  - Burn Compensation Pool

Treasury enforces controlled distribution of all tokens.

---

# 9. Pool Tagging

Treasury pools are identified and managed using hashed pool identifiers.

Defined pools include:

- REWARDS_POOL  
- BURN_COMPENSATION_POOL  

Pools are tagged within the treasury to allocate token inventory for specific purposes.

Execution function:

tagPoolInventory(poolId, amount)

Operational Requirements:

- Caller must have appropriate authorization  
- Treasury must hold sufficient token balance  
- Pool must not already be initialized with conflicting state  

Common Failure Causes:

- Insufficient treasury balance  
- Unauthorized caller  
- Attempt to re-tag an already initialized pool  

All pool tagging operations must be verified on-chain after execution.

---

# 10. Minting

Token minting is restricted and executed through a protocol-controlled function.

Execution function:

protocolMint(address to, uint256 amount)

Key Characteristics:

- Standard ERC20 mint() is not exposed  
- Only authorized contracts or roles can mint tokens  
- Minting is governed by protocol-level controls  

Initial Mint:

- 1,000,000,000 APASS minted to Treasury  

Operational Notes:

- All minted tokens are routed through the treasury  
- Direct minting to external wallets is not permitted  
- Minting beyond initial supply must remain within the maximum supply cap  
- All mint operations must be auditable on-chain

---

# 11. Market Operations Allocation

A portion of tokens is allocated for market operations.

Execution flow:

1. Allocate tokens to Rewards Pool
2. Authorize distributor
3. Release tokens to deployer or designated wallet

Example:

- 190,000,000 APASS allocated for:
  - CEX listing
  - Market maker provisioning
  - Liquidity seeding

---

# 12. Distribution Flow

Tokens follow a strict path:

Mint → Treasury → Pool → Authorized Distributor → External Wallet

Direct transfer bypassing this flow is not supported.

---

# 13. Verification

Contracts are verified using Hardhat verify plugin.

Steps:

1. Ensure correct config (CommonJS vs ESM compatibility)
2. Verify implementation contracts
3. Verify proxy (if applicable)

Known issues:

- Etherscan API v1 deprecation
- Missing OpenZeppelin imports
- Explorer API limitations

---

# 14. Common Errors and Resolutions

Error: GS013  
Cause: Safe transaction failure or permission issue  
Resolution: Verify role assignments and Safe configuration  

Error: 0x82b42900  
Cause: Access control violation or invalid state  
Resolution: Check caller authorization and contract state  

Error: insufficient funds  
Cause: Not enough BNB for gas  
Resolution: Fund deployer wallet  

Error: token.mint is not a function  
Cause: Minting is restricted to protocolMint  
Resolution: Use correct function  

---

# 15. Operational Notes

- All critical actions are permissioned
- Treasury is the single source of truth
- Distribution is policy-driven
- System is designed for phased activation

---

# 16. Deployment Status

The APASS protocol has been successfully deployed on BNB Smart Chain Mainnet.

Completed:

- Contract deployment
- Role configuration
- Treasury initialization
- Router configuration
- Initial minting
- Pool tagging
- Market allocation setup

The system is now operational and ready for integration and market activation.

---

# 17. Deployed Contracts (BSC Mainnet)

APASSToken: 0x8f5c7779D860558AD236dC45C247c55b5D7BB3D5  
APASSAccessManager (Implementation): 0x1233b2D383Aee736C562392aC8e919873ad9dC6b  
APASSProtocol Proxy: 0xa4Dd345Af15eB45F0c0efe32c5AE969C901d5f04  
Verification Router: 0x03fEb9637Ff65fcA7708c97Cdb4737a09C45e8cf  
Treasury: 0xd1049ca00F5F5E05939d39e7909101793901C318  

Deployer Address: 0xaB9b9EFFD7E5b6c47Ba705e5F48A5d6b2C727A54  
Operational Signer: 0x1cc74471d473D5dC4437Ff7017E2111dcf6AE2A5  
SAFE (Multisig): 0x6200f4f4a83C97e0008a1C78aF873d51a4F97d5F  

---

# 18. Ownership and Control

Administrative control is enforced through role-based access control.

- DEFAULT_ADMIN_ROLE is assigned to a controlled authority (SAFE / authorized operator)
- No unrestricted public administrative access exists
- All sensitive operations require explicit role authorization

This ensures that no single actor can unilaterally control protocol state without authorization.

---

# 19. Upgradeability

Core contracts are deployed using the UUPS upgradeable proxy pattern.

- Proxy contracts delegate logic to implementation contracts
- Implementation contracts are separately deployed and verified
- Upgrades require authorized roles
- Upgrade operations are executed on-chain and are auditable

---

# 20. Deterministic Deployment

Deployment is executed using scripted Hardhat workflows.

- All parameters are defined in scripts
- Environment variables control sensitive inputs
- Deployment steps are repeatable and deterministic

This ensures reproducibility across environments.

---

# 21. Deployment Security Controls

The following controls are enforced:

- Private keys are managed via environment variables
- No secrets are hardcoded in source code
- Role assignments are validated after execution
- Treasury operations are permission-restricted
- Unauthorized calls revert at contract level

---

# 22. Post-Deployment Validation

The following checks must be completed:

- Contract addresses verified on BscScan
- Implementation contracts verified
- Roles correctly assigned
- Treasury balance confirmed
- Router configuration validated
- Pools successfully tagged
- Reward distribution paths tested
- Unauthorized access attempts revert

---

# 23. Observability

All critical operations emit on-chain events, including:

- Role assignments
- Minting operations
- Pool tagging
- Token distribution

These events provide full transparency and enable external monitoring.

---

# 24. Network Configuration

Deployment executed on:

- Network: BNB Smart Chain Mainnet
- Chain ID: 56

All parameters and configurations are production-specific.
