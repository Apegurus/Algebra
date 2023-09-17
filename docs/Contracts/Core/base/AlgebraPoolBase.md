

# AlgebraPoolBase


Algebra pool base abstract contract

Contains state variables, immutables and common internal functions

*Developer note: Decoupling into a separate abstract contract simplifies testing*

## Modifiers
### onlyValidTicks

```solidity
modifier onlyValidTicks(int24 bottomTick, int24 topTick)
```

Check that the lower and upper ticks do not violate the boundaries of allowed ticks and are specified in the correct order

| Name | Type | Description |
| ---- | ---- | ----------- |
| bottomTick | int24 |  |
| topTick | int24 |  |


## Structs
### GlobalState

The struct with important state values of pool

*Developer note: fits into one storage slot*

```solidity
struct GlobalState {
  uint160 price;
  int24 tick;
  uint16 lastFee;
  uint8 pluginConfig;
  uint16 communityFee;
  bool unlocked;
}
```


## Variables
### uint128 maxLiquidityPerTick constant

The maximum amount of position liquidity that can use any tick in the range

*Developer note: This parameter is enforced per tick to prevent liquidity from overflowing a uint128 at any point, and
also prevents out-of-range liquidity from being used to prevent adding in-range liquidity to a pool*

### address factory immutable

The Algebra factory contract, which must adhere to the IAlgebraFactory interface


### address token0 immutable

The first of the two tokens of the pool, sorted by address


### address token1 immutable

The second of the two tokens of the pool, sorted by address


### address communityVault immutable

The contract to which community fees are transferred


### uint256 totalFeeGrowth0Token 

The fee growth as a Q128.128 fees of token0 collected per unit of liquidity for the entire life of the pool

*Developer note: This value can overflow the uint256*

### uint256 totalFeeGrowth1Token 

The fee growth as a Q128.128 fees of token1 collected per unit of liquidity for the entire life of the pool

*Developer note: This value can overflow the uint256*

### struct AlgebraPoolBase.GlobalState globalState 

The globalState structure in the pool stores many values but requires only one slot
and is exposed as a single method to save gas when accessed externally.

*Developer note: **important security note: caller should check &#x60;unlocked&#x60; flag to prevent read-only reentrancy***

### mapping(int24 &#x3D;&gt; struct TickManagement.Tick) ticks 

Look up information about a specific tick in the pool

*Developer note: **important security note: caller should check reentrancy lock to prevent read-only reentrancy***

### uint32 communityFeeLastTimestamp 

The timestamp of the last sending of tokens to community vault


### address plugin 

Returns the address of currently used plugin

*Developer note: The plugin is subject to change*

### mapping(int16 &#x3D;&gt; uint256) tickTable 

Returns 256 packed tick initialized boolean values. See TickTree for more information


### int24 nextTickGlobal 

The next initialized tick after current global tick

*Developer note: **important security note: caller should check reentrancy lock to prevent read-only reentrancy***

### int24 prevTickGlobal 

The previous initialized tick before (or at) current global tick

*Developer note: **important security note: caller should check reentrancy lock to prevent read-only reentrancy***

### uint128 liquidity 

The currently in range liquidity available to the pool

*Developer note: This value has no relationship to the total liquidity across all ticks.
Returned value cannot exceed type(uint128).max
**important security note: caller should check reentrancy lock to prevent read-only reentrancy***

### int24 tickSpacing 

The current tick spacing

*Developer note: Ticks can only be initialized by new mints at multiples of this value
e.g.: a tickSpacing of 60 means ticks can be initialized every 60th tick, i.e., ..., -120, -60, 0, 60, 120, ...
However, tickspacing can be changed after the ticks have been initialized.
This value is an int24 to avoid casting even though it is always positive.*


## Functions
### safelyGetStateOfAMM

```solidity
function safelyGetStateOfAMM() external view returns (uint160 sqrtPrice, int24 tick, uint16 lastFee, uint8 pluginConfig, uint128 activeLiquidity, int24 nextTick, int24 previousTick)
```

Safely get most important state values of Algebra Integral AMM

*Developer note: safe from read-only reentrancy getter function*

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| sqrtPrice | uint160 | The current price of the pool as a sqrt(dToken1/dToken0) Q64.96 value |
| tick | int24 | The current global tick of the pool. May not always be equal to SqrtTickMath.getTickAtSqrtRatio(price) if the price is on a tick boundary |
| lastFee | uint16 | The current (last known) pool fee value in hundredths of a bip, i.e. 1e-6 (so '100' is '0.01%'). May be obsolete if using dynamic fee plugin |
| pluginConfig | uint8 | The current plugin config as bitmap. Each bit is responsible for enabling/disabling the hooks, the last bit turns on/off dynamic fees logic |
| activeLiquidity | uint128 | The currently in-range liquidity available to the pool |
| nextTick | int24 | The next initialized tick after current global tick |
| previousTick | int24 | The previous initialized tick before (or at) current global tick |

### isUnlocked

```solidity
function isUnlocked() external view returns (bool unlocked)
```

Allows to easily get current reentrancy lock status

*Developer note: can be used to prevent read-only reentrancy.
This method just returns &#x60;globalState.unlocked&#x60; value*

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| unlocked | bool | Reentrancy lock flag, true if the pool currently is unlocked, otherwise - false |

### getCommunityFeePending

```solidity
function getCommunityFeePending() external view returns (uint128, uint128)
```

The amounts of token0 and token1 that will be sent to the vault

*Developer note: Will be sent COMMUNITY_FEE_TRANSFER_FREQUENCY after communityFeeLastTimestamp*

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| [0] | uint128 |  |
| [1] | uint128 |  |

### fee

```solidity
function fee() external view returns (uint16 currentFee)
```

The current pool fee value

*Developer note: In case dynamic fee is enabled in the pool, this method will call the plugin to get the current fee.
If the plugin implements complex fee logic, this method may return an incorrect value or revert.
In this case, see the plugin implementation and related documentation.
**important security note: caller should check reentrancy lock to prevent read-only reentrancy***

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| currentFee | uint16 | The current pool fee value in hundredths of a bip, i.e. 1e-6 |

