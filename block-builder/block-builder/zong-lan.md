# 總覽

## 介紹

Block Builder是BlockRazor的BSC區塊構建服務。Block Builder基於全球多地部署和驗證者建立低延遲通信，運行多種區塊構建算法，實現高勝率出塊，目前支持bundle提交、隱私交易提交、bundle模擬、bundle追蹤等特性。目前Block Builder在BSC上的历史累计出块率37%，全链第一，实时出块率查詢請前往[https://dune.com/bnbchain/bnb-smart-chain-mev-stats](https://dune.com/bnbchain/bnb-smart-chain-mev-stats)



## RPC端點

| 全球通用                                                               |
| ------------------------------------------------------------------ |
| [https://rpc.blockrazor.builders](https://rpc.blockrazor.builders) |

該端點支持來自全球用戶的低時延請求，建議優先接入。

<table><thead><tr><th width="127">地區</th><th width="146">可用區（AWS）</th><th>RPC端點</th></tr></thead><tbody><tr><td>東京</td><td>apne1-az4</td><td><a href="https://tokyo.builder.blockrazor.io">https://tokyo.builder.blockrazor.io</a></td></tr><tr><td>法蘭克福</td><td>euc1-az2</td><td><a href="https://frankfurt.builder.blockrazor.io">https://frankfurt.builder.blockrazor.io</a></td></tr><tr><td>弗吉尼亞</td><td>use1-az4</td><td><a href="https://virginia.builder.blockrazor.io">https://virginia.builder.blockrazor.io</a></td></tr><tr><td>都柏林</td><td>euw1-az1</td><td><a href="https://dublin.builder.blockrazor.io">https://dublin.builder.blockrazor.io</a></td></tr></tbody></table>

為進一步達到最低請求延遲，BlockRazor提供特定地區的RPC端點。建議在已接入全球通用端點的基礎上，盡可能將Bot多點部署於Builder所在地區並完成接入。

