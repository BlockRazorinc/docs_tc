# Send Bundle

### 接口說明

本接口用於接收用戶提交的bundle，方法名為`eth_sendBundle`。&#x20;

BlockRazor MEV服務的區塊構建算法傾向於向BlockRazor Builder EOA轉賬原生代幣（BNB）數量更多的bundle，當前BlockRazor Builder EOA地址為0x1266C6bE60392A8Ff346E8d5ECCd3E69dD9c5F20。\
bundle中交易的gas price需不小於BSC Validator要求的最低標準（當前為0.05 gwei）。

對於Tier 2 / Tier 1 / Tier0用戶，可以在bundle中包含0 gwei的交易，但bundle中交易（public mempool中的交易除外）的平均gasPrice仍需不小於0.05 gwei。



### 流控說明

<table><thead><tr><th width="212"></th><th width="132.26953125">Tier 4</th><th>Tier 3</th><th width="135.984375">Tier 2</th><th>Tier 1</th><th>Tier0</th></tr></thead><tbody><tr><td>BPS</td><td>不限</td><td>不限</td><td>不限</td><td>不限</td><td>不限</td></tr><tr><td>每區塊可接受bundle數</td><td>不限</td><td>不限</td><td>不限</td><td>不限</td><td>不限</td></tr><tr><td>bundle包含0 gwei交易</td><td>-</td><td>-</td><td>支持</td><td>支持</td><td>支持</td></tr></tbody></table>



### 請求參數

<table><thead><tr><th width="163">參數</th><th width="85.73046875">必選</th><th width="136">格式</th><th width="124">示例</th><th>描述</th></tr></thead><tbody><tr><td>txs</td><td>是</td><td>array[hex]</td><td>["0x…4b", "0x…5c"]</td><td>經過簽名的raw transaction列表</td></tr><tr><td>maxBlockNumber</td><td>否</td><td>uint64</td><td>39177941</td><td>該bundle有效的最大區塊號，默認為當前區塊號+100</td></tr><tr><td>minTimestamp</td><td>否</td><td>uint64</td><td>1710229370</td><td>期望bundle有效的最小Unix秒級時間戳</td></tr><tr><td>maxTimestamp</td><td>否</td><td>uint64</td><td>1710829390</td><td>期望bundle有效的最大Unix秒級時間戳</td></tr><tr><td>revertingTxHashes</td><td>否</td><td>array[hash]</td><td>["0x…c7", "0x…b7"]</td><td>允許revert的交易哈希列表</td></tr><tr><td>noMerge</td><td>否</td><td>bool</td><td>false</td><td>bundle merge可提升區塊價值，加快bundle上鏈速度，如不設置則默認為false（允許bundle merge）</td></tr><tr><td>positionFirst</td><td>否</td><td>bool</td><td>false</td><td>嚴格按優先費用對bundle排序。若該項设置为false（默認），則到達较晚的bundle可能被添加至區塊尾部区域，以保證其在當前區塊成功上鏈。</td></tr></tbody></table>



### 請求示例

{% tabs %}
{% tab title="HTTPS" %}
```bash
curl https://virginia.builder.blockrazor.io \
  -H 'content-type: application/json' \
  -H 'Authorization: <auth-token>' \
  --data '{
    "jsonrpc": "2.0",
    "id": "1",
    "method": "eth_sendBundle",
    "params": [{
      "txs": [
        "0x...4b",
        "0x...5c"
      ],
      "maxBlockNumber": 39177941,
      "minTimestamp": 1710229370,
      "maxTimestamp": 1710829390,
      "revertingTxHashes": [
        "0x44b89abe860142d3c3bda789cf955b69ba00b71882cd968ec407a70f4719ff06",
        "0x7d7652c685e9fda4fe2e41bad017519cffeed8ba03d59aa6401284be2ec4244c"
      ],
      "noMerge": false
    }]
  }'
```
{% endtab %}
{% endtabs %}



### 返回示例

```json
{
 "jsonrpc":"2.0",
 "id":"1",
 "result":"0xa06b……f7e8ec" //bundle哈希
}‍
```

```json
{
  "jsonrpc":"2.0",
  "id":"1",
  "error":{
    "code":-38000,
    "message":"the maxBlockNumber should not be smaller than currentBlockNum"
    }
}
```



