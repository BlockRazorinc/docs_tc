# Send RawTxBatch

### 接口說明

{% hint style="info" %}
該接口不具備MEV保護能力，通過該接口發送的交易會在公開mempool廣播。
{% endhint %}

用於在BSC高性能网络上廣播一批rawtx，方法名`SendTxs`



### 端點

<table><thead><tr><th width="154">地區</th><th width="218">可用區（AWS）</th><th>Relay地址</th></tr></thead><tbody><tr><td>法蘭克福</td><td>euc1-az2</td><td>35.157.64.49:50051</td></tr><tr><td>東京</td><td>apne1-az4</td><td>54.249.93.63:50051</td></tr><tr><td>愛爾蘭</td><td>euw1-az1</td><td>3.248.65.151:50051</td></tr><tr><td>弗吉尼亞</td><td>use1-az4</td><td>52.205.173.134:50051</td></tr></tbody></table>



### 流控說明

<table><thead><tr><th width="117.41015625"></th><th width="89.5859375">Tier 4</th><th width="88.046875">Tier 3</th><th>Tier 2</th><th>Tier 1</th><th>Tier 0</th></tr></thead><tbody><tr><td>BPS</td><td>-</td><td>-</td><td>2 batches / 1s</td><td>4 batches / 1s</td><td>12 batches / 1s</td></tr><tr><td>每batch包含交易數</td><td>-</td><td>-</td><td>10 Txs / batch</td><td>10 Txs / batch</td><td>20 Txs / batch</td></tr></tbody></table>



### 請求參數

<table><thead><tr><th width="148">參數</th><th width="59">必選</th><th width="137">格式</th><th width="215">示例</th><th>描述</th></tr></thead><tbody><tr><td>Transactions</td><td>是</td><td>Array[String]</td><td>["0xf8……8219", "0xcb……2ac0"]</td><td>經過簽名的raw tx列表</td></tr></tbody></table>



### 請求示例

{% tabs %}
{% tab title="Go" %}
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

	// BlockRazor relay endpoint address
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

	// use the Gateway client connection interface
	client := pb.NewGatewayClient(conn)

	// create context
	ctx := context.Background()

	// replace with your address
	from_private_address1 := "6c0456……8b8003"
	from_private_address2 := "42b565……44d05c"
	to_public_address := "0x4321……3f1c66"

	// replace with your transaction data
	nonce1 := uint64(1)
	nonce2 := uint64(1)
	toAddress := common.HexToAddress(to_public_address)
	var data []byte
	gasPrice := big.NewInt(1e9)
	gasLimit := uint64(22000)
	value := big.NewInt(0)
	chainid := types.NewEIP155Signer(big.NewInt(56))

	// create new transaction
	tx1 := types.NewTransaction(nonce1, toAddress, value, gasLimit, gasPrice, data)
	tx2 := types.NewTransaction(nonce2, toAddress, value, gasLimit, gasPrice, data)

	privateKey1, err := crypto.HexToECDSA(from_private_address1)
	if err != nil {
		fmt.Println("fail to casting private key to ECDSA")
		return
	}

	privateKey2, err := crypto.HexToECDSA(from_private_address2)
	if err != nil {
		fmt.Println("fail to casting private key to ECDSA")
		return
	}

	// sign transaction by private key
	signedTx1, err := types.SignTx(tx1, chainid, privateKey1)
	if err != nil {
		fmt.Println("fail to sign transaction")
		return
	}

	signedTx2, err := types.SignTx(tx2, chainid, privateKey2)
	if err != nil {
		fmt.Println("fail to sign transaction")
		return
	}

	// use rlp to encode your transaction
	body, _ := rlp.EncodeToBytes([]types.Transaction{*signedTx1, *signedTx2})

	// encode []byte to string
	encode_txs := hex.EncodeToString(body)

	// send raw tx batch by BlockRazor
	res, err := client.SendTxs(ctx, &pb.SendTxsRequest{Transactions: encode_txs})

	if err != nil {
		fmt.Println("failed to send raw tx batch: ", err)
		return
	} else {
		fmt.Println("raw tx batch sent by BlockRazor, tx hashes are ", res.TxHashs)
	}

}
```
{% endtab %}
{% endtabs %}



### 返回示例

**正常**

```json
{
 "tx_hash":["0xdb95……584813", "0x887d……7aba19"]
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

