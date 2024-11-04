Client: FRAX Protocol
Title: Smart Contract Audit Report
Target: FRAXStablecoin
Version: 1.0
Classification: Public
Status: Draft
Date Created: November 4, 2024

# Table of contents
1. INTRODUCTION
   1.1 Disclaimer
   1.2 About Me
   1.3 Skills  
   1.4 Links
   1.5 FRAX Protocol
   1.6 FRAXStablecoin
   1.7 Scope
   1.8 Roles
   1.9 System Overview

2. CONTRACT REVIEW
3. FINDINGS
   3.1 Qualitative Analysis
   3.2 Summary
   3.3 Recommendations
4. CONCLUSION

# 1.5 FRAX Protocol
FRAX is a fractional-algorithmic stablecoin protocol that aims to maintain a price target of $1. The protocol uses a dynamic collateral ratio system that adjusts based on market conditions, combining both collateralized and algorithmic mechanisms to maintain stability.

# 1.6 FRAXStablecoin
The FRAXStablecoin contract is the core implementation of the FRAX protocol's stablecoin. It inherits from ERC20Custom and implements a sophisticated price stability mechanism through dynamic collateral ratios and a pool-based minting/burning system.

# 1.7 Scope
(Table: 1.7: FRAX Stablecoin Audit Scope)

Files in scope | SLOC
---------------|------
Contracts: 1 | 
FRAXStablecoin.sol | 450
Imports: 11 |
Context.sol | -
IERC20.sol | -
ERC20Custom.sol | -
ERC20.sol | -
SafeMath.sol | -
Owned.sol | -
FXS.sol | -
FraxPool.sol | -
UniswapPairOracle.sol | -
ChainlinkETHUSDPriceConsumer.sol | -
AccessControl.sol | -

# 1.8 Roles

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

# 1.9 System Overview

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

# 2.0 CONTRACT REVIEW

The FRAXStablecoin contract implements several key mechanisms:

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

# 3.0 FINDINGS

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
- No limit on number of pools
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

Issue Type | Count | Details
-----------|--------|----------
Critical | 2 | Oracle risks, Access control
Major | 2 | Pool risks, Price mechanism
Minor | 3 | Gas optimization, Documentation, Input validation

## 3.3 Recommendations

1. **Oracle Security**
- Implement oracle heartbeat checks
- Add backup price sources
- Implement circuit breakers

2. **Access Control**
- Add timelock delays
- Implement multi-signature requirements
- Distribute admin powers

3. **Pool Safety**
- Add pool contract validation
- Implement pool limits
- Optimize collateral calculations

# 4.0 CONCLUSION

The FRAXStablecoin contract demonstrates a sophisticated approach to algorithmic stablecoin design. The implementation shows careful consideration of price stability mechanisms and governance controls. However, several critical issues need addressing before mainnet deployment.

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

This concludes the security review of the FRAXStablecoin smart contract.
