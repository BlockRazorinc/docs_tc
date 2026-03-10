# 項目方

### RPC端點

{% hint style="info" %}
項目方可發送至[項目專屬RPC端點](../../rpc/ethereum/ji-cheng-rpc.md)
{% endhint %}

https://eth.blockrazor.xyz



### 請求參數

<table><thead><tr><th width="184">參數</th><th width="74">必選</th><th width="105">格式</th><th width="177.79296875">示例</th><th>備註</th></tr></thead><tbody><tr><td>txs</td><td>是</td><td>[]bytes</td><td>[ "0xf84a……e54284" ]</td><td>raw txs，最多允許設置50筆</td></tr><tr><td>revertingTxHashes</td><td>否</td><td>[]hash</td><td>["0x1f23……0abb1e"]</td><td>允許revert的交易哈希，是txs的子集</td></tr><tr><td>maxBlockNumber</td><td>否</td><td>uint64</td><td>39177941</td><td>該bundle有效的最大區塊號，默認為當前區塊號+100</td></tr></tbody></table>



### 請求示例

```json
curl -X POST -H "Content-Type: application/json" --data '{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "eth_sendBundle",
  "params": [{
    "Txs": [
"0xf8668203988405f5e100825208942ee393c739036a7660ec11bf2101d537eb52f3ac80808193a06836d5f4052376dc3114794da5fedd7a5b8090ddae0ec45dfa66c234fcabb6efa07cf17dad992c33e8e3e4d82355cfe12d3be2560fc2b0873c36dce398088d8e4f"
    ]
    "revertingTxHashes":[],
    "maxBlockNumber":62934913
  }]
}' https://eth.blockrazor.xyz
```



### 返回示例

正常

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x11111111..."
}
```

異常

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "jsonerror": {
	  "code": -38000,
	  "message": "nonce too low: address 0x9Abae1b279A4Be25AEaE49a33e807cDd3cCFFa0C, tx: 0 state: 45"
  }
}
```

