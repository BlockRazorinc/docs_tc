# Send Transaction

### 介紹

{% hint style="warning" %}
Solana發送交易的服務不和訂閱計劃綁定，可前往 [Authentication](../../../../authentication.md) 獲取API KEY，默认限流为3 TPS。如需提升限流標準，請[聯繫](https://discord.com/invite/qqJuwRb8Nh)我們，我們會在第一時間處理
{% endhint %}

`Send Transaction` 用於在Solana上發送已簽名的交易，支持HTTP和gRPC協議。



### 端點

**HTTP**

<table><thead><tr><th width="108.7109375">地区</th><th>URL</th></tr></thead><tbody><tr><td>法蘭克福</td><td>http://frankfurt.solana.blockrazor.xyz:443/sendTransaction</td></tr><tr><td>紐約</td><td>http://newyork.solana.blockrazor.xyz:443/sendTransaction</td></tr><tr><td>東京</td><td>http://tokyo.solana.blockrazor.xyz:443/sendTransaction</td></tr><tr><td>阿姆斯特丹</td><td>http://amsterdam.solana.blockrazor.xyz:443/sendTransaction</td></tr><tr><td>倫敦</td><td>http://london.solana.blockrazor.xyz:443/sendTransaction</td></tr></tbody></table>

#### gRPC

<table><thead><tr><th width="115.80859375">地区</th><th width="544.58984375">URL</th></tr></thead><tbody><tr><td>法蘭克福</td><td>frankfurt.solana-grpc.blockrazor.xyz:80</td></tr><tr><td>紐約</td><td>newyork.solana-grpc.blockrazor.xyz:80</td></tr><tr><td>東京</td><td>tokyo.solana-grpc.blockrazor.xyz:80</td></tr><tr><td>阿姆斯特丹</td><td>amsterdam.solana-grpc.blockrazor.xyz:80</td></tr><tr><td>倫敦</td><td>london.solana-grpc.blockrazor.xyz:80</td></tr></tbody></table>



### 流控說明

{% hint style="info" %}
Solana發交易服務已不和訂閱計劃綁定，可前往 [Authentication](../../../../authentication.md) 獲取API KEY，默认限流为3 TPS。如需提升Solana發交易的限流標準，請[聯繫](https://discord.com/invite/qqJuwRb8Nh)我們，我們會在第一時間處理
{% endhint %}



### 交易構建示例

* [Go](go.md)
* [Rust](rust.md)
* [JS](js.md)



### 请求参数

<table><thead><tr><th width="103.18359375">字段</th><th width="77.1875">必填</th><th width="125.62890625">示例</th><th>備注</th></tr></thead><tbody><tr><td>transaction</td><td>是</td><td>"4hXTCk……tAnaAT"</td><td>已完成簽名的交易，兼容base 64和base 58的編碼格式，建議用base 64</td></tr><tr><td>mode</td><td>否</td><td>"fast"<br>"sandwichMitigation"</td><td>BlockRazor支持fast和sandwichMitigation兩種模式，默認為fast模式。<br><br>在fast模式中，交易會基於全球分布式高性能網絡和高質量SWQoS質押鏈路被飽和式發送，以最低延遲到達Leader節點。<br><br>在sandwichMitigation模式中，交易會被發往BlockRazor高度信任的SWQoS質押鏈路，同時交易會跳過黑名單Leader(經BlockRazor三明治監測機制動態精確識別)的slot。在此模式下，<strong>請不要用</strong>durable nonce發送交易，這會使三明治保護失效。</td></tr><tr><td>safeWindow</td><td>否</td><td>3</td><td>sandwichMitigation模式中用於確定交易發送時機的參數，數字代表從當前slot起連續白名單驗證者的slot數量，比如設定3，則交易會僅在當前起連續3個slot都屬於白名單驗證者時發送。<br><br>safeWindow的參數範圍是3-13，數字越大防治三明治攻擊效果越好，但可能會對上鏈速度有一定影響。如不設定，則默認為3。</td></tr><tr><td>revertProtection</td><td>否</td><td>false</td><td>默認為false。如設置為true，交易不會在鏈上執行失敗，但上鏈速度会受到影响且存在无法上链的可能，請根據實際需求謹慎選擇開啓。</td></tr></tbody></table>



### **Priority** Fee

Priority Fee是Solana在Base Fee（發送交易的最低成本，交易中每包含一個簽名花費5000 Lamports）基礎上的額外交易費用。由於計算資源有限，Leader節點在出塊時主要按交易價值對交易進行排序，Priority Fee越高的交易被優先納入下個區塊的概率越高。建議在發送交易時將CU Price至少設置為1,000,000。



### Tip

在構建交易時，需在交易中添加Tip轉賬指令（建議將Tip指令放在靠前位置），用於進一步加速交易上鍊。BlockRazor不從Tip中收取服務費。Tip指令轉賬金額至少為100,000 Lamports（0.0001 Sol），建議將Tip置為[`getTransactionfee`](../../../../streams/network-fee-stream/solana/get-transactionfee.md)返回的推薦值，接收Tip的账户地址為：

```
"FjmZZrFvhnqqb9ThCuMVnENaM3JGVuGWNyCAxRJcFpg9",
"6No2i3aawzHsjtThw81iq1EXPJN6rh8eSJCLaYZfKDTG",
"A9cWowVAiHe9pJfKAj3TJiN9VpbzMUq6E4kEvf5mUT22",
"Gywj98ophM7GmkDdaWs4isqZnDdFCW7B46TXmKfvyqSm",
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
```

{% hint style="info" %}
為盡量避免因地址佔用引起交易處理性能下降，導致交易延遲，請盡量在發交易時輪換Tip賬戶地址。
{% endhint %}



### Keep Alive

請發送 POST 請求到健康檢查端點以保持連線活躍，請求示例如下：

{% tabs %}
{% tab title="CURL" %}
```bash
curl -X POST 'http://frankfurt.solana.blockrazor.xyz:443/health' \
-H "Content-Type: application/json" \
-H "apikey: <auth_token>" \
-d ""
```
{% endtab %}
{% endtabs %}

