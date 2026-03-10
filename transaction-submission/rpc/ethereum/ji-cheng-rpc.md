# 集成RPC

### 介紹

{% hint style="info" %}
項目方無需訂閱計劃即可使用項目專屬RPC
{% endhint %}

BlockRazor RPC可以為Ethereum上的Wallet、DEX、Trading Bot等項目提供MEV保護，支持返利至項目方用戶地址或項目方指定地址。

BlockRazor RPC為每個項目方提供專屬的RPC URL，支持可視化自定義RPC域名、交易披露、返利地址和Revert保護等參數，便於項目方低成本快速集成RPC。



### 如何將RPC集成到項目中

#### 1. 配置RPC

1. 在[blockrazor.io](https://blockrazor.io/)完成註冊並登錄控制台
2. 在RPC模塊下點擊RPC配置頁，查看專屬RPC的配置信息
3. 點擊 **更新**，進入配置更新頁，根據需求調整參數，參數含義見如下表格

<table><thead><tr><th width="183">參數</th><th>含義</th></tr></thead><tbody><tr><td>默認RPC URL</td><td>每個賬號默認自動生成1個Ethereum RPC和1個BSC RPC。默認RPC由系統自動生成，URL無法修改</td></tr><tr><td>自定義RPC URL</td><td>自定義RPC URL支持修改三級域名，可用於向項目的終端用戶推廣，引導用戶在錢包中添加自定義RPC</td></tr><tr><td>披露</td><td>系統默認將交易的hash和logs分享給Searcher，分享字段越多獲得返利的可能性越大，請在評估交易隱私披露必要性後謹慎操作</td></tr><tr><td>返利地址</td><td>默認返利地址為tx.origin，即會將返利返給交易的發起者，可修改為固定返利地址（EOA）</td></tr><tr><td>Revert保護</td><td>默認開revert保護，如交易在實際執行中發現revert，則不會納入區塊。為確保快速納入區塊，建議在以太坊的交易中設置一定的priority fee。</td></tr></tbody></table>

4. 點擊  **確認**，系統將實時更新RPC配置

#### 2.集成RPC

1. 找到配置文件或代碼：打開項目工程，在DApp項目中找到配置RPC節點的文件或代碼段。這可能是一個配置文件，如.env、config.js、truffle-config.js等，或者是直接在代碼中硬編碼
2. 修改RPC URL：將配置文件或代碼中的RPC URL修改為Scutum RPC URL
3. 測試連接：更改後，在本地運行DApp或相應的測試腳本來確保新的RPC URL可以正常工作，可以使用如`web3.eth.net.isListening()`或`ethers.provider.pollingInterval`等方法來檢查連接是否成功
4. 部署更新：如測試通過，可以將變更部署至生產環境

{% tabs %}
{% tab title="JavaScript" %}
```javascript
// 引入Web3
const Web3 = require('web3');

// 創建web3實例並連接到RPC URL
const web3 = new Web3('https://ethereum-rpc.publicnode.com'); // 可在此处将RPC URL替换为Scutum RPC URL

// 檢查連接
web3.eth.net.isListening()
  .then((listening) => {
    console.log('Web3 connected: ', listening);
  })
  .catch((err) => {
    console.error('Web3 connection error: ', err);
  });
```
{% endtab %}
{% endtabs %}

#### 3. 查詢交易

1. 登錄[blockrazor.io](https://blockrazor.io)
2. 在RPC模塊下點擊【返利】查看返利情況，點擊【交易】查看項目交易情況
