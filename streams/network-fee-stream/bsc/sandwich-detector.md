---
hidden: true
---

# Sandwich Detector

## 介紹

Sandwich Detector為项目方（如钱包、DEX和Trading Bot）提供BSC交易的三明治攻擊監測服務，用戶可基於gRPC實時訂閱BSC上的三明治攻擊交易信息，訂閱端點: bsc.sandwich-detector.blockrazor.me:443



## 訂閱計劃

<table><thead><tr><th width="176.69140625"></th><th width="99.60546875">Tier 4</th><th width="99.58203125">Tier 3</th><th width="100.49609375">Tier 2</th><th width="99.81640625">Tier 1</th><th width="99.61328125">Tier 0</th></tr></thead><tbody><tr><td>Sandwich Detector</td><td>-</td><td>-</td><td>-</td><td>-</td><td>支持</td></tr></tbody></table>

Sandwich Detector服務會向訂閱Tier 0計劃的項目方（如錢包、DEX和Trading Bot）開放白名單，白名單用戶可實時訂閱BSC交易的三明治攻擊交易信息，請在訂閱Tier 0計劃後與我們[聯繫](https://discord.com/invite/qqJuwRb8Nh)



## 請求參數

<table><thead><tr><th width="94.30859375">参数</th><th width="70.4921875">必選</th><th width="93.66015625">格式</th><th width="190.74609375">示例</th><th>备注</th></tr></thead><tbody><tr><td>from</td><td>否</td><td>String</td><td>""0x3879……42d82a"</td><td>通過被攻擊交易的發起者地址篩選三明治交易信息</td></tr><tr><td>to</td><td>否</td><td>String</td><td>"0xda77……7098a2"</td><td>通過被攻擊交易交互的合約地址篩選三明治交易信息</td></tr><tr><td>method</td><td>否</td><td>String</td><td>"0xac9650d8"</td><td>通過被攻擊交易調用的方法篩選三明治交易信息</td></tr></tbody></table>



## 請求示例

{% tabs %}
{% tab title="grpccurl" %}
```sh
# without filter
grpcurl -plaintext=false -import-path /path/to/proto -proto bscstream.proto -d ''  bsc.sandwich-detector.blockrazor.me:443 bscstream.TxStreamService/SubscribeTxStream
# with filter
grpcurl -plaintext=false -import-path /path/to/proto -proto bscstream.proto -d '{"to":"0xaa..bb"}'  bsc.sandwich-detector.blockrazor.me:443 bscstream.TxStreamService/SubscribeTxStream
```
{% endtab %}

{% tab title="Go" %}
```go
package main

import (
	"context"
	"crypto/tls"
	"log"

	pb "bscstream/proto/bscstream"

	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"
)

func main() {
	for {
		conn, err := grpc.Dial(
			"bsc.sandwich-detector.blockrazor.me:443",
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
			From:   "",
			To:     "",
			Method: "",
		})
		if err != nil {
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

			log.Printf("+%v\n", resp)
		}
	}
}
```
{% endtab %}
{% endtabs %}



### **Proto**

```go
// bscstream.proto
syntax = "proto3";

package bscstream;

option go_package = "proto/bscstream";

service TxStreamService {
  rpc SubscribeTxStream(TxFilter) returns (stream TxStreamResponse);
}

message TxFilter {
  string from = 1;   // (optional) from of victim tx to filter
  string to = 2;     // (optional) to of victim tx to filter
  string method = 3; // (optional) method of victim tx fo filter
}

message TxStreamResponse {
  repeated string txs = 1;         // all sandwich tx hash
  string victimtx = 2;             // victim tx hash
  repeated string attackertxs = 3; // frontrun and backrun tx hash
  string from = 4;                 // from of victim hash
  string to = 5;                   // to of victim hash
  string method = 6;               // method of victim hash
  repeated string tokens = 7;      // sandwiched token
  uint64 block = 8;                // block number of txs

  bool heartbeat = 9;              // if this resp is heartbeat, to keep stream alive if no sandwich data available
}
```



## 返回示例

```json
{
  "txs": [
    "0xee5b2cfb7a5120206450b7edf2ac824ea810442d982b71923240115ef906acfa",
    "0xb2c9048de1af5be6733b3ed3d1ea4ac74c3d7a9e6c58a9e07ebdac7b8708399b",
    "0x046455f9207b5209489cfada58bfe593acdca58d4a730210e552e56199bfcfe8"
  ],
  "victimtx": "0xb2c9048de1af5be6733b3ed3d1ea4ac74c3d7a9e6c58a9e07ebdac7b8708399b",
  "attackertxs": [
    "0xee5b2cfb7a5120206450b7edf2ac824ea810442d982b71923240115ef906acfa",
    "0x046455f9207b5209489cfada58bfe593acdca58d4a730210e552e56199bfcfe8"
  ],
  "from": "0x387965d0b2dfbebb24aa02e0a068d79a0b42d82a",
  "to": "0xda77c035e4d5a748b4ab6674327fa446f17098a2",
  "method": "0xac9650d8",
  "tokens": [
    "0xdd1b2aaf66a030315a821e7588cf691d73b64444",
    "0xbb4cdb9cbd36b01bd1cbaebf2de08d9173bc095c"
  ],
  "block": "47993864"
}
```

