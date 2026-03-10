# 解析FlashBlock

FlashBlock的解析方法如下：

```go
// ParseFlashBlockByte decompresses the brotli-compressed binary data of a flash block.
func ParseFlashBlockByte(data []byte) (string, error) {
	br := brotli.NewReader(bytes.NewReader(data))
	var buf bytes.Buffer
	_, err := buf.ReadFrom(br)
	if err != nil {
		return "", err
	}
	return buf.String(), nil
}
```

[查看](https://github.com/BlockRazorinc/base-api-client-go/blob/1d46c2983420d6da645992a9f3ed51688f7dac88/main.go#L242C1-L242C56)示例
