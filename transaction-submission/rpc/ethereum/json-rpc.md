# JSON-RPC

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

RPC支持發送Bundle，詳見[Bundle](../../bundle/ethereum/xiang-mu-fang.md)



### JSON-RPC方法

RPC支持以太坊節點原生的JSON-RPC方法，可參考[https://ethereum.org/zh/developers/docs/apis/json-rpc/#json-rpc-methods](https://ethereum.org/zh/developers/docs/apis/json-rpc/#json-rpc-methods)



