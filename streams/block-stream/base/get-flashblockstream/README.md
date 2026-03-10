# Get FlashBlockStream

## 介紹

本方法用於獲取Base的FlashBlock數據，支持gRPC和WebSocket協議。

Flashblocks 是 Base 上每 200 毫秒流式傳輸的“子區塊”（sub-blocks），實現比標準 2 秒區塊時間快 10 倍 的交易預確認。這些子區塊被稱為 Flashblocks，包含大約一個完整區塊交易數據的 10%，允許應用程序幾乎即時獲得交易反饋，適用於需要低延遲的場景。



## 端點

{% tabs %}
{% tab title="gRPC" %}
<table><thead><tr><th width="136.83203125">地區</th><th>端點</th></tr></thead><tbody><tr><td>法蘭克福</td><td>frankfurt.grpc.base.blockrazor.xyz:80</td></tr><tr><td>弗吉尼亞</td><td>virginia.grpc.base.blockrazor.xyz:80</td></tr><tr><td>東京</td><td>tokyo.grpc.base.blockrazor.xyz:80</td></tr></tbody></table>
{% endtab %}

{% tab title="WebSocket" %}
<table><thead><tr><th width="139.7421875">地區</th><th>端點</th></tr></thead><tbody><tr><td>法蘭克福</td><td>ws://frankfurt.base.blockrazor.xyz:81/ws</td></tr><tr><td>弗吉尼亞</td><td>ws://virginia.base.blockrazor.xyz:81/ws</td></tr><tr><td>日本</td><td>ws://tokyo.base.blockrazor.xyz:81/ws</td></tr></tbody></table>
{% endtab %}
{% endtabs %}



## 限流

<table><thead><tr><th width="171.44140625"></th><th width="97.92578125">Tier 4</th><th width="90.2578125">Tier 3</th><th width="94.734375">Tier 2</th><th width="129.59765625">Tier 1</th><th>Tier 0</th></tr></thead><tbody><tr><td>FlashBlockStream</td><td>-</td><td>-</td><td>-</td><td>✅</td><td>✅</td></tr></tbody></table>



## 請求示例

{% tabs %}
{% tab title="gRPC" %}
[查看](https://github.com/BlockRazorinc/base-api-client-go/blob/c4bec3d65e55ffb0da07253fa78aefe1b1c07e33/main.go#L93)示例

```go
// GetFlashBlockStream provides a simplified example of subscribing to and processing the flash block stream.
// Note: This function attempts to connect and subscribe only once. For production use, implement your own reconnection logic.
func GetFlashBlockStream(authToken string) {
	log.Printf("[FlashStream] Attempting to connect to gRPC server at %s...", grpcAddr)

	// Establish a connection to the gRPC server with a timeout.
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	conn, err := grpc.DialContext(ctx, grpcAddr,
		grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Printf("[FlashStream] Failed to connect to gRPC server: %v", err)
		return
	}
	defer conn.Close()

	log.Println("[FlashStream] Successfully connected to gRPC server.")
	client := basepb.NewBaseApiClient(conn)

	// Create a new context with authentication metadata for the stream subscription.
	streamCtx := metadata.NewOutgoingContext(context.Background(), metadata.Pairs("authorization", authToken))
	stream, err := client.GetRawFlashBlockStream(streamCtx, &basepb.GetRawFlashBlocksStreamRequest{})
	if err != nil {
		log.Printf("[FlashStream] Failed to subscribe to stream: %v", err)
		return
	}

	log.Println("[FlashStream] Subscription successful. Waiting for new flash blocks...")

	// Loop indefinitely to receive messages from the stream.
	for {
		block, err := stream.Recv()
		if err != nil {
			if err == io.EOF {
				log.Println("[FlashStream] Stream closed by the server (EOF).")
			} else {
				log.Printf("[FlashStream] An error occurred while receiving data: %v", err)
			}
			break // Exit the loop on error or stream closure.
		}

		// Process the received flash block data.
		jsonString, err := ParseFlashBlockByte(block.Message)
		if err != nil {
			log.Printf("[FlashStream] Failed to parse flash block data: %v", err)
			continue
		}

		var jsonMap map[string]interface{}
		if err := json.Unmarshal([]byte(jsonString), &jsonMap); err != nil {
			log.Printf("[FlashStream] Failed to unmarshal flash block JSON: %v", err)
			continue
		}
		printPretty(jsonMap)
	}
}
```
{% endtab %}

{% tab title="WebSocket-Go" %}
[查看](https://github.com/BlockRazorinc/base-api-client-go/blob/c4bec3d65e55ffb0da07253fa78aefe1b1c07e33/main.go#L179)示例

```go
// GetWebSocketFlashBlockStream provides a simplified example of using the WebSocket API to subscribe to the flash block stream.
// Note: This function attempts to connect only once and will exit on a read error. For production use, implement your own reconnection logic.
func GetWebSocketFlashBlockStream(authToken string) {
	// Set up the HTTP header with the authorization token.
	header := http.Header{}
	header.Set("Authorization", authToken)

	// Dial the WebSocket server.
	dialer := websocket.DefaultDialer
	conn, resp, err := dialer.Dial(websocketAddr, header)
	if err != nil {
		if resp != nil {
			log.Fatalf("[WebSocket] Dial failed: %v (HTTP status: %s)", err, resp.Status)
		}
		log.Fatalf("[WebSocket] Dial failed: %v", err)
	}
	defer conn.Close()
	log.Printf("[WebSocket] Successfully connected to %s", websocketAddr)

	// Prepare the JSON-RPC subscription request.
	req := map[string]interface{}{
		"jsonrpc": "2.0",
		"method":  "subscribe_FlashBlock",
		"params":  []interface{}{},
		"id":      1,
	}
	reqB, _ := json.Marshal(req)
	if err := conn.WriteMessage(websocket.TextMessage, reqB); err != nil {
		log.Fatalf("[WebSocket] Failed to send subscription request: %v", err)
	}
	log.Printf("[WebSocket] Subscription request sent: %s", string(reqB))

	// Loop indefinitely to read messages from the server.
	for {
		msgType, msg, err := conn.ReadMessage()
		if err != nil {
			log.Printf("[WebSocket] Error reading message: %v", err)
			return // Exit the function on any read error.
		}
		if msgType != websocket.TextMessage && msgType != websocket.BinaryMessage {
			continue // Ignore messages that are not text or binary.
		}

		// Parse the outer JSON-RPC response wrapper.
		var outer = &FlashBlockWebSocketResponse{}
		if err := json.Unmarshal(msg, &outer); err != nil {
			log.Printf("[WebSocket] Failed to parse JSON from server: %v\nRaw data: %s", err, string(msg))
			continue
		}

		// Extract the "result" field, which contains the actual flash block data, and process it.
		resultRaw := outer.Result
		if jsonString, err := ParseFlashBlockByte(resultRaw); err == nil {
			printPretty(jsonString)
		} else {
			log.Printf("[WebSocket] Received message without a valid result field. Raw data: %s", string(msg))
		}
	}
}
```
{% endtab %}

{% tab title="WebSocket-Cli" %}
```bash
wscat -H "Authorization: <AUTH_HEADER>" \
  -c ws://frankfurt.base.blockrazor.xyz:81/ws \
  --wait 1000 \
  --execute '{"jsonrpc": "2.0", "id": 1, "method": "subscribe_FlashBlock", "params": []}'
```
{% endtab %}

{% tab title="JS" %}
```javascript
const WebSocket = require('ws'); // WebSocket library for Node.js, install via npm(npm install ws) if not already installed
const zlib = require('zlib'); // Node.js built-in library for compression/decompression

// --- Configuration Parameters ---
// Replace with your actual authorization token
const AUTH_TOKEN = "<YOUR_AUTH_TOKEN>";
const WS_URL = "<WEBSOCKET_URL>"; // websocket URL

const SUBSCRIPTION_MESSAGE = {
    "jsonrpc": "2.0",
    "id": 1,
    "method": "subscribe_FlashBlock",
    "params": []
};
const WAIT_TIME_MS = 1000; // Time to wait before sending the subscription message


/**
 * Decompresses a Buffer using the Brotli algorithm.
 * @param {Buffer} dataBuffer - The data buffer to be decompressed.
 * @returns {Promise<string>} - A promise that resolves with the decompressed string.
 */
function ParseBrotliData(dataBuffer) {
    return new Promise((resolve, reject) => {
        // zlib.brotliDecompress is used for Brotli decompression
        zlib.brotliDecompress(dataBuffer, (err, decompressedBuffer) => {
            if (err) {
                // Return error if decompression fails
                return reject(new Error("Brotli decompression failed."));
            }
            // Convert the decompressed buffer to a string and resolve
            resolve(decompressedBuffer.toString('utf8'));
        });
    });
}

// --- WebSocket Connection and Operations ---

function connectWebSocket() {
    console.log(`Attempting to connect to: ${WS_URL}`);

    // Create custom Headers for authorization
    const headers = {
        'Authorization': AUTH_TOKEN
    };

    // Establish WebSocket connection, passing headers
    const ws = new WebSocket(WS_URL, {
        headers: headers
    });

    // 1. Connection established successfully
    ws.on('open', () => {
        console.log('✅ Connection established.');

        // Mimicking --wait 1000, wait 1 second before sending the message
        setTimeout(() => {
            const messageString = JSON.stringify(SUBSCRIPTION_MESSAGE);
            
            console.log(`➡️ Sending subscription message (after waiting ${WAIT_TIME_MS}ms):`);
            console.log(messageString);
            
            ws.send(messageString);
        }, WAIT_TIME_MS);
    });

    // 2. Message received
    ws.on('message', async (data) => {
        // Data is a Buffer received from the server, which we suspect is uncompressed JSON string.
        const rawJsonString = data.toString('utf8');
        
        try {
            // STEP 1: Parse the outer JSON-RPC wrapper
            const rpcResponse = JSON.parse(rawJsonString);

            // Check if 'result' or 'params' field exists and contains the Base64 data
            const base64Data = rpcResponse.result || (rpcResponse.params && rpcResponse.params.data);

            if (!base64Data || typeof base64Data !== 'string') {
                console.log(`\n⬅️ Received non-compressed JSON message:`);
                console.log(JSON.stringify(rpcResponse, null, 2));
                return;
            }

            console.log(`\n⬅️ Received compressed data in JSON wrapper. Base64 length: ${base64Data.length}`);

            // STEP 2: Decode Base64 string back into raw Buffer
            const rawBrotliBuffer = Buffer.from(base64Data, 'base64');
            console.log(`   Decoded Base64 to Buffer. Brotli Buffer size: ${rawBrotliBuffer.length} bytes.`);
            
            // STEP 3: Decompress the raw Brotli Buffer
            const decompressedString = await ParseBrotliData(rawBrotliBuffer);
            
            console.log("🌟 Successfully decompressed and parsed message:");

            // STEP 4: Parse the inner JSON content
            const finalData = JSON.parse(decompressedString);
            console.log(JSON.stringify(finalData, null, 2));

        } catch (e) {
            // Handle any error during the 4 steps (JSON parse, Base64 decode, Brotli decompress, final JSON parse)
            console.error("\n❌ Error during processing compressed payload:");
            console.error(`   Error message: ${e.message}`);
            console.log(`   Raw received string (first 200 chars): ${rawJsonString.substring(0, 200)}...`);
        }
    });

    // 3. Connection closed
    ws.on('close', (code, reason) => {
        console.log(`\n❌ Connection closed. Code: ${code}, Reason: ${reason.toString()}`);
    });

    // 4. Connection error
    ws.on('error', (error) => {
        console.error(`\n🔥 An error occurred: ${error.message}`);
    });
}

// Start the client
connectWebSocket();
```
{% endtab %}
{% endtabs %}

#### [proto](https://github.com/BlockRazorinc/base-api-client-go/blob/1d46c2983420d6da645992a9f3ed51688f7dac88/proto/BaseApi.proto)

```go
syntax = "proto3";

option go_package = "./basepb";


import "google/protobuf/wrappers.proto";

message BaseBlock {
  string parent_hash = 1;
  string fee_recipient = 2;
  bytes state_root = 3;
  bytes receipts_root = 4;
  bytes logs_bloom = 5;
  bytes prev_randao = 6;
  uint64 block_number = 7;
  uint64 gas_limit = 8;
  uint64 gas_used = 9;
  uint64 timestamp = 10;
  bytes extra_data = 11;
  repeated uint64 base_fee_per_gas = 12;
  string block_hash = 13;
  repeated bytes transactions = 14;

  repeated Withdrawal withdrawals = 15;
  google.protobuf.UInt64Value blob_gas_used = 16;
  google.protobuf.UInt64Value excess_blob_gas = 17;
  google.protobuf.BytesValue withdrawals_root = 18;
}

message Withdrawal {
  uint64 index = 1;
  uint64 validator = 2;
  bytes address = 3;
  uint64 amount = 4;
}

message GetRawFlashBlocksStreamRequest {
}

message GetBlockStreamRequest {
}

message SendTransactionRequest {
  string rawTransaction = 1;
}

message SendTransactionResponse {
  string txHash = 1;
}

message FlashBlockStrRequest {
}

message RawFlashBlockStrResponse {
  bytes message = 1;
}

service BaseApi {
  rpc SendTransaction(SendTransactionRequest) returns (SendTransactionResponse);
  rpc GetBlockStream(GetBlockStreamRequest) returns (stream BaseBlock);
  rpc GetRawFlashBlockStream(GetRawFlashBlocksStreamRequest) returns (stream RawFlashBlockStrResponse);
}
```



## 返回示例

**正常**

```json
{
   message: "185329……7e04b7"
}
```



**异常**

```
rpc error: code = Unknown desc = Authentication information is missing. Please provide a valid auth token
```

