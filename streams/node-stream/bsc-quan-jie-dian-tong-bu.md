# BSC全節點同步

### 介紹

{% hint style="info" %}
全節點同步服務支持單獨採購。如訂閱計劃達到Tier 2及以上，則可按計劃等級免費獲得額度（允許連接的全節點數量）。
{% endhint %}

全節點高速同步服務基於高性能網絡為Ethereum客戶端提供低時延同步服務。與訂閱區塊數據流不同，用戶的Ethereum客戶端可直接與就近地區的relay建立P2P連接，relay將最新區塊通過對等節點網絡同步至Ethereum客戶端，用戶可在第一時間獲得最新區塊事件與世界狀態。



### 價格 & 免费額度

{% hint style="info" %}
全節點同步服務已支持單獨採購，單個enode的價格詳見以下表格。如訂閱計劃達到Tier 2及以上，則可按計劃等級免費獲得enode額度（允許連接的全節點數量）
{% endhint %}

#### 單獨採購價格

| 服務週期  | 折扣   | 價格                |
| ----- | ---- | ----------------- |
| 1 個月  | 100% | $500(1 \* $500)   |
| 3 個月  | 95%  | $1425(3 \* $475)  |
| 6 個月  | 90%  | $2700(6 \* $450)  |
| 9 個月  | 85%  | $3825(9 \* $425)  |
| 12 個月 | 80%  | $4800(12 \* $400) |

#### 訂閱計劃免費額度

<table><thead><tr><th width="194"></th><th width="100.1015625">Tier 4</th><th width="93.2109375">Tier 3</th><th>Tier 2</th><th>Tier 1</th><th>Tier 0</th></tr></thead><tbody><tr><td>允許連接的全節點數量</td><td>-</td><td>-</td><td>2</td><td>5</td><td>30</td></tr></tbody></table>



### Relay IP

<table><thead><tr><th width="154">地區</th><th width="218">可用區（AWS）</th><th>Relay地址</th></tr></thead><tbody><tr><td>法蘭克福</td><td>euc1-az2</td><td>35.157.64.49:50051</td></tr><tr><td>東京</td><td>apne1-az4</td><td>54.249.93.63:50051</td></tr><tr><td>愛爾蘭</td><td>euw1-az1</td><td>3.248.65.151:50051</td></tr><tr><td>弗吉尼亞</td><td>use1-az4</td><td>52.205.173.134:50051</td></tr></tbody></table>



### 使用說明

#### 步驟1：添加Enode

**免費添加Enode**

1. 前往[https://www.blockrazor.io/](https://www.blockrazor.io/)，點擊右上角的【註冊】，完成註冊
2. 登錄控制台，前往【訂閱】，選擇Tier 2及以上計劃，點擊【開始訂閱】

<figure><img src="../../.gitbook/assets/image (74).png" alt="" width="563"><figcaption></figcaption></figure>

3. 確認服務週期和支付方式，完成支付
4. 前往【服務】 - 【全節點同步】，點擊【添加Enode】

<figure><img src="../../.gitbook/assets/image (75).png" alt="" width="563"><figcaption></figcaption></figure>

5. 輸入需要連接relay的Ethereum客戶端Enode，選擇離Ethereum客戶端最近的地區，點擊【確認】，完成添加

<figure><img src="../../.gitbook/assets/image (76).png" alt="" width="563"><figcaption></figcaption></figure>

6. 回到Enode列表，點擊【複製Relay Enode】



**單獨採購Enode**

1. 前往[https://www.blockrazor.io/](https://www.blockrazor.io/)，點擊右上角的【註冊】，完成註冊
2. 登錄控制台，前往【服務】 - 【全節點同步】，點擊【添加Enode】

<figure><img src="../../.gitbook/assets/image (77).png" alt="" width="563"><figcaption></figcaption></figure>

3. 輸入需要連接relay的Ethereum客戶端Enode，選擇離Ethereum客戶端最近的地區，點擊【確認】

<figure><img src="../../.gitbook/assets/image (78).png" alt="" width="563"><figcaption></figcaption></figure>

4. 確認服務週期和支付方式，完成支付
5. 回到Enode列表，點擊【複製Relay Enode】（注:Enode列表僅在支付完成後顯示）



#### 步驟2：向relay開放端口

{% hint style="info" %}
如果你的Ethereum客戶端部署於AWS等雲服務，需在雲環境中額外配置安全組（security group）的入端(inbound)規則。
{% endhint %}

1. 進入自己的Ethereum客戶端所在服务器，設置防火牆允許Relay訪問

```
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="52.205.173.134" port port="30311" protocol="tcp" accept'
```

* source address是Relay的IP地址，可以在 [Relay IP](bsc-quan-jie-dian-tong-bu.md#relay-ip) 中查詢
* port是Ethereum客戶端允許Relay訪問的端口，一般默認為30311，用戶可根據自己節點配置修改

2. 重載防火牆配置，以使配置生效

```
sudo firewall-cmd --reload
```



#### 步骤3：设置Relay为TrustedNode（以Geth節點為例）

為確保Geth節點和Relay可以保持持續連接，建議在Geth節點的config.toml中添加Relay Enode

1. 在 config.toml文件中，找到 Node.P2P中的TrustedNodes字段，添加在步驟2中獲取的Relay Enode

```scheme
[Node.P2P]
TrustedNodes = ["enode://b5b4e5aa8d8f4568af755af6da0d4642b6475d8d87c3470632bdecab8f54e4e2936ec8ae0d6f34cff8b052235e81a281912c17dfcdbf40d6d3c281b78ada4134"]
```

2. 重啓Geth節點，指定config.toml啓動，`--config config.toml`



#### 步驟4：查詢連接狀態（以在Geth節點中開啓admin namespace為例）

1. 等待10分鐘，進入Geth節點， 執行命令，查看連接狀態

```
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"admin_peers","params":[],"id":1}' http://localhost:8545
```

2. 在返回的数据中查詢相應地區的Relay Enode地址（可前往控制台複製獲取），如查詢到地址存在則證明連接成功

```json
[
    {
        "enode": "enode://9ddacbcca0dc1d1b112d470552acc795fce5c3e9f50983fcd5cee7b47289914295acaef3163bea819bcc967461978425def13595deb7de4063295c40e593f320@52.205.173.134:53754",
        "id": "8be29a75ac2cf81e3aa37ccc119630a9dfc43c88d7b5200398a466f5ef9097c4",
        "name": "Geth/v1.4.5/linux-amd64/go1.21.7",
        "caps": [
            "eth/68"
        ],
        "network": {
            "localAddress": "127.0.0.1:30311",
            "remoteAddress": "52.205.173.134:53754",
            "inbound": true,
            "trusted": false,
            "static": false
        },
        "protocols": {
            "eth": {
                "version": 68
            }
        }
    }
]
```

{% hint style="info" %}
如經查詢發現連接狀態異常，有可能是因為節點間的網絡通信出現問題，請前往[Discord](https://discord.com/invite/qqJuwRb8Nh)與我們取得聯系。
{% endhint %}

