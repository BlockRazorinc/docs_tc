# Geyser Stream

## 產品介紹

**什麼是Geyser Stream**

Geyser是Solana驗證者的插件機制，通過該插件，Solana的賬戶、槽位、區塊和交易數據可以實時傳輸到外部數據存儲介質。Geyser Stream是BlockRazor基於Yellowstone gRPC(Geyser插件)推出的Solana高性能數據流服務，支持客戶端通過gRPC協議以極低延遲訂閱到實時Solana數據流。



**Geyser Stream主要有什麼用**

交易分析：監聽指定賬戶的交易，可用於跟單聰明錢賬戶交易或狙擊pump.fun新幣

賬戶分析：監聽指定賬戶餘額變化，可用於監聽DEX(如Raydium、Orca 或 Jupiter)指定池子賬戶的餘額，以計算代幣對價格

區塊 & 槽位分析：監聽區塊和槽位，可用於獲取網絡共識和健康狀態



**Geyser Stream的關鍵特性**

高性能：Geyser Stream部署於高性能服務器以極低延遲實時向客戶端推送gRPC數據流

數據完整性：支持最近500個slot(200 s)的交易重放，保證“斷連續傳”

高穩定性：Geyser Stream服務多雲多實例運行，支持無縫故障轉移，保障長時間穩定



## 端點

<table><thead><tr><th width="171.1796875">地區</th><th>端點</th></tr></thead><tbody><tr><td>法蘭克福</td><td>geyserstream-frankfurt.blockrazor.xyz:443</td></tr><tr><td>東京</td><td>geyserstream-tokyo.blockrazor.xyz:443</td></tr></tbody></table>



## 如何集成Geyser Stream

Geyser Stream僅支持Tier2 - Solana及以上等級的用戶使用，auth獲取步驟詳見[Authentication](../../../../authentication.md)，多語言客戶端集成詳見：

* [CLI](cli.md)
* [Go](go.md)
* [Rust](rust.md)
* [JS](js.md)



## 週期流量

Geyser Stream根據訂閱計劃週期為用戶提供流量，訂閱Tier2 - Solana及以上等級計劃的用戶可免費獲得相應週期流量，關係如下：

<table><thead><tr><th width="275.84765625">訂閱計劃等級</th><th>週期流量</th></tr></thead><tbody><tr><td>Tier4 - Solana</td><td>-</td></tr><tr><td>Tier3 - Solana</td><td>-</td></tr><tr><td>Tier2 - Solana</td><td>20 TiB</td></tr><tr><td>Tier1</td><td>50 TiB</td></tr><tr><td>Custom</td><td>Custom</td></tr></tbody></table>

如當前週期內的附贈流量消耗殆盡，則可繼續採購週期流量，價格如下：

| 週期流量    | 折扣   | 折後價 / 週期 |
| ------- | ---- | -------- |
| 5 TiB   | 100% | $250     |
| 10 TiB  | 100% | $500     |
| 50 TiB  | 100% | $2500    |
| 100 TiB | 95%  | $4750    |
| 150 TiB | 90%  | $6750    |
| 200 TiB | 85%  | $8500    |
| 250 TiB | 80%  | $10000   |

採購週期流量的步驟如下：

1. 登錄BlockRazor控制台，前往服務 - Solana - Geyser Stream
2. 點擊【採購流量】，選擇週期流量規格，點擊【確認】
3. 選擇服務週期（註：週期流量每個週期的起止時間和訂閱計劃週期保持一致）和支付方式
4. 完成賬單支付
5. 回到服務 - Solana - Geyser Stream，在總覽中查看週期流量總額

