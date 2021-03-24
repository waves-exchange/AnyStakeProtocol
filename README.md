# AnyStakeProtocol
Any Stake Protocol is designed to unify callable methods and data state between different 'staking-like' dapps

## Table of contents
- [AnyStake Interface](#anystake-interface)
- [AnyStake Data State Specification](#anystake-data-state-specification)
- [LP Product Implementation](lp-product-implementation)
- [Algo Product Implementation](algo-product-implementation)

## AnyStake Interface


## AnyStake Data State Specification
#### Internal Asset Id
Because of key length restriction (max 100 symbols) we need to introduce _$internalBaseAssetId_ and bidirectional mappings between it and real assetId

Num| Key           | Type        | Optional? | Format and Description |
---| ------------- |------------ |---------- | ---------------------- |
1  | `%s%s%d__mappings__internal2baseAssetId__${internalBaseAssetId}`|String|No|Simple|
2  | `%s%s%s__mappings__baseAsset2internalId__${baseAssetId}`|String|No|Simple|

#### Config Data State
Version_#0001

Type | Format   |
---- | ---- |
Key  | `%s%s%s__config__asset__${realBaseAssetId}` |
Value| `%s%d%d%d%d__${shareAssetId}__${internalBaseAssetId}__${decimalsMultBothTokens}__${decimalsMultPrice}__${getDelayBlocks}__...extensions...` |

#### submitPut operation
Version_#0001

Type | Format   |
---- | ---- |
Key  | `%s%d%s%s__P__${internalBaseAssetId}__${userAddress}__${txId}` |
Value| `%s%d%d%d%d%d%d%d__${status}__${inBaseTokensAmount}__${price}__${outShareTokensAmount}__${startHeight}__${startTimestamp}__${endHeight}__${endTimestamp}` |


#### submitGet operation
Version_#0001

Type | Format   |
---- | ---- |
Key  | `%s%d%s%s__G__${internalBaseAssetId}__${userAddress}__${txId}` |
Value| `%s%d%d%d%d%d%d%d__${status}__${inShareTokensAmount}__${price}__${outBaseTokensAmount}__${startHeight}__${startTimestamp}__${endHeight}__${endTimestamp}` |

#### Other information in state
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

#### PUT operation assumptions
It is always one step operation. (No need to in claim/execute/withdraw by user)
* ${status} - always FINISHED
* ${startHeight} == ${endHeight}
* ${startTimestamp} == ${endTimestamp}

#### Mainnet Contract
* Dapp: `3P6SFR9ZZwKHZw5mMDZxpXHEhg1CXjBb51y` [wavesexplorer](https://wavesexplorer.com/address/3P6SFR9ZZwKHZw5mMDZxpXHEhg1CXjBb51y/tx) OR [w8io](https://w8io.ru/3P6SFR9ZZwKHZw5mMDZxpXHEhg1CXjBb51y)
  * USDTLP token id: `9AT2kEi8C4AYxV1qKxtQTVpD5i54jCPvaNNRP6VzRtYZ` [wavesexplorer](https://wavesexplorer.com/assets/9AT2kEi8C4AYxV1qKxtQTVpD5i54jCPvaNNRP6VzRtYZ) OR  [w8io](https://w8io.ru/9AT2kEi8C4AYxV1qKxtQTVpD5i54jCPvaNNRP6VzRtYZ)
  * USDCLP token id: `CrjhbC9gRezwvBZ1XwPQqRwx4BWzoyMHGFNUVdn22ep6` [wavesexplorer](https://wavesexplorer.com/assets/CrjhbC9gRezwvBZ1XwPQqRwx4BWzoyMHGFNUVdn22ep6) OR  [w8io](https://w8io.ru/CrjhbC9gRezwvBZ1XwPQqRwx4BWzoyMHGFNUVdn22ep6)
  * BTCLP token id: `DazN41oAedqwGZ8aabf4nJQwJNZhsEgPH3YQWDtPsdeV` [wavesexplorer](https://wavesexplorer.com/assets/DazN41oAedqwGZ8aabf4nJQwJNZhsEgPH3YQWDtPsdeV) OR  [w8io](https://w8io.ru/DazN41oAedqwGZ8aabf4nJQwJNZhsEgPH3YQWDtPsdeV)
  * ETHLP token id: `ELzXTgPa6GGYyLtitn2oWDWQ9joyTFEueNtF4kxg95dM` [wavesexplorer](https://wavesexplorer.com/assets/ELzXTgPa6GGYyLtitn2oWDWQ9joyTFEueNtF4kxg95dM) OR  [w8io](https://w8io.ru/ELzXTgPa6GGYyLtitn2oWDWQ9joyTFEueNtF4kxg95dM)

#### Testnet Contract
* [3Mzt645zA6u2QG6jRPoo6H6CK89kVggFgNi](https://testnet.wavesexplorer.com/address/3Mzt645zA6u2QG6jRPoo6H6CK89kVggFgNi/tx)

## Algo Product Implementation