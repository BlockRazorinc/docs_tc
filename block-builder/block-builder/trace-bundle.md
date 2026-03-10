# Trace Bundle

### 介紹

本方法支持通過bundle hash查詢該 bundle 的當前狀態，請在向builder發送 bundle 後 5 分鐘進行查詢。端點：[https://bsc-bundle-stats.blockrazor.io/](https://bsc-bundle-stats.blockrazor.io/)



### 流控说明

<table><thead><tr><th width="125.88671875"></th><th>Tier 4</th><th>Tier 3</th><th>Tier 2</th><th>Tier 1</th><th>Tier 0</th></tr></thead><tbody><tr><td>交易筆數 / 天</td><td>4筆 / 天</td><td>8筆 / 天</td><td>40筆 / 天</td><td>200筆 / 天</td><td>不限</td></tr></tbody></table>



### 請求參數

<table><thead><tr><th width="84.25390625">參數</th><th width="76.70703125">必選</th><th width="74.328125">格式</th><th>示例</th><th>描述</th></tr></thead><tbody><tr><td>hash</td><td>是</td><td>hash</td><td>0x25f9……317097</td><td>bundle hash</td></tr></tbody></table>



### 請求示例

{% tabs %}
{% tab title="HTTPS" %}
```bash
curl -X GET "https://bsc-bundle-stats.blockrazor.io/bundlestate?hash=0x25f9fc35e978709195c00e864b9a19fb41ad5c5c5b8a3e003813ae9727317097" \
     -H "Content-Type: application/json" \
     -H "Authorization: M2ZiZj……JhODA1"
```
{% endtab %}
{% endtabs %}



### 返回示例

**正常**

```json
{
  "bundle": {
    "timestamp": "2025-04-07T09:04:40Z", // builder接收bundle的時間（UTC）
    "bundleHash": "0x25f9fc35e978709195c00e864b9a19fb41ad5c5c5b8a3e003813ae9727317097", // bundle哈希
    "state": "onchain", // bundle已上鏈
    "blockNumber": 48144954, // bundle所在的區塊號
    "priority": "358911000000000" // bundle價值，單位為wei
  }
}
```

**异常- failed**

```json
{
  "bundle": {
    "timestamp": "2025-04-08T05:21:10Z", // builder接收bundle的時間（UTC）
    "bundleHash": "0xe06a923bce1f46b2a0602b8fb2263dffde22f21277b0b71d84b79aff6a58772b", // bundle哈希
    "state": "failed", // builder已收到bundle，但bundle未上鏈
    "err": "non-reverting tx in bundle failed" // 具體未上鏈的原因
  }
}
```

**异常- not found**

```json
{
  "bundle": {
    "bundleHash": "0xabc", // 請求時的bundle hash
    "state": "not found" // 未查詢到bundle
  }
}
```



