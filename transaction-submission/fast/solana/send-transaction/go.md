---
description: 本文档介绍如何通过Go构建、发送Solana交易
---

# Go

{% hint style="info" %}
Solana發送交易的服務不和訂閱計劃綁定，可前往 [Authentication](../../../../authentication.md) 獲取API KEY，默认限流为1 TPS。如需提升限流標準，請[聯繫](https://discord.com/invite/qqJuwRb8Nh)我們，我們會在第一時間處理
{% endhint %}

## HTTP

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
	"bytes"
	"context"
	"encoding/json"
	"fmt"
	"io"
	"math/rand"
	"net/http"
	"time"

	"github.com/gagliardetto/solana-go"
	"github.com/gagliardetto/solana-go/programs/system"
	"github.com/gagliardetto/solana-go/rpc"
)

const (
	httpEndpoint     = "http://frankfurt.solana.blockrazor.xyz:443/sendTransaction"
	healthEndpoint   = "http://frankfurt.solana.blockrazor.xyz:443/health"
	mainNetRPC       = ""
	authKey          = ""
	privateKey       = ""
	publicKey        = ""
	amount           = 200_000
	tipAmount        = 1_000_000
	mode             = "fast"
	safeWindow       = 5
	revertProtection = false
)

var tipAccounts = []string{
	"Gywj98ophM7GmkDdaWs4isqZnDdFCW7B46TXmKfvyqSm",
	"FjmZZrFvhnqqb9ThCuMVnENaM3JGVuGWNyCAxRJcFpg9",
	"6No2i3aawzHsjtThw81iq1EXPJN6rh8eSJCLaYZfKDTG",
	"A9cWowVAiHe9pJfKAj3TJiN9VpbzMUq6E4kEvf5mUT22",
	"68Pwb4jS7eZATjDfhmTXgRJjCiZmw1L7Huy4HNpnxJ3o",
	"4ABhJh5rZPjv63RBJBuyWzBK3g9gWMUQdTZP2kiW31V9",
	"B2M4NG5eyZp5SBQrSdtemzk5TqVuaWGQnowGaCBt8GyM",
	"5jA59cXMKQqZAVdtopv8q3yyw9SYfiE3vUCbt7p8MfVf",
	"5YktoWygr1Bp9wiS1xtMtUki1PeYuuzuCF98tqwYxf61",
	"295Avbam4qGShBYK7E9H5Ldew4B3WyJGmgmXfiWdeeyV",
	"EDi4rSy2LZgKJX74mbLTFk4mxoTgT6F7HxxzG2HBAFyK",
	"BnGKHAC386n4Qmv9xtpBVbRaUTKixjBe3oagkPFKtoy6",
	"Dd7K2Fp7AtoN8xCghKDRmyqr5U169t48Tw5fEd3wT9mq",
	"AP6qExwrbRgBAVaehg4b5xHENX815sMabtBzUzVB4v8S",
}

var httpClient = &http.Client{
	Timeout: 10 * time.Second,
}

type SendRequest struct {
	Transaction      string `json:"transaction"`
	Mode             string `json:"mode"`
	SafeWindow       int    `json:"safeWindow"`
	RevertProtection bool   `json:"revertProtection"`
}

type SendResponse struct {
	Signature string `json:"signature"`
}
type HealthResponse struct {
	Result string `json:"result"`
}

func main() {
	// Pre-warm: perform an initial health check to establish the HTTP connection
	err := pingHealth()
	if err != nil {
		fmt.Printf("health check failed: %v\n", err)
	}
	// Start a background goroutine to periodically send /health requests
	// For low-frequency users, this keeps the HTTP connection alive (warm)
	go func() {
		for {
			err := pingHealth()
			if err != nil {
				fmt.Printf("health check failed: %v\n", err)
			}
			time.Sleep(30 * time.Second)
		}
	}()
	// send transactions
	if err := sendTx(); err != nil {
		fmt.Printf("send tx failed: %v\n", err)
	}
}

func pingHealth() error {
	req, err := http.NewRequest("GET", healthEndpoint, nil)
	if err != nil {
		return err
	}
	req.Header.Set("Content-Type", "application/json")
	req.Header.Set("apikey", authKey)
	resp, err := httpClient.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()
	// ⚠️ Important Note:
	// According to the Go net/http documentation, in order for the underlying TCP connection
	// to be reused (i.e. kept alive), the response body must be fully read and closed.
	// Otherwise, the Transport may not reuse the connection for future requests.
	// Reference: https://pkg.go.dev/net/http#Response
	// > "The default HTTP client's Transport may not reuse HTTP/1.x 'keep-alive' TCP connections
	//    if the Body is not read to completion and closed."

	// Read the full response body to enable connection reuse
	bodyBytes, _ := io.ReadAll(resp.Body)
	var healthRes HealthResponse
	if err := json.Unmarshal(bodyBytes, &healthRes); err != nil {
		return fmt.Errorf("decode error: %v", err)
	}

	return nil
}

func sendTx() error {
	account, err := solana.WalletFromPrivateKeyBase58(privateKey)
	if err != nil {
		return err
	}
	receivePub := solana.MustPublicKeyFromBase58(publicKey)
	tipPub := solana.MustPublicKeyFromBase58(tipAccounts[rand.Intn(len(tipAccounts))])

	rpcClient := rpc.New(mainNetRPC)
	blockhash, err := rpcClient.GetLatestBlockhash(context.TODO(), rpc.CommitmentFinalized)
	if err != nil {
		return fmt.Errorf("[get blockhash] %v", err)
	}

	transferIx := system.NewTransferInstruction(amount, account.PublicKey(), receivePub).Build()
	tipIx := system.NewTransferInstruction(tipAmount, account.PublicKey(), tipPub).Build()

	tx, err := solana.NewTransaction(
		[]solana.Instruction{tipIx, transferIx},
		blockhash.Value.Blockhash,
		solana.TransactionPayer(account.PublicKey()),
	)
	if err != nil {
		return fmt.Errorf("build tx error: %v", err)
	}

	_, err = tx.Sign(func(key solana.PublicKey) *solana.PrivateKey {
		if account.PublicKey().Equals(key) {
			return &account.PrivateKey
		}
		return nil
	})
	if err != nil {
		return fmt.Errorf("sign tx error: %v", err)
	}

	txBase64, err := tx.ToBase64()
	if err != nil {
		return err
	}

	reqBody := SendRequest{
		Transaction:      txBase64,
		Mode:             mode,
		SafeWindow:       safeWindow,
		RevertProtection: revertProtection,
	}
	jsonBody, _ := json.Marshal(reqBody)

	httpReq, err := http.NewRequest("POST", httpEndpoint, bytes.NewBuffer(jsonBody))
	if err != nil {
		return err
	}
	httpReq.Header.Set("Content-Type", "application/json")
	httpReq.Header.Set("apikey", authKey)
	resp, err := httpClient.Do(httpReq)
	if err != nil {
		return fmt.Errorf("send http error: %v", err)
	}
	defer resp.Body.Close()

	bodyBytes, _ := io.ReadAll(resp.Body)
	var sendRes SendResponse
	if err := json.Unmarshal(bodyBytes, &sendRes); err != nil {
		return fmt.Errorf("decode error: %v", err)
	}
	fmt.Printf("[send tx] response: %+v\n", sendRes)
	return nil
}
```
{% endtab %}

{% tab title="curl" %}
```javascript
// 以下是fast模式的请求示例

curl --request POST \
  --url http://frankfurt.solana.blockrazor.xyz:443/sendTransaction \
  --header 'Content-Type: application/json' \
  --header 'apikey: $auth_token' \
  --data '{
  "transaction":"$base64_tx",
  "mode":"fast",
  "revertProtection":false
}'
```
{% endtab %}
{% endtabs %}



### 返回示例

**正常**

```http
{"signature":"2DkHpsZxwbGDHkCujRMqd1jXMPaXfzn2JpwxZ43PoTk6Cj7hpxDp8VHESNCeuh95nVMmsWV4RGGWCkmGERZCTWHL","error":""}
```



**異常**

```http
{"signature":"","error":"error: Authentication information is missing. Please provide a valid auth token"}
```



## gRPC

{% hint style="info" %}
[fast模式](https://github.com/BlockRazorinc/solana-trader-client-go/blob/main/example/grpc/mode_fast/main.go)\
[sandwichMitigation模式](https://github.com/BlockRazorinc/solana-trader-client-go/tree/main/example/grpc/mode_sandwichMitigation)
{% endhint %}

```go
package main

import (
	"context"
	"fmt"
	"math/rand"
	"time"

	pb "github.com/BlockRazorinc/solana-trader-client-go/pb/serverpb"

	"github.com/gagliardetto/solana-go"
	"github.com/gagliardetto/solana-go/programs/system"
	"github.com/gagliardetto/solana-go/rpc"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
)

const (
	// BlockRazor relay endpoint address
	blzRelayEndpoint = "frankfurt.solana-grpc.blockrazor.xyz:80"
	// replace your solana rpc endpoint
	mainNetRPC = ""
	// replace your authKey
	authKey = ""
	// relace your private key(base58)
	privateKey = ""
	// publicKey(base58)
	publicKey = ""
	// transfer amount
	amount = 200_000
	// send mode
	mode = "fast"// set to "sandwichMitigation" to mitigate sandwich attacks on tx
	// safeWindow
	safeWindow = 3 // only take effect in sandwichMitigation mod
	// revertProtection
	revertProtection = false
	// tip amount
	tipAmount = 1_000_000
)

var tipAccounts = []string{
	"Gywj98ophM7GmkDdaWs4isqZnDdFCW7B46TXmKfvyqSm",
	"FjmZZrFvhnqqb9ThCuMVnENaM3JGVuGWNyCAxRJcFpg9",
	"6No2i3aawzHsjtThw81iq1EXPJN6rh8eSJCLaYZfKDTG",
	"A9cWowVAiHe9pJfKAj3TJiN9VpbzMUq6E4kEvf5mUT22",
	"68Pwb4jS7eZATjDfhmTXgRJjCiZmw1L7Huy4HNpnxJ3o",
	"4ABhJh5rZPjv63RBJBuyWzBK3g9gWMUQdTZP2kiW31V9",
	"B2M4NG5eyZp5SBQrSdtemzk5TqVuaWGQnowGaCBt8GyM",
	"5jA59cXMKQqZAVdtopv8q3yyw9SYfiE3vUCbt7p8MfVf",
	"5YktoWygr1Bp9wiS1xtMtUki1PeYuuzuCF98tqwYxf61",
	"295Avbam4qGShBYK7E9H5Ldew4B3WyJGmgmXfiWdeeyV",
	"EDi4rSy2LZgKJX74mbLTFk4mxoTgT6F7HxxzG2HBAFyK",
	"BnGKHAC386n4Qmv9xtpBVbRaUTKixjBe3oagkPFKtoy6",
	"Dd7K2Fp7AtoN8xCghKDRmyqr5U169t48Tw5fEd3wT9mq",
	"AP6qExwrbRgBAVaehg4b5xHENX815sMabtBzUzVB4v8S",
}

func main() {
	var err error
	account, err := solana.WalletFromPrivateKeyBase58(privateKey)
	receivePub := solana.MustPublicKeyFromBase58(publicKey)

	// setup grpc connect
	conn, err := grpc.NewClient(blzRelayEndpoint,
		grpc.WithTransportCredentials(insecure.NewCredentials()),
		grpc.WithPerRPCCredentials(&Authentication{authKey}),
	)
	if err != nil {
		panic(fmt.Sprintf("connect error: %v", err))
	}

	// use the Gateway client connection interface
	client := pb.NewServerClient(conn)

	// Pre-warm: perform an initial health check to establish the grpc connection
	err = pingHealth(client)
	if err != nil {
		fmt.Printf("health check failed: %v\n", err)
	}
	// Start a background goroutine to periodically send /health requests
	// For low-frequency users, this keeps the grpc connection alive (warm)
	go func() {
		for {
			err := pingHealth(client)
			if err != nil {
				fmt.Printf("health check failed: %v\n", err)
			}
			time.Sleep(30 * time.Second)
		}
	}()

	// new rpc client and get latest block hash
	rpcClient := rpc.New(mainNetRPC)
	blockhash, err := rpcClient.GetLatestBlockhash(context.TODO(), rpc.CommitmentFinalized)
	if err != nil {
		panic(fmt.Sprintf("[get latest block hash] error: %v", err))
	}

	tipAccount := tipAccounts[rand.Intn(len(tipAccounts))]

	// construct instruction
	transferIx := system.NewTransferInstruction(amount, account.PublicKey(), receivePub).Build()
	tipIx := system.NewTransferInstruction(tipAmount, account.PublicKey(), solana.MustPublicKeyFromBase58(tipAccount)).Build()

	// construct transation, replace your transation
	tx, err := solana.NewTransaction(
		[]solana.Instruction{tipIx, transferIx},
		blockhash.Value.Blockhash,
		solana.TransactionPayer(account.PublicKey()),
	)
	if err != nil {
		panic(fmt.Sprintf("new tx error: %v", err))
	}

	// transaction sign
	_, err = tx.Sign(
		func(key solana.PublicKey) *solana.PrivateKey {
			if account.PublicKey().Equals(key) {
				return &account.PrivateKey
			}
			return nil
		},
	)
	if err != nil {
		panic(fmt.Sprintf("sign tx error: %v", err))
	}

	txBase64, _ := tx.ToBase64()
	sendRes, err := client.SendTransaction(context.TODO(), &pb.SendRequest{
		Transaction: 	  txBase64,
		Mode:             mode,
		SafeWindow:       safeWindow,
		RevertProtection: revertProtection,
	})
	if err != nil {
		panic(fmt.Sprintf("[send tx] error: %v", err))
	}

	fmt.Printf("[send tx] response: %+v \n", sendRes)
	return
}

type Authentication struct {
	apiKey string
}

func (a *Authentication) GetRequestMetadata(context.Context, ...string) (map[string]string, error) {
	return map[string]string{"apikey": a.apiKey}, nil
}

func (a *Authentication) RequireTransportSecurity() bool {
	return false
}

func pingHealth(client pb.ServerClient) error {
	// grpc request warmup
	healthRes, err := client.GetHealth(context.Background(), &pb.HealthRequest{})
	if err != nil {
		return err
	}
	fmt.Printf("[health] response: %+v \n", healthRes.Status)
	return err
}
```



### Proto

```go
syntax = "proto3";

package serverpb;

option go_package = "./pb/serverpb";


service Server {
    rpc SendTransaction(SendRequest) returns(SendResponse) {};

    rpc GetHealth(HealthRequest) returns(HealthResponse) {};
}

message SendRequest {
    string transaction = 1;
    string mode = 2;
    int32 safeWindow = 3; // only take effect in sandwichMitigation mode
    bool revertProtection = 4;
}

message SendResponse {
    string signature = 1;
}

message HealthRequest {
}

message HealthResponse {
    string status = 1;
}
```



### 返回示例

**正常**

```go
signature:"2PCgCdD5gm4852ooyT2LQqiTgcau28hRNWCqBHCJ33EhhBsuAzAxYGLbS8kAmAMu7DW8JSrrKtaCGMZqQbtbGpgx" 
```



**異常**

```go
error: rpc error: code = Unknown desc = Insufficient tip, please increase the tip amount and try again, at least 1000000 lamports
```

