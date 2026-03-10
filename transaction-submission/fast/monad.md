# Monad

### 介紹

Fast模式基於BlockRazor的全球高性能網絡實現交易最低延遲上鏈，適合對交易上鏈速度存在極致要求的用戶。Fast模式的交易發送方法和`eth_sendRawTransaction`兼容。

{% hint style="info" %}
Fast模式不和訂閱計劃綁定，但每筆交易中需要包含轉賬至0x9D70AC39166ca154307a93fa6b595CF7962fe8e5的tip，下限金額取 0.5 MON 或 MaxPriorityFee × 0.05 MON 的較大者
{% endhint %}



### 端點

http://monad-fast.blockrazor.io



### 限流

Fast模式不和訂閱計劃綁定，限流默認統一為10 TPS，如需提升TPS，請於我們聯繫



### 請求示例

```json
curl http://monad-fast.blockrazor.io \
  -X POST \
  -H "Authorization: Bearer <auth>" \
  -H "Content-Type: application/json" \
  --data '
    {
    "jsonrpc": "2.0",
    "method": "eth_sendRawTransaction",
    "params": [
      "Signed Transaction"
    ],
    "id": 1
  }
  '
```



### 返回示例

**正常**

```json
{
 "jsonrpc":"2.0",
 "id":"1",
 "result":"0xa06b……f7e8ec"  // 交易哈希
}‍
```



**異常**

```json
{
  "jsonrpc":"2.0",
  "id":"1",
  "error":{
    "code":-32000,
    "message":"Tip verification failed"
    }
}
```



