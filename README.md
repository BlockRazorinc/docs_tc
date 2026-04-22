# 關於BlockRazor

### 介紹

[BlockRazor](https://blockrazor.io/)是一家Web3交易基礎設施供應商，基於分佈式網絡通信技術和架構為錢包、去中心化交易所、Trading Bot、Searcher和量化交易系統等用戶提供服務。BlockRazor已在Ethereum、BSC、Solana和Base等主流公鏈推出Transaction Submission, Streams和Block Builder等服務，幫助客戶實現交易的最佳執行。

### Transaction Submission

BlockRazor基於客戶訴求提供RPC、Gas Sponsor、Fast和Bundle等多種交易發送模式。

* RPC：提供標準JSON-RPC方法，為交易提供MEV保護，支持實時返利
* Gas Sponsor：為原生代幣不足以支付交易費的用戶提供gas贊助，提升用戶交易體驗
* Fast：利用全球加速網絡提升交易上鏈速度，適合對交易速度有極致要求的用戶
* Bundle：通過Bundle在Backrun、跟單、狙擊等交易場景下實現和信號交易同區塊上鏈

### Streams

BlockRazor提供Mempool、Block Stream、Network Fee Stream和Block Stream等多種數據流，

* Mempool：低延遲推送Mempool pending交易，第一時間監聽Backrun、跟單、狙擊等多場景下的信號交易
* Block Stream：低延遲向Trading程序推送區塊，第一時間監聽已確認的信號交易或普通交易
* Network Fee Stream：基於最近歷史區塊數據實時推送Gas Price、Priority Fee或Tip
* Node Stream：低延遲向全節點同步數據，第一時間同步世界狀態

### Block Builder

Block Builder提供基于PBS架構的BSC 区块构建服务，基於全球多點部署和多種區塊構建算法最大化區塊價值量，實現高勝率出塊。目前BlockRazor的[Block Builder](block-builder/block-builder/)歷史累計出塊率37%，BSC全鏈第一，查看實時出塊率可前往[https://dune.com/bnbchain/bnb-smart-chain-mev-stats](https://dune.com/bnbchain/bnb-smart-chain-mev-stats)。

