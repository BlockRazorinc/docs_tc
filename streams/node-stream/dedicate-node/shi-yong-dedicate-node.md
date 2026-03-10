# 使用Dedicate Node

### 1. 獲取端點

<figure><img src="../../../.gitbook/assets/image (63).png" alt=""><figcaption><p><a href="https://www.blockrazor.io/#/login">登錄</a>控制台，前往【Dedicate Node】獲取端點URL</p></figcaption></figure>

### 2. 調用JSON RPC方法

#### 標準JSON RPC方法

BSC Dedicate Node（Geth）的標準JSON RPC方法於Ethereum Geth完全兼容，具體調用方法請參考[https://ethereum.org/en/developers/docs/apis/json-rpc/#json-rpc-methods](https://ethereum.org/en/developers/docs/apis/json-rpc/#json-rpc-methods)



#### 定製JSON RPC方法

BSC Dedicate Node（Geth）支持bundle模擬，方法名`eth_simulateBundles`，請求和返回示例如下：

**請求**

```
curl -X POST <dedicate node url> \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_simulateBundles",
    "params": [
      [
        {
          "from": "0x03Ae1e3082dD5E338e8Ae8572A34bcdF8be40362",
          "to": "0x55d398326f99059fF775485246999027B3197955",
          "data": "0x18160ddd",
          "gas": "0x100000"
        }
      ],
      "latest" // 指定模擬的區塊高度或狀態，latest表示最新已打包的區塊
    ]
  }'
```



**返回**

```
{"jsonrpc":"2.0","id":1,"result":[{"return":"0x","gasUsed":21160}]}
```

```
{"jsonrpc":"2.0","id":1,"result":[{"return":"0x","gasUsed":21070,"revertReason":"execution reverted"}]}
```

```
{"jsonrpc":"2.0","id":1,"result":[{"return":"0x","gasUsed":0,"error":"err: intrinsic gas too low: have 0, want 21000 (supplied gas 0)"}]}
```



