

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

- <a href="#intro">1. INTRODUCTION</a>
  - <a href="#Disclaim">1.1 Disclaimer</a>
  - <a href="#About">1.2 About Me </a>
  - <a href="#Skills">1.3 Skills</a>
  - <a href="#links">1.4 Links</a>
  - <a href="#Cpg">1.5 FRAX Protocol</a>
  - <a href="#Gbd">1.6 FRAXStablecoin</a>
  - <a href="#scope">1.7 Scope</a>
  - <a href="#roles">1.8 Roles</a>
  - <a href="#overview">1.9 System Overview</a>
- <a href="#review">2.0 CONTRACT REVIEW</a>
- <a href="#findings">3.0 FINDINGS</a>
  - <a href="#Qanalysis">3.1 Qualitative Analysis</a>
  - <a href="#summary">3.2 Summary</a>
  - <a href="#recom">3.3 Recommendations</a>
- <a href="#conclusion">4.0 CONCLUSION</a>

<h2 id="intro">1.0 INTRODUCTION </h2>

## 1.1 Disclaimer
This report is intended solely for the use of the client and should not be shared without prior consent. The findings and recommendations are based on the code review conducted on the specified contracts.

## 1.2 About Me
I am Osiyomeoh Aleonomoh, a software engineer and Solidity developer with a focus on blockchain technology and smart contract security.

## 1.3 Skills
- Solidity development
- Smart contract auditing
- Blockchain architecture
- Risk analysis and mitigation

## 1.4 Links
- [GitHub Profile](https://github.com/Osiyomeoh)
- [LinkedIn Profile](https://www.linkedin.com/in/samuel-aleonomoh-047495162/)

## 1.5 FRAX Protocol
FRAX is a fractional-algorithmic stablecoin protocol that aims to maintain a price target of $1. The protocol uses a dynamic collateral ratio system that adjusts based on market conditions, combining both collateralized and algorithmic mechanisms to maintain stability.

## 1.6 FRAXStablecoin
The FRAXStablecoin contract is the core implementation of the FRAX protocol's stablecoin. It inherits from ERC20Custom and implements a sophisticated price stability mechanism through dynamic collateral ratios and a pool-based minting/burning system.

## 1.7 Scope
| Files in Scope              | SLOC  |
|------------------------------|-------|
| Contracts: 1                 |       |
| FRAXStablecoin.sol           | 450   |
| Imports: 11                  |       |
| Context.sol                  | -     |
| IERC20.sol                   | -     |
| ERC20Custom.sol              | -     |
| ERC20.sol                    | -     |
| SafeMath.sol                 | -     |
| Owned.sol                    | -     |
| FXS.sol                      | -     |
| FraxPool.sol                 | -     |
| UniswapPairOracle.sol        | -     |
| ChainlinkETHUSDPriceConsumer.sol | -  |
| AccessControl.sol            | -     |

## 1.8 Roles
**Owner**
- Set system parameters
- Add/remove pools
- Set oracles
- Set timelock and controller addresses

**Timelock (Governance)**
- Set system parameters
- Add/remove pools
- Set oracles
- Set timelock and controller addresses
- Toggle collateral ratio

**Controller**
- Set system parameters
- Add/remove pools
- Set oracles

**Pools**
- Mint FRAX tokens
- Burn FRAX tokens

**Collateral Ratio Pauser**
- Toggle collateral ratio adjustments

## 1.9 System Overview
**Core Components:**

**FRAXStablecoin.sol**
- Main stablecoin implementation
- Manages collateral ratio
- Handles price oracle integration
- Controls minting/burning through pools
- Implements governance controls

**Price Oracles:**
- ChainlinkETHUSDPriceConsumer: Provides ETH/USD price feed
- UniswapPairOracle: Provides FRAX/ETH and FXS/ETH prices

**Pool System:**
- FraxPool: Handles minting and redemption of FRAX tokens
- Manages collateral deposits and withdrawals

**Access Control:**
- Implements role-based access control
- Manages permissions for system operations
- Controls administrative functions

<h2 id="review">2.0 CONTRACT REVIEW</h2>

The FRAXStablecoin contract implements several key mechanisms:
# FRAX Stablecoin Technical Contract Review
**Version:** 1.0  
**Date:** November 5, 2024  
**Contract:** FRAXStablecoin.sol

## 1. Import Analysis

### Core Ethereum Standards
```solidity
import "../Common/Context.sol";
```
- Provides `_msgSender()` and `_msgData()` functions
- Used for consistent message context handling across contracts

```solidity
import "../ERC20/IERC20.sol";
```
- Standard ERC20 interface
- Defines basic token functionality (transfer, approve, etc.)

```solidity
import "../ERC20/ERC20Custom.sol";
```
- Custom implementation of ERC20 standard
- Base contract for FRAX token functionality

```solidity
import "../ERC20/ERC20.sol";
```
- Standard ERC20 implementation
- Used as reference for basic token operations

### Utility Libraries
```solidity
import "../Math/SafeMath.sol";
```
- Prevents integer overflow/underflow
- Used in all mathematical operations within contract

### Access Control
```solidity
import "../Staking/Owned.sol";
```
- Implements basic ownership functionality
- Provides `onlyOwner` modifier

```solidity
import "../Governance/AccessControl.sol";
```
- Role-based access control system
- Manages different permission levels

### Protocol Components
```solidity
import "../FXS/FXS.sol";
```
- FXS token contract interface
- Used for protocol governance token integration

```solidity
import "./Pools/FraxPool.sol";
```
- Pool contract interface
- Handles minting/burning operations

### Oracle Integrations
```solidity
import "../Oracle/UniswapPairOracle.sol";
```
- Uniswap price oracle interface
- Used for FRAX/ETH and FXS/ETH price feeds

```solidity
import "../Oracle/ChainlinkETHUSDPriceConsumer.sol";
```
- Chainlink ETH/USD price feed consumer
- Provides external price reference

## 2. State Variables

### Core Configuration
```solidity
string public symbol;
string public name;
uint8 public constant decimals = 18;
uint256 public constant genesis_supply = 2000000e18;
```
- Basic token configuration
- Initial supply set to 2M FRAX (test value)

### Access Control
```solidity
address public creator_address;
address public timelock_address;
address public controller_address;
address public DEFAULT_ADMIN_ADDRESS;
bytes32 public constant COLLATERAL_RATIO_PAUSER;
```
- Defines key administrative addresses
- Controls system permissions

### Protocol Parameters
```solidity
uint256 public global_collateral_ratio;
uint256 public redemption_fee;
uint256 public minting_fee;
uint256 public frax_step;
uint256 public refresh_cooldown;
uint256 public price_target;
uint256 public price_band;
```
- Core economic parameters
- Controls stability mechanism

## 3. Function Analysis

### Constructor
```solidity
constructor(
    string memory _name,
    string memory _symbol,
    address _creator_address,
    address _timelock_address
) public Owned(_creator_address)
```
**Purpose:** Initializes the FRAX contract
**Operations:**
- Sets token name and symbol
- Assigns initial admin roles
- Mints genesis supply
- Sets default parameters
- Validates timelock address

### Price Oracle Functions

#### `oracle_price(PriceChoice choice)`
```solidity
function oracle_price(PriceChoice choice) internal view returns (uint256)
```
**Purpose:** Calculates USD price for FRAX or FXS
**Process:**
1. Gets ETH/USD price from Chainlink
2. Gets token/ETH price from Uniswap
3. Calculates token/USD price
**Security Considerations:**
- No oracle freshness check
- Potential price manipulation risks

#### `frax_price()`
```solidity
function frax_price() public view returns (uint256)
```
**Purpose:** Gets current FRAX/USD price
**Usage:**
- Called by stability mechanism
- Used for CR adjustments

#### `fxs_price()`
```solidity
function fxs_price() public view returns (uint256)
```
**Purpose:** Gets current FXS/USD price
**Usage:**
- Used for protocol calculations
- Informs stability decisions

### Collateral Management

#### `refreshCollateralRatio()`
```solidity
function refreshCollateralRatio() public
```
**Purpose:** Adjusts global collateral ratio
**Process:**
1. Checks cooldown period
2. Gets current FRAX price
3. Adjusts CR based on price bands
4. Updates timestamp
**Security Considerations:**
- Public access risk
- Front-running vulnerability

#### `globalCollateralValue()`
```solidity
function globalCollateralValue() public view returns (uint256)
```
**Purpose:** Calculates total collateral value
**Process:**
1. Iterates through all pools
2. Sums collateral balances
3. Returns total in USD
**Gas Optimization:**
- Consider caching array length
- Potential for gas limit issues

### Pool Management

#### `addPool(address pool_address)`
```solidity
function addPool(address pool_address) public onlyByOwnerGovernanceOrController
```
**Purpose:** Adds new pool to system
**Checks:**
- Non-zero address
- Pool not already added
**Security:**
- No pool contract validation
- Centralized control

#### `removePool(address pool_address)`
```solidity
function removePool(address pool_address) public onlyByOwnerGovernanceOrController
```
**Purpose:** Removes pool from system
**Process:**
1. Removes from mapping
2. Nullifies array entry
**Security:**
- No active pool balance check
- Potential stuck funds risk

### Token Operations

#### `pool_mint(address m_address, uint256 m_amount)`
```solidity
function pool_mint(address m_address, uint256 m_amount) public onlyPools
```
**Purpose:** Mints new FRAX tokens
**Security:**
- Only callable by pools
- Emits minting event

#### `pool_burn_from(address b_address, uint256 b_amount)`
```solidity
function pool_burn_from(address b_address, uint256 b_amount) public onlyPools
```
**Purpose:** Burns FRAX tokens
**Security:**
- Only callable by pools
- Emits burning event

### Parameter Management Functions

#### Fee Management
```solidity
function setRedemptionFee(uint256 red_fee)
function setMintingFee(uint256 min_fee)
```
**Purpose:** Updates protocol fees
**Security Issues:**
- No maximum fee limits
- No validation checks

#### Price Band Management
```solidity
function setPriceTarget(uint256 _new_price_target)
function setPriceBand(uint256 _price_band)
```
**Purpose:** Controls price stability parameters
**Security Issues:**
- No bounds checking
- Potential stability risks

### Oracle Management
```solidity
function setFRAXEthOracle(address _frax_oracle_addr, address _weth_address)
function setFXSEthOracle(address _fxs_oracle_addr, address _weth_address)
```
**Purpose:** Updates oracle addresses
**Security:**
- Address validation
- No oracle interface check



## Price Oracle System
```solidity
function oracle_price(PriceChoice choice) internal view returns (uint256)
```
- Combines Chainlink and Uniswap oracle prices
- Uses ETH as an intermediary for price calculations
- Converts prices to standard precision (1e6)

## Collateral Ratio Management
```solidity
function refreshCollateralRatio() public
```
- Adjusts collateral ratio based on FRAX price
- Uses configurable step size (default 0.25%)
- Implements cooldown period between adjustments
- Targets $1 price with configurable bands

## Pool System
```solidity
function pool_mint(address m_address, uint256 m_amount) public onlyPools
function pool_burn_from(address b_address, uint256 b_amount) public onlyPools
```
- Controlled minting/burning through whitelisted pools
- Manages global collateral tracking
- Implements fee mechanism for minting/redemption

<h2 id="findings">3.0 FINDINGS</h2>

## 3.1 Qualitative Analysis

### Critical Issues
1. **Oracle Dependency**
```solidity
function oracle_price(PriceChoice choice) internal view returns (uint256)
```
- Heavy reliance on external price feeds
- No fallback mechanism for oracle failures
- Risk of price manipulation through Uniswap pools

2. **Access Control Centralization**
```solidity
modifier onlyByOwnerGovernanceOrController()
```
- Concentrated control among few roles
- No time-delay on critical parameter changes
- Single address can pause collateral ratio

### Major Issues
1. **Pool Management Risks**
```solidity
function addPool(address pool_address) public onlyByOwnerGovernanceOrController
```
- No limit on the number of pools
- Potential gas issues in globalCollateralValue()
- No validation of pool contract code

2. **Price Band Vulnerabilities**
```solidity
uint256 public price_band;
```
- Configurable without restrictions
- Could allow extreme collateral ratio changes
- No emergency pause mechanism

## 3.2 Summary

| Issue Type | Count | Details                     |
|------------|-------|-----------------------------|
| Critical   | 2     | Oracle risks, Access control|
| Major      | 2     | Pool risks, Price mechanism |
| Minor      | 3     | Gas optimization, Documentation, Input validation |

## 3.3 Recommendations

1. **Oracle Security**
   - Implement oracle heartbeat checks
   - Add backup price sources
   - Implement circuit breakers for oracle updates

2. **Access Control**
   - Add timelock delays for critical parameter changes
   - Implement multi-signature requirements for administrative actions
   - Distribute administrative powers to enhance security

3. **Pool Safety**
   - Add pool contract validation to ensure proper functionality
   - Implement limits on the number of active pools to prevent gas issues
   - Optimize collateral calculations to reduce computational costs

<h2 id="conclusion">4.0 CONCLUSION</h2>

The FRAXStablecoin contract demonstrates a sophisticated approach to algorithmic stablecoin design. The implementation shows careful consideration of price stability mechanisms and governance controls. However, several critical issues need addressing before mainnet deployment.

**Key Strengths:**
- Well-designed collateral ratio mechanism
- Flexible pool system
- Comprehensive event logging

**Primary Concerns:**
- Oracle dependencies pose risks to stability
- Centralized control can lead to governance issues
- Gas optimization is necessary for efficient operation

**Final Recommendation:** HOLD - Requires critical fixes before deployment

**Risk Level:** HIGH

