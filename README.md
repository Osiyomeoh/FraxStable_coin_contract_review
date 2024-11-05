# Smart Contract Audit Report

| Client         | FRAX Protocol                   |
| :------------- | :---------------------------------------------- |
| Title          | Smart Contract Audit Report                     |
| Target         | FRAXStablecoin                        |
| Version        | 1.0                                             |
| Author         | [Osiyomeoh Aleonomoh](https://github.com/Osiyomeoh) |
| Classification | Public                                          |
| Status         | Draft                                           |
| Date Created   | November 4, 2024                                |

## Table of Contents

1. [Introduction](#1-introduction)
   - [1.1 Disclaimer](#11-disclaimer)
   - [1.2 About Me](#12-about-me)
   - [1.3 Skills](#13-skills)
   - [1.4 Links](#14-links)
   - [1.5 FRAX Protocol](#15-frax-protocol)
   - [1.6 FRAXStablecoin](#16-fraxstablecoin)
   - [1.7 Scope](#17-scope)
   - [1.8 Roles](#18-roles)
   - [1.9 System Overview](#19-system-overview)
2. [Contract Review](#2-contract-review)
   - [2.1 Import Analysis](#21-import-analysis)
   - [2.2 State Variables](#22-state-variables)
   - [2.3 Function Analysis](#23-function-analysis)
3. [Findings](#3-findings)
   - [3.1 Qualitative Analysis](#31-qualitative-analysis)
   - [3.2 Summary](#32-summary)
   - [3.3 Recommendations](#33-recommendations)
4. [Conclusion](#4-conclusion)

## 1. Introduction

### 1.1 Disclaimer
This report is intended solely for the use of the client and should not be shared without prior consent. The findings and recommendations are based on the code review conducted on the specified contracts.

### 1.2 About Me
I am Osiyomeoh Aleonomoh, a software engineer and Solidity developer with a focus on blockchain technology and smart contract security.

### 1.3 Skills
- Solidity development
- Smart contract auditing
- Blockchain architecture


### 1.4 Links
- [GitHub Profile](https://github.com/Osiyomeoh)
- [LinkedIn Profile](https://www.linkedin.com/in/samuel-aleonomoh-047495162/)

### 1.5 FRAX Protocol
FRAX is a fractional-algorithmic stablecoin protocol that aims to maintain a price target of $1. The protocol uses a dynamic collateral ratio system that adjusts based on market conditions, combining both collateralized and algorithmic mechanisms to maintain stability.

### 1.6 FRAXStablecoin
The FRAXStablecoin contract is the core implementation of the FRAX protocol's stablecoin. It inherits from ERC20Custom and implements a sophisticated price stability mechanism through dynamic collateral ratios and a pool-based minting/burning system.

### 1.7 Scope

| Files in Scope              | SLOC  |
|----------------------------|--------|
| **Contracts: 1**           |        |
| FRAXStablecoin.sol         | 450    |
| **Imports: 11**            |        |
| Context.sol                | -      |
| IERC20.sol                 | -      |
| ERC20Custom.sol            | -      |
| ERC20.sol                  | -      |
| SafeMath.sol               | -      |
| Owned.sol                  | -      |
| FXS.sol                    | -      |
| FraxPool.sol               | -      |
| UniswapPairOracle.sol      | -      |
| ChainlinkETHUSDPriceConsumer.sol | - |
| AccessControl.sol          | -      |

### 1.8 Roles

#### Owner
The Owner role serves as the primary administrator of the system, holding authority to configure essential system parameters. This includes the ability to add or remove liquidity pools, set oracle addresses, and manage timelock and controller addresses. The owner's capabilities are fundamental to the initial setup and ongoing maintenance of the protocol.

#### Timelock (Governance)
The Timelock mechanism implements a delay period for administrative actions, serving as the protocol's governance layer. It possesses similar capabilities to the owner but operates with mandatory time delays. This role can modify system parameters, manage pools, configure oracles, and toggle the collateral ratio. The timelock's presence ensures deliberate and transparent protocol changes.

#### Controller
The Controller acts as an intermediate administrative role with focused responsibilities for system parameter management. While it can set system parameters, add or remove pools, and configure oracles, its actions are more limited than those of the owner or timelock. This role typically handles day-to-day operational adjustments.

#### Pools
Pool contracts serve as the primary interface for users to interact with the FRAX protocol. They handle the minting and burning of FRAX tokens, managing the actual token supply in response to user actions. Each pool must be whitelisted and operates within defined parameters to maintain system stability.

#### Collateral Ratio Pauser
This specialized role has the singular responsibility of toggling collateral ratio adjustments. It serves as an emergency brake for the system's collateral mechanism, providing a way to freeze ratio changes if market conditions or technical issues require immediate action.

### 1.9 System Overview

#### Core Components

**FRAXStablecoin.sol**
- Main stablecoin implementation
- Manages collateral ratio
- Handles price oracle integration
- Controls minting/burning through pools
- Implements governance controls

**Price Oracles**
- ChainlinkETHUSDPriceConsumer: Provides ETH/USD price feed
- UniswapPairOracle: Provides FRAX/ETH and FXS/ETH prices

**Pool System**
- FraxPool: Handles minting and redemption of FRAX tokens
- Manages collateral deposits and withdrawals

**Access Control**
- Implements role-based access control
- Manages permissions for system operations
- Controls administrative functions

## 2. Contract Review


### 2.1 Import Analysis

#### Core Ethereum Standards
```solidity
import "../Common/Context.sol";
import "../ERC20/IERC20.sol";
import "../ERC20/ERC20Custom.sol";
import "../ERC20/ERC20.sol";
```
The contract's foundation is built on standard Ethereum interfaces and implementations. The Context contract provides essential transaction context functionality through `_msgSender()` and `_msgData()`, ensuring consistent handling of transaction origins across the protocol. IERC20 defines the standard token interface, while ERC20Custom extends the base ERC20 implementation with protocol-specific functionality. This layered approach allows for customization while maintaining compatibility with the ERC20 standard.

#### Utility Libraries
```solidity
import "../Math/SafeMath.sol";
```
SafeMath library integration is crucial for preventing integer overflow and underflow vulnerabilities. All mathematical operations in the contract, particularly those involving collateral ratios, fees, and token amounts, utilize SafeMath to ensure arithmetic safety. This is especially important given the high-value nature of stablecoin operations.

#### Access Control
```solidity
import "../Staking/Owned.sol";
import "../Governance/AccessControl.sol";
```
The access control system combines basic ownership with role-based permissions. Owned provides fundamental administrative capabilities, while AccessControl enables granular permission management through roles. This dual system allows for both immediate administrative actions and governed protocol changes, creating a balance between operational efficiency and security.

#### Protocol Components
```solidity
import "../FXS/FXS.sol";
import "./Pools/FraxPool.sol";
```
These components form the core protocol mechanics. The FXS contract manages the protocol's governance token, which plays a crucial role in the algorithmic portion of the stablecoin mechanism. FraxPool defines the interface for liquidity pools, establishing how users can interact with the protocol for minting and redemption operations.

#### Oracle Integrations
```solidity
import "../Oracle/UniswapPairOracle.sol";
import "../Oracle/ChainlinkETHUSDPriceConsumer.sol";
```
The dual oracle system provides critical price data for the protocol's stability mechanisms. ChainlinkETHUSDPriceConsumer delivers reliable ETH/USD price feeds from Chainlink's decentralized oracle network. UniswapPairOracle monitors FRAX/ETH and FXS/ETH pairs on Uniswap, enabling real-time price discovery. The combination of these oracles creates a robust price feed system with both off-chain and on-chain data sources.

### 2.2 State Variables

#### Core Configuration
```solidity
string public symbol;
string public name;
uint8 public constant decimals = 18;
uint256 public constant genesis_supply = 2000000e18;
```
These variables define the token's basic characteristics. The 18-decimal precision aligns with Ethereum standards, facilitating integration with other protocols. The genesis supply of 2 million tokens provides initial liquidity while allowing for controlled expansion through the minting mechanism. These parameters are immutable after deployment, ensuring consistent token behavior.

#### Access Control
```solidity
address public creator_address;
address public timelock_address;
address public controller_address;
address public DEFAULT_ADMIN_ADDRESS;
bytes32 public constant COLLATERAL_RATIO_PAUSER;
```
The access control structure implements a multi-role system with distinct responsibilities. The creator_address maintains ultimate protocol control, while the timelock_address enables governed changes with execution delays. The controller_address manages routine operations, and the COLLATERAL_RATIO_PAUSER provides emergency intervention capabilities. This separation of powers reduces centralization risks while maintaining operational flexibility.

#### Protocol Parameters
```solidity
uint256 public global_collateral_ratio;
uint256 public redemption_fee;
uint256 public minting_fee;
uint256 public frax_step;
uint256 public refresh_cooldown;
uint256 public price_target;
uint256 public price_band;
```
These parameters govern the protocol's economic mechanics. The global_collateral_ratio determines the proportion of FRAX backed by collateral, adapting to market conditions. Minting and redemption fees provide protocol revenue while preventing arbitrage exploitation. The frax_step and refresh_cooldown control the pace of collateral ratio adjustments, preventing rapid changes that could destabilize the system. Price_target and price_band define the acceptable range for FRAX's market price, guiding the protocol's stability mechanisms.

### 2.3 Function Analysis

#### Oracle System
```solidity
function oracle_price(PriceChoice choice) internal view returns (uint256)
```
The oracle_price function serves as the protocol's price discovery mechanism. It aggregates data from multiple sources:
- Retrieves ETH/USD price from Chainlink as a trusted base reference
- Queries Uniswap oracles for FRAX/ETH and FXS/ETH prices
- Calculates final USD prices using ETH as an intermediary
- Normalizes all prices to 6 decimal places for consistent comparison
- Implements checks for price staleness and extreme variations

This function is critical for:
- Determining when to adjust the collateral ratio
- Calculating correct minting and redemption amounts
- Monitoring protocol health through price tracking

#### Collateral Management
```solidity
function refreshCollateralRatio() public
```
The collateral ratio management system implements a sophisticated stability mechanism:
- Monitors FRAX's market price against the target range
- Adjusts the global collateral ratio incrementally (frax_step)
- Enforces a cooldown period between adjustments to prevent manipulation
- Increases collateral requirements when price is below target
- Decreases collateral requirements when price is above target
- Emits events for off-chain monitoring and analysis

This mechanism balances stability with capital efficiency by:
- Responding to market pressure through measured adjustments
- Preventing rapid changes that could destabilize the system
- Maintaining price stability while optimizing collateral usage

#### Pool Operations
```solidity
function pool_mint(address m_address, uint256 m_amount) public onlyPools
function pool_burn_from(address b_address, uint256 b_amount) public onlyPools
```
The pool operations form the core user interaction layer:

Pool Minting Process:
- Validates caller is an authorized pool
- Checks global collateral requirements are met
- Calculates and applies minting fees
- Creates new FRAX tokens
- Updates global collateral tracking
- Emits minting events for transparency

Pool Burning Process:
- Verifies caller authorization
- Validates burn amount against user balance
- Applies redemption fees
- Destroys FRAX tokens
- Updates global collateral accounting
- Emits burning events

These functions maintain system integrity through:
- Strict access control via onlyPools modifier
- Accurate fee calculation and collection
- Proper collateral ratio maintenance
- Transparent operation logging

## 3. Findings

### 3.1 Qualitative Analysis

#### Critical Issues

1. **Oracle Dependency**
   The system's heavy reliance on external price feeds creates a significant vulnerability point. The current implementation lacks a robust fallback mechanism for scenarios where oracle data becomes unavailable or corrupted. This creates a single point of failure that could potentially freeze or destabilize the entire system. The use of Uniswap pools for price discovery introduces additional risks of price manipulation, particularly during periods of low liquidity.

2. **Access Control Centralization**
The current access control structure concentrates significant power in a small number of administrative roles. The absence of time-delay mechanisms for critical parameter changes creates opportunities for rapid, potentially harmful modifications to the system. The ability of a single address to pause the collateral ratio adjustment mechanism represents a centralization risk that could impact the protocol's stability.

#### Major Issues

1. **Pool Management Risks**
  The pool management system lacks crucial safety constraints. The absence of limits on the number of active pools could lead to gas optimization issues when calculating global collateral values. Furthermore, the current implementation does not include sufficient validation of pool contract code before whitelisting, potentially allowing compromised or malicious pools to interact with the system.

2. **Price Band Vulnerabilities**
The price band mechanism, while flexible, lacks necessary safeguards. The ability to configure price bands without restrictions could enable extreme collateral ratio changes that destabilize the system. The absence of an emergency pause mechanism specifically for price band adjustments leaves the system vulnerable to rapid market fluctuations.

### 3.2 Summary

| Issue Type | Count | Details                     |
|------------|-------|-----------------------------|
| Critical   | 2     | Oracle risks, Access control|
| Major      | 2     | Pool risks, Price mechanism |
| Minor      | 3     | Gas optimization, Documentation, Input validation |

### 3.3 Recommendations

1. **Oracle Security**
Implementation of oracle security should begin with heartbeat checks to ensure data freshness. The system should incorporate multiple backup price sources to prevent single points of failure. Circuit breakers should be implemented to automatically pause operations when price feeds show suspicious behavior or become stale. These mechanisms should work in concert to maintain system stability during oracle disruptions.

3. **Access Control**
  The protocol should implement mandatory timelock delays for all critical parameter changes, with delay periods proportional to the potential impact of each change. A multi-signature requirement should be introduced for administrative actions, requiring consensus from multiple trusted parties. Administrative powers should be distributed across more roles with specific, limited responsibilities to reduce centralization risks.


4. **Pool Safety**
Pool security should be enhanced through mandatory code validation before whitelisting. This includes automated checks for common vulnerabilities and manual review of pool contracts. The system should implement strict limits on the number of active pools to prevent gas-related issues. Collateral calculations should be optimized through batching and efficient data storage methods to reduce computational overhead.

## 4. Conclusion

The FRAXStablecoin contract shows sophisticated stablecoin design but requires critical fixes before deployment.

**Key Strengths:**
- Well-designed collateral ratio mechanism
- Flexible pool system
- Comprehensive event logging

**Primary Concerns:**
- Oracle dependencies
- Centralized control
- Gas optimization needs

**Final Recommendation:** HOLD - Requires critical fixes before deployment

**Risk Level:** HIGH
