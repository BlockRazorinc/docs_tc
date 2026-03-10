# Call Bundle

### 接口說明

本接口用於接收用戶提交的模擬bundle的請求，方法名為`eth_callBundle`。



### 流控说明

<table><thead><tr><th width="176.4296875"></th><th width="99.47265625">Tier 4</th><th width="113.46484375">Tier 3</th><th width="94.48046875">Tier 2</th><th width="104.7421875">Tier 1</th><th width="117.92578125">Tier 0</th></tr></thead><tbody><tr><td><code>eth_callBundle</code></td><td>-</td><td>支持</td><td>支持</td><td>支持</td><td>支持</td></tr></tbody></table>



### 请求参数

<table><thead><tr><th width="159">參數</th><th width="62">必選</th><th width="113">格式</th><th>示例</th><th>描述</th></tr></thead><tbody><tr><td>txs</td><td>是</td><td>array[hex]</td><td>["0x…4b", "0x…5c"]</td><td>經過簽名的raw transaction列表</td></tr><tr><td>blockNumber</td><td>是</td><td>number</td><td>39177941</td><td>當前區塊號+1</td></tr></tbody></table>



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
  "method": "eth_callBundle",
  "params": [
    {
      "txs":["0x…4b"],
      "blockNumber":39177941
    }
  ]
  }'
```
{% endtab %}
{% endtabs %}



## 返回示例

#### **正常**

{% tabs %}
{% tab title="HTTPS" %}
```json
{  
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "bundleHash": "0xb13bb92ebee57b42b7e8e91b41891e1693a449dcdb8530c6bffe684157e988da",
    "ethSentToBuilder": "63003000000000",
    "gasFees": "21000000000000",
    "results": [
      {
        "ethSentToBuilder": "63003000000000",
        "fromAddress": "0x12994B3004Daab21035EDa1D4b31F7F63D606128",
        "gasFees": "21000000000000",
        "gasPrice": "4000142857",
        "gasUsed": 21000,
        "toAddress": "0x1266C6bE60392A8Ff346E8d5ECCd3E69dD9c5F20",
        "txHash": "0x0fbaa913aa0ffc22bba63a81ca2958ad14d630928c5a5240130fec7037605648",
        "value": "0x"
      }
    ],
    "stateBlockNumber": 756983,
    "totalGasUsed": 21000
  }
}
```
{% endtab %}
{% endtabs %}



#### 异常

**auth token缺失**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "missing auth token"
  }
}
```

**觸發限流**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "rate limit exceeded, try again later"
  }
}
```

**nonce错误**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "err: nonce too low: address 0x6c85F133fa06Fe5eb185743FB6c79f4a7cb9C076, tx: 28 state: 29; txhash 0x300c95d2b3086ddc1836de1e8c87878916d332ff42e8312cd404264ab8cdcd18"
  }
}
```

**执行失败**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "bundleHash": "0x592da70a510720d149567bfbc5935a05780963607c3c75b93a9190bb60818f21",
    "ethSentToBuilder": "0",
    "gasFees": "218070000000000",
    "results": [
      {
        "error": "execution reverted",
        "ethSentToBuilder": "0",
        "fromAddress": "0xb0b10B09780aa6A315158EF724404aa1497e9E6E",
        "gasFees": "218070000000000",
        "gasPrice": "10000000000",
        "gasUsed": 21807,
        "revert": "unlock error: locked",
        "toAddress": "0xdA51Cf6ed22740FD8fAfbBe61577A577915e7526",
        "txHash": "0xa09745617e0a212c5da379ea066f64e33f5215ce235d653bbbc1359b38d78baa",
      }
    ],
    "stateBlockNumber": 29115,
    "totalGasUsed": 21807
  }
}
```



