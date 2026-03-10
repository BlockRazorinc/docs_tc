# Send PrivateTransaction

## 接口說明

本接口用於接收用戶提交的隱私交易，方法名為`eth_sendPrivateTransaction`



## 請求參數

<table><thead><tr><th width="183">参数</th><th width="81">必选</th><th width="82">格式</th><th width="106">示例</th><th>描述</th></tr></thead><tbody><tr><td>transaction</td><td>是</td><td>String</td><td>"0x…4b"</td><td>經過簽名的raw transaction</td></tr></tbody></table>



## 請求示例

{% tabs %}
{% tab title="HTTPS" %}
```bash
curl https://virginia.builder.blockrazor.io \
  -H 'content-type: application/json' \
  -H 'Authorization: <auth-token>' \
  --data '{
  "jsonrpc": "2.0",
  "id": "1",
  "method": "eth_sendPrivateTransaction",
  "params": ["0x…9c"]
}'
```
{% endtab %}
{% endtabs %}



### Proto

```go
syntax = "proto3";

package sendbundle;

option go_package = "internal/ethapi/sendbundle;sendbundle";

service BundleService {
  rpc SendBundle (SendBundleArgs) returns (SendBundleResponse);
  rpc SendTransaction (SendTransactionArgs) returns (SendTransactionResponse);
}

message SendBundleArgs {
  repeated bytes txs = 1;
  uint64 maxBlockNumber = 2;
  uint64 minTimestamp = 3;
  uint64 maxTimestamp = 4;
  repeated string revertingTxHashes = 5;
  repeated string droppingTxHashes = 6;
}

message SendTransactionArgs {
  bytes tx = 1;
}

message SendBundleResponse {
    string result = 1;
}

message SendTransactionResponse {
    string result = 1;
}
```



## 返回示例

```json
{
 "jsonrpc":"2.0",
 "id":"1",
 "result":"0xa06b……f7e8ec"  // 交易哈希
}‍
```

```json
{
  "jsonrpc":"2.0",
  "id":"1",
  "error":{
    "code":-32000,
    "message":"nonce too low: next nonce 57, tx nonce 56"
    }
}
```



