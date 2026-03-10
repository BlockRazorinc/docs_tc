# GetAllGasPriceStream

## **介紹**

`GetAllGasPriceStream` 供BSC交易gas price訂閱服務，基於gRPC流按百分位向用戶推送BSC交易在最近上鏈區塊中的gas price，端點域名：<mark style="color:$primary;">grpc.bsc-fee.blockrazor.me:443</mark>



## **限流**

<table><thead><tr><th width="104.73828125"></th><th>Tier 4</th><th>Tier 3</th><th>Tier 2</th><th>Tier 1</th><th>Tier 0</th></tr></thead><tbody><tr><td>Streams</td><td>-</td><td>-</td><td>-</td><td>-</td><td>10</td></tr></tbody></table>



## 請求參數

<table><thead><tr><th width="123.12890625">字段</th><th width="64.1875">必選</th><th width="82.46484375">格式</th><th width="64.1875">示例</th><th>備注</th></tr></thead><tbody><tr><td>blockRange</td><td>是</td><td>int</td><td>20</td><td>統計最近N個區塊中交易的gas price，N取值範圍1-20</td></tr></tbody></table>



## 請求示例

```go
package main

import (
	"context"
	"crypto/tls"
	"log"

	pb "fee-test/bscfeepb"

	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"
)

const (
	gRPCEndpoint = "grpc.bsc-fee.blockrazor.xyz:443"
	auth         = "your_auth"
)

func main() {
	conn, err := grpc.Dial(
		gRPCEndpoint,
		grpc.WithTransportCredentials(credentials.NewTLS(&tls.Config{})),
		grpc.WithPerRPCCredentials(&Authentication{auth}),
	)
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	client := pb.NewServerClient(conn)

	req := &pb.BlockRange{
		BlockRange: 5,
	}
	stream, err := client.GetAllGasPriceStream(context.Background(), req)
	if err != nil {
		panic(err)
	}

	for {
		message, err := stream.Recv()
		if err != nil {
			log.Printf("Failed to receive message: %v", err)
			break
		}
		log.Printf("Received Time: %s\n", message.Time.AsTime().Format("2006-01-02 15:04:05"))
		for _, gasPrice := range message.AllGasPrice {
			log.Printf("Received message percentile: %d\n", gasPrice.Percentile)
			log.Printf("Received message value: %s\n", gasPrice.Value)
		}
	}
}

type Authentication struct {
	auth string
}

func (a *Authentication) GetRequestMetadata(context.Context, ...string) (map[string]string, error) {
	return map[string]string{"apiKey": a.auth}, nil
}

func (a *Authentication) RequireTransportSecurity() bool {
	return false
}
```



### Proto

```json
syntax = "proto3";

package bscfeepb;

import "google/protobuf/timestamp.proto";

option go_package = "./pb/bscfeepb";

service Server {
    rpc GetAllGasPriceStream(BlockRange) returns(stream AllGasPriceStreamResponse) {};
}

message FeeValue {
    int32 percentile = 1;
    string value = 2;
}

message BlockRange {
    int32 blockRange = 2;
}

message AllGasPriceStreamResponse {
    repeated FeeValue allGasPrice = 1;
    google.protobuf.Timestamp time = 2;
}

```



## 返回示例

```json
2025/04/08 10:18:21 Received Time: 2025-04-08 02:18:21
2025/04/08 10:18:21 Received message percentile: 25
2025/04/08 10:18:21 Received message value: 1000000000
2025/04/08 10:18:21 Received message percentile: 50
2025/04/08 10:18:21 Received message value: 1100000000
2025/04/08 10:18:21 Received message percentile: 75
2025/04/08 10:18:21 Received message value: 2020000000
2025/04/08 10:18:21 Received message percentile: 95
2025/04/08 10:18:21 Received message value: 5000000000
2025/04/08 10:18:21 Received message percentile: 99
2025/04/08 10:18:21 Received message value: 20000000000
```





