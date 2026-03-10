# Get TransactionFee

## **介紹**

`Get TransactionFee` 用於聚合獲取Solana交易的priority fee和tip，支持gRPC協議，端點域名：<mark style="color:$primary;">grpc.solana-fee.blockrazor.me:443</mark>



## 流控說明

<table><thead><tr><th width="115.97265625"></th><th>Tier 4</th><th>Tier 3</th><th>Tier 2</th><th>Tier 1</th><th>Tier 0</th></tr></thead><tbody><tr><td>QPS</td><td>1</td><td>5</td><td>10</td><td>50</td><td>100</td></tr></tbody></table>



## 請求參數

<table><thead><tr><th width="112.46484375" valign="middle">字段</th><th width="62.78515625">必選</th><th width="92.390625">格式</th><th>示例</th><th>備注</th></tr></thead><tbody><tr><td valign="middle">accounts</td><td>否</td><td>string[]</td><td>["DH4xma……HFtNYJ"]</td><td>不指定account則統計指定slot區間內全部交易的priority fee和tip</td></tr><tr><td valign="middle">percentile</td><td>是</td><td>int</td><td>50</td><td>獲取指定分位的priority fee和tip，枚舉值：25、50、75、95、99</td></tr><tr><td valign="middle">slotRange</td><td>是</td><td>int</td><td>150</td><td>統計最近N個confirmed slot中交易的priorityFee和tip，N取值範圍1-150</td></tr></tbody></table>



## 請求示例

```go
package main

import (
	"context"
	"crypto/tls"
	"log"

	// directory of the generated code using the provided proto file
	pb "fee-test/feepb"

	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"
)

const (
	gRPCEndpoint = "grpc.solana-fee.blockrazor.xyz:443" // endpoint address
	auth         = "your auth" // auth to be verified
	testAccount1 = "DH4xmaWDnTzKXehVaPSNy9tMKJxnYL5Mo5U3oTHFtNYJ" // query priority fee and tip of transactions involving a specified account 
	testAccount2 = "CAPhoEse9xEH95XmdnJjYrZdNCA8xfUWdy3aWymHa1Vj" 
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

	// use the gRPC client
	client := pb.NewServerClient(conn)

	req := &pb.TransactionFee{
		Accounts:   []string{}, //do not set accounts
		Percentile: 75,
		SlotRange:  150,
	}
  
  // send gRPC request1
	resp, err := client.GetTransactionFee(context.Background(), req)
	if err != nil {
		panic(err)
	}
	log.Printf("Response PriorityFee Percentile: %+v", resp.PriorityFee.Percentile)
	log.Printf("Response PriorityFee Value: %+v", resp.PriorityFee.Value)
	log.Printf("Response Tip Percentile: %+v", resp.Tip.Percentile)
	log.Printf("Response Tip Value: %+v", resp.Tip.Value)
	
	req2 := &pb.TransactionFee{
		Accounts:   []string{testAccount1, testAccount2}, //set specified accounts
		Percentile: 75,
		SlotRange:  150,
	}
	
	// send gRPC request2
	resp2, err := client.GetTransactionFee(context.Background(), req2)
	if err != nil {
		panic(err)
	}
	log.Printf("Response2 PriorityFee Percentile: %+v", resp2.PriorityFee.Percentile)
	log.Printf("Response2 PriorityFee Value: %+v", resp2.PriorityFee.Value)
	log.Printf("Response2 Tip Percentile: %+v", resp2.Tip.Percentile)
	log.Printf("Response2 Tip Value: %+v", resp2.Tip.Value)
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

package feepb;

option go_package = "./pb/feepb";

service Server {
    rpc GetTransactionFee(TransactionFee) returns(TransactionFeeResponse) {};
}

message TransactionFee {
    repeated string accounts = 1;
    int32 percentile = 2;
    int32 slotRange = 3;
}

message TransactionFeeResponse {
    FeeValue priorityFee = 1;
    FeeValue tip = 2;
}

message FeeValue {
    int32 percentile = 1;
    double value = 2;
}
```



## 返回示例

```json
Response PriorityFee Percentile: 75
Response PriorityFee Value: 10000 // 對應分位的priority fee，單位為micro-lamports
Response Tip Percentile: 75 
Response Tip Value: 0.0001 // 對應分位的tip，單位為Sol
Response2 PriorityFee Percentile: 75
Response2 PriorityFee Value: 432313.2435833086
Response2 Tip Percentile: 75
Response2 Tip Value: 0.0001
```







