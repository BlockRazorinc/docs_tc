# 量化交易系统

搬磚套利是一種低風險的量化交易策略。DEX 和 CEX 上的同一個代幣，由於流動性、市場機制、交易速度等因素，同一時間在價格上可能存在差異，搬磚套利策略在價格較低的交易所買入資產，在價格較高的交易所賣出，從中賺取價差。

雖然搬磚套利風險低，依然存在影響策略穩定盈利的因素——“確定性”。



### 價差確定性

價差決定套利空間，決定價差的是目標代幣在CEX與DEX上的價格——雙邊實時變化的價格。價差確定性指的是，在雙邊價格趨勢實時變化的同時，確保目標代幣存在價差。

**DEX價格確定性**

DEX價格的確定性取決於查詢方式。可以使用DEX聚合器API（比如1inch）查詢多個DEX的匯總價格，但可能存在數據查詢延遲，且API通常會設置限流，需付費才能解除。

一種方式是自建全節點，同步最新區塊，直接從DEX合約查詢交易對價格。限流問題得以解決，但需在第一時間同步到最新區塊。

**CEX價格確定性**

與DEX價格按區塊更新不同，CEX上交易對的價格處於持續實時變化中，這會造成一定的價差風險。

假設DEX所在的鏈，每隔3s出一個區塊，搬磚套利策略在1000ms發現價差信號，在1600ms完成DEX交易構建併發出，剩下的1400ms，CEX上交易對的價格變化是量化策略無法控制的，如果在此期間CEX價格發生劇烈波動，抹平價差甚至導致負向價差，策略就會面臨困損。

為消除價差風險，增強CEX價格確定性，搬磚套利策略無法控制的時間段需要盡量被縮短，策略需“壓秒”發送，在區塊的最後時限發出交易。



### 上鏈確定性

確定目標代幣存在價差後，搬磚套利策略會在CEX和DEX上同時下單，與採用訂單薄撮合機制的CEX不同，DEX交易可能由於無法被納入下個區塊，導致價差被抹平。增強DEX交易的上鏈確定性也是策略盈利的關鍵。

提升DEX交易上鏈確定性，也可以理解為提升DEX交易的上鏈速度，詳見[Swap交易要素分析](https://www.blockrazor.io/#/blogs/20250331swap)。對於量化交易系統，建議直接對接專業的RPC，釋放出時間和精力專注於核心策略。



### BlockRazor如何提升確定性

針對DEX價格確定性，量化交易系統可以使用[Node Stream](../streams/node-stream/)，第一時間同步最新區塊，低延遲獲取最新的DEX交易對價格。

針對CEX價格確定性，量化交易系統可以建立和BlockRazor Builder的直連通道，在區塊“最後時刻”，以極低延遲發送交易至BlockRazor Builder，最大程度降低CEX價格波動風險。BlockRazor Builder是BSC上出塊率第一的Builder，如有意向和BlockRazor Builder建立直連通道，請與我們[聯繫](https://discord.com/invite/qqJuwRb8Nh)。

針對上鏈確定性，量化交易系統可以將策略交易提交至[RPC](../transaction-submission/rpc/)。基於全球分布式網絡，RPC能以極低延遲接收來自錢包客戶端的交易，再將交易第一時間轉發給地理位置就近的鏈上主流Builder，在網絡層面實現端到端低延遲轉發的同時，提升交易上鏈速度。

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption><p>benchmark客戶端分別向BlockRazor RPC、A RPC和B RPC發送交易，記錄交易發送時的最新區塊號和上鏈區塊號的差值，差值越小，表示交易上鏈速度越快。</p></figcaption></figure>

圖表數據表明，提交至BlockRazor RPC的交易在下個區塊內上鏈的概率高達95%，100%的交易在兩個區塊內上鏈，交易上鏈的速度和穩定性遠超其他RPC。



### 如何使用BlockRazor的服務

#### Node Stream

1. ​[註冊](https://www.blockrazor.io/#/register)BlockRazor
2. ​[登錄](https://www.blockrazor.io/#/login)BlockRazor，訂閱BlockRazor的Tier2或Tier1計劃
3. 配置[Node Stream](../streams/node-stream/)服务

#### 對接BlockRazor RPC

BlockRazor RPC會為項目方生成專屬的RPC URL，專屬RPC支持原生的JSON RPC方法，具體步驟如下：

1. [註冊](https://www.blockrazor.io/#/register)BlockRazor
2. ​[登錄](https://www.blockrazor.io/#/login)BlockRazor控制台
3. 前往RPC模塊，查看專屬RPC（目前支持Ethereum和BSC），一鍵可視化配置RPC參數，複製RPC URL
4. 前往量化交易系統項目工程，找到鏈的RPC配置文件，將默認端點替換為專屬RPC URL
5. 更新發佈項目
6. 執行搬磚套利策略，將交易以[`eth_sendRawtransaction`](../transaction-submission/rpc/ethereum/json-rpc.md#eth_sendrawtransaction)提交至RPC，如策略涉及多筆交易，則以[Bundle](../transaction-submission/bundle/)提交至RPC
7. [登錄](https://www.blockrazor.io/#/login)BlockRazor控制台，查看返利和交易情況

#### Builder直連通道

* 如果需要和BlockRazor Builder建立直連通道，請前往[Discord](https://discord.com/invite/qqJuwRb8Nh)和我們取得聯繫。

