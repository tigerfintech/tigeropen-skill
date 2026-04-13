# Tiger OpenAPI Go SDK — Quickstart

> Go SDK 快速入门 / Quick Start for Go SDK
> GitHub: https://github.com/tigerfintech/openapi-sdks

## 安装 / Installation

```bash
go get github.com/tigerfintech/openapi-sdks/go@latest
```

要求 / Requirements: Go 1.20+

---

## 配置 / Configuration

### 方式一：代码直接设置 / Method 1: Code assignment

```go
import (
    "github.com/tigerfintech/openapi-sdks/go/config"
)

cfg, err := config.NewClientConfig(
    config.WithTigerID("your_tiger_id"),
    config.WithPrivateKey("your_rsa_private_key"),  // PEM 格式私钥字符串
    config.WithAccount("your_account"),
)
```

### 方式二：从 properties 文件加载 / Method 2: Properties file

```go
cfg, err := config.NewClientConfig(
    config.WithPropertiesFile("tiger_openapi_config.properties"),
)
```

配置文件格式 / Config file format:
```properties
tiger_id=your_tiger_id
private_key=your_rsa_private_key
account=your_account
```

### 方式三：环境变量 / Method 3: Environment variables

```bash
export TIGEROPEN_TIGER_ID=your_tiger_id
export TIGEROPEN_PRIVATE_KEY=your_rsa_private_key
export TIGEROPEN_ACCOUNT=your_account
```

### 配置项说明 / Config Options

| 选项 Option | 函数 Function | 说明 | 必填 |
|------------|--------------|------|------|
| Tiger ID | `WithTigerID` | 开发者 ID | ✅ |
| 私钥 | `WithPrivateKey` | RSA 私钥 PEM 字符串 | ✅ |
| 账户 | `WithAccount` | 交易账户号 | ✅ |
| 配置文件 | `WithPropertiesFile` | .properties 文件路径 | - |
| 语言 | `WithLanguage` | `zh_CN` / `en_US` | - |
| 超时 | `WithTimeout` | 请求超时时长（默认 15s） | - |
| 沙箱 | `WithSandboxDebug` | 使用沙箱环境（模拟） | - |

---

## 客户端创建 / Create Clients

```go
import (
    "github.com/tigerfintech/openapi-sdks/go/client"
    "github.com/tigerfintech/openapi-sdks/go/quote"
    "github.com/tigerfintech/openapi-sdks/go/trade"
    "github.com/tigerfintech/openapi-sdks/go/push"
)

httpClient := client.NewHttpClient(cfg)

qc := quote.NewQuoteClient(httpClient)       // 行情客户端
tc := trade.NewTradeClient(httpClient, cfg.Account) // 交易客户端
pc := push.NewPushClient(cfg)               // 推送客户端
```

---

## 通用 API 调用 / Generic API Call

当 SDK 尚未封装某个 API 时，使用 `httpClient.ExecuteRaw` 直接调用：

```go
result, err := httpClient.ExecuteRaw("market_state", map[string]interface{}{
    "market": "US",
})
if err != nil {
    log.Fatal(err)
}
fmt.Println(string(result))
```

---

## 错误处理 / Error Handling

```go
result, err := qc.QuoteRealTime([]string{"AAPL"})
if err != nil {
    // API 业务错误或网络错误
    log.Printf("error: %v", err)
    return
}
// result 是 json.RawMessage，需反序列化
var data []map[string]interface{}
json.Unmarshal(result, &data)
```

---

## 前置条件 / Prerequisites

1. 老虎证券账户 + 开发者 API 权限：https://developer.itigerup.com/
2. 准备好 `tiger_id`、RSA 私钥、`account`
3. 行情数据需要对应的行情权限；期权行情需要期权行情权限

---

## FAQ

**Q: 私钥如何生成?**
参考官方文档 https://docs.itigerup.com/docs/prepare，使用 RSA-2048 生成密钥对，上传公钥到开发者后台。

**Q: 返回值格式?**
所有 API 返回 `(json.RawMessage, error)`，需要自行 `json.Unmarshal` 解析。

**Q: 模拟账户和实盘账户区别?**
模拟账户在开发者后台申请，`sandboxDebug=true` 连接沙箱环境；实盘账户直接使用生产域名。
