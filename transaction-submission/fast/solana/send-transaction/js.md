---
description: 本文档介绍如何通过JS构建、发送Solana交易
---

# JS

{% hint style="info" %}
Solana發送交易的服務不和訂閱計劃綁定，可前往 [Authentication](../../../../authentication.md) 獲取API KEY，默认限流为1 TPS。如需提升限流標準，請[聯繫](https://discord.com/invite/qqJuwRb8Nh)我們，我們會在第一時間處理
{% endhint %}

## HTTP

{% tabs %}
{% tab title="JS" %}
```javascript
const axios = require('axios');
const web3 = require('@solana/web3.js');
const bs58 = require('bs58');

// ------------------ Configuration Constants ------------------
// BlockRazor relay endpoint address
const httpEndpoint = "http://frankfurt.solana.blockrazor.xyz:443/sendTransaction";
const healthEndpoint = "http://frankfurt.solana.blockrazor.xyz:443/health";
// Replace with your Solana RPC endpoint
const mainNetRPC = "";
// Replace with your authKey
const authKey = "";
// Replace with your private key (base58)
const privateKey = "";
// Replace with your target public key
const publicKey = "";
// Send mode
const mode = "fast";
// Safe window
const safeWindow = 5; // only take effect in sandwichMitigation mod
// Revert protection
const revertProtection = false;
// Transaction amount
const amount = 200_000;
// Tip amount
const tipAmount = 1000000;

const tipAccounts = [
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
];

// ------------------ Axios HTTP Client (Connection Reuse Enabled) ------------------
const httpClient = axios.create({
    timeout: 10000,
    headers: {
        'Content-Type': 'application/json',
        'apikey': authKey,
    },
    httpAgent: new (require('http').Agent)({ keepAlive: true }),
    httpsAgent: new (require('https').Agent)({ keepAlive: true }),
});

// ------------------ Periodic Health Ping to Keep Connection Alive ------------------
async function pingHealth() {
    try {
        const res = await httpClient.get(healthEndpoint);
        console.log(`Health result:`, res.data);
    } catch (err) {
        console.error('Health check failed:', err.message);
    }
}

// ------------------ Build and Send Transaction ------------------
async function sendTx() {
    const senderPrivateKey = new Uint8Array(bs58.decode(privateKey));
    const senderKeypair = web3.Keypair.fromSecretKey(senderPrivateKey);
    const receiver = new web3.PublicKey(publicKey);
    const tipAccount = new web3.PublicKey(tipAccounts[Math.floor(Math.random() * tipAccounts.length)]);

    const connection = new web3.Connection(mainNetRPC);
    const { blockhash } = await connection.getLatestBlockhash('finalized');

    const tx = new web3.Transaction()
        .add(web3.SystemProgram.transfer({
            fromPubkey: senderKeypair.publicKey,
            toPubkey: tipAccount,
            lamports: tipAmount,
        }))
        .add(web3.SystemProgram.transfer({
            fromPubkey: senderKeypair.publicKey,
            toPubkey: receiver,
            lamports: amount,
        }));

    tx.recentBlockhash = blockhash;
    tx.feePayer = senderKeypair.publicKey;
    tx.sign(senderKeypair);

    const serialized = tx.serialize();
    const base64Tx = serialized.toString('base64');

    const payload = {
        transaction: base64Tx,
        mode: mode,
        safeWindow: safeWindow,
		RevertProtection: revertProtection,
    };

    try {
        const res = await httpClient.post(httpEndpoint, payload);
        console.log('[send tx] response:', res.data);
    } catch (err) {
        console.error('SendTx failed:', err.response?.data || err.message);
    }
}

// ------------------ Main Entry ------------------
(async () => {
    // Initial health check (establish connection)
    await pingHealth();

    // Periodically send /health to keep connection alive
    setInterval(pingHealth, 30 * 1000);
    sendTx().catch(console.error);
})();
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

```
{"signature":"2DkHpsZxwbGDHkCujRMqd1jXMPaXfzn2JpwxZ43PoTk6Cj7hpxDp8VHESNCeuh95nVMmsWV4RGGWCkmGERZCTWHL","error":""}
```



**異常**

```
{"signature":"","error":"error: Authentication information is missing. Please provide a valid auth token"}
```



## gRPC

{% hint style="info" %}
[fast模式](https://github.com/BlockRazorinc/solana-trader-client-js/blob/main/mode_grpc_fast.js)\
[sandwichMitigation模式](https://github.com/BlockRazorinc/solana-trader-client-js/blob/main/mode_grpc_sandwichMitigation.js)
{% endhint %}

```javascript
const web3 = require("@solana/web3.js");
const bs58 = require("bs58");

const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

const PROTO_PATH = __dirname + '/server.proto';
const packageDefinition = protoLoader.loadSync(
    PROTO_PATH,
    {
        keepCase: true,
        longs: String,
        enums: String,
        defaults: true,
        oneofs: true
    }
);

const serverProto = grpc.loadPackageDefinition(packageDefinition).serverpb;

// BlockRazor relay endpoint address
const blzRelayEndpoint = "frankfurt.solana-grpc.blockrazor.xyz:80";
// replace your solana rpc endpoint
const mainNetRPC = "";
// replace your authKey
const authKey = "";
// relace your private key(base58)
const privateKey = "";
// send mode
const mode = "fast"; // set to "sandwichMitigation" to mitigate sandwich attacks on tx
// safeWindow
const safeWindow = 3; // only take effect in sandwichMitigation mode
// Revert protection
const revertProtection = false;
// tip amount
const tipAmount = 1000000;

const tipAccounts = [
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
];

const client = new serverProto.Server(
    blzRelayEndpoint,
    grpc.credentials.createInsecure()
);

var meta = new grpc.Metadata();

function getRandomAccount() {
    const randomIndex = Math.floor(Math.random() * tipAccounts.length);
    return tipAccounts[randomIndex];
}

// ------------------ Periodic Health Ping to Keep Connection Alive ------------------
async function pingHealth() {
    client.getHealth({}, meta, (err, response) => {
        if (err) {
            console.error('[get health] error:', err);
            return;
        }

        console.log('[get health] response:', response);
    });
}

(async () => {
    meta.add('apikey', authKey);

    // Initial health check (establish connection)
    await pingHealth();

    // Periodically send /health to keep connection alive
    setInterval(pingHealth, 30 * 1000);

    const senderPrivateKey = new Uint8Array(bs58.decode(privateKey));
    const senderKeypair = web3.Keypair.fromSecretKey(senderPrivateKey);

    const tipAccount = getRandomAccount();
    const recipientPublicKey = new web3.PublicKey(tipAccount);

    const transaction = new web3.Transaction().add(
        web3.SystemProgram.transfer({
            fromPubkey: senderKeypair.publicKey,
            toPubkey: recipientPublicKey,
            lamports: tipAmount,
        })
    );

    const connection = new web3.Connection(mainNetRPC);

    transaction.recentBlockhash = (await connection.getLatestBlockhash()).blockhash;
    transaction.feePayer = senderKeypair.publicKey;
    transaction.sign(senderKeypair);

    const serializedTransaction = transaction.serialize();
    const base64Tx = serializedTransaction.toString('base64');

    client.SendTransaction({ transaction: base64Tx, mode: mode, safeWindow: safeWindow, revertProtection: revertProtection}, meta, (err, response) => {
        if (err) {
            console.error('[send tx] error:', err);
            return;
        }

        console.log('[send tx] response:', response);
    });
})();
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

```javascript
[send tx] response: {signature: '3WbpmLkpgKC13XTFZDUCvjvC3uo5KincG3DTEgjWerQi1B1Frf4zwkDsEtnxBVqW3EgD1b38YWsTKAyFonRR6xSM'}
```



**異常**

```javascript
[send tx] error: Error: 2 UNKNOWN: Insufficient tip, please increase the tip amount and try again, at least 1000000 lamports
```

