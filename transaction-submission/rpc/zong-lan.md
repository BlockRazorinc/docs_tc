# 總覽

### 產品介紹

BlockRazor RPC是一款為用戶提供MEV保護、支持返利的RPC產品，目前已支持BSC和Ethereum。\
用戶提交的交易如由普通RPC處理，會在節點P2P網絡間公開廣播，交易極容易受到MEV攻擊，導致因交易產生的潛在返利空間被壓榨乾淨。\
BlockRazor RPC相比普通RPC，可以實現對交易的隱私保護，從而抵御三明治攻擊和搶跑攻擊。同時, BlockRazor RPC支持用戶自定義披露交易數據，從中產生的收益講實時返利至用戶賬戶。



### 產品優勢

#### 高安全性

BlockRazor RPC支持Ethereum和BSC上全類型交易的隱私保護，抵御三明治攻擊和搶跑攻擊。

#### 快速打包

BlockRazor RPC支持backrun bundle競拍，以實現交易價值最大化，同時藉助BlockRazor高性能網絡的技術將交易以極低延遲發往Ethereum和BSC上的頭部區塊構建者。

#### 實時返利

交易潛在收益會實時返利給用戶。返利交易會在RPC內部提前構建完成，緊隨用戶交易發往區塊構建者，納入同一個區塊中。

#### 極速集成

交易者可[一鍵添加RPC到錢包](../../yong-hu-an-li/ge-ren-jiao-yi-zhe.md)，項目方可[自定義RPC URL和交易隱私數據](bsc/ji-cheng-rpc.md)，快速完成RPC集成。



### 常見問題

**如何確保交易在獲得返利的同時抵御MEV攻擊**

* 交易會根據自定義披露配置分享給Searcher，Searcher只能對交易執行無害的backrun策略，以此在抵御三明治和搶跑攻擊的同時，也可以將backrun產生的收益返利給用戶

**我會在什麼時候以什麼形式收到返利？**

* 如交易存在執行backrun策略的機會，返利由Searcher調用智能合約完成，返利交易和項目交易會在同一個區塊內執行成功，以達到實時返利的目的。​

**交易存在revert的可能性嗎？**

* 如果BlockRazor RPC在實際執行過程中發現交易revert且revert保護已經開啟，則交易不會納入區塊中。

**各個角色間的利潤分配是怎麼樣的？**

* 如交易存在利润空间且在競價成功後納入區塊，BlockRazor RPC會將競價的部分返利給用戶，剩餘一部分用於支付BlockRazor RPC手續費，另一部分用於向builder支付bribe以使交易盡快上鏈。



### New to MEV

#### MEV

MEV全稱是 Maximal Extractable Value，指的是在新區塊生產過程中，MEV機器人會對mempool中的交易重新排序，在利用價格波動賺取利潤的同時，實現區塊整體價值的最大化。在MEV策略中，三明治攻擊對用戶的傷害最大，機器人會在目標交易（通常是DEX的SWAP交易）的前後分別構建一筆攻擊交易，導致用戶收到的加密貨幣數量相比預期減少。

#### Searcher

Searcher是指那些尋找並利用MEV機會的獨立網絡參與者，他們通過運行複雜的算法來檢測盈利的 MEV 機會，構建交易組成[bundle](zong-lan.md#bundle)提交給[MEV Protect RPC](zong-lan.md#rpc)或區塊構建者，以期將bundle納入區塊從中獲利。在Searcher執行的MEV策略中，三明治（sandwich）攻擊和搶跑（frontrunning）攻擊對用戶傷害最大，而backrun策略由於其在用戶交易後序執行，對用戶無害。

#### Sandwich

Sandwich是一種惡意的MEV策略，這種攻擊通常發生在去中心化交易所（DEX）上，當一個用戶（比如Alice）想要在DEX上購買一個代幣時，一個惡意的交易者（比如Bob）可以偵測到這個交易，並在Alice的交易前後分別插入自己的兩個交易來「夾擊」它。

Bob首先在Alice的交易之前買入代幣A（Bob會在Alice的交易前插入高gas費用的交易，以確保被優先處理，此類前置交易的攻擊也被稱為frontrunning），這會推高代幣A的價格。然後，在Alice的交易之後，Bob再賣出代幣A，此時由於價格已經被推高，Bob可以從中獲利。這種攻擊方式不僅影響了Alice原本交易的執行價格，還讓Bob通過操縱價格獲得了利潤。

#### Backrun

Backrun是一種無害的MEV策略，backrun策略的執行者會在一筆高價值交易後立即策略性地執行一筆Backrun交易，利用因前置交易價格波動產生的套利機會來獲取利潤。由Backrun交易在用戶交易之後執行，不會對前置交易造成影響。

#### Bundle

在MEV中，Searcher通過將用戶交易和策略交易按一定順序構建成bundle（交易捆綁包）來捕獲套利機會。bundle具備原子性，如果bundle中的任意一筆交易模擬執行失敗，則整個bundle都將不會執行，這意味著bundle中的交易要麼全部執行成功，要麼全部執行失敗。

#### RPC

RPC（遠程過程調用 Remote Procedure Call）是一種協議，它允許一個程序（客戶端）通過網絡向另一個程序（服務器）請求服務，而無需瞭解底層網絡技術的細節。在區塊鏈的上下文中，RPC通常用於節點通信和客戶端交互，區塊鏈客戶端（如錢包或DApp前端）通過RPC與區塊鏈節點交互，發送交易、查詢賬戶餘額、獲取區塊信息等。



### 隱私聲明

BlockRazor RPC 不跟蹤任何類型的用戶信息（例如 IP 地址、位置等）。僅保留區塊鏈上公開的信息，如交易的時間戳。

