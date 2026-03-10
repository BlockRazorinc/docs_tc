# GetGasPriceStream

## **介紹**

`GetGasPriceStream` 供BSC交易gas price訂閱服務，基於gRPC流以指定百分位向用戶推送BSC交易在最近上鏈區塊中的gas price，端點域名：<mark style="color:$primary;">grpc.bsc-fee.blockrazor.me:443</mark>



## **限流**

<table><thead><tr><th width="101.1328125"></th><th width="127.58203125">Tier 4</th><th width="140.89453125">Tier 3</th><th width="122.78515625">Tier 2</th><th width="127.6875">Tier 1</th><th width="139.5390625">Tier 0</th></tr></thead><tbody><tr><td>Streams</td><td>-</td><td>-</td><td>-</td><td>-</td><td>10</td></tr></tbody></table>



## 請求參數

<table><thead><tr><th width="124.2265625">字段</th><th width="67.33984375">必選</th><th width="82.46484375">格式</th><th width="82.48046875">示例</th><th>備注</th></tr></thead><tbody><tr><td>percentile</td><td>是</td><td>int</td><td>50</td><td>獲取指定分位的gas price，枚舉值：25、50、75、95、99</td></tr><tr><td>blockRange</td><td>是</td><td>int</td><td>20</td><td>統計最近N個區塊中交易的gas price，N取值範圍1-20</td></tr></tbody></table>



## 請求示例

```go
package main

import (
	"context"
	"crypto/tls"
	"log"

	// directory of the generated code using the provided proto file
	pb "fee-test/bscfeepb"

	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"
)

const (
	gRPCEndpoint = "grpc.bsc-fee.blockrazor.xyz:443" // endpoint address
	auth         = "your_auth" // auth to be verified
)

func main() {
	// open gRPC connection to endpoint
	conn, err := grpc.Dial(
		gRPCEndpoint,
		grpc.WithTransportCredentials(credentials.NewTLS(&tls.Config{})),
		grpc.WithPerRPCCredentials(&Authentication{auth}),
	)
	if err != nil {
		panic(err)
	}
	defer conn.Close()
	
	// create the gRPC client
	client := pb.NewServerClient(conn)

	// send gRPC request
	req := &pb.TransactionFee{
		Percentile: 95,
		BlockRange: 10,
	}
	stream, err := client.GetGasPriceStream(context.Background(), req)
	if err != nil {
		panic(err)
	}

	for {
		message, err := stream.Recv()
		if err != nil {
			log.Printf("Failed to receive message: %v", err)
			break
		}
		log.Printf("Received message: %s\n", message.Time.AsTime().Format("2006-01-02 15:04:05"))
		log.Printf("Received GasPrice: %v\n", message.GasPrice.Percentile)
		log.Printf("Received GasPrice: %v\n", message.GasPrice.Value)
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
    rpc GetGasPriceStream(TransactionFee) returns(stream TransactionFeeStreamResponse) {};
}

message TransactionFee {
    int32 percentile = 1;
    int32 blockRange = 2;
}

message FeeValue {
    int32 percentile = 1;
    string value = 2;
}

message TransactionFeeStreamResponse {
    FeeValue gasPrice = 1;
    google.protobuf.Timestamp time = 2;
}
```



## 返回示例

```json
2025/03/26 10:25:42 Received message: 2025-03-26 02:25:42
2025/03/26 10:25:42 Received GasPrice: 95
2025/03/26 10:25:42 Received GasPrice: 1000000000 // 对应分位的gas price，单位为wei
2025/03/26 10:25:45 Received message: 2025-03-26 02:25:45
2025/03/26 10:25:45 Received GasPrice: 95
2025/03/26 10:25:45 Received GasPrice: 1000000000
```







