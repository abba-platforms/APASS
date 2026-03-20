// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/// @title Africa Tourism Pass (APASS) Protocol         
/// @author [Simon Kapenda](https://linkedin.com/in/simonkapenda)      

/*
  APASS Protocol v0.7

  Production hardening applied:
  - dispute amounts derived from protocol state, not admin input
  - burnedAmount stored on-chain by txId for dispute compensation
  - reward reversal uses stored record values only
  - rewards ledger has external pause/unpause
  - terminal throttling uses bytes32 keys, not address casts
  - merchant active check centralized in registry semantic method
  - pending redemption liability tracked on-chain
  - active dispute compensation exposure tracked on-chain
  - delayed claim reservation remains, with dispute-aware settlement blocking

  Operational assumptions:
  - grossAmount, discountApplied, and spendAmount are normalized reference amounts in minor units.
  - spendAmount MUST equal grossAmount - discountApplied.
  - holder signature is collected before merchant/operator submits verification.
  - production deployment must still use multisig/timelock admin custody and pass compile/test/upgrade validation.
*/

import {Initializable} from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import {UUPSUpgradeable} from "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
import {AccessControlUpgradeable} from "@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol";
import {PausableUpgradeable} from "@openzeppelin/contracts-upgradeable/utils/PausableUpgradeable.sol";
import {ReentrancyGuardUpgradeable} from "@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol";
import {ERC20Upgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import {EIP712Upgradeable} from "@openzeppelin/contracts-upgradeable/utils/cryptography/EIP712Upgradeable.sol";
import {SignatureChecker} from "@openzeppelin/contracts/utils/cryptography/SignatureChecker.sol";

library APASSRoles {
    bytes32 internal constant GOVERNANCE_ROLE = keccak256("GOVERNANCE_ROLE");
    bytes32 internal constant MERCHANT_ADMIN_ROLE = keccak256("MERCHANT_ADMIN_ROLE");
    bytes32 internal constant VERIFIER_ADMIN_ROLE = keccak256("VERIFIER_ADMIN_ROLE");
    bytes32 internal constant TREASURY_ROLE = keccak256("TREASURY_ROLE");
    bytes32 internal constant REDEMPTION_ADMIN_ROLE = keccak256("REDEMPTION_ADMIN_ROLE");
    bytes32 internal constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
    bytes32 internal constant UPGRADER_ROLE = keccak256("UPGRADER_ROLE");
    bytes32 internal constant TOKEN_ADMIN_ROLE = keccak256("TOKEN_ADMIN_ROLE");
    bytes32 internal constant BURN_ADMIN_ROLE = keccak256("BURN_ADMIN_ROLE");
    bytes32 internal constant REWARDS_ADMIN_ROLE = keccak256("REWARDS_ADMIN_ROLE");
    bytes32 internal constant DISPUTE_ADMIN_ROLE = keccak256("DISPUTE_ADMIN_ROLE");
}

interface IAPASSAccessManager {
    function hasRole(bytes32 role, address account) external view returns (bool);
}

interface IMerchantRegistry {
    function isAuthorizedMerchant(address merchant) external view returns (bool);
    function getMerchantCountryCode(address merchant) external view returns (bytes32);
    function getMerchantCategoryCode(address merchant) external view returns (uint16);
    function merchantIdOf(address merchant) external view returns (bytes32);
    function isAuthorizedOperator(address merchant, address operator) external view returns (bool);
    function isAuthorizedTerminal(address merchant, bytes32 terminalRef) external view returns (bool);
}

interface IAPASSToken is IERC20 {
    function totalSupply() external view returns (uint256);
    function protocolMint(address to, uint256 amount) external;
    function protocolBurn(address from, uint256 amount) external;
}

interface IAPASSRewardsLedger {
    function calculatePoints(uint256 spendAmount, uint16 categoryCode) external view returns (uint256);
    function recordReward(
        bytes32 txId,
        bytes32 receiptRef,
        bytes32 bookingRef,
        address holder,
        address merchant,
        uint256 grossAmount,
        uint256 discountApplied,
        uint256 spendAmount,
        uint256 pointsEarned,
        bytes32 countryCode,
        uint16 categoryCode,
        bytes32 currencyCode
    ) external;
    function freezeReward(bytes32 txId) external;
    function unfreezeReward(bytes32 txId) external;
    function reverseReward(bytes32 txId) external;
    function reservePoints(address holder, uint256 points) external;
    function unreservePoints(address holder, uint256 points) external;
    function consumeReservedPoints(address holder, uint256 points) external;
    function totalPointsEarned(address holder) external view returns (uint256);
    function totalPointsRedeemed(address holder) external view returns (uint256);
    function totalPointsFrozen(address holder) external view returns (uint256);
    function totalPointsReserved(address holder) external view returns (uint256);
    function availablePoints(address holder) external view returns (uint256);
    function rewardInfo(bytes32 txId) external view returns (
        bytes32 receiptRef,
        bytes32 bookingRef,
        address holder,
        address merchant,
        uint256 grossAmount,
        uint256 discountApplied,
        uint256 spendAmount,
        uint256 pointsEarned,
        bytes32 countryCode,
        uint16 categoryCode,
        bytes32 currencyCode,
        bool frozen,
        bool reversed,
        uint64 timestamp
    );
}

interface IAPASSTreasury {
    function releaseRewardTokens(address to, uint256 amount) external;
    function compensateBurn(address to, uint256 amount) external;
    function addPendingRewardLiability(uint256 amount) external;
    function reducePendingRewardLiability(uint256 amount) external;
    function addActiveDisputeCompensation(uint256 amount) external;
    function reduceActiveDisputeCompensation(uint256 amount) external;
}

interface IAPASSDisputeRegistry {
    function hasActiveDispute(address holder) external view returns (bool);
}

contract APASSAccessManager is Initializable, AccessControlUpgradeable, PausableUpgradeable, UUPSUpgradeable {
    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() { _disableInitializers(); }

    function initialize(
        address admin,
        address governance,
        address upgrader,
        address pauser,
        address tokenAdmin,
        address merchantAdmin,
        address burnAdmin,
        address rewardsAdmin,
        address treasuryAdmin,
        address redemptionAdmin,
        address verifierAdmin,
        address disputeAdmin
    ) external initializer {
        __AccessControl_init();
        __Pausable_init();
        __UUPSUpgradeable_init();
        _grantRole(DEFAULT_ADMIN_ROLE, admin);
        _grantRole(APASSRoles.GOVERNANCE_ROLE, governance);
        _grantRole(APASSRoles.UPGRADER_ROLE, upgrader);
        _grantRole(APASSRoles.PAUSER_ROLE, pauser);
        _grantRole(APASSRoles.TOKEN_ADMIN_ROLE, tokenAdmin);
        _grantRole(APASSRoles.MERCHANT_ADMIN_ROLE, merchantAdmin);
        _grantRole(APASSRoles.BURN_ADMIN_ROLE, burnAdmin);
        _grantRole(APASSRoles.REWARDS_ADMIN_ROLE, rewardsAdmin);
        _grantRole(APASSRoles.TREASURY_ROLE, treasuryAdmin);
        _grantRole(APASSRoles.REDEMPTION_ADMIN_ROLE, redemptionAdmin);
        _grantRole(APASSRoles.VERIFIER_ADMIN_ROLE, verifierAdmin);
        _grantRole(APASSRoles.DISPUTE_ADMIN_ROLE, disputeAdmin);
    }

    function pause() external onlyRole(APASSRoles.PAUSER_ROLE) { _pause(); }
    function unpause() external onlyRole(APASSRoles.PAUSER_ROLE) { _unpause(); }
    function _authorizeUpgrade(address newImplementation) internal override onlyRole(APASSRoles.UPGRADER_ROLE) {}
    uint256[50] private __gap;
}

contract APASSToken is Initializable, ERC20Upgradeable, PausableUpgradeable, UUPSUpgradeable {
    error Unauthorized();
    error MaxSupplyExceeded();
    error ZeroAddress();

    event AccessManagerUpdated(address indexed oldManager, address indexed newManager);
    event BurnControllerUpdated(address indexed oldController, address indexed newController);
    event TreasuryUpdated(address indexed oldTreasury, address indexed newTreasury);
    event TokensMinted(address indexed to, uint256 amount);
    event TokensBurned(address indexed from, uint256 amount);

    uint256 public constant MAX_SUPPLY = 100_000_000_000 ether;
    address public accessManager;
    address public burnController;
    address public treasury;

    modifier onlyRole(bytes32 role) { if (!IAPASSAccessManager(accessManager).hasRole(role, msg.sender)) revert Unauthorized(); _; }
    modifier onlyBurnController() { if (msg.sender != burnController) revert Unauthorized(); _; }
    modifier onlyTreasury() { if (msg.sender != treasury) revert Unauthorized(); _; }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() { _disableInitializers(); }

    function initialize(address accessManager_, address treasury_, uint256 initialTreasuryMint) external initializer {
        if (accessManager_ == address(0) || treasury_ == address(0)) revert ZeroAddress();
        __ERC20_init("Africa Tourism Pass", "APASS");
        __Pausable_init();
        __UUPSUpgradeable_init();
        accessManager = accessManager_;
        treasury = treasury_;
        if (initialTreasuryMint > MAX_SUPPLY) revert MaxSupplyExceeded();
        _mint(treasury_, initialTreasuryMint);
        emit TokensMinted(treasury_, initialTreasuryMint);
    }

    function setAccessManager(address newManager) external onlyRole(APASSRoles.GOVERNANCE_ROLE) {
        if (newManager == address(0)) revert ZeroAddress();
        emit AccessManagerUpdated(accessManager, newManager);
        accessManager = newManager;
    }

    function setBurnController(address newController) external onlyRole(APASSRoles.TOKEN_ADMIN_ROLE) {
        if (newController == address(0)) revert ZeroAddress();
        emit BurnControllerUpdated(burnController, newController);
        burnController = newController;
    }

    function setTreasury(address newTreasury) external onlyRole(APASSRoles.TOKEN_ADMIN_ROLE) {
        if (newTreasury == address(0)) revert ZeroAddress();
        emit TreasuryUpdated(treasury, newTreasury);
        treasury = newTreasury;
    }

    function protocolMint(address to, uint256 amount) external onlyTreasury {
        if (totalSupply() + amount > MAX_SUPPLY) revert MaxSupplyExceeded();
        _mint(to, amount);
        emit TokensMinted(to, amount);
    }

    function protocolBurn(address from, uint256 amount) external onlyBurnController {
        _burn(from, amount);
        emit TokensBurned(from, amount);
    }

    function pause() external onlyRole(APASSRoles.PAUSER_ROLE) { _pause(); }
    function unpause() external onlyRole(APASSRoles.PAUSER_ROLE) { _unpause(); }
    function _update(address from, address to, uint256 amount) internal override whenNotPaused { super._update(from, to, amount); }
    function _authorizeUpgrade(address newImplementation) internal override onlyRole(APASSRoles.UPGRADER_ROLE) {}
    uint256[50] private __gap;
}

contract MerchantRegistry is Initializable, PausableUpgradeable, UUPSUpgradeable {
    error Unauthorized();
    error MerchantExists();
    error MerchantNotFound();
    error ZeroAddress();

    enum MerchantLifecycle { None, Active, Suspended, Probation, UnderReview, Restricted }

    struct Merchant {
        bytes32 merchantId;
        address merchantWallet;
        bytes32 countryCode;
        uint16 categoryCode;
        MerchantLifecycle status;
        uint64 registeredAt;
    }

    event MerchantRegistered(bytes32 indexed merchantId, address indexed merchantWallet, bytes32 indexed countryCode, uint16 categoryCode);
    event MerchantWalletUpdated(bytes32 indexed merchantId, address indexed oldWallet, address indexed newWallet);
    event MerchantStatusUpdated(bytes32 indexed merchantId, address indexed merchantWallet, MerchantLifecycle status);
    event MerchantOperatorUpdated(bytes32 indexed merchantId, address indexed operator, bool active);
    event MerchantTerminalUpdated(bytes32 indexed merchantId, bytes32 indexed terminalRef, bool active);
    event AccessManagerUpdated(address indexed oldManager, address indexed newManager);

    address public accessManager;
    mapping(address => Merchant) private _merchantByWallet;
    mapping(bytes32 => address) public merchantWalletById;
    mapping(address => mapping(address => bool)) private _merchantOperatorAllowed;
    mapping(address => mapping(bytes32 => bool)) private _merchantTerminalAllowed;

    modifier onlyRole(bytes32 role) { if (!IAPASSAccessManager(accessManager).hasRole(role, msg.sender)) revert Unauthorized(); _; }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() { _disableInitializers(); }

    function initialize(address accessManager_) external initializer {
        if (accessManager_ == address(0)) revert ZeroAddress();
        __Pausable_init();
        __UUPSUpgradeable_init();
        accessManager = accessManager_;
    }

    function setAccessManager(address newManager) external onlyRole(APASSRoles.GOVERNANCE_ROLE) {
        if (newManager == address(0)) revert ZeroAddress();
        emit AccessManagerUpdated(accessManager, newManager);
        accessManager = newManager;
    }

    function registerMerchant(bytes32 merchantId, address merchantWallet, bytes32 countryCode, uint16 categoryCode) external onlyRole(APASSRoles.MERCHANT_ADMIN_ROLE) whenNotPaused {
        if (merchantWallet == address(0)) revert ZeroAddress();
        if (_merchantByWallet[merchantWallet].merchantWallet != address(0) || merchantWalletById[merchantId] != address(0)) revert MerchantExists();
        _merchantByWallet[merchantWallet] = Merchant(merchantId, merchantWallet, countryCode, categoryCode, MerchantLifecycle.Active, uint64(block.timestamp));
        merchantWalletById[merchantId] = merchantWallet;
        emit MerchantRegistered(merchantId, merchantWallet, countryCode, categoryCode);
    }

    function updateMerchantWallet(bytes32 merchantId, address newWallet) external onlyRole(APASSRoles.MERCHANT_ADMIN_ROLE) whenNotPaused {
        if (newWallet == address(0)) revert ZeroAddress();
        address currentWallet = merchantWalletById[merchantId];
        if (currentWallet == address(0)) revert MerchantNotFound();
        if (_merchantByWallet[newWallet].merchantWallet != address(0)) revert MerchantExists();
        Merchant memory m = _merchantByWallet[currentWallet];
        delete _merchantByWallet[currentWallet];
        m.merchantWallet = newWallet;
        _merchantByWallet[newWallet] = m;
        merchantWalletById[merchantId] = newWallet;
        emit MerchantWalletUpdated(merchantId, currentWallet, newWallet);
    }

    function setMerchantStatus(address merchantWallet, MerchantLifecycle status) external onlyRole(APASSRoles.MERCHANT_ADMIN_ROLE) whenNotPaused {
        Merchant storage m = _merchantByWallet[merchantWallet];
        if (m.merchantWallet == address(0)) revert MerchantNotFound();
        m.status = status;
        emit MerchantStatusUpdated(m.merchantId, merchantWallet, status);
    }

    function setMerchantOperator(address merchantWallet, address operator, bool active) external onlyRole(APASSRoles.MERCHANT_ADMIN_ROLE) whenNotPaused {
        if (_merchantByWallet[merchantWallet].merchantWallet == address(0)) revert MerchantNotFound();
        _merchantOperatorAllowed[merchantWallet][operator] = active;
        emit MerchantOperatorUpdated(_merchantByWallet[merchantWallet].merchantId, operator, active);
    }

    function setMerchantTerminal(address merchantWallet, bytes32 terminalRef, bool active) external onlyRole(APASSRoles.MERCHANT_ADMIN_ROLE) whenNotPaused {
        if (_merchantByWallet[merchantWallet].merchantWallet == address(0)) revert MerchantNotFound();
        _merchantTerminalAllowed[merchantWallet][terminalRef] = active;
        emit MerchantTerminalUpdated(_merchantByWallet[merchantWallet].merchantId, terminalRef, active);
    }

    function merchantIdOf(address merchant) external view returns (bytes32) { return _merchantByWallet[merchant].merchantId; }
    function isAuthorizedMerchant(address merchant) external view returns (bool) { return _merchantByWallet[merchant].status == MerchantLifecycle.Active; }
    function merchantStatus(address merchant) external view returns (uint8) { return uint8(_merchantByWallet[merchant].status); }
    function isAuthorizedOperator(address merchant, address operator) external view returns (bool) { return _merchantOperatorAllowed[merchant][operator]; }
    function isAuthorizedTerminal(address merchant, bytes32 terminalRef) external view returns (bool) { return _merchantTerminalAllowed[merchant][terminalRef]; }
    function getMerchantCountryCode(address merchant) external view returns (bytes32) { return _merchantByWallet[merchant].countryCode; }
    function getMerchantCategoryCode(address merchant) external view returns (uint16) { return _merchantByWallet[merchant].categoryCode; }

    function pause() external onlyRole(APASSRoles.PAUSER_ROLE) { _pause(); }
    function unpause() external onlyRole(APASSRoles.PAUSER_ROLE) { _unpause(); }
    function _authorizeUpgrade(address newImplementation) internal override onlyRole(APASSRoles.UPGRADER_ROLE) {}
    uint256[50] private __gap;
}

contract APASSBurnController is Initializable, PausableUpgradeable, UUPSUpgradeable {
    error Unauthorized();
    error ZeroAddress();
    error InsufficientBalanceForBurn();

    event BurnScheduleUpdated(uint256 mediumSpendThreshold, uint256 largeSpendThreshold, uint256 minBurnAmount, uint256 mediumBurnAmount, uint256 largeBurnAmount);
    event MinimumSupplyFloorUpdated(uint256 oldFloor, uint256 newFloor);
    event TokenUpdated(address indexed oldToken, address indexed newToken);
    event AccessManagerUpdated(address indexed oldManager, address indexed newManager);
    event RouterUpdated(address indexed oldRouter, address indexed newRouter);
    event BurnExecuted(address indexed holder, uint256 spendAmount, uint256 burnedAmount);
    event BurnSkippedAtFloor(address indexed holder, uint256 spendAmount);

    address public accessManager;
    address public router;
    IAPASSToken public token;
    uint256 public minimumSupplyFloor;
    uint256 public minBurnAmount;
    uint256 public mediumBurnAmount;
    uint256 public largeBurnAmount;
    uint256 public mediumSpendThreshold;
    uint256 public largeSpendThreshold;

    modifier onlyRole(bytes32 role) { if (!IAPASSAccessManager(accessManager).hasRole(role, msg.sender)) revert Unauthorized(); _; }
    modifier onlyRouter() { if (msg.sender != router) revert Unauthorized(); _; }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() { _disableInitializers(); }

    function initialize(address accessManager_, address token_, uint256 minimumSupplyFloor_, uint256 mediumSpendThreshold_, uint256 largeSpendThreshold_, uint256 minBurnAmount_, uint256 mediumBurnAmount_, uint256 largeBurnAmount_) external initializer {
        if (accessManager_ == address(0) || token_ == address(0)) revert ZeroAddress();
        __Pausable_init();
        __UUPSUpgradeable_init();
        accessManager = accessManager_;
        token = IAPASSToken(token_);
        minimumSupplyFloor = minimumSupplyFloor_;
        mediumSpendThreshold = mediumSpendThreshold_;
        largeSpendThreshold = largeSpendThreshold_;
        minBurnAmount = minBurnAmount_;
        mediumBurnAmount = mediumBurnAmount_;
        largeBurnAmount = largeBurnAmount_;
    }

    function setAccessManager(address newManager) external onlyRole(APASSRoles.GOVERNANCE_ROLE) { if (newManager == address(0)) revert ZeroAddress(); emit AccessManagerUpdated(accessManager, newManager); accessManager = newManager; }
    function setRouter(address newRouter) external onlyRole(APASSRoles.BURN_ADMIN_ROLE) { if (newRouter == address(0)) revert ZeroAddress(); emit RouterUpdated(router, newRouter); router = newRouter; }
    function setToken(address newToken) external onlyRole(APASSRoles.BURN_ADMIN_ROLE) { if (newToken == address(0)) revert ZeroAddress(); emit TokenUpdated(address(token), newToken); token = IAPASSToken(newToken); }
    function setBurnSchedule(uint256 mediumSpendThreshold_, uint256 largeSpendThreshold_, uint256 minBurnAmount_, uint256 mediumBurnAmount_, uint256 largeBurnAmount_) external onlyRole(APASSRoles.BURN_ADMIN_ROLE) {
        mediumSpendThreshold = mediumSpendThreshold_;
        largeSpendThreshold = largeSpendThreshold_;
        minBurnAmount = minBurnAmount_;
        mediumBurnAmount = mediumBurnAmount_;
        largeBurnAmount = largeBurnAmount_;
        emit BurnScheduleUpdated(mediumSpendThreshold_, largeSpendThreshold_, minBurnAmount_, mediumBurnAmount_, largeBurnAmount_);
    }
    function setMinimumSupplyFloor(uint256 newFloor) external onlyRole(APASSRoles.BURN_ADMIN_ROLE) { emit MinimumSupplyFloorUpdated(minimumSupplyFloor, newFloor); minimumSupplyFloor = newFloor; }
    function computeBurn(uint256 spendAmount) public view returns (uint256) {
        if (spendAmount >= largeSpendThreshold) return largeBurnAmount;
        if (spendAmount >= mediumSpendThreshold) return mediumBurnAmount;
        return minBurnAmount;
    }
    function executeBurn(address holder, uint256 spendAmount) external onlyRouter whenNotPaused returns (uint256 burned) {
        burned = computeBurn(spendAmount);
        uint256 supply = token.totalSupply();
        if (supply <= minimumSupplyFloor || supply - burned < minimumSupplyFloor) { emit BurnSkippedAtFloor(holder, spendAmount); return 0; }
        if (token.balanceOf(holder) < burned) revert InsufficientBalanceForBurn();
        token.protocolBurn(holder, burned);
        emit BurnExecuted(holder, spendAmount, burned);
    }
    function pause() external onlyRole(APASSRoles.PAUSER_ROLE) { _pause(); }
    function unpause() external onlyRole(APASSRoles.PAUSER_ROLE) { _unpause(); }
    function _authorizeUpgrade(address newImplementation) internal override onlyRole(APASSRoles.UPGRADER_ROLE) {}
    uint256[50] private __gap;
}

contract APASSRewardsLedger is Initializable, PausableUpgradeable, UUPSUpgradeable {
    error Unauthorized();
    error RewardAlreadyRecorded();
    error InsufficientPoints();
    error ZeroAddress();
    error AlreadyReversed();

    struct RewardRecord {
        bytes32 txId;
        bytes32 receiptRef;
        bytes32 bookingRef;
        address holder;
        address merchant;
        uint256 grossAmount;
        uint256 discountApplied;
        uint256 spendAmount;
        uint256 pointsEarned;
        bytes32 countryCode;
        uint16 categoryCode;
        bytes32 currencyCode;
        bool frozen;
        bool reversed;
        uint64 timestamp;
    }

    event AccessManagerUpdated(address indexed oldManager, address indexed newManager);
    event RouterUpdated(address indexed oldRouter, address indexed newRouter);
    event DisputeRegistryUpdated(address indexed oldRegistry, address indexed newRegistry);
    event RedemptionUpdated(address indexed oldRedemption, address indexed newRedemption);
    event RewardRuleUpdated(uint256 oldBps, uint256 newBps);
    event CategoryMultiplierUpdated(uint16 indexed categoryCode, uint256 oldMultiplierBps, uint256 newMultiplierBps);
    event RewardRecorded(bytes32 indexed txId, bytes32 indexed receiptRef, bytes32 indexed bookingRef, address holder, address merchant, uint256 spendAmount, uint256 pointsEarned);
    event RewardFrozen(bytes32 indexed txId, address indexed holder, uint256 pointsAmount);
    event RewardUnfrozen(bytes32 indexed txId, address indexed holder, uint256 pointsAmount);
    event RewardReversed(bytes32 indexed txId, address indexed holder, uint256 pointsAmount, uint256 spendAmount);
    event PointsReserved(address indexed holder, uint256 points);
    event ReservedPointsReleased(address indexed holder, uint256 points);
    event ReservedPointsConsumed(address indexed holder, uint256 points);

    address public accessManager;
    address public router;
    address public redemption;
    address public disputeRegistry;
    uint256 public basePointsRateBps;

    mapping(uint16 => uint256) public categoryMultiplierBps;
    mapping(bytes32 => bool) public rewardRecorded;
    mapping(address => uint256) public totalPointsEarned;
    mapping(address => uint256) public totalPointsRedeemed;
    mapping(address => uint256) public totalPointsFrozen;
    mapping(address => uint256) public totalPointsReserved;
    mapping(address => uint256) public totalVerifiedSpend;
    mapping(bytes32 => RewardRecord) public rewardByTxId;

    modifier onlyRole(bytes32 role) { if (!IAPASSAccessManager(accessManager).hasRole(role, msg.sender)) revert Unauthorized(); _; }
    modifier onlyRouter() { if (msg.sender != router) revert Unauthorized(); _; }
    modifier onlyRedemption() { if (msg.sender != redemption) revert Unauthorized(); _; }
    modifier onlyDisputeRegistry() { if (msg.sender != disputeRegistry) revert Unauthorized(); _; }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() { _disableInitializers(); }

    function initialize(address accessManager_, uint256 basePointsRateBps_) external initializer {
        if (accessManager_ == address(0)) revert ZeroAddress();
        __Pausable_init();
        __UUPSUpgradeable_init();
        accessManager = accessManager_;
        basePointsRateBps = basePointsRateBps_;
    }

    function setAccessManager(address newManager) external onlyRole(APASSRoles.GOVERNANCE_ROLE) { if (newManager == address(0)) revert ZeroAddress(); emit AccessManagerUpdated(accessManager, newManager); accessManager = newManager; }
    function setRouter(address newRouter) external onlyRole(APASSRoles.REWARDS_ADMIN_ROLE) { if (newRouter == address(0)) revert ZeroAddress(); emit RouterUpdated(router, newRouter); router = newRouter; }
    function setRedemption(address newRedemption) external onlyRole(APASSRoles.REWARDS_ADMIN_ROLE) { if (newRedemption == address(0)) revert ZeroAddress(); emit RedemptionUpdated(redemption, newRedemption); redemption = newRedemption; }
    function setDisputeRegistry(address newRegistry) external onlyRole(APASSRoles.REWARDS_ADMIN_ROLE) { if (newRegistry == address(0)) revert ZeroAddress(); emit DisputeRegistryUpdated(disputeRegistry, newRegistry); disputeRegistry = newRegistry; }
    function setBasePointsRateBps(uint256 newBps) external onlyRole(APASSRoles.REWARDS_ADMIN_ROLE) { emit RewardRuleUpdated(basePointsRateBps, newBps); basePointsRateBps = newBps; }
    function setCategoryMultiplierBps(uint16 categoryCode, uint256 newMultiplierBps) external onlyRole(APASSRoles.REWARDS_ADMIN_ROLE) { emit CategoryMultiplierUpdated(categoryCode, categoryMultiplierBps[categoryCode], newMultiplierBps); categoryMultiplierBps[categoryCode] = newMultiplierBps; }

    function calculatePoints(uint256 spendAmount, uint16 categoryCode) public view returns (uint256) {
        uint256 multiplier = categoryMultiplierBps[categoryCode];
        if (multiplier == 0) multiplier = 10_000;
        return (spendAmount * basePointsRateBps * multiplier) / 100_000_000;
    }

    function availablePoints(address holder) public view returns (uint256) {
        uint256 used = totalPointsRedeemed[holder] + totalPointsFrozen[holder] + totalPointsReserved[holder];
        uint256 earned = totalPointsEarned[holder];
        if (earned <= used) return 0;
        return earned - used;
    }

    function rewardInfo(bytes32 txId) external view returns (
        bytes32 receiptRef,
        bytes32 bookingRef,
        address holder,
        address merchant,
        uint256 grossAmount,
        uint256 discountApplied,
        uint256 spendAmount,
        uint256 pointsEarned,
        bytes32 countryCode,
        uint16 categoryCode,
        bytes32 currencyCode,
        bool frozen,
        bool reversed,
        uint64 timestamp
    ) {
        RewardRecord memory r = rewardByTxId[txId];
        return (r.receiptRef, r.bookingRef, r.holder, r.merchant, r.grossAmount, r.discountApplied, r.spendAmount, r.pointsEarned, r.countryCode, r.categoryCode, r.currencyCode, r.frozen, r.reversed, r.timestamp);
    }

    function recordReward(bytes32 txId, bytes32 receiptRef, bytes32 bookingRef, address holder, address merchant, uint256 grossAmount, uint256 discountApplied, uint256 spendAmount, uint256 pointsEarned, bytes32 countryCode, uint16 categoryCode, bytes32 currencyCode) external onlyRouter whenNotPaused {
        if (rewardRecorded[txId]) revert RewardAlreadyRecorded();
        rewardRecorded[txId] = true;
        totalPointsEarned[holder] += pointsEarned;
        totalVerifiedSpend[holder] += spendAmount;
        rewardByTxId[txId] = RewardRecord(txId, receiptRef, bookingRef, holder, merchant, grossAmount, discountApplied, spendAmount, pointsEarned, countryCode, categoryCode, currencyCode, false, false, uint64(block.timestamp));
        emit RewardRecorded(txId, receiptRef, bookingRef, holder, merchant, spendAmount, pointsEarned);
    }

    function freezeReward(bytes32 txId) external onlyDisputeRegistry whenNotPaused {
        RewardRecord storage record = rewardByTxId[txId];
        if (!record.frozen) {
            record.frozen = true;
            totalPointsFrozen[record.holder] += record.pointsEarned;
            emit RewardFrozen(txId, record.holder, record.pointsEarned);
        }
    }

    function unfreezeReward(bytes32 txId) external onlyDisputeRegistry whenNotPaused {
        RewardRecord storage record = rewardByTxId[txId];
        if (record.frozen) {
            record.frozen = false;
            totalPointsFrozen[record.holder] -= record.pointsEarned;
            emit RewardUnfrozen(txId, record.holder, record.pointsEarned);
        }
    }

    function reverseReward(bytes32 txId) external onlyDisputeRegistry whenNotPaused {
        RewardRecord storage record = rewardByTxId[txId];
        if (record.reversed) revert AlreadyReversed();
        if (record.frozen) {
            record.frozen = false;
            totalPointsFrozen[record.holder] -= record.pointsEarned;
        }
        record.reversed = true;
        totalPointsEarned[record.holder] -= record.pointsEarned;
        totalVerifiedSpend[record.holder] -= record.spendAmount;
        emit RewardReversed(txId, record.holder, record.pointsEarned, record.spendAmount);
    }

    function reservePoints(address holder, uint256 points) external onlyRedemption whenNotPaused {
        if (availablePoints(holder) < points) revert InsufficientPoints();
        totalPointsReserved[holder] += points;
        emit PointsReserved(holder, points);
    }

    function unreservePoints(address holder, uint256 points) external onlyRedemption whenNotPaused {
        totalPointsReserved[holder] -= points;
        emit ReservedPointsReleased(holder, points);
    }

    function consumeReservedPoints(address holder, uint256 points) external onlyRedemption whenNotPaused {
        totalPointsReserved[holder] -= points;
        totalPointsRedeemed[holder] += points;
        emit ReservedPointsConsumed(holder, points);
    }

    function pause() external onlyRole(APASSRoles.PAUSER_ROLE) { _pause(); }
    function unpause() external onlyRole(APASSRoles.PAUSER_ROLE) { _unpause(); }
    function _authorizeUpgrade(address newImplementation) internal override onlyRole(APASSRoles.UPGRADER_ROLE) {}
    uint256[50] private __gap;
}

contract APASSTreasury is Initializable, PausableUpgradeable, ReentrancyGuardUpgradeable, UUPSUpgradeable {
    using SafeERC20 for IERC20;

    error Unauthorized();
    error ZeroAddress();
    error InsufficientPoolBalance();
    error APASSRecoveryBlocked();

    event AccessManagerUpdated(address indexed oldManager, address indexed newManager);
    event TokenUpdated(address indexed oldToken, address indexed newToken);
    event DistributorUpdated(address indexed distributor, bool authorized);
    event PoolInventoryTagged(bytes32 indexed pool, uint256 amount);
    event PoolInventoryRebalanced(bytes32 indexed fromPool, bytes32 indexed toPool, uint256 amount);
    event RewardTokensReleased(address indexed to, uint256 amount);
    event BurnCompensated(address indexed to, uint256 amount);
    event PendingRewardLiabilityAdded(uint256 amount, uint256 newTotal);
    event PendingRewardLiabilityReduced(uint256 amount, uint256 newTotal);
    event ActiveDisputeCompensationAdded(uint256 amount, uint256 newTotal);
    event ActiveDisputeCompensationReduced(uint256 amount, uint256 newTotal);
    event ExternalTokenRecovered(address indexed token, address indexed to, uint256 amount);

    bytes32 public constant REWARDS_POOL = keccak256("REWARDS_POOL");
    bytes32 public constant ECOSYSTEM_POOL = keccak256("ECOSYSTEM_POOL");
    bytes32 public constant LIQUIDITY_POOL = keccak256("LIQUIDITY_POOL");
    bytes32 public constant GOVERNANCE_POOL = keccak256("GOVERNANCE_POOL");
    bytes32 public constant OPERATIONS_POOL = keccak256("OPERATIONS_POOL");
    bytes32 public constant BURN_COMPENSATION_POOL = keccak256("BURN_COMPENSATION_POOL");

    address public accessManager;
    IAPASSToken public token;
    mapping(bytes32 => uint256) public poolBalances;
    mapping(address => bool) public authorizedDistributor;
    uint256 public pendingRewardLiability;
    uint256 public activeDisputeCompensation;

    modifier onlyRole(bytes32 role) { if (!IAPASSAccessManager(accessManager).hasRole(role, msg.sender)) revert Unauthorized(); _; }
    modifier onlyDistributor() { if (!authorizedDistributor[msg.sender]) revert Unauthorized(); _; }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() { _disableInitializers(); }

    function initialize(address accessManager_, address token_) external initializer {
        if (accessManager_ == address(0) || token_ == address(0)) revert ZeroAddress();
        __Pausable_init();
        __ReentrancyGuard_init();
        __UUPSUpgradeable_init();
        accessManager = accessManager_;
        token = IAPASSToken(token_);
    }

    function setAccessManager(address newManager) external onlyRole(APASSRoles.GOVERNANCE_ROLE) { if (newManager == address(0)) revert ZeroAddress(); emit AccessManagerUpdated(accessManager, newManager); accessManager = newManager; }
    function setToken(address newToken) external onlyRole(APASSRoles.TREASURY_ROLE) { if (newToken == address(0)) revert ZeroAddress(); emit TokenUpdated(address(token), newToken); token = IAPASSToken(newToken); }
    function setAuthorizedDistributor(address distributor, bool authorized) external onlyRole(APASSRoles.TREASURY_ROLE) { authorizedDistributor[distributor] = authorized; emit DistributorUpdated(distributor, authorized); }

    function tagPoolInventory(bytes32 pool, uint256 amount) external onlyRole(APASSRoles.TREASURY_ROLE) whenNotPaused {
        if (freeInventory() < amount) revert InsufficientPoolBalance();
        poolBalances[pool] += amount;
        emit PoolInventoryTagged(pool, amount);
    }

    function rebalancePoolInventory(bytes32 fromPool, bytes32 toPool, uint256 amount) external onlyRole(APASSRoles.TREASURY_ROLE) whenNotPaused {
        if (poolBalances[fromPool] < amount) revert InsufficientPoolBalance();
        poolBalances[fromPool] -= amount;
        poolBalances[toPool] += amount;
        emit PoolInventoryRebalanced(fromPool, toPool, amount);
    }

    function totalTaggedInventory() public view returns (uint256 total) {
        total = poolBalances[REWARDS_POOL] + poolBalances[ECOSYSTEM_POOL] + poolBalances[LIQUIDITY_POOL] + poolBalances[GOVERNANCE_POOL] + poolBalances[OPERATIONS_POOL] + poolBalances[BURN_COMPENSATION_POOL];
    }

    function freeInventory() public view returns (uint256) { return token.balanceOf(address(this)) - totalTaggedInventory(); }
    function rewardPoolSolvent(uint256 amount) external view returns (bool) { return poolBalances[REWARDS_POOL] >= amount; }
    function burnCompensationSolvent(uint256 amount) external view returns (bool) { return poolBalances[BURN_COMPENSATION_POOL] >= amount; }
    function pendingRewardCoverage() external view returns (bool) { return poolBalances[REWARDS_POOL] >= pendingRewardLiability; }
    function activeDisputeCoverage() external view returns (bool) { return poolBalances[BURN_COMPENSATION_POOL] >= activeDisputeCompensation; }

    function addPendingRewardLiability(uint256 amount) external onlyDistributor {
        pendingRewardLiability += amount;
        emit PendingRewardLiabilityAdded(amount, pendingRewardLiability);
    }

    function reducePendingRewardLiability(uint256 amount) external onlyDistributor {
        pendingRewardLiability -= amount;
        emit PendingRewardLiabilityReduced(amount, pendingRewardLiability);
    }

    function addActiveDisputeCompensation(uint256 amount) external onlyDistributor {
        activeDisputeCompensation += amount;
        emit ActiveDisputeCompensationAdded(amount, activeDisputeCompensation);
    }

    function reduceActiveDisputeCompensation(uint256 amount) external onlyDistributor {
        activeDisputeCompensation -= amount;
        emit ActiveDisputeCompensationReduced(amount, activeDisputeCompensation);
    }

    function releaseRewardTokens(address to, uint256 amount) external onlyDistributor nonReentrant whenNotPaused {
        if (poolBalances[REWARDS_POOL] < amount) revert InsufficientPoolBalance();
        poolBalances[REWARDS_POOL] -= amount;
        IERC20(address(token)).safeTransfer(to, amount);
        emit RewardTokensReleased(to, amount);
    }

    function compensateBurn(address to, uint256 amount) external onlyDistributor nonReentrant whenNotPaused {
        if (poolBalances[BURN_COMPENSATION_POOL] < amount) revert InsufficientPoolBalance();
        poolBalances[BURN_COMPENSATION_POOL] -= amount;
        IERC20(address(token)).safeTransfer(to, amount);
        emit BurnCompensated(to, amount);
    }

    function recoverUnsupportedToken(address externalToken, address to, uint256 amount) external onlyRole(APASSRoles.TREASURY_ROLE) {
        if (externalToken == address(token)) revert APASSRecoveryBlocked();
        IERC20(externalToken).safeTransfer(to, amount);
        emit ExternalTokenRecovered(externalToken, to, amount);
    }

    function pause() external onlyRole(APASSRoles.PAUSER_ROLE) { _pause(); }
    function unpause() external onlyRole(APASSRoles.PAUSER_ROLE) { _unpause(); }
    function _authorizeUpgrade(address newImplementation) internal override onlyRole(APASSRoles.UPGRADER_ROLE) {}
    uint256[50] private __gap;
}

contract APASSDisputeRegistry is Initializable, PausableUpgradeable, ReentrancyGuardUpgradeable, UUPSUpgradeable {
    error Unauthorized();
    error ZeroAddress();
    error DisputeExists();
    error InvalidStatus();
    error VerificationNotFound();

    enum DisputeStatus { None, Open, Frozen, ResolvedNoAction, Reversed }

    struct Dispute {
        bytes32 txId;
        address holder;
        address merchant;
        uint256 pointsAmount;
        uint256 spendAmount;
        uint256 burnedTokenAmount;
        bytes32 evidenceHash;
        bytes32 caseRef;
        DisputeStatus status;
        uint64 openedAt;
        uint64 resolvedAt;
    }

    event AccessManagerUpdated(address indexed oldManager, address indexed newManager);
    event LedgerUpdated(address indexed oldLedger, address indexed newLedger);
    event TreasuryUpdated(address indexed oldTreasury, address indexed newTreasury);
    event RouterUpdated(address indexed oldRouter, address indexed newRouter);
    event DisputeOpened(bytes32 indexed txId, bytes32 indexed caseRef, address indexed holder, address merchant, uint256 pointsAmount, uint256 spendAmount, uint256 burnedTokenAmount, bytes32 evidenceHash);
    event DisputeFrozen(bytes32 indexed txId);
    event DisputeResolved(bytes32 indexed txId, DisputeStatus status);

    address public accessManager;
    APASSRewardsLedger public rewardsLedger;
    APASSTreasury public treasury;
    APASSVerificationRouter public router;
    mapping(bytes32 => Dispute) public disputeByTxId;
    mapping(address => uint256) public activeDisputeCount;

    modifier onlyRole(bytes32 role) { if (!IAPASSAccessManager(accessManager).hasRole(role, msg.sender)) revert Unauthorized(); _; }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() { _disableInitializers(); }

    function initialize(address accessManager_, address rewardsLedger_, address treasury_, address router_) external initializer {
        if (accessManager_ == address(0) || rewardsLedger_ == address(0) || treasury_ == address(0) || router_ == address(0)) revert ZeroAddress();
        __Pausable_init();
        __ReentrancyGuard_init();
        __UUPSUpgradeable_init();
        accessManager = accessManager_;
        rewardsLedger = APASSRewardsLedger(rewardsLedger_);
        treasury = APASSTreasury(treasury_);
        router = APASSVerificationRouter(router_);
    }

    function setAccessManager(address newManager) external onlyRole(APASSRoles.GOVERNANCE_ROLE) { if (newManager == address(0)) revert ZeroAddress(); emit AccessManagerUpdated(accessManager, newManager); accessManager = newManager; }
    function setRewardsLedger(address newLedger) external onlyRole(APASSRoles.DISPUTE_ADMIN_ROLE) { if (newLedger == address(0)) revert ZeroAddress(); emit LedgerUpdated(address(rewardsLedger), newLedger); rewardsLedger = APASSRewardsLedger(newLedger); }
    function setTreasury(address newTreasury) external onlyRole(APASSRoles.DISPUTE_ADMIN_ROLE) { if (newTreasury == address(0)) revert ZeroAddress(); emit TreasuryUpdated(address(treasury), newTreasury); treasury = APASSTreasury(newTreasury); }
    function setRouter(address newRouter) external onlyRole(APASSRoles.DISPUTE_ADMIN_ROLE) { if (newRouter == address(0)) revert ZeroAddress(); emit RouterUpdated(address(router), newRouter); router = APASSVerificationRouter(newRouter); }
    function hasActiveDispute(address holder) external view returns (bool) { return activeDisputeCount[holder] != 0; }

    function openDispute(bytes32 txId, bytes32 caseRef, bytes32 evidenceHash) external onlyRole(APASSRoles.DISPUTE_ADMIN_ROLE) whenNotPaused {
        if (disputeByTxId[txId].status != DisputeStatus.None) revert DisputeExists();
        (
            bytes32 receiptRef,
            bytes32 bookingRef,
            address holder,
            address merchant,
            uint256 grossAmount,
            uint256 discountApplied,
            uint256 spendAmount,
            uint256 pointsEarned,
            bytes32 countryCode,
            uint16 categoryCode,
            bytes32 currencyCode,
            bool frozen,
            bool reversed,
            uint64 timestamp
        ) = rewardsLedger.rewardInfo(txId);
        receiptRef; bookingRef; grossAmount; discountApplied; countryCode; categoryCode; currencyCode; frozen; reversed; timestamp;
        if (holder == address(0)) revert VerificationNotFound();
        uint256 burnedAmount = router.burnedAmountByTxId(txId);
        disputeByTxId[txId] = Dispute(txId, holder, merchant, pointsEarned, spendAmount, burnedAmount, evidenceHash, caseRef, DisputeStatus.Open, uint64(block.timestamp), 0);
        activeDisputeCount[holder] += 1;
        if (burnedAmount != 0) treasury.addActiveDisputeCompensation(burnedAmount);
        emit DisputeOpened(txId, caseRef, holder, merchant, pointsEarned, spendAmount, burnedAmount, evidenceHash);
    }

    function freezeDispute(bytes32 txId) external onlyRole(APASSRoles.DISPUTE_ADMIN_ROLE) whenNotPaused {
        Dispute storage d = disputeByTxId[txId];
        if (d.status != DisputeStatus.Open) revert InvalidStatus();
        rewardsLedger.freezeReward(txId);
        d.status = DisputeStatus.Frozen;
        emit DisputeFrozen(txId);
    }

    function resolveNoAction(bytes32 txId) external onlyRole(APASSRoles.DISPUTE_ADMIN_ROLE) whenNotPaused {
        Dispute storage d = disputeByTxId[txId];
        if (d.status != DisputeStatus.Open && d.status != DisputeStatus.Frozen) revert InvalidStatus();
        if (d.status == DisputeStatus.Frozen) rewardsLedger.unfreezeReward(txId);
        d.status = DisputeStatus.ResolvedNoAction;
        d.resolvedAt = uint64(block.timestamp);
        activeDisputeCount[d.holder] -= 1;
        if (d.burnedTokenAmount != 0) treasury.reduceActiveDisputeCompensation(d.burnedTokenAmount);
        emit DisputeResolved(txId, d.status);
    }

    function reverseDisputedTransaction(bytes32 txId) external onlyRole(APASSRoles.DISPUTE_ADMIN_ROLE) nonReentrant whenNotPaused {
        Dispute storage d = disputeByTxId[txId];
        if (d.status != DisputeStatus.Open && d.status != DisputeStatus.Frozen) revert InvalidStatus();
        rewardsLedger.reverseReward(txId);
        if (d.burnedTokenAmount != 0) {
            treasury.compensateBurn(d.holder, d.burnedTokenAmount);
            treasury.reduceActiveDisputeCompensation(d.burnedTokenAmount);
        }
        d.status = DisputeStatus.Reversed;
        d.resolvedAt = uint64(block.timestamp);
        activeDisputeCount[d.holder] -= 1;
        emit DisputeResolved(txId, d.status);
    }

    function pause() external onlyRole(APASSRoles.PAUSER_ROLE) { _pause(); }
    function unpause() external onlyRole(APASSRoles.PAUSER_ROLE) { _unpause(); }
    function _authorizeUpgrade(address newImplementation) internal override onlyRole(APASSRoles.UPGRADER_ROLE) {}
    uint256[50] private __gap;
}

contract APASSRedemption is Initializable, PausableUpgradeable, ReentrancyGuardUpgradeable, UUPSUpgradeable {
    error Unauthorized();
    error ZeroAddress();
    error InvalidRate();
    error ZeroOutput();
    error ClaimNotFound();
    error ClaimNotClaimable();
    error ClaimAlreadySettled();
    error HolderUnderDispute();

    struct Claim {
        uint256 id;
        address holder;
        uint256 reservedPoints;
        uint256 tokenAmount;
        uint64 createdAt;
        uint64 claimableAt;
        bool settled;
        bool cancelled;
    }

    event AccessManagerUpdated(address indexed oldManager, address indexed newManager);
    event LedgerUpdated(address indexed oldLedger, address indexed newLedger);
    event TreasuryUpdated(address indexed oldTreasury, address indexed newTreasury);
    event DisputeRegistryUpdated(address indexed oldRegistry, address indexed newRegistry);
    event RedemptionRateUpdated(uint256 oldPointUnit, uint256 newPointUnit, uint256 oldNumerator, uint256 newNumerator);
    event RedemptionDelayUpdated(uint256 oldDelay, uint256 newDelay);
    event RedemptionRequested(uint256 indexed claimId, address indexed holder, uint256 pointsReserved, uint256 tokenAmount, uint64 claimableAt);
    event RedemptionCancelled(uint256 indexed claimId, address indexed holder, uint256 pointsReleased);
    event RedemptionFinalized(uint256 indexed claimId, address indexed holder, uint256 pointsConsumed, uint256 tokensReleased);

    address public accessManager;
    IAPASSRewardsLedger public rewardsLedger;
    IAPASSTreasury public treasury;
    IAPASSDisputeRegistry public disputeRegistry;
    uint256 public redemptionPointUnit;
    uint256 public tokensPerPointNumerator;
    uint256 public redemptionDelay;
    uint256 public nextClaimId;
    mapping(uint256 => Claim) public claimById;

    modifier onlyRole(bytes32 role) { if (!IAPASSAccessManager(accessManager).hasRole(role, msg.sender)) revert Unauthorized(); _; }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() { _disableInitializers(); }

    function initialize(address accessManager_, address rewardsLedger_, address treasury_, address disputeRegistry_, uint256 redemptionPointUnit_, uint256 tokensPerPointNumerator_, uint256 redemptionDelay_) external initializer {
        if (accessManager_ == address(0) || rewardsLedger_ == address(0) || treasury_ == address(0) || disputeRegistry_ == address(0)) revert ZeroAddress();
        if (redemptionPointUnit_ == 0 || tokensPerPointNumerator_ == 0) revert InvalidRate();
        __Pausable_init();
        __ReentrancyGuard_init();
        __UUPSUpgradeable_init();
        accessManager = accessManager_;
        rewardsLedger = IAPASSRewardsLedger(rewardsLedger_);
        treasury = IAPASSTreasury(treasury_);
        disputeRegistry = IAPASSDisputeRegistry(disputeRegistry_);
        redemptionPointUnit = redemptionPointUnit_;
        tokensPerPointNumerator = tokensPerPointNumerator_;
        redemptionDelay = redemptionDelay_;
    }

    function setAccessManager(address newManager) external onlyRole(APASSRoles.GOVERNANCE_ROLE) { if (newManager == address(0)) revert ZeroAddress(); emit AccessManagerUpdated(accessManager, newManager); accessManager = newManager; }
    function setRewardsLedger(address newLedger) external onlyRole(APASSRoles.REDEMPTION_ADMIN_ROLE) { if (newLedger == address(0)) revert ZeroAddress(); emit LedgerUpdated(address(rewardsLedger), newLedger); rewardsLedger = IAPASSRewardsLedger(newLedger); }
    function setTreasury(address newTreasury) external onlyRole(APASSRoles.REDEMPTION_ADMIN_ROLE) { if (newTreasury == address(0)) revert ZeroAddress(); emit TreasuryUpdated(address(treasury), newTreasury); treasury = IAPASSTreasury(newTreasury); }
    function setDisputeRegistry(address newRegistry) external onlyRole(APASSRoles.REDEMPTION_ADMIN_ROLE) { if (newRegistry == address(0)) revert ZeroAddress(); emit DisputeRegistryUpdated(address(disputeRegistry), newRegistry); disputeRegistry = IAPASSDisputeRegistry(newRegistry); }
    function setRedemptionRate(uint256 newPointUnit, uint256 newTokensPerPointNumerator) external onlyRole(APASSRoles.REDEMPTION_ADMIN_ROLE) { if (newPointUnit == 0 || newTokensPerPointNumerator == 0) revert InvalidRate(); emit RedemptionRateUpdated(redemptionPointUnit, newPointUnit, tokensPerPointNumerator, newTokensPerPointNumerator); redemptionPointUnit = newPointUnit; tokensPerPointNumerator = newTokensPerPointNumerator; }
    function setRedemptionDelay(uint256 newDelay) external onlyRole(APASSRoles.REDEMPTION_ADMIN_ROLE) { emit RedemptionDelayUpdated(redemptionDelay, newDelay); redemptionDelay = newDelay; }

    function quoteRedemption(address holder, uint256 pointsToRedeem) public view returns (uint256 tokenAmount) {
        uint256 available = rewardsLedger.availablePoints(holder);
        if (pointsToRedeem > available) return 0;
        tokenAmount = (pointsToRedeem * tokensPerPointNumerator) / redemptionPointUnit;
    }

    function requestRedemption(uint256 pointsToRedeem) external nonReentrant whenNotPaused returns (uint256 claimId, uint256 tokenAmount) {
        if (disputeRegistry.hasActiveDispute(msg.sender)) revert HolderUnderDispute();
        tokenAmount = quoteRedemption(msg.sender, pointsToRedeem);
        if (tokenAmount == 0) revert ZeroOutput();
        rewardsLedger.reservePoints(msg.sender, pointsToRedeem);
        treasury.addPendingRewardLiability(tokenAmount);
        claimId = ++nextClaimId;
        claimById[claimId] = Claim(claimId, msg.sender, pointsToRedeem, tokenAmount, uint64(block.timestamp), uint64(block.timestamp + redemptionDelay), false, false);
        emit RedemptionRequested(claimId, msg.sender, pointsToRedeem, tokenAmount, uint64(block.timestamp + redemptionDelay));
    }

    function cancelRedemption(uint256 claimId) external nonReentrant whenNotPaused {
        Claim storage c = claimById[claimId];
        if (c.holder == address(0)) revert ClaimNotFound();
        if (c.settled || c.cancelled) revert ClaimAlreadySettled();
        if (msg.sender != c.holder && !IAPASSAccessManager(accessManager).hasRole(APASSRoles.REDEMPTION_ADMIN_ROLE, msg.sender)) revert Unauthorized();
        c.cancelled = true;
        rewardsLedger.unreservePoints(c.holder, c.reservedPoints);
        treasury.reducePendingRewardLiability(c.tokenAmount);
        emit RedemptionCancelled(claimId, c.holder, c.reservedPoints);
    }

    function finalizeRedemption(uint256 claimId) external nonReentrant whenNotPaused returns (uint256 tokenAmount) {
        Claim storage c = claimById[claimId];
        if (c.holder == address(0)) revert ClaimNotFound();
        if (c.settled || c.cancelled) revert ClaimAlreadySettled();
        if (block.timestamp < c.claimableAt) revert ClaimNotClaimable();
        if (disputeRegistry.hasActiveDispute(c.holder)) revert HolderUnderDispute();
        c.settled = true;
        rewardsLedger.consumeReservedPoints(c.holder, c.reservedPoints);
        treasury.reducePendingRewardLiability(c.tokenAmount);
        treasury.releaseRewardTokens(c.holder, c.tokenAmount);
        emit RedemptionFinalized(claimId, c.holder, c.reservedPoints, c.tokenAmount);
        return c.tokenAmount;
    }

    function pause() external onlyRole(APASSRoles.PAUSER_ROLE) { _pause(); }
    function unpause() external onlyRole(APASSRoles.PAUSER_ROLE) { _unpause(); }
    function _authorizeUpgrade(address newImplementation) internal override onlyRole(APASSRoles.UPGRADER_ROLE) {}
    uint256[50] private __gap;
}

contract APASSVerificationRouter is Initializable, PausableUpgradeable, ReentrancyGuardUpgradeable, EIP712Upgradeable, UUPSUpgradeable {
    error Unauthorized();
    error ZeroAddress();
    error MerchantNotAuthorized();
    error TerminalNotAuthorized();
    error TransactionAlreadyProcessed();
    error ReceiptAlreadyUsed();
    error HolderHasNoAPASS();
    error HolderBelowQualificationBalance();
    error HolderCannotCoverBurn();
    error InvalidSpendAmount();
    error TimestampOutsideWindow();
    error InvalidOperator();
    error InvalidHolderSignature();
    error DiscountOutOfBounds();
    error HolderRateLimited();
    error MerchantRateLimited();
    error TerminalRateLimited();
    error PairRateLimited();

    bytes32 public constant VERIFY_TYPEHASH = keccak256(
        "Verification(bytes32 txId,bytes32 receiptRef,bytes32 bookingRef,bytes32 terminalRef,address holder,address merchant,uint256 grossAmount,uint256 discountApplied,uint256 spendAmount,bytes32 currencyCode,uint64 merchantTimestamp,uint64 expiry,bytes32 nonceKey)"
    );

    event AccessManagerUpdated(address indexed oldManager, address indexed newManager);
    event RegistryUpdated(address indexed oldRegistry, address indexed newRegistry);
    event TokenUpdated(address indexed oldToken, address indexed newToken);
    event BurnControllerUpdated(address indexed oldController, address indexed newController);
    event RewardsLedgerUpdated(address indexed oldLedger, address indexed newLedger);
    event MaxVerificationDelayUpdated(uint256 oldDelay, uint256 newDelay);
    event DiscountBoundsUpdated(uint256 oldMin, uint256 oldMax, uint256 newMin, uint256 newMax);
    event MerchantLimitUpdated(uint256 oldCount, uint256 oldSpend, uint256 newCount, uint256 newSpend);
    event HolderLimitUpdated(uint256 oldCount, uint256 oldSpend, uint256 newCount, uint256 newSpend);
    event QualificationBalanceUpdated(uint256 oldMin, uint256 newMin);
    event SingleTransactionCapsUpdated(uint256 oldGrossCap, uint256 oldDiscountCap, uint256 newGrossCap, uint256 newDiscountCap);
    event ShortWindowLimitsUpdated(uint256 oldWindow, uint256 oldCount, uint256 newWindow, uint256 newCount);
    event PairWindowLimitsUpdated(uint256 oldWindow, uint256 oldCount, uint256 newWindow, uint256 newCount);
    event VerificationDigestCancelled(address indexed holder, bytes32 indexed digest);
    event TourismTransactionVerified(
        bytes32 indexed txId,
        bytes32 indexed merchantId,
        bytes32 indexed receiptRef,
        address holder,
        address merchant,
        uint256 grossAmount,
        uint256 discountApplied,
        uint256 spendAmount,
        uint256 burnedAmount,
        uint256 pointsEarned,
        bytes32 countryCode,
        uint16 categoryCode,
        bytes32 currencyCode,
        bytes32 bookingRef,
        bytes32 terminalRef,
        uint64 merchantTimestamp,
        address operator
    );

    struct DailyUsage {
        uint64 dayIndex;
        uint64 count;
        uint256 spend;
    }

    struct WindowUsage {
        uint64 windowStart;
        uint64 count;
    }

    address public accessManager;
    IMerchantRegistry public registry;
    IAPASSToken public token;
    APASSBurnController public burnController;
    IAPASSRewardsLedger public rewardsLedger;
    uint256 public maxVerificationDelay;
    uint256 public minDiscountBps;
    uint256 public maxDiscountBps;
    uint256 public maxMerchantDailyCount;
    uint256 public maxMerchantDailySpend;
    uint256 public maxHolderDailyCount;
    uint256 public maxHolderDailySpend;
    uint256 public minQualificationBalance;
    uint256 public maxSingleTransactionGross;
    uint256 public maxSingleTransactionDiscount;
    uint256 public shortWindowSeconds;
    uint256 public maxShortWindowCount;
    uint256 public pairWindowSeconds;
    uint256 public maxPairWindowCount;

    mapping(bytes32 => bool) public processedTxIds;
    mapping(bytes32 => bool) public usedReceiptRef;
    mapping(bytes32 => bool) public usedVerificationDigest;
    mapping(address => mapping(bytes32 => bool)) public usedNonceKey;
    mapping(bytes32 => bool) public cancelledDigest;
    mapping(address => DailyUsage) public merchantDailyUsage;
    mapping(address => DailyUsage) public holderDailyUsage;
    mapping(bytes32 => WindowUsage) public terminalShortUsage;
    mapping(bytes32 => WindowUsage) public pairShortUsage;
    mapping(bytes32 => uint256) public burnedAmountByTxId;

    modifier onlyRole(bytes32 role) { if (!IAPASSAccessManager(accessManager).hasRole(role, msg.sender)) revert Unauthorized(); _; }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() { _disableInitializers(); }

    function initialize(address accessManager_, address registry_, address token_, address burnController_, address rewardsLedger_, uint256 maxVerificationDelay_, uint256 minDiscountBps_, uint256 maxDiscountBps_) external initializer {
        if (accessManager_ == address(0) || registry_ == address(0) || token_ == address(0) || burnController_ == address(0) || rewardsLedger_ == address(0)) revert ZeroAddress();
        __Pausable_init();
        __ReentrancyGuard_init();
        __EIP712_init("Africa Tourism Pass Verification", "1");
        __UUPSUpgradeable_init();
        accessManager = accessManager_;
        registry = IMerchantRegistry(registry_);
        token = IAPASSToken(token_);
        burnController = APASSBurnController(burnController_);
        rewardsLedger = IAPASSRewardsLedger(rewardsLedger_);
        maxVerificationDelay = maxVerificationDelay_;
        minDiscountBps = minDiscountBps_;
        maxDiscountBps = maxDiscountBps_;
    }

    function setAccessManager(address newManager) external onlyRole(APASSRoles.GOVERNANCE_ROLE) { if (newManager == address(0)) revert ZeroAddress(); emit AccessManagerUpdated(accessManager, newManager); accessManager = newManager; }
    function setRegistry(address newRegistry) external onlyRole(APASSRoles.VERIFIER_ADMIN_ROLE) { if (newRegistry == address(0)) revert ZeroAddress(); emit RegistryUpdated(address(registry), newRegistry); registry = IMerchantRegistry(newRegistry); }
    function setToken(address newToken) external onlyRole(APASSRoles.VERIFIER_ADMIN_ROLE) { if (newToken == address(0)) revert ZeroAddress(); emit TokenUpdated(address(token), newToken); token = IAPASSToken(newToken); }
    function setBurnController(address newBurnController) external onlyRole(APASSRoles.VERIFIER_ADMIN_ROLE) { if (newBurnController == address(0)) revert ZeroAddress(); emit BurnControllerUpdated(address(burnController), newBurnController); burnController = APASSBurnController(newBurnController); }
    function setRewardsLedger(address newRewardsLedger) external onlyRole(APASSRoles.VERIFIER_ADMIN_ROLE) { if (newRewardsLedger == address(0)) revert ZeroAddress(); emit RewardsLedgerUpdated(address(rewardsLedger), newRewardsLedger); rewardsLedger = IAPASSRewardsLedger(newRewardsLedger); }
    function setMaxVerificationDelay(uint256 newDelay) external onlyRole(APASSRoles.VERIFIER_ADMIN_ROLE) { emit MaxVerificationDelayUpdated(maxVerificationDelay, newDelay); maxVerificationDelay = newDelay; }
    function setDiscountBounds(uint256 newMinDiscountBps, uint256 newMaxDiscountBps) external onlyRole(APASSRoles.VERIFIER_ADMIN_ROLE) { emit DiscountBoundsUpdated(minDiscountBps, maxDiscountBps, newMinDiscountBps, newMaxDiscountBps); minDiscountBps = newMinDiscountBps; maxDiscountBps = newMaxDiscountBps; }
    function setMerchantDailyLimits(uint256 newMaxCount, uint256 newMaxSpend) external onlyRole(APASSRoles.VERIFIER_ADMIN_ROLE) { emit MerchantLimitUpdated(maxMerchantDailyCount, maxMerchantDailySpend, newMaxCount, newMaxSpend); maxMerchantDailyCount = newMaxCount; maxMerchantDailySpend = newMaxSpend; }
    function setHolderDailyLimits(uint256 newMaxCount, uint256 newMaxSpend) external onlyRole(APASSRoles.VERIFIER_ADMIN_ROLE) { emit HolderLimitUpdated(maxHolderDailyCount, maxHolderDailySpend, newMaxCount, newMaxSpend); maxHolderDailyCount = newMaxCount; maxHolderDailySpend = newMaxSpend; }
    function setQualificationBalance(uint256 newMinQualificationBalance) external onlyRole(APASSRoles.VERIFIER_ADMIN_ROLE) { emit QualificationBalanceUpdated(minQualificationBalance, newMinQualificationBalance); minQualificationBalance = newMinQualificationBalance; }
    function setSingleTransactionCaps(uint256 newMaxSingleTransactionGross, uint256 newMaxSingleTransactionDiscount) external onlyRole(APASSRoles.VERIFIER_ADMIN_ROLE) { emit SingleTransactionCapsUpdated(maxSingleTransactionGross, maxSingleTransactionDiscount, newMaxSingleTransactionGross, newMaxSingleTransactionDiscount); maxSingleTransactionGross = newMaxSingleTransactionGross; maxSingleTransactionDiscount = newMaxSingleTransactionDiscount; }
    function setShortWindowLimits(uint256 newWindowSeconds, uint256 newMaxCount) external onlyRole(APASSRoles.VERIFIER_ADMIN_ROLE) { emit ShortWindowLimitsUpdated(shortWindowSeconds, maxShortWindowCount, newWindowSeconds, newMaxCount); shortWindowSeconds = newWindowSeconds; maxShortWindowCount = newMaxCount; }
    function setPairWindowLimits(uint256 newWindowSeconds, uint256 newMaxCount) external onlyRole(APASSRoles.VERIFIER_ADMIN_ROLE) { emit PairWindowLimitsUpdated(pairWindowSeconds, maxPairWindowCount, newWindowSeconds, newMaxCount); pairWindowSeconds = newWindowSeconds; maxPairWindowCount = newMaxCount; }

    function buildVerificationDigest(bytes32 txId, bytes32 receiptRef, bytes32 bookingRef, bytes32 terminalRef, address holder, address merchant, uint256 grossAmount, uint256 discountApplied, uint256 spendAmount, bytes32 currencyCode, uint64 merchantTimestamp, uint64 expiry, bytes32 nonceKey) public view returns (bytes32) {
        return _hashTypedDataV4(keccak256(abi.encode(VERIFY_TYPEHASH, txId, receiptRef, bookingRef, terminalRef, holder, merchant, grossAmount, discountApplied, spendAmount, currencyCode, merchantTimestamp, expiry, nonceKey)));
    }

    function cancelVerificationDigest(bytes32 digest) external whenNotPaused { cancelledDigest[digest] = true; emit VerificationDigestCancelled(msg.sender, digest); }

    function verifyTransaction(bytes32 txId, bytes32 receiptRef, bytes32 bookingRef, bytes32 terminalRef, address holder, address merchant, uint256 grossAmount, uint256 discountApplied, uint256 spendAmount, bytes32 currencyCode, uint64 merchantTimestamp, uint64 expiry, bytes32 nonceKey, bytes calldata holderSignature) external nonReentrant whenNotPaused {
        bool operatorAllowed = (msg.sender == merchant) || registry.isAuthorizedOperator(merchant, msg.sender);
        if (!operatorAllowed) revert InvalidOperator();
        if (!registry.isAuthorizedMerchant(merchant)) revert MerchantNotAuthorized();
        if (!registry.isAuthorizedTerminal(merchant, terminalRef)) revert TerminalNotAuthorized();
        if (processedTxIds[txId]) revert TransactionAlreadyProcessed();
        if (usedReceiptRef[receiptRef]) revert ReceiptAlreadyUsed();
        if (usedNonceKey[holder][nonceKey]) revert TransactionAlreadyProcessed();
        if (token.balanceOf(holder) == 0) revert HolderHasNoAPASS();
        if (token.balanceOf(holder) < minQualificationBalance) revert HolderBelowQualificationBalance();
        if (grossAmount == 0 || spendAmount == 0 || grossAmount < discountApplied || spendAmount != grossAmount - discountApplied) revert InvalidSpendAmount();
        if (maxSingleTransactionGross != 0 && grossAmount > maxSingleTransactionGross) revert InvalidSpendAmount();
        if (maxSingleTransactionDiscount != 0 && discountApplied > maxSingleTransactionDiscount) revert DiscountOutOfBounds();
        if (merchantTimestamp > block.timestamp || block.timestamp - merchantTimestamp > maxVerificationDelay || expiry < block.timestamp) revert TimestampOutsideWindow();

        uint256 discountBps = (discountApplied * 10_000) / grossAmount;
        if (discountBps < minDiscountBps || discountBps > maxDiscountBps) revert DiscountOutOfBounds();

        bytes32 digest = buildVerificationDigest(txId, receiptRef, bookingRef, terminalRef, holder, merchant, grossAmount, discountApplied, spendAmount, currencyCode, merchantTimestamp, expiry, nonceKey);
        if (cancelledDigest[digest] || usedVerificationDigest[digest]) revert TransactionAlreadyProcessed();
        if (!SignatureChecker.isValidSignatureNow(holder, digest, holderSignature)) revert InvalidHolderSignature();

        uint256 expectedBurn = burnController.computeBurn(spendAmount);
        if (expectedBurn != 0 && token.balanceOf(holder) < expectedBurn) revert HolderCannotCoverBurn();

        _consumeDailyLimit(merchantDailyUsage, merchant, spendAmount, maxMerchantDailyCount, maxMerchantDailySpend, true);
        _consumeDailyLimit(holderDailyUsage, holder, spendAmount, maxHolderDailyCount, maxHolderDailySpend, false);
        _consumeShortWindow(terminalShortUsage, _terminalKey(merchant, terminalRef));
        _consumePairWindow(_pairKey(holder, merchant));

        processedTxIds[txId] = true;
        usedReceiptRef[receiptRef] = true;
        usedVerificationDigest[digest] = true;
        usedNonceKey[holder][nonceKey] = true;

        bytes32 merchantId = registry.merchantIdOf(merchant);
        bytes32 countryCode = registry.getMerchantCountryCode(merchant);
        uint16 categoryCode = registry.getMerchantCategoryCode(merchant);
        uint256 burnedAmount = burnController.executeBurn(holder, spendAmount);
        burnedAmountByTxId[txId] = burnedAmount;
        uint256 pointsEarned = rewardsLedger.calculatePoints(spendAmount, categoryCode);

        rewardsLedger.recordReward(txId, receiptRef, bookingRef, holder, merchant, grossAmount, discountApplied, spendAmount, pointsEarned, countryCode, categoryCode, currencyCode);

        emit TourismTransactionVerified(txId, merchantId, receiptRef, holder, merchant, grossAmount, discountApplied, spendAmount, burnedAmount, pointsEarned, countryCode, categoryCode, currencyCode, bookingRef, terminalRef, merchantTimestamp, msg.sender);
    }

    function _terminalKey(address merchant, bytes32 terminalRef) internal pure returns (bytes32) {
        return keccak256(abi.encode(merchant, terminalRef));
    }

    function _pairKey(address holder, address merchant) internal pure returns (bytes32) {
        return keccak256(abi.encode(holder, merchant));
    }

    function _consumeDailyLimit(mapping(address => DailyUsage) storage usageMap, address subject, uint256 spendAmount, uint256 maxCount, uint256 maxSpend, bool merchantSide) internal {
        DailyUsage storage usage = usageMap[subject];
        uint64 dayIndex = uint64(block.timestamp / 1 days);
        if (usage.dayIndex != dayIndex) { usage.dayIndex = dayIndex; usage.count = 0; usage.spend = 0; }
        if (maxCount != 0 && usage.count + 1 > maxCount) { if (merchantSide) revert MerchantRateLimited(); revert HolderRateLimited(); }
        if (maxSpend != 0 && usage.spend + spendAmount > maxSpend) { if (merchantSide) revert MerchantRateLimited(); revert HolderRateLimited(); }
        usage.count += 1;
        usage.spend += spendAmount;
    }

    function _consumeShortWindow(mapping(bytes32 => WindowUsage) storage usageMap, bytes32 subject) internal {
        if (shortWindowSeconds == 0 || maxShortWindowCount == 0) return;
        WindowUsage storage usage = usageMap[subject];
        if (usage.windowStart == 0 || block.timestamp >= usage.windowStart + shortWindowSeconds) { usage.windowStart = uint64(block.timestamp); usage.count = 0; }
        if (usage.count + 1 > maxShortWindowCount) revert TerminalRateLimited();
        usage.count += 1;
    }

    function _consumePairWindow(bytes32 subject) internal {
        if (pairWindowSeconds == 0 || maxPairWindowCount == 0) return;
        WindowUsage storage usage = pairShortUsage[subject];
        if (usage.windowStart == 0 || block.timestamp >= usage.windowStart + pairWindowSeconds) { usage.windowStart = uint64(block.timestamp); usage.count = 0; }
        if (usage.count + 1 > maxPairWindowCount) revert PairRateLimited();
        usage.count += 1;
    }

    function pause() external onlyRole(APASSRoles.PAUSER_ROLE) { _pause(); }
    function unpause() external onlyRole(APASSRoles.PAUSER_ROLE) { _unpause(); }
    function _authorizeUpgrade(address newImplementation) internal override onlyRole(APASSRoles.UPGRADER_ROLE) {}
    uint256[50] private __gap;
}
