# Monad

### 介紹

{% hint style="info" %}
目前Monad RPC僅支持`eth_sendRawTransaction`方法。
{% endhint %}

本API用於在Monad主網上發送已簽名的交易，支持HTTP/HTTPS協議。



### 端點

{% tabs %}
{% tab title="HTTP" %}
<table><thead><tr><th width="151.77734375">地區</th><th>端點</th></tr></thead><tbody><tr><td>Frankfurt</td><td>http://frankfurt-monad.blockrazor.io</td></tr><tr><td>Virginia</td><td>http://virginia-monad.blockrazor.io</td></tr><tr><td>Tokyo</td><td>http://tokyo-monad.blockrazor.io</td></tr></tbody></table>
{% endtab %}

{% tab title="HTTPS" %}
<table><thead><tr><th width="133.1953125">地區</th><th>端點</th></tr></thead><tbody><tr><td>Frankfurt</td><td>https://frankfurt-monad.blockrazor.io</td></tr><tr><td>Virginia</td><td>https://virginia-monad.blockrazor.io</td></tr><tr><td>Tokyo</td><td>https://tokyo-monad.blockrazor.io</td></tr></tbody></table>
{% endtab %}
{% endtabs %}



### 流控说明

{% hint style="warning" %}
Monad發送交易的服務目前不和訂閱計劃綁定，如需使用該服務，請[聯繫](https://discord.com/invite/qqJuwRb8Nh)我們，我們會在第一時間處理
{% endhint %}



### 請求示例

{% tabs %}
{% tab title="CURL" %}
```bash
curl -X POST http://frankfurt-monad.blockrazor.io \
  -H "Authorization: <auth>" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_sendRawTransaction",
    "params": [
      "0xd46e……e8b2c1"
    ]
  }'
```
{% endtab %}

{% tab title="Go" %}
```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
)

type JsonRpcRequest struct {
	JsonRpc string        `json:"jsonrpc"`
	ID      int           `json:"id"`
	Method  string        `json:"method"`
	Params  []interface{} `json:"params"`
}

func main() {
	// Define request parameters and configuration
	url := "http://frankfurt-monad.blockrazor.io"
	authToken := "<auth-token>"        // Replace with your actual authorization token
	rawTransaction := "0xd46e……e8b2c1" // Replace with your actual raw transaction data

	// Construct the JSON-RPC request body
	requestBody := JsonRpcRequest{
		JsonRpc: "2.0",
		ID:      1,
		Method:  "eth_sendRawTransaction",
		Params:  []interface{}{rawTransaction},
	}

	// Marshal the struct into JSON format
	jsonBody, err := json.Marshal(requestBody)
	if err != nil {
		log.Fatalf("Error marshaling JSON: %v", err)
	}

	// Create HTTP client (Keep-Alive is supported by default)
	client := &http.Client{}

	// Create a new POST request
	req, err := http.NewRequest("POST", url, bytes.NewBuffer(jsonBody))
	if err != nil {
		log.Fatalf("Error creating request: %v", err)
	}

	// Set request Headers
	req.Header.Set("Authorization", authToken)
	req.Header.Set("Content-Type", "application/json")
	// Go's http.Client uses Keep-Alive by default when the connection can be reused.
	// Explicitly setting 'Connection: keep-alive' is optional,
	// but can be kept if the server has strict requirements for connection behavior.
	// The default Go behavior is usually sufficient in this case.
	// req.Header.Set("Connection", "keep-alive")

	// Send the request
	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("Error sending request: %v", err)
	}
	defer resp.Body.Close()

	// Process the response
	bodyBytes, _ := io.ReadAll(resp.Body)
	fmt.Printf("Status Code: %d\n", resp.StatusCode)
	fmt.Printf("Response Body: %s\n", bodyBytes)
	fmt.Printf("Connection Header: %s\n", resp.Header.Get("Connection"))
	fmt.Printf("Keep-Alive successful (requires server support).\n")

	// Second request
	// The client will attempt to use the same underlying TCP connection
	// established by the first request, thus implementing Keep-Alive.
	fmt.Println("\n--- Sending Request 2 (Connection Reuse Test) ---")

	// Reset the request body to prepare for resending
	req2, err := http.NewRequest("POST", url, bytes.NewBuffer(jsonBody))
	if err != nil {
		log.Fatalf("Error creating request 2: %v", err)
	}

	// Copy the same Headers
	req2.Header.Set("Authorization", authToken)
	req2.Header.Set("Content-Type", "application/json")

	// ⚠️ Crucial: Continue using the existing client, do not create a new one.
	resp2, err := client.Do(req2)
	if err != nil {
		log.Fatalf("Error sending request 2: %v", err)
	}
	defer resp2.Body.Close()

	bodyBytes2, _ := io.ReadAll(resp2.Body)
	fmt.Printf("Status Code 2: %d\n", resp2.StatusCode)
	fmt.Printf("Response Body 2: %s\n", bodyBytes2)
	fmt.Printf("Connection Header 2: %s\n", resp2.Header.Get("Connection"))
}
```
{% endtab %}

{% tab title="JS" %}
```javascript
// Import Node.js built-in modules
const http = require('http');

// --- Configuration ---
const url = "http://frankfurt-monad.blockrazor.io";
const authToken = "<auth-token>";         // Replace with your actual authorization token
const rawTransaction = "0xd46e……e8b2c1"; // Replace with your actual raw transaction data

// Define the JSON-RPC request body
const requestBody = {
    jsonrpc: "2.0",
    id: 1,
    method: "eth_sendRawTransaction",
    params: [rawTransaction]
};

// Stringify the request body into a JSON string
const jsonBody = JSON.stringify(requestBody);

// Create an HTTP Agent to control the connection pool, Keep-Alive is enabled by default
// Similar to the default http.Client in Go
const keepAliveAgent = new http.Agent({
    keepAlive: true,
    maxSockets: 5, // Max number of sockets can be adjusted as needed
});

async function sendJsonRpcRequest(requestUrl, body, token, agent, requestNumber) {
    console.log(`\n--- Sending Request ${requestNumber} ---`);
    try {
        const response = await fetch(requestUrl, {
            method: 'POST',
            // Note: fetch in Node.js requires explicit specification of the agent for connection reuse control
            agent: agent,
            headers: {
                'Authorization': token,
                'Content-Type': 'application/json',
                // 'Connection': 'keep-alive' is enabled by default, but can be explicitly added
            },
            body: body
        });

        // Check response status
        if (!response.ok) {
            console.error(`Request ${requestNumber} failed with status: ${response.status}`);
        }

        const responseText = await response.text();
        
        console.log(`Status Code ${requestNumber}: ${response.status}`);
        console.log(`Response Body ${requestNumber}: ${responseText}`);
        console.log(`Connection Header ${requestNumber}: ${response.headers.get('connection') || 'N/A'}`);
        // ⚠️ It's hard to directly confirm underlying TCP connection reuse in Node.js like in Go,
        // we rely on the Agent and server support.
        console.log(`Keep-Alive successful (requires server support and Agent configuration).`);

        return response;

    } catch (error) {
        console.error(`Error sending request ${requestNumber}:`, error.message);
        // Exit program or handle error
        process.exit(1); 
    }
}

// Main execution function
async function main() {
    // First request
    await sendJsonRpcRequest(url, jsonBody, authToken, keepAliveAgent, 1);

    // Second request
    // The client will attempt to use the same underlying TCP connection established by the first request,
    // thus implementing Keep-Alive.
    // ⚠️ Note: We reuse the keepAliveAgent object, not the request object itself.
    await sendJsonRpcRequest(url, jsonBody, authToken, keepAliveAgent, 2);

    // After completion, destroy the Agent to free up resources
    keepAliveAgent.destroy();
}

// Execute the main function
main();
```
{% endtab %}
{% endtabs %}



### 返回示例

**正常**

{% code overflow="wrap" %}
```json
{"jsonrpc":"2.0","id":0,"result":"0xfb2c5fc7d7b92e2b8ba43f079ce68b67c66e42633f6bf10ab762ace2b5ec47f6"}
```
{% endcode %}

**異常**

```json
{"jsonrpc":"2.0","error":{"code":-32603,"message":"Transaction nonce too low"},"id":1}
```



