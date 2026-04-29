---
description: ransaction
---

# sui\_executeTransactionBlock

### 介紹

Fast模式基於BlockRazor的全球高性能網絡實現交易最低延遲上鏈，適合對交易上鏈速度存在極致要求的用戶。Fast模式支持HTTP協議和gRPC協議，HTTP協議的交易發送方法和[`sui_executeTransactionBlock`](https://docs.sui.io/sui-api-ref?q=grpc#sui_executetransactionblock) 兼容，gRPC協議的交易發送方法和[`sui/rpc/v2/transaction_execution_service.proto`](https://docs.sui.io/references/fullnode-protocol#sui_rpc_v2_transaction-execution-service-proto) 兼容。

{% hint style="info" %}
Fast模式不和訂閱計劃綁定，但每筆交易中需要包含tip，下限金額取 0.001 SUI 或 Gas Budget × 0.05 SUI 的較大者。

Tip方法：可集成[TS](ts.md)或[Go](go.md)中的BlockRazor SDK，調用`AddTip`方法。
{% endhint %}



### 端點

sui-fast.blockrazor.io:80



### 限流

Fast模式不和訂閱計劃綁定，限流默認統一為10 TPS，如需提升TPS，請於我們[聯繫](https://discord.com/invite/qqJuwRb8Nh)。



### 請求示例

* [TS](ts.md)
* [Go](go.md)

