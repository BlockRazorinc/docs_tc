# Send RawTx

### 接口說明

{% hint style="info" %}
該接口不具備MEV保護能力，通過該接口發送的交易會在公開mempool廣播。
{% endhint %}

本接口用於在BSC高性能网络上廣播单笔rawtx，方法名`SendTx` ，支持HTTP和gRPC協議。其中基於HTTP協議發送交易面向Tier 1等級或以上用戶開放，請在[訂閱](https://blockrazor.io/#/pricing)後和我們[聯繫](https://discord.com/invite/qqJuwRb8Nh)。



### 端點

<table><thead><tr><th width="154">地區</th><th width="218">可用區（AWS）</th><th>Relay地址</th></tr></thead><tbody><tr><td>法蘭克福</td><td>euc1-az2</td><td>35.157.64.49:50051</td></tr><tr><td>東京</td><td>apne1-az4</td><td>54.249.93.63:50051</td></tr><tr><td>愛爾蘭</td><td>euw1-az1</td><td>3.248.65.151:50051</td></tr><tr><td>弗吉尼亞</td><td>use1-az4</td><td>52.205.173.134:50051</td></tr></tbody></table>



### 流控說明

|        | Tier 4       | Tier 3       | Tier 2       | Tier 1       | Tier 0       |
| ------ | ------------ | ------------ | ------------ | ------------ | ------------ |
| TPS    | 10 Txs / 5s  | 10 Txs / 5s  | 50 Txs / 5s  | 100 Txs / 5s | 200 Txs / 5s |
| 每日交易上限 | 10 Txs       | 10 Txs       | 不限           | 不限           | 不限           |



### 請求參數

<table><thead><tr><th width="147">參數</th><th width="66">必選</th><th width="91">格式</th><th>示例</th><th>描述</th></tr></thead><tbody><tr><td>Transaction</td><td>是</td><td>String</td><td>"0xf8……8219"</td><td>經過簽名的rawtx</td></tr></tbody></table>



### 請求示例

{% tabs %}
{% tab title="gRPC" %}
```go
package main

import (
	"context"
	"encoding/hex"
	"fmt"
	"math/big"

	// directory of the generated code using the provided relay.proto file
	pb "github.com/BlockRazorinc/relay_example/protobuf"
	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/core/types"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/ethereum/go-ethereum/rlp"
	"google.golang.org/grpc"
)

// auth will be used to verify the credential
type Authentication struct {
	apiKey string
}

func (a *Authentication) GetRequestMetadata(context.Context, ...string) (map[string]string, error) {
	return map[string]string{"apiKey": a.apiKey}, nil
}

func (a *Authentication) RequireTransportSecurity() bool {
	return false
}

func main() {

	// blockrazor relay endpoint address
	blzrelayEndPoint := "ip:port"

	// auth will be used to verify the credential
	auth := Authentication{
		"your auth token",
	}

	// open gRPC connection to BlockRazor relay
	var err error
	conn, err := grpc.Dial(blzrelayEndPoint, grpc.WithInsecure(), grpc.WithPerRPCCredentials(&auth), grpc.WithWriteBufferSize(0), grpc.WithInitialConnWindowSize(128*1024))
	if err != nil {
		fmt.Println("error: ", err)
		return
	}

	// use the Gateway client connection interface.
	client := pb.NewGatewayClient(conn)

	// create context
	ctx := context.Background()

	// replace with your address
	from_private_address := "42b565……44d05c"
	to_public_address := "0x4321……3f1c66"

	// replace with your transaction data
	nonce := uint64(1)
	toAddress := common.HexToAddress(to_public_address)
	var data []byte
	gasPrice := big.NewInt(1e9)
	gasLimit := uint64(22000)
	value := big.NewInt(0)
	chainid := types.NewEIP155Signer(big.NewInt(56))

	// create new transaction
	tx := types.NewTransaction(nonce, toAddress, value, gasLimit, gasPrice, data)

	privateKey, err := crypto.HexToECDSA(from_private_address)
	if err != nil {
		fmt.Println("fail to casting private key to ECDSA")
		return
	}

	// sign transaction by private key
	signedTx, err := types.SignTx(tx, chainid, privateKey)
	if err != nil {
		fmt.Println("fail to sign transaction")
		return
	}

	// use rlp to encode your transaction
	body, _ := rlp.EncodeToBytes([]types.Transaction{*signedTx})

	// encode []byte to string
	encode_tx := hex.EncodeToString(body)

	// send raw tx by BlockRazor
	res, err := client.SendTx(ctx, &pb.SendTxRequest{Transaction: encode_tx})

	if err != nil {
		fmt.Println("failed to send raw tx: ", err)
		return
	} else {
		fmt.Println("raw tx sent by BlockRazor, tx hash is ", res.TxHash)
	}

}
```
{% endtab %}

{% tab title="HTTP" %}
```bash
curl <Endpoint_URL> \
    -X POST \
    -H "Content-Type: application/json" \
    -d '{"jsonrpc":"2.0","method":"eth_sendRawTransaction","params":["0x02f8……869514"],"id":1}'
```
{% endtab %}
{% endtabs %}



### 返回示例

**正常**

```json
{
 "tx_hash":"0x2944……b2188f"
}
```



**异常**

```
rpc error: code = Unknown desc = invalid transaction format
```



### Proto

`relay.proto`文件代碼如下：

```
syntax = "proto3";

package blockchain;

option go_package = "/Users/code/relay/grpcServer"; 
service Gateway {
  rpc SendTx (SendTxRequest) returns (SendTxReply) {}
  rpc SendTxs (SendTxsRequest) returns (SendTxsReply) {}
  rpc NewTxs (TxsRequest) returns (stream TxsReply){}
  rpc NewBlocks (BlocksRequest) returns (stream BlocksReply){}
}

message TxsRequest{
  bool node_validation = 1;
}

message Tx{
  bytes from = 1;
  int64 timestamp = 2;
  bytes raw_tx = 3;
}

message TxsReply{
   Tx tx = 1;
}

message BlocksRequest{
  bool node_validation = 1;
}

message BlockHeader{
  string parent_hash = 1;
  string sha3_uncles = 2;
  string miner = 3;
  string state_root = 4;
  string transactions_root = 5;
  string receipts_root = 6;
  string logs_bloom = 7;
  string difficulty = 8;
  string number = 9;
  uint64 gas_limit = 10;
  uint64 gas_used = 11;
  uint64 timestamp = 12;
  bytes extra_data = 13;
  string mix_hash = 14;
  uint64 nonce = 15;
  uint64 base_fee_per_gas = 16;
  string withdrawals_root = 17;
  uint64 blob_gas_used = 18;
  uint64 excess_blob_gas = 19;
  string parent_beacon_block_root = 20;
}

message NextValidator{
  string block_height = 1;
  string coinbase = 2;
}

message BlocksReply{
  string hash = 1;
  BlockHeader header = 2;
  repeated NextValidator nextValidator = 3;
  repeated Tx txs = 4;
}

message Transaction {
  string content = 1;
}

message Transactions {
  repeated Transaction transactions = 1;
}

message SendTxRequest {
  string transaction = 1;
}

message SendTxsRequest {
  string transactions = 1;
}

message SendTxReply {
  string tx_hash = 1;
}

message SendTxsReply {
  repeated string tx_hashs = 1;
}
```



### 代码示例

[https://github.com/BlockRazorinc/relay\_example](https://github.com/BlockRazorinc/relay_example)

