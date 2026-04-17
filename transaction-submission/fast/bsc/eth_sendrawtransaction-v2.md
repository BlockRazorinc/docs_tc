# eth\_sendRawTransaction v2

### 介紹

Fast模式基於BlockRazor的全球高性能網絡實現交易最低延遲上鏈，適合對交易上鏈速度存在極致要求的用戶。相比[Send RawTx](send-rawtx.md) ，发往`eth_sendRawTransaction`的交易不會通過mempool廣播，在確保速度的同時具備隱私性。

{% hint style="info" %}
`eth_sendRawTransaction`不和訂閱計劃綁定，但每筆交易中需要包含轉賬至0x9D70AC39166ca154307a93fa6b595CF7962fe8e5的tip，金額至少為0.000025 BNB 或 Transaction Fee 的5%
{% endhint %}

與[eth\_sendRawTransaction](eth_sendrawtransaction.md)相比，`eth_sendRawTransaction v2`提供了一種更精簡、更迅速的交易提交途徑。

* 繞過 CORS 預檢： 它消除了通常由 OPTIONS 預檢請求所引起的延遲（大約 50-100 毫秒）。
* 純文本而非 JSON： 採用簡單的純文本傳輸，避免了與解析 JSON 相關的計算負擔。此外，由此產生的較小數據包尺寸有助於縮短網路傳輸時間並降低成本。



### 端點

http://bsc-fast.blockrazor.io/v2/sendRawTransaction



### 限流

`eth_sendRawTransaction v2`不和訂閱計劃綁定，限流默認統一為10 TPS，如需提升TPS，請於我們聯繫



### 請求示例

{% tabs %}
{% tab title="CURL" %}
```bash
curl -X POST 'bsc-fast.blockrazor.io/v2/sendRawTransaction?auth=<auth_token>'
-H "Content-Type: text/plain" \
-d "<raw_tx>"
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
注意：

* 認證 (auth) 參數必須以 URI 參數的形式填入URL
* 請求中唯一允許的header是 `Content-Type: text/plain`
{% endhint %}



### 返回示例

**正常**

```json
{
	"code": 0,
	"message": "success",
	"data": {
		"txHash": "0xd2ebb523f400dd33ebf946a1280426196eed72c9a63b7e1734dd6f8e2f5a81dc"
	}
}
```

**異常**

```json
{
	"code": -32600,
	"message": "auth token is invalid",
	"data": null
}
```

