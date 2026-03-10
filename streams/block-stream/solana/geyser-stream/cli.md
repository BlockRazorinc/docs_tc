# CLI

### 快速開始

{% stepper %}
{% step %}
#### 創建目錄

```
mkdir geyser-stream
cd geyser-stream
```
{% endstep %}

{% step %}
#### 下載Client文件

```
# Mac
curl -L -o client https://github.com/BlockRazorinc/geyserstream-client-go/releases/download/v1.0.0/client-darwin-arm64
```

```
# Ubuntu-22.04
curl -L -o client https://github.com/BlockRazorinc/geyserstream-client-go/releases/download/v1.0.0/client-ubuntu-22.04
```

```
# Ubuntu-24.04
curl -L -o client https://github.com/BlockRazorinc/geyserstream-client-go/releases/download/v1.0.0/client-ubuntu-24.04
```
{% endstep %}

{% step %}
#### 賦予Client文件可執行權限

```
chmod +x client
```
{% endstep %}

{% step %}
#### 執行Client文件

```
# 訂閱交易
./client -e "https://geyserstream-frankfurt.blockrazor.xyz" --x-token "$AUTH_TOKEN" subscribe --transactions --transactions-vote false --transactions-failed false
```

```
# 訂閱賬戶
./client -e "https://geyserstream-frankfurt.blockrazor.xyz" --x-token "$AUTH_TOKEN" subscribe --accounts --accounts-owner 11111111111111111111111111111111
```

```
# 訂閱區塊
./client -e "https://geyserstream-frankfurt.blockrazor.xyz" --x-token "$AUTH_TOKEN" subscribe --blocks
```
{% endstep %}
{% endstepper %}
