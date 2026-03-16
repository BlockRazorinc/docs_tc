# Go

### 快速開始

請參考[README.md](https://github.com/BlockRazorinc/geyserstream-client-go/blob/main/README.md)



### 代碼示例

```go
package main

import (
	"context"
	"crypto/x509"
	"encoding/json"
	"io"
	"log"
	"time"

	"github.com/BlockRazorinc/geyserstream-client-go/pb"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"
	"google.golang.org/grpc/credentials/insecure"
	"google.golang.org/grpc/keepalive"
	"google.golang.org/grpc/metadata"
)

var (
	// grpc server address
	grpcAddr = "geyserstream-tokyo.blockrazor.xyz:443"
	// auth token
	token = ""

	// Subscribe to block update
	subscriptBlocks = false

	// Subscribe to accounts
	subscriptAccounts   = false
	accountsFilter      = []string{}
	accountOwnersFilter = []string{}

	// Subscribe to transactions, required for tx_account_include/tx_account_exclude and vote/failed.
	subscriptTransactions       = false
	failedTransactions          = true
	voteTransactions            = true
	transactionsAccountsInclude = []string{}
	transactionsAccountsExclude = []string{}
)

var kacp = keepalive.ClientParameters{
	Time:                10 * time.Second, // send pings every 10 seconds if there is no activity
	Timeout:             time.Second,      // wait 1 second for ping ack before considering the connection dead
	PermitWithoutStream: true,             // send pings even without active streams
}

func grpc_connect(address string, plaintext bool) *grpc.ClientConn {
	var opts []grpc.DialOption
	if plaintext {
		opts = append(opts, grpc.WithTransportCredentials(insecure.NewCredentials()))
	} else {
		pool, _ := x509.SystemCertPool()
		creds := credentials.NewClientTLSFromCert(pool, "")
		opts = append(opts, grpc.WithTransportCredentials(creds))
	}

	opts = append(opts, grpc.WithKeepaliveParams(kacp))

	log.Println("Starting grpc client, connecting to", address)
	conn, err := grpc.Dial(address, opts...)
	if err != nil {
		log.Fatalf("fail to dial: %v", err)
	}

	return conn
}

func grpc_subscribe(conn *grpc.ClientConn) {
	var err error
	client := pb.NewGeyserClient(conn)

	var subscription pb.SubscribeRequest

	if subscriptBlocks {
		if subscription.Blocks == nil {
			subscription.Blocks = make(map[string]*pb.SubscribeRequestFilterBlocks)
		}
		subscription.Blocks["blocks"] = &pb.SubscribeRequestFilterBlocks{}
	}

	if (len(accountsFilter)+len(accountOwnersFilter)) > 0 || (subscriptAccounts) {
		if subscription.Accounts == nil {
			subscription.Accounts = make(map[string]*pb.SubscribeRequestFilterAccounts)
		}

		subscription.Accounts["account_sub"] = &pb.SubscribeRequestFilterAccounts{}

		if len(accountsFilter) > 0 {
			subscription.Accounts["account_sub"].Account = accountsFilter
		}

		if len(accountOwnersFilter) > 0 {
			subscription.Accounts["account_sub"].Owner = accountOwnersFilter
		}
	}

	// Set up the transactions subscription
	if subscription.Transactions == nil {
		subscription.Transactions = make(map[string]*pb.SubscribeRequestFilterTransactions)
	}

	// Subscribe to generic transaction stream
	if subscriptTransactions {
		subscription.Transactions["transactions_sub"] = &pb.SubscribeRequestFilterTransactions{
			Failed: &failedTransactions,
			Vote:   &voteTransactions,
		}

		subscription.Transactions["transactions_sub"].AccountInclude = transactionsAccountsInclude
		subscription.Transactions["transactions_sub"].AccountExclude = transactionsAccountsExclude
	}

	subscriptionJson, err := json.Marshal(&subscription)
	if err != nil {
		log.Printf("Failed to marshal subscription request: %v", subscriptionJson)
	}
	log.Printf("Subscription request: %s", string(subscriptionJson))

	// Set up the subscription request
	ctx := context.Background()
	if token != "" {
		md := metadata.New(map[string]string{"x-token": token})
		ctx = metadata.NewOutgoingContext(ctx, md)
	}

	stream, err := client.Subscribe(ctx)
	if err != nil {
		log.Fatalf("%v", err)
	}
	err = stream.Send(&subscription)
	if err != nil {
		log.Fatalf("%v", err)
	}

	for {
		resp, err := stream.Recv()
		timestamp := time.Now().UnixNano()

		if err == io.EOF {
			return
		}
		if err != nil {
			log.Fatalf("Error occurred in receiving update: %v", err)
		}

		log.Printf("%v %v", timestamp, resp)
	}
}
func main() {
	conn := grpc_connect(grpcAddr, false)
	defer conn.Close()

	grpc_subscribe(conn)
}
```

