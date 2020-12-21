# AnyStakeProtocol

## Testnet Contract
[3MxbLiDcxhjARQZ37Ec3PcoXWMcBgYV1tBh](https://testnet.wavesexplorer.com/address/3MxbLiDcxhjARQZ37Ec3PcoXWMcBgYV1tBh/tx)

## Contract Data State Specification
### Internal Asset Id
Because of key length restriction (max 100 symbols) we need to introduce _$internalBaseAssetId_ and bidirectional mappings between it and real assetId

Num| Key           | Type        | Optional? | Format and Description |
---| ------------- |------------ |---------- | ---------------------- |
1  | `%s%s%s__mappings__internal2baseAssetId__${internalBaseAssetId}`|String|No|Simple|
2  | `%s%s%s__mappings__baseAsset2internalId__${baseAssetId}`|String|No|Simple|

### Config Data State
Version_#0001

Type | Format   |
---- | ---- |
Key  | `%s%s%s__config__asset__${realBaseAssetId}` |
Value| `%s%s%d%d%d__${shareAssetId}__${internalBaseAssetId}__${decimalsMultBothTokens}__${decimalsMultPrice}__${getDelayBlocks}` |

### PUT operation
Version_#0001

Type | Format   |
---- | ---- |
Key  | `%s%s%s%s__P__${internalBaseAssetId}__${userAddress}__${txId}` |
Value| `%s%d%d%d%d%d%d%d__${status}__${inBaseTokensAmount}__${price}__${outShareTokensAmount}__${startHeight}__${startTimestamp}__${endHeight}__${endTimestamp}` |

Assumptions:
* ${status} - always FINISHED
* ${startHeight} == ${endHeight}
* ${startTimestamp} == ${endTimestamp}

### GetRequest operation
Version_#0001

Type | Format   |
---- | ---- |
Key  | `%s%s%s%s__G__${internalBaseAssetId}__${userAddress}__${txId}` |
Value| `%s%d%d%d%d%d%d%d__${status}__${inShareTokensAmount}__${price}__${outBaseTokensAmount}__${startHeight}__${startTimestamp}__${endHeight}__${endTimestamp}` |

### Other information in state
Num| Key           | Type        | Optional? | Format | Description |
---| ------------- |------------ |---------- | -------| ------------|
1|`%s%s%s__mappings__share2baseAssetId__${shareAssetId}`|String|NO|Simple|mapping between ${shareAssetId} and real ${baseAssetId}|
2|`%s%s%s__mappings__baseAsset2shareId__${baseAssetId}` |String|NO|Simple|mapping between ${baseAssetId} and real ${shareAssetId}|
3|`%s%s%s%s__total__locked__${internalBaseAssetId}__${userAddress}`|String|YES|`%d%d__${sharedTokenAmount}__${baseTokenAmount}`||
4|`%s%s%s__total__locked__${internalBaseAssetId}`|String|NO|`%d%d__${sharedTokenAmount}__${baseTokenAmount}`||
5|`%s%s%s%d%d__price__history__${intrenalBaseAssetId}__${height}__${timestamp}`|Integer|YES|Simple|It is difficult to use ${decimalsMult} from asset config because assets with 0 decimals will have bad price (300/200=1.5 ~ 1) ${decimalsMultPrice} has been introduce to resolve this problem|
6|`%s%s%s__price__last__${internalBaseAssetId}`|Integer|NO|Simple||
