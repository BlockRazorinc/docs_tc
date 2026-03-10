# 个人交易者

### 介紹

{% hint style="info" %}
個人交易者無需訂閱計劃即可使用通用RPC
{% endhint %}

對於經常在DEX進行交易的用戶，存在極大概率以滑點上限兌換到目標加密貨幣，導致利益在無形中受到損失。个人交易者可以一鍵添加RPC到錢包。從此發起的SWAP交易都可以受到BlockRazor RPC的深度保護，減少滑點損失，同時能以一定概率實時收到交易返利。目前BlockRazor支持個人交易者添加Ethereum和BSC的RPC。

存在差異化需求的用戶可根據需求[選擇不同模式的通用RPC](ge-ren-jiao-yi-zhe.md#ru-he-zi-ding-yi-tian-jia-rpc-dao-qian-bao)進行添加。如有任何問題，請前往[Discord](https://discord.com/invite/qqJuwRb8Nh)與我們聯繫。



### 如何快速添加RPC到錢包

#### Metamask

1. 前往[rpc.blockrazor.io](https://rpc.blockrazor.io)，點擊開啓保護

<figure><img src="../.gitbook/assets/image (35).png" alt="" width="563"><figcaption></figcaption></figure>

2. 點擊連接狐狸錢包，在喚起的狐狸錢包中完成連接

<figure><img src="../.gitbook/assets/image (37).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (47).png" alt="" width="355"><figcaption></figcaption></figure>

3. 點擊【一鍵添加RPC】，在喚起的狐狸錢包中批准添加新的網絡

<figure><img src="../.gitbook/assets/image (39).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (31).png" alt="" width="351"><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (49).png" alt="" width="318"><figcaption></figcaption></figure>



#### 其他錢包（以Coinbase為例）

1. 點擊瀏覽器插件，喚起Coinbase錢包
2. 點擊【設置】 - 【網絡】&#x20;

<figure><img src="../.gitbook/assets/image (50).png" alt="" width="366"><figcaption></figcaption></figure>

3. 點擊【+】添加網絡

<figure><img src="../.gitbook/assets/image (51).png" alt="" width="371"><figcaption></figcaption></figure>

4. 填寫[通用RPC網絡信息](ge-ren-jiao-yi-zhe.md#tong-yong-rpc-wang-luo-xin-xi)，點擊【保存】

<figure><img src="../.gitbook/assets/image (30).png" alt="" width="351"><figcaption></figcaption></figure>

### 如何自定义添加RPC到錢包

#### Metamask

1. 前往[rpc.blockrazor.io](https://rpc.blockrazor.io)，點擊開啓保護
2. 點擊連接狐狸錢包，在喚起的狐狸錢包中完成連接
3. 點擊【自定義添加RPC】，選擇[通用RPC模式](ge-ren-jiao-yi-zhe.md#tong-yong-rpc-mo-shi-dui-bi)

<figure><img src="../.gitbook/assets/image (32).png" alt="" width="563"><figcaption></figcaption></figure>

4. 確認需要添加的RPC模式，點擊【自定義添加RPC】，在喚起的狐狸錢包中批准添加新的網絡

#### 其他钱包（以Coinbase為例）

1. 點擊瀏覽器插件，喚起Coinbase錢包
2. 點擊【設置】 - 【網絡】&#x20;
3. 點擊【+】添加網絡
4. 填寫[通用RPC網絡信息](ge-ren-jiao-yi-zhe.md#tong-yong-rpc-wang-luo-xin-xi)。如需選擇RPC模式，詳見[通用RPC模式對比](ge-ren-jiao-yi-zhe.md#tong-yong-rpc-mo-shi-dui-bi)。

<figure><img src="../.gitbook/assets/image (29).png" alt="" width="357"><figcaption></figcaption></figure>

5. 點擊 **儲存**，完成RPC添加

### 通用RPC網絡信息

<table><thead><tr><th width="164"></th><th>Ethereum</th><th>BSC</th></tr></thead><tbody><tr><td>網絡名稱</td><td>Ethereum Mainnet</td><td>BNB Smart Chain Mainnet</td></tr><tr><td>RPC網址</td><td>https://eth.blockrazor.xyz</td><td>https://bsc.blockrazor.xyz</td></tr><tr><td>鏈ID</td><td>1</td><td>56</td></tr><tr><td>貨幣符號</td><td>ETH</td><td>BNB</td></tr><tr><td>瀏覽器網址</td><td><a href="https://etherscan.io">https://etherscan.io</a></td><td><a href="https://bscscan.com">https://bscscan.com</a></td></tr></tbody></table>

### 通用RPC模式對比

<table><thead><tr><th width="161"></th><th width="186">default</th><th width="196">fullprivacy</th><th>maxbackrun</th></tr></thead><tbody><tr><td>BSC</td><td>https://bsc.blockrazor.xyz</td><td>https://bsc.blockrazor.xyz/fullprivacy</td><td>https://bsc.blockrazor.xyz/maxbackrun</td></tr><tr><td>Ethereum</td><td>https://eth.blockrazor.xyz</td><td>https://eth.blockrazor.xyz/fullprivacy</td><td>https://eth.blockrazor.xyz/maxbackrun</td></tr><tr><td>MEV保護</td><td>保護</td><td>保護</td><td>保護</td></tr><tr><td>交易隱私</td><td>最小程度披露</td><td>全隱私</td><td>最大程度披露</td></tr><tr><td>返利可能性</td><td>中等</td><td>無返利</td><td>高</td></tr><tr><td>返利比例</td><td>支持</td><td>無返利</td><td>支持</td></tr><tr><td>Revert保護</td><td>不保護</td><td>保護</td><td>保護</td></tr></tbody></table>

**default**

* default模式下，提交至Scutum通用RPC的交易，僅向Searcher披露必要的交易數據（hash和logs & state），以在最大程度保護交易隱私的前提下贏得返利機會。同時，為保證交易納入區塊的速度，交易不做revert保護（與從錢包官方RPC提交一致）。

**fullprivacy**

* fullprivacy模式下，提交至Scutum通用RPC的交易，不會披露任何交易數據，Scutum會直接轉發交易至主流builder。由於未披露任何交易數據，交易也不會收到返利，因此無需設置返利比例。該模式下的交易同時會受到revert保護，為保證納入區塊的速度，建議在發送交易時設置一定的交易priority fee（Ethereum）。

**maxbackrun**

* maxbackrun模式下，提交至Scutum通用RPC的交易，會在隱私保護的前提下披露必要的交易數據（hash、to、calldata、functionSelector、logs & state），以最大可能獲得返利。該模式下的交易同時會受到revert保護，為保證納入區塊的速度，建議在發送交易時設置一定的交易priority fee（Ethereum）。

