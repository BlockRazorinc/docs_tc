# Searcher

### 介绍

Searcher可以在BlockRazor RPC提供的Private Mempool上[訂閱交易](../../../streams/mempool/private-mempool/)執行backrun策略，再將[backrun bundle](searcher.md#qing-qiu-shi-li)發送至BlockRazor RPC參與競拍獲得收益。

另外，Searcher也可以跳過bundle訂閱，直接將bundle發送至BlockRazor RPC，憑借高性能網絡的網絡加速服務，BlockRazor RPC可以極低延遲將bundle轉發至主流builders，无需重复对接builder接口。



### RPC端點

{% hint style="info" %}
請將訂閱bundle的域名與發送bundle的域名保持一致。如訂閱https://jp-ethscutum.blockrazor.xyz/stream，則將bundle發送至https://jp-ethscutum.blockrazor.xyz
{% endhint %}

<table><thead><tr><th width="148.99609375">地區</th><th>端點</th></tr></thead><tbody><tr><td>東京</td><td>https://jp-ethscutum.blockrazor.xyz</td></tr><tr><td>紐約</td><td>https://us-ethscutum.blockrazor.xyz</td></tr></tbody></table>



### eth\_sendBid

{% hint style="info" %}
為保護隱私交易，`eth_sendBid`採用准入制，如需發送bid請於我們[聯繫](https://discord.com/invite/qqJuwRb8Nh)。
{% endhint %}

#### 請求參數

<table><thead><tr><th width="128.52734375">參數</th><th width="67.5546875">必選</th><th width="116.62890625">格式</th><th width="129.56640625">示例</th><th>備註</th></tr></thead><tbody><tr><td>txs</td><td>是</td><td>[]string</td><td>["tx_hash", "rawTxHex"]</td><td><ul><li>tx_hash為數據流中的用戶交易，rawTxHex為自行構建的backrun交易</li><li>如backrun對象是一個bundle，請填寫bundle中的所有tx_hash</li></ul></td></tr><tr><td>blockNumber</td><td>否</td><td>hex string</td><td>"0x102286B"</td><td>一个十六进制编码的区块号，表示bundle在该区块号之前有效。默认值为当前区块号 + 100</td></tr></tbody></table>

#### Bid機制

BlockRazor會將Bid的`tx_hash`替換為rawTxHex，並在添加`refundPercent`和`refundRecipient`後立即轉發至Builder出塊。對於Builder而言，Bid價值量為backrun交易的`priority fee + coinbase.transfer()` ，由於同一筆用戶交易的`refundPercent`相同，因此價值量最高的Bid將成功上鏈。



#### 請求示例

**Backrun Originator Transaction**

```bash
curl https://eth.blockrazor.xyz \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  --data '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_sendBid",
    "params": [
      {
        "txs": [
          "0x3a3c086bad1e765076986814939d2be70399e85ff75e4da8ec730aaf6ce3fea1",
          "0x02f86d0182149c808411e1a300830484e294fef10de0823f58df4f5f24856ab4274ededa6a5c8084c179306cc001a075d1f8fea39352c2da065aeef85be4b24789953f57a62ca7ad29b45bfc1e362aa043de44543083d0dafb80eaedd5764d6ef03336ff028ef02f238bccd5dfa9ece9"
        ],
        "blockNumber": "0x102286B"
      }
    ]
  }'
```

**Backrun Originator Bundle**

```bash
curl https://eth.blockrazor.xyz \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  --data '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_sendBid",
    "params": [
      {
        "txs": [
          "0x3a3c086bad1e765076986814939d2be70399e85ff75e4da8ec730aaf6ce3fea1",
          "0x47816242f30167d1525a98801355f49370fc83049cb070d5d3ae9fd6c83d2d9d",
          "0x02f86d0182149c808411e1a300830484e294fef10de0823f58df4f5f24856ab4274ededa6a5c8084c179306cc001a075d1f8fea39352c2da065aeef85be4b24789953f57a62ca7ad29b45bfc1e362aa043de44543083d0dafb80eaedd5764d6ef03336ff028ef02f238bccd5dfa9ece9"
        ],
        "blockNumber": "0x102286B"
      }
    ]
  }'
```



#### 返回示例

```json
{
    "id": 1,
    "jsonrpc": "2.0",
    "result": "0x164d7d41f24b7f333af3b4a70b690cf93f636227165ea2b699fbb7eed09c46c7"
}
```



### eth\_sendBundle

Searcher也可以作為項目方發送raw bundle，詳見[項目方](xiang-mu-fang.md)











