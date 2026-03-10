# BSC

### 介紹

BSC Private Mempool推送BSC的隱私交易流數據。數據流基於SSE協議構建，數據流統一以bundle形式推送。數據流中的交易統一經脫敏處理，僅披露經授權允許披露的交易數據。

Private Mempool可應用於Backrun、跟單、狙擊等多種場景。

為避免由於網絡波動導致的數據斷流，建議建立重連機制。



### RPC端點

{% hint style="info" %}
請將訂閱bundle的域名與發送bundle的域名保持一致。如訂閱https://jp-bscscutum.blockrazor.xyz/stream，則將bundle發送至https://jp-bscscutum.blockrazor.xyz
{% endhint %}

{% hint style="info" %}
不同地區推送的隱私數據流不同，建議同時訂閱3個端點
{% endhint %}

<table><thead><tr><th width="107.42578125">地區</th><th>端點</th></tr></thead><tbody><tr><td>東京</td><td>https://jp-bscscutum.blockrazor.xyz/stream</td></tr><tr><td>紐約</td><td>https://us-bscscutum.blockrazor.xyz/stream</td></tr><tr><td>法蘭克福</td><td>https://ger-bscscutum.blockrazor.xyz/stream</td></tr><tr><td>都柏林</td><td>https://ire-bscscutum.blockrazor.xyz/stream</td></tr></tbody></table>



### Authentication

為對API的請求做認證，請設置auth token，請求示例如下：

```json
curl -X GET \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer <token>" \
    --data '{}' \
    https://jp-bscscutum.blockrazor.xyz/stream
```

```
https://jp-bscscutum.blockrazor.xyz/stream?token=<token>
```

示例中的\<token>需在註冊BlockRazor後獲取，步驟如下：

1. 前往[https://www.blockrazor.io](https://www.blockrazor.io)，在網頁右上角點擊【註冊】，系統跳轉至註冊頁
2. 在註冊頁輸入郵箱和密碼，點擊【註冊】，系統會向郵箱發送賬戶激活郵件
3. 前往郵箱，查看賬戶激活郵件，點擊賬戶激活鏈接
4. 成賬戶激活，前往登錄，查看賬戶信息，複製auth token



### 流控說明

|     | Tier 4 | Tier 3 | Tier 2    | Tier 1    | Tier 0    |
| --- | ------ | ------ | --------- | --------- | --------- |
| 數據流 | -      | -      | 2 streams | 2 streams | 2 streams |



### Bundle類型

**Raw Bundle**

Raw Bundle是指尚未被跟隨策略交易的bundle，Raw Bundle中的交易來源於兩個渠道：

* 通過RPC `eth_sendRawTransaction`提交的交易，會由BlockRazor RPC自動構建為bundle推送至Private Mempool，該場景下的Raw Bundle僅包含一筆交易；
* 通過RPC `eth_sendMevBundle`提交的Raw Bundle，交易來自於公開內存池或自行構建，該場景下的Raw Bundle至多可包含50筆交易。

**Followed Bundle**

客戶端在對Raw Bundle執行backrun、跟單或狙擊策略後，可以通過開啟hint繼續將bundle披露至Private Mempool以執行嵌套的backrun策略。此時Private Mempool中的該類Bundle稱為Followed Bundle，包含raw bundle中的全部交易，以及1筆策略交易。



### 數據流結構

**Bundle**

<table><thead><tr><th width="210">參數</th><th width="166">格式</th><th>備注</th></tr></thead><tbody><tr><td>chainID</td><td>string</td><td>ETH: 1, BSC:56</td></tr><tr><td>hash</td><td>string</td><td>bundle hash,Private Mempool數據推流統一以bundle形式呈現</td></tr><tr><td><a href="bsc.md#tx">txs</a></td><td><a href="bsc.md#tx">[]tx</a></td><td>bundle中包含的交易</td></tr><tr><td>nextBlockNumber</td><td>uint64</td><td>該bundle所在區塊號</td></tr><tr><td>maxBlockNumber</td><td>uint64</td><td>該bundle有效的最大區塊號</td></tr><tr><td>proxyBidContract</td><td>string</td><td>bundle競拍代理合約地址，競拍方法调用詳見 <a href="/broken/pages/9FTJWMwZJ35WZYpzRmdD">Backrun</a></td></tr><tr><td>refundAddress</td><td>string</td><td>競拍方法的入參， 競拍金額將按比例返利至refundAddress</td></tr><tr><td>refundCfg</td><td>int</td><td>競拍方法的入參</td></tr><tr><td>state</td><td><a href="bsc.md#state">[]state</a></td><td>虛擬機狀態對象的數據變化，<a href="bsc.md#shu-ju-liu-shi-li-bao-han-state">查看數據流示例</a></td></tr></tbody></table>

**txs**

<table><thead><tr><th width="212">參數</th><th width="166">格式</th><th>備注</th></tr></thead><tbody><tr><td>hash</td><td>string</td><td>交易哈希</td></tr><tr><td>from</td><td>string</td><td>交易的發起方地址</td></tr><tr><td>to</td><td>string</td><td>交易的接收方地址</td></tr><tr><td>value</td><td>hex</td><td>交易value</td></tr><tr><td>nonce</td><td>uint64</td><td>交易nonce</td></tr><tr><td>calldata</td><td>string</td><td>交易calldata</td></tr><tr><td>functionSelector</td><td>string</td><td>合約函數簽名哈希的前4個字節</td></tr><tr><td>logs</td><td><a href="bsc.md#log">[]log</a></td><td>交易在執行過程中拋出的事件日誌</td></tr></tbody></table>

**log**

<table><thead><tr><th width="210">參數</th><th width="164">格式</th><th>備注</th></tr></thead><tbody><tr><td>address</td><td>string</td><td>触发事件的智能合约地址</td></tr><tr><td>topics</td><td>[]string</td><td>事件日志的topcis</td></tr><tr><td>data</td><td>string</td><td>非索引参数的存储区域</td></tr></tbody></table>



#### **state**

{% hint style="info" %}
默認數據推流中不包含state，如需獲取，請將訂閱地址修改為

Ethereum：https://ethscutum.blockrazor.xyz/stream?state=true

BSC：https://jp-bscscutum.blockrazor.xyz/stream?state=true
{% endhint %}

<table><thead><tr><th width="212">參數</th><th width="166">格式</th><th>備註</th></tr></thead><tbody><tr><td>"0x7C3b……3cb9E2"</td><td>[]string</td><td>數據發生變化的狀態對象地址，可以是一個EOA地址或智能合約地址</td></tr><tr><td>"0x935b……6cf608"</td><td>string</td><td>狀態對象數據發生變化的Key</td></tr><tr><td>"0x0000……3ffc00"</td><td>string</td><td>狀態對象數據變化後的Value</td></tr></tbody></table>



### 數據流示例（默認）

```json
{
    "chainID":"56" //ETH: 1, BSC:56
    "hash":"0x2ba4c05436d4a48a0ce30341a3164b34b31c091a28ed62618f7b0512aba41f51" // bundle hash
    "txs":[{
          "hash":"0x2ba4c05436d4a48a0ce30341a3164b34b31c091a28ed62618f7b0512aba41f51"
          "from":"0xB4647b856CB9C3856d559C885Bed8B43e0846a47"
          "to":"0x0000000000000000000000000000000000001000"
          "value":"0x1c4eda9192000"
          "nonce":88036
          "calldata":"0xf340fa01000000000000000000000000b4647b856cb9c3856d559c885bed8b43e0846a47"
          "functionSelector":"0xe47d166c"
          "logs":[
              {
                "address": "0x6c1bcf1b99d9f0819459dad661795802d232437e",
                "topics": ["0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67", "0x0000000000000000000000000000000000000000000000000000000000000000", "0x0000000000000000000000000000000000000000000000000000000000000000"],
                "data": "0x"
              }
              {
                "address": "0x6c1bcf1b99d9f0819459dad661795802d232437e",
                "topics": ["0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67", "0x0000000000000000000000000000000000000000000000000000000000000000", "0x0000000000000000000000000000000000000000000000000000000000000000"],
                "data": "0x"
              }
          ]
    }]
    "nextBlockNumber":39177841  //該bundle所在區塊號
    "maxBlockNumber":39177941  //該bundle有效的最大區塊號
    "proxyBidContract":"0x74Ce839c6aDff544139f27C1257D34944B794605" //Backrun競拍合約地址，調用合約的proxyBid方法可進行競拍
    "refundAddress":"0x6c1bcf1b99d9f0819459dad661795802d232437e", //返利接收地址，競拍金額將按比例返利至refundAddress
    "refundCfg":10380050 //返利配置
}
```



### 數據流示例（包含state）

{% hint style="info" %}
默認數據推流中不包含state，如需獲取，請將訂閱地址修改為

Ethereum：[https://eth.blockrazor.xyz/stream?state=true](https://eth.blockrazor.xyz/stream?state=true)

BSC：[https://bsc.blockrazor.xyz/stream?state=true](https://bsc.blockrazor.xyz/stream?state=true)
{% endhint %}

```json
{
    "chainID":"56" //ETH: 1, BSC:56
    "hash":"0x2ba4c05436d4a48a0ce30341a3164b34b31c091a28ed62618f7b0512aba41f51" // bundle hash
    "txs":[{
          "hash":"0x2ba4c05436d4a48a0ce30341a3164b34b31c091a28ed62618f7b0512aba41f51"
          "from":"0xB4647b856CB9C3856d559C885Bed8B43e0846a47"
          "to":"0x0000000000000000000000000000000000001000"
          "value":"0x1c4eda9192000"
          "nonce":88036
          "calldata":"0xf340fa01000000000000000000000000b4647b856cb9c3856d559c885bed8b43e0846a47"
          "functionSelector":"0xe47d166c"
          "logs":[
              {
                "address": "0x6c1bcf1b99d9f0819459dad661795802d232437e",
                "topics": ["0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67", "0x0000000000000000000000000000000000000000000000000000000000000000", "0x0000000000000000000000000000000000000000000000000000000000000000"],
                "data": "0x"
              }
              {
                "address": "0x6c1bcf1b99d9f0819459dad661795802d232437e",
                "topics": ["0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67", "0x0000000000000000000000000000000000000000000000000000000000000000", "0x0000000000000000000000000000000000000000000000000000000000000000"],
                "data": "0x"
              }
          ]
    }]
    "nextBlockNumber":39177841  //該bundle所在區塊號
    "maxBlockNumber":39177941  //該bundle有效的最大區塊號
    "proxyBidContract":"0x74Ce839c6aDff544139f27C1257D34944B794605" //Scutum的bundle競拍合約地址，調用合約的proxyBid方法可進行競拍
    "refundAddress":"0x6c1bcf1b99d9f0819459dad661795802d232437e", //返利接收地址，競拍金額將按比例返利至refundAddress
    "refundCfg":10380050 //返利配置
    "state": {
	"0x7C3b00CB3B40Cc77d88329A58574E29cFA3cb9E2": { //數據發生變化的狀態對象地址，可以是一個EOA地址或智能合約地址      
	      "0x935b605129a438014d6ae0692623c5e1fbf83d5a631f5a0f8489a301966cf608": "0x00000000000000000000000000000000000000000000010c86a7e418723ffc00"
	      //"狀態對象數據發生變化的Key":"狀態對象數據變化後的Value"
            }      
      }
}
```

