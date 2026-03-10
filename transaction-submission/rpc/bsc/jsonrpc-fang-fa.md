# JSON-RPC方法

### eth\_sendRawTransaction

RPC的`eth_sendRawTransaction` 兼容原生的JSON-RPC方法，无需进行额外修改。如需修改交易披露、返利地址和revert保护等参数，可前往[控制台](https://www.blockrazor.io/#/login)配置专属RPC。

#### 請求參數

<table><thead><tr><th width="124">參數</th><th width="65">必選</th><th width="110">格式</th><th width="180">示例</th><th>備註</th></tr></thead><tbody><tr><td>-</td><td>是</td><td>bytes</td><td>"0xd46e……445675"</td><td>raw tx</td></tr></tbody></table>

#### 請求示例

```json
curl -X POST -H "Content-Type: application/json" --data '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "eth_sendRawTransaction",
    "params": [
        "0xd46e……445675"
    ]
}' https://bsc.blockrazor.xyz
```

#### 返回示例

**正常**

```json
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0xe670……527331"
}
```



**異常**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
	"code": -32000,
	"message": "rlp: element is larger than containing list"
  }
}
```



### eth\_sendMevBundle

RPC支持發送Bundle，詳見[Bundle](../../bundle/bsc/xiang-mu-fang.md)



### scutum\_queryTxProcessStatus

用於實時查詢BlockRazor RPC對於交易的處理狀態，目前支持查询BSC交易。

#### 请求示例

```json
curl -X POST -H "Content-Type: application/json" --data '{
	"id": 1,
	"jsonrpc": "2.0",
	"method": "scutum_queryTxProcessStatus",
	"params": ["0xf84a……e54284"]
}'<ETH_NODE_URL>
```

#### 返回示例

正常

```json
{
  "jsonrpc": "2.0",
  "result": "{"msg\":\"tx is included on chain\",\"status\":\"included\"}",
  "id": "1"
} // 交易已經在鏈上執行
```

```json
{
  "jsonrpc": "2.0",
  "result": "{"msg\":\"tx is expired and discarded\",\"status\":\"expired\"}",
  "id": "1"
} // 交易由於已過期被廢棄（交易發出時的區塊高度+100小於最新區塊高度）
```

```json
{
  "jsonrpc": "2.0",
  "result": "{"msg\":\"nonce too high\",\"status\":\"pending\"}",
  "id": "1"
} // Scutum正在處理該筆交易，交易由於nonce過高被放入隊列中
```

```json
{
  "jsonrpc": "2.0",
  "result": "{"msg\":\"tx is pending\",\"status\":\"pending\"}",
  "id": "1"
} // Scutum正在處理該筆交易
```

```json
{
  "jsonrpc": "2.0",
  "result": "{"msg\":\"simulation error: xxxxxxx\",\"status\":\"failed\"}",
  "id": "1"
} // 由於計算失敗，Scutum無法處理該筆交易
```

異常

```json
{
  "jsonrpc": "2.0",
  "error": "{\"code\":-32000,\"message\":\"tx not found\"}",
  "id": "1"
} //交易未發往Scutum或者已超過Scutum處理時限
```



### 其他JSON-RPC方法

RPC支持BSC節點原生的JSON-RPC方法，可參考[https://ethereum.org/zh/developers/docs/apis/json-rpc/#json-rpc-methods](https://ethereum.org/zh/developers/docs/apis/json-rpc/#json-rpc-methods)

