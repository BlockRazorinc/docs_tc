---
hidden: true
---

# Sandwich Detector

## 介紹

Sandwich Detector為項目方（如錢包、DEX和Trading Bot）提供Solana交易的三明治攻擊監測服務，用戶可基於gRPC實時訂閱Solana上的三明治攻擊交易信息，訂閱端點: solana.sandwich-detector.blockrazor.me:443



## 訂閱計劃

<table><thead><tr><th width="176.69140625"></th><th width="99.60546875">Tier 4</th><th width="99.58203125">Tier 3</th><th width="100.49609375">Tier 2</th><th width="99.81640625">Tier 1</th><th width="99.61328125">Tier 0</th></tr></thead><tbody><tr><td>Sandwich Detector</td><td>-</td><td>-</td><td>-</td><td>-</td><td>支持</td></tr></tbody></table>

Sandwich Detector服務會向訂閱Tier 0計劃的项目方（如錢包、DEX和Trading Bot）開放白名單，白名單用戶可實時訂閱Solana交易的三明治攻擊交易信息，請在訂閱Tier 0計劃後與我們[聯繫](https://discord.com/invite/qqJuwRb8Nh)



## 請求參數

<table><thead><tr><th width="167.59375">参数</th><th width="70.4921875">必選</th><th width="93.66015625">格式</th><th>示例</th><th>备注</th></tr></thead><tbody><tr><td>signer</td><td>否</td><td>String</td><td>"6yf14f……iwPZ3V"</td><td>通過被攻擊交易的發起者賬戶地址篩選三明治交易信息</td></tr><tr><td>relevantProgramId</td><td>否</td><td>String</td><td>"CAMMCz……KgrWqK"</td><td>通過被攻擊交易交互的programId篩選三明治交易信息</td></tr></tbody></table>



## 請求示例

{% tabs %}
{% tab title="grpccurl" %}
```sh
# without filter
grpcurl -plaintext=false -import-path /path/to/proto -proto solstream.proto -d ''  solana.sandwich-detector.blockrazor.me:443 solstream.TxStreamService/SubscribeTxStream
# with filter
grpcurl -plaintext=false -import-path /path/to/proto -proto solstream.proto -d '{"relevantProgramId":"aa..bb"}'  solana.sandwich-detector.blockrazor.me:443 solstream.TxStreamService/SubscribeTxStream
```
{% endtab %}

{% tab title="Go" %}
```go
package main

import (
	"context"
	"crypto/tls"
	"log"

	pb "solstream/proto/solstream"

	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"
)

func main() {
	for {
		conn, err := grpc.Dial(
			"solana.sandwich-detector.blockrazor.me:443",
			grpc.WithTransportCredentials(credentials.NewTLS(&tls.Config{})),
			grpc.WithContextDialer(func(ctx context.Context, addr string) (net.Conn, error) { //use ipv4
				d := net.Dialer{DualStack: false}
				return d.DialContext(ctx, "tcp4", addr)
			}),
		)
		if err != nil {
			log.Printf("Failed to connect: %v.\n", err)
			continue
		}

		client := pb.NewTxStreamServiceClient(conn)
		stream, err := client.SubscribeTxStream(context.Background(), &pb.TxFilter{
			// Empty for no filter
			Signer:            "",
			RelevantProgramId: "",
		})
		if err != nil {
			log.Printf("Error subscribing to stream: %v.\n", err)
			conn.Close()
			continue
		}

		log.Println("Successfully subscribed to the stream.")

		for {
			resp, err := stream.Recv()
			if err != nil {
				log.Printf("Error receiving stream data: %v. Reconnecting...\n", err)
				conn.Close()
				break
			}
			log.Println(resp)

		}
	}

}

```
{% endtab %}
{% endtabs %}

### **Proto**

```go
// solstream.proto
syntax = "proto3";

package solstream;

option go_package = "proto/solstream";

service TxStreamService {
  rpc SubscribeTxStream(TxFilter) returns (stream TxStreamResponse);
}

message TxFilter {
  string signer = 1;            // (optional) signer of victim tx to filter
  string relevantProgramId = 2; // (optional) interacted program id of victim tx to filter. For example, programId of some router
}

message TxStreamResponse {
  uint64 slot = 1;                 // slot of sandwich txs
  repeated string txs = 2;         // all sandwich tx signatures
  string victimtx = 3;             // victim tx signature
  repeated string attackertxs = 4; // frontrun and backrun tx signature
  string signer = 5;               // signaer of victim tx

  bool heartbeat = 6;              // if this resp is heartbeat, to keep stream alive if no sandwich data available
}
```



## 返回示例

```json
{
  "slot": "330748316",
  "txs": [
    "3gDiqwyYnqLtBrkdF5Y6RJiattctYjKu2NfjcAupDBRkhodh9a22XjPYYdhf83FpGUnBe2g1oeLxfpNYG8QyCrUt",
    "4joomCxakJxi9vhJ6mRuFStyiUyetaB2X1HMj4g3VUSTMQkPgMTAZ7NaLNMejvfu9zdp7gXp2fM5UEgVK5PohVrP",
    "4QgGbPiaNyhgMrkeFrNjZaPNDQTvXqc1THVQJXvbN26gBt8GxQikBzk5fiQyjvffLt8PJUGNw21GU55iErmQcGkC"
  ],
  "victimtx": "4joomCxakJxi9vhJ6mRuFStyiUyetaB2X1HMj4g3VUSTMQkPgMTAZ7NaLNMejvfu9zdp7gXp2fM5UEgVK5PohVrP",
  "attackertxs": [
    "3gDiqwyYnqLtBrkdF5Y6RJiattctYjKu2NfjcAupDBRkhodh9a22XjPYYdhf83FpGUnBe2g1oeLxfpNYG8QyCrUt",
    "4QgGbPiaNyhgMrkeFrNjZaPNDQTvXqc1THVQJXvbN26gBt8GxQikBzk5fiQyjvffLt8PJUGNw21GU55iErmQcGkC"
  ],
  "signer": "Ejhath43fHYtWLbSMfVHCL8eJiLkV22K6w1Ni5xzXofX" // signer of victim tx
}
```





