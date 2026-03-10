# Trading Bot

在 Solana 與 EVM 高頻交易生態中，Trading Bot的成功取決於兩個維度：信號獲取速度以及交易執行速度。BlockRazor 基於上述維度為Trading Bot提供底層技術以及基礎設施支持。本案例以Solana的狙擊場景為例，介紹BlockRazor如何確保Trading Bot在激烈的鏈上博弈中始終處於領先地位。



### 痛點分析

#### **狙擊信號的滯後性**

對於狙擊新開盤代幣Bot 而言，傳統的公共節點數據傳輸存在嚴重的物理延遲且由於缺少質押處於接收區塊鏈路的末端。當 Bot 接收到新幣部署或流動性添加的信號時，往往已經錯過了最佳的狙擊時機，導致買入成本大幅攀升。

#### 交易執行端的風險與阻礙

* 交易延遲：對於追求交易亞秒級成交的 Bot 來說，狙擊交易上鏈速度慢意味著用戶利潤回吐。
* 三明治攻擊：在進行大額 Snipe 或 Swap 時，Bot 自身的交易意圖若暴露，極易成為 MEV 機器人或惡意驗證者的攻擊目標。
* 交易執行受阻：Bot 頻繁切換錢包或操作新帳戶時，可能用於缺乏原生代幣（如 SOL）導致關鍵交易因賬戶租金不足或交易費不足失敗。



### 產品服務

BlockRazor基于以上痛点为Trading Bot针对性提供产品服务。

#### 狙擊信號監聽

針對狙擊信號的滯後性，BlockRazor提供Geyser Stream、Shred Stream等服務，基於物理鏈路和交易鏈路的雙重優化，幫助Trading Bot以極致速度獲取到狙擊信號。同時，BlockRazor根據不同鏈的特性提供極速交易監聽服務。對於Ethereum和BSC等可以監聽到pending交易的鏈，BlockRazor提供Public Mempool和Private Mempool用於pending聰敏錢交易的監聽，以及New Blocks用於已上鏈聰敏錢交易的監聽。

<table><thead><tr><th width="142.38671875">鏈</th><th width="286.75">pending信號交易</th><th>confirmed信號交易</th></tr></thead><tbody><tr><td>Ethereum</td><td><a href="../streams/mempool/public-mempool.md">Public Mempool</a><br><a href="../streams/mempool/private-mempool/">Private Mempool</a></td><td>-</td></tr><tr><td>BSC</td><td><a href="../streams/mempool/public-mempool.md">Public Mempool</a><br><a href="../streams/mempool/private-mempool/">Private Mempool</a></td><td><a href="../streams/block-stream/bsc/newblocks.md">New Blocks</a></td></tr><tr><td>Solana</td><td>-</td><td><a href="../streams/block-stream/solana/geyser-stream/">Geyser Stream</a><br><a href="../streams/block-stream/solana/shred-stream.md">Shred Stream</a></td></tr><tr><td>Base</td><td>-</td><td><a href="../streams/block-stream/base/get-flashblockstream/">FlashBlock Stream</a></td></tr></tbody></table>

#### 多模式交易執行優化

[Fast](../transaction-submission/fast/)：利用全球加速的高性能網絡和高質押驗證者的SWQoS鏈路實現交易最低延遲上鏈。

[Gas Sponsor](../transaction-submission/gas-sponsor.md)：通過贊助交易費和租金使Trading Bot的用戶能夠進行狙擊交易而無需支付SOL，集成Gas Sponsor可以為Trading Bot帶來額外收益：

* 引增量交易流量：Gas Sponsor 可用作強大的品牌推廣和營銷工具，以吸引增量的交易流量並提升整體交易活躍度
* 增加收入: 更多用戶完成狙擊，整體交易量更高，為項目方帶來更多的手續費收入

[Sandwich Mitigation](../transaction-submission/fast/solana/)：為交易提供三明治保護機制，實時監測、跳過黑名單驗證者，保護交易利潤不受滑點侵蝕

[Bundle](../transaction-submission/bundle/)：在監聽到開盤交易後，Trading Bot可以利用Bundle讓狙擊交易緊貼在開盤交易後上鏈，幫助用戶抹平由於上鏈延遲導致的價格波動風險。



### Benchmark

BlockRazor 旨在成為 Trading Bot 最堅實的基礎設施合作夥伴，通過集成我們的服務，Trading Bot 可以獲得卓越的鏈上監聽與執行效能，benchmark如下：

* [Shred Stream](https://blockrazor.io/#/blogs/20250818shredbenchmark)
* [Fast](https://blockrazor.io/#/blogs/20250801Benchmarking)
* [端到端交易延遲](https://blockrazor.io/#/blogs/20250826e2etest)

