# Gas Sponsor

{% hint style="info" %}
BlockRazor Gas Sponsor目前支持BSC、Solana和Ethereum，如需接入Gas Sponsor服務請於我們[聯繫](https://discord.gg/qqJuwRb8Nh)。
{% endhint %}

### Gas Sponsor介绍

Gas Sponsor通過gas贊助使錢包用戶能夠進行代幣互換（Swap）而無需支付任何原生區塊鏈貨幣（比如ETH、BNB、SOL等）。對於錢包項目方集成Gas Sponsor可以帶來以下收益：

* 引增量交易流量：Gas Sponsor 可用作強大的品牌推廣和營銷工具，以吸引增量的交易流量並提升整體交易活躍度
* 消除交易障礙：用戶不再需要僅僅為了支付 Gas 費用而專門獲取原生代幣（例如，ETH、BNB、SOL），這消除了交易中最大的障礙，從而實現了真正無縫和無摩擦的交易體驗
* 增加收入: 更多用戶完成SWAP，整體交易量更高，為項目方帶來更多的SWAP費用收入

### Gas Sponsor特性

* 與原生交易接口 `eth_sendRawTransaction` (EVM) / `sendTransaction` (Solana)完全兼容，只需簡單地更改端點即可實現兼容
* 支持交易過濾和成本控制，可針對交易進行篩選和成本管理
* 交易以完全隱私的方式轉發，交易被完全屏蔽，有效抵御潛在的 MEV攻擊
* 交易通過 BlockRazor 的全球高性能網絡路由，以最低的延遲被打包納入區塊

### Gas Sponsor流程（以Ethereum為例）

<figure><img src="../.gitbook/assets/image (1).png" alt="" width="563"><figcaption></figcaption></figure>

1. 用戶在客戶端發起一筆交易
2. 客戶端調用 `pm_isSponsorable` 接口，以檢查該交易是否符合被贊助（代付 Gas）的資格
3. 如果 `pm_isSponsorable` 返回為 `true`（符合資格），用戶對交易進行簽名
4. 客戶端調用 `eth_sendRawTransaction` 接口，提交已簽名交易
5. BlockRazor 以bundle方式在用戶交易前附加一筆贊助交易，將bundle轉發給Builders
6. 用戶交易以贊助bundle形式被打包上鏈

