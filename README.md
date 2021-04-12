# AnyStakeProtocol
Any Stake Protocol is designed to unify callable methods and data state between different 'staking-like' dapps

## Table of contents
- [AnyStake Interface](#anystake-interface)
- [AnyStake Data State Specification](#anystake-data-state-specification)
- Products
    - [LP Product Implementation](#lp-product-implementation)
    - [ALGO Product Implementation](#algo-product-implementation)
    - [LAMBO Product Implementation](#lambo-product-implementation)

## AnyStake Interface
```python
@Callable(i)
func adminRegisterAsset() = {}
 
# payment in $baseAssetId
@Callable(i)
func submitPut() = {}
 
# payment in $shareAssetId
@Callable(i)
func submitGet() = {}
 
@Callable(i)
func executePut(baseTokenStr: String, userAddressStr: String, txId: String) = {}
 
@Callable(i)
func executeGet(baseTokenStr: String, userAddressStr: String, txId: String) = {}
 
# if amount > 0 then payment in $baseAssetId
# if amount < 0 then empty payments
# if amount = 0 then error
@Callable(i)
func topUpBalance(baseAssetId: String, amount: Int) = {}
 
@Callable(i)
func currentSysParamsREST(baseTokenId: String)={}
# includes current price
```

## AnyStake Data State Specification
#### Internal Asset Id
Because of key length restriction (max 100 symbols) we need to introduce _$internalBaseAssetId_ and bidirectional mappings between it and real assetId

Num| Key           | Type        | Optional? | Format and Description |
---| ------------- |------------ |---------- | ---------------------- |
1  | `%s%s%d__mappings__internal2baseAssetId__${internalBaseAssetId}`|String|No|Simple|
2  | `%s%s%s__mappings__baseAsset2internalId__${baseAssetId}`|String|No|Simple|

#### Config V_01

Type | Format   |
---- | ---- |
Key  | `%s%s%s__config__asset__${realBaseAssetId}` |
Value| `%s%d%d%d%d__${shareAssetId}__${internalBaseAssetId}__${decimalsMultBothTokens}__${decimalsMultPrice}__${getDelayBlocks}__...dataExtensions...` |

Extensions:
* [ALGO config](#algo-config)

#### submitPut operation V_01

Type | Format   |
---- | ---- |
Key  | `%s%d%s%s__P__${internalBaseAssetId}__${userAddress}__${txId}` |
Value| `%s%d%d%d%d%d%d%d__${status}__${inBaseTokensAmount}__${price}__${outShareTokensAmount}__${startHeight}__${startTimestamp}__${endHeight}__${endTimestamp}...dataExtensions...` |

Extensions:
* [LP submitPut](#lp-submitput-and-submitget)
* [ALGO submitPut](#algo-submitput)

#### submitGet operation V_01

Type | Format   |
---- | ---- |
Key  | `%s%d%s%s__G__${internalBaseAssetId}__${userAddress}__${txId}` |
Value| `%s%d%d%d%d%d%d%d__${status}__${inShareTokensAmount}__${price}__${outBaseTokensAmount}__${startHeight}__${startTimestamp}__${endHeight}__${endTimestamp}...dataExtensions...` |

Extensions:
* [LP submitGet](#lp-submitput-and-submitget)
* [ALGO submitGet](#algo-submitget)

#### Other information
Num| Key           | Type        | Optional? | Format | Description |
---| ------------- |------------ |---------- | -------| ------------|
1|`%s%s%s__mappings__share2baseAssetId__${shareAssetId}`|String|NO|Simple|mapping between ${shareAssetId} and real ${baseAssetId}|
2|`%s%s%s__mappings__baseAsset2shareId__${baseAssetId}` |String|NO|Simple|mapping between ${baseAssetId} and real ${shareAssetId}|
3|`%s%s%d%s__total__locked__${internalBaseAssetId}__${userAddress}`|String|YES|`%d%d__${sharedTokenAmount}__${baseTokenAmount}`||
4|`%s%s%d__total__locked__${internalBaseAssetId}`|String|NO|`%d%d__${sharedTokenAmount}__${baseTokenAmount}`||
5|`%s%s%d%d%d__price__history__${intrenalBaseAssetId}__${height}__${timestamp}`|Integer|YES|Simple|It is difficult to use ${decimalsMult} from asset config because assets with 0 decimals will have bad price (300/200=1.5 ~ 1) ${decimalsMultPrice} has been introduce to resolve this problem|
6|`%s%s%d__price__last__${internalBaseAssetId}`|Integer|NO|Simple||
7|`%s%s%d__shutdown__put__${internalBaseAssetId}`|Boolean|Yes|Simple. If true then put operation is blocked||


## LP Product Implementation

### Mainnet Contract
* Dapp: `3P6SFR9ZZwKHZw5mMDZxpXHEhg1CXjBb51y` [wavesexplorer](https://wavesexplorer.com/address/3P6SFR9ZZwKHZw5mMDZxpXHEhg1CXjBb51y/tx) OR [w8io](https://w8io.ru/3P6SFR9ZZwKHZw5mMDZxpXHEhg1CXjBb51y)
  * USDTLP token id: `9AT2kEi8C4AYxV1qKxtQTVpD5i54jCPvaNNRP6VzRtYZ` [wavesexplorer](https://wavesexplorer.com/assets/9AT2kEi8C4AYxV1qKxtQTVpD5i54jCPvaNNRP6VzRtYZ) OR  [w8io](https://w8io.ru/9AT2kEi8C4AYxV1qKxtQTVpD5i54jCPvaNNRP6VzRtYZ)
  * USDCLP token id: `CrjhbC9gRezwvBZ1XwPQqRwx4BWzoyMHGFNUVdn22ep6` [wavesexplorer](https://wavesexplorer.com/assets/CrjhbC9gRezwvBZ1XwPQqRwx4BWzoyMHGFNUVdn22ep6) OR  [w8io](https://w8io.ru/CrjhbC9gRezwvBZ1XwPQqRwx4BWzoyMHGFNUVdn22ep6)
  * BTCLP token id: `DazN41oAedqwGZ8aabf4nJQwJNZhsEgPH3YQWDtPsdeV` [wavesexplorer](https://wavesexplorer.com/assets/DazN41oAedqwGZ8aabf4nJQwJNZhsEgPH3YQWDtPsdeV) OR  [w8io](https://w8io.ru/DazN41oAedqwGZ8aabf4nJQwJNZhsEgPH3YQWDtPsdeV)
  * ETHLP token id: `ELzXTgPa6GGYyLtitn2oWDWQ9joyTFEueNtF4kxg95dM` [wavesexplorer](https://wavesexplorer.com/assets/ELzXTgPa6GGYyLtitn2oWDWQ9joyTFEueNtF4kxg95dM) OR  [w8io](https://w8io.ru/ELzXTgPa6GGYyLtitn2oWDWQ9joyTFEueNtF4kxg95dM)

### Testnet Contract
* [3Mzt645zA6u2QG6jRPoo6H6CK89kVggFgNi](https://testnet.wavesexplorer.com/address/3Mzt645zA6u2QG6jRPoo6H6CK89kVggFgNi/tx)

### LP submitPut and submitGet
#### Data State Extensions
No extensions, see corresponding
* [submitPut V_01](#submitput-operation-v_01)
* [submitGet V_01](#submitget-operation-v_01)

#### Assumptions
* both one step operations (No need to send claim/execute/withdraw by user)
* ${status} - always FINISHED
* ${startHeight} == ${endHeight}
* ${startTimestamp} == ${endTimestamp}


## ALGO Product Implementation
### Mainnet Contract
* Dapp: `3PKaRXj6pBb23A8k965eEgBAhhJ4FSDKS5e` [wavesexplorer](https://wavesexplorer.com/address/3PKaRXj6pBb23A8k965eEgBAhhJ4FSDKS5e/tx) OR [w8io](https://w8io.ru/3PKaRXj6pBb23A8k965eEgBAhhJ4FSDKS5e)
  * AUSDTLPM token id: `3iAUM9xnKdu2TWBXRqPvYoGsbiZEbDPP1okUf5v9RJNz` [wavesexplorer](https://wavesexplorer.com/assets/3iAUM9xnKdu2TWBXRqPvYoGsbiZEbDPP1okUf5v9RJNz) OR  [w8io](https://w8io.ru/3iAUM9xnKdu2TWBXRqPvYoGsbiZEbDPP1okUf5v9RJNz)


### Testnet Contract
* [3NC9wWawxuFG6a3sZdfckGwoMeVhLFjZFwH](https://testnet.wavesexplorer.com/address/3NC9wWawxuFG6a3sZdfckGwoMeVhLFjZFwH/tx)
### ALGO config
#### Data State Extensions
Type | Format   |
---- | ---- |
Key  | [Config V_01](#config-v_01).key |
Value| [Config V_01](#config-v_01).val `__${TopupIntervalInBlocks}__${TopupMaxNegativePart}__${TopupManagerAddress}__${SubmitLimitsBaseMax}__${SubmitLimitsBaseReset}__${SubmitLimitsShareMax}__${SubmitLimitsShareReset}` |

### ALGO submitPut

#### Data State Extensions
Type | Format   |
---- | ---- |
Key  | [submitPut V_01](#submitput-operation-v_01).key |
Value| [submitPut V_01](#submitput-operation-v_01).val`__${topUpIdxUnlock}` |

#### Assumptions
* two step operation
* the following data is available only after corresponding `executeXxx` call
   * `${endHeight}`
   * `${endTimestamp}`
   * `${price}`
   * `${outShareTokensAmount}`

### ALGO submitGet

#### Data State Extensions
Type | Format   |
---- | ---- |
Key  | [submitGet V_01](#submitget-operation-v_01).key |
Value| [submitGet V_01](#submitget-operation-v_01).val`__${topUpIdxUnlock}` |

#### Assumptions
* two step operation
* the following data is available only after corresponding `executeXxx` call
   * `${endHeight}`
   * `${endTimestamp}`
   * `${price}`
   * `${outBaseTokensAmount}`

## LAMBO Product Implementation

### Mainnet Contract
* Dapp: `3P3hCvE9ZfeMnZE6kXzR6YBzxhxM8J6PE7K` [wavesexplorer](https://wavesexplorer.com/address/3P3hCvE9ZfeMnZE6kXzR6YBzxhxM8J6PE7K/tx) OR [w8io](https://w8io.ru/3P3hCvE9ZfeMnZE6kXzR6YBzxhxM8J6PE7K)
  * USDTLAMBO token id: `TBD` [wavesexplorer](https://wavesexplorer.com/assets/TBD) OR  [w8io](https://w8io.ru/TBD)

### Testnet Contract
* [3NAkRz8VVS1aizMWiZ7Hxs3N9vrfhpR6579](https://testnet.wavesexplorer.com/address/3NAkRz8VVS1aizMWiZ7Hxs3N9vrfhpR6579/tx)

### LAMBO config
same as [ALGO config](#algo-config)

### LAMBO submitPut
same as [ALGO submitPut](#algo-submitput)

### LAMBO submitGet
with one exception as [ALGO submitGet](#algo-submitget):
* `${endHeight}` is available right away after `submitGet` and calculated as `${currentBlockchainHeight} + .${getDelayBlocks}` (see [Config V_01](#config-v_01)). It is used to block operation till this height.
