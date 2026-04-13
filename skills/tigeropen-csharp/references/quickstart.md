# Tiger OpenAPI C# SDK — Quickstart / 快速入门

> C# SDK 快速入门 / Quick Start for C# SDK
> GitHub: https://github.com/tigerfintech/openapi-cs-sdk

## 安装 / Installation

### NuGet

```bash
dotnet add package TigerOpenAPI
```

或在 `.csproj` 中添加：

```xml
<PackageReference Include="TigerOpenAPI" Version="*" />
```

---

## 配置 / Configuration

### 配置文件 / Config File

在项目目录创建 `tiger_openapi_config.properties`：

```properties
tiger_id=your_tiger_id
private_key_pk8=MIIEvQIBADANBgkqhkiG9w0BAQEFAASC...
account=your_account_number
license=TBNZ
```

> ⚠️ **配置文件字段名必须是 `private_key_pk8`（PKCS#8 DER base64，无 PEM 头尾行）**，而非 `private_key`。
> The config key must be `private_key_pk8` (PKCS#8 DER base64 without PEM header/footer lines), NOT `private_key`.
>
> 如果持有 PKCS#1 格式（PEM 文件），先转换：
> ```bash
> openssl pkcs8 -topk8 -inform PEM -outform DER -nocrypt -in pk1.pem | base64 | tr -d '\n'
> ```

### 代码加载配置 / Load Config in Code

```csharp
using TigerOpenAPI.Config;
using TigerOpenAPI.Common.Enum;

// 从配置文件目录加载（ConfigFilePath 是目录路径，不是文件全路径）
TigerConfig config = new TigerConfig()
{
    ConfigFilePath = "path/to/config/dir/",
    Language = Language.en_US,
    TimeZone = CustomTimeZone.HK_ZONE
};
```

`TigerConfig` 主要属性：

| 属性 Property | 说明 | 默认值 |
|--------------|------|-------|
| `ConfigFilePath` | 配置文件所在目录路径 | — |
| `TigerId` | Tiger ID（可代替配置文件） | — |
| `PrivateKey` | RSA 私钥（可代替配置文件） | — |
| `DefaultAccount` | 默认账户号（交易请求自动填入） | 从配置文件读取 |
| `Environment` | `Env.PROD`（生产）/ `Env.SANDBOX`（沙箱） | `Env.PROD` |
| `Language` | `Language.zh_CN` / `Language.en_US` | `Language.zh_CN` |
| `TimeZone` | `CustomTimeZone.HK_ZONE` 等 | — |
| `FailRetryCounts` | HTTP 重试次数（0–5，Polly 指数退避） | 2 |
| `AutoGrabPermission` | 自动申请行情权限 | `false` |
| `UseFullTick` | 使用完整逐笔数据 | `false` |
| `IsSslSocket` | 推送连接使用 SSL | `true` |

---

## 创建客户端 / Create Clients

```csharp
using TigerOpenAPI.Quote;
using TigerOpenAPI.Trade;
using TigerOpenAPI.Push;

// 行情客户端 / Quote client
QuoteClient quoteClient = new QuoteClient(config);

// 交易客户端 / Trade client
TradeClient tradeClient = new TradeClient(config);

// 推送客户端（单例）/ Push client (singleton)
PushClient pushClient = PushClient.GetInstance()
    .Config(config)
    .ApiComposeCallback(new MyCallback());
```

---

## API 调用模式 / API Call Pattern

C# SDK 使用统一的 `TigerRequest<TResponse>` 泛型模式调用所有 API：

```csharp
using TigerOpenAPI.Model;
using TigerOpenAPI.Quote;
using TigerOpenAPI.Quote.Model;
using TigerOpenAPI.Quote.Response;

// 1. 构造请求 / Build request
var request = new TigerRequest<QuoteRealTimeQuoteResponse>()
{
    ApiMethodName = QuoteApiService.QUOTE_REAL_TIME,  // API 名称常量
    ModelValue = new QuoteSymbolModel()
    {
        Symbols = new List<string> { "AAPL", "TSLA" }
    }
};

// 2. 同步执行 / Sync execute
QuoteRealTimeQuoteResponse response = quoteClient.Execute(request);

// 3. 异步执行 / Async execute
QuoteRealTimeQuoteResponse asyncResponse = await quoteClient.ExecuteAsync(request);
```

---

## 错误处理 / Error Handling

```csharp
using TigerOpenAPI.Common.Exceptions;

try
{
    var request = new TigerRequest<QuoteRealTimeResponse>()
    {
        ApiMethodName = QuoteApiService.QUOTE_REAL_TIME,
        ModelValue = new QuoteRealTimeModel()
        {
            Symbols = new List<string> { "AAPL" }
        }
    };
    var response = await quoteClient.ExecuteAsync(request);

    if (response?.Data != null)
    {
        foreach (var item in response.Data)
        {
            Console.WriteLine($"{item.Symbol}: {item.LatestPrice}");
        }
    }
}
catch (ApiException ex)
{
    Console.WriteLine($"API Error [{ex.Code}]: {ex.Message}");
}
catch (Exception ex)
{
    Console.WriteLine($"Error: {ex.Message}");
}
```

---

## 直接调用 API / Raw API Call

当 SDK 封装方法不满足需求时，可直接使用基类 `TigerClient.Execute()`：

```csharp
using TigerOpenAPI.Common;
using TigerOpenAPI.Common.Model;

// QuoteClient 和 TradeClient 均继承自 TigerClient
// 使用 QuoteApiService / TradeApiService 中的常量作为 ApiMethodName
var rawRequest = new TigerRequest<RawResponse>()
{
    ApiMethodName = QuoteApiService.MARKET_STATE,
    ModelValue = new MarketStateModel() { Market = "US" }
};
var rawResponse = quoteClient.Execute(rawRequest);
```

---

## 模拟账户 vs 实盘 / Paper vs Live

C# SDK 的 `TradeClient` 会自动根据账户号判断是否为模拟账户并路由到不同地址：

```csharp
// 模拟账户（Paper）：账户号以特定前缀标识，自动路由
// 实盘账户（Live）：正常路由到生产环境

// 查询账户列表以确认账户类型 / Query accounts to verify type
var accountsRequest = new TigerRequest<AccountsResponse>()
{
    ApiMethodName = TradeApiService.ACCOUNTS,
    ModelValue = new AccountModel()
};
var accounts = await tradeClient.ExecuteAsync(accountsRequest);
```

---

## 前置条件 / Prerequisites

1. 老虎证券账户 + 开发者 API 权限：https://developer.itigerup.com/
2. 准备好 `tiger_id`、RSA 私钥（2048 位）、账户号
3. 行情数据需要对应市场的行情权限
4. .NET >= 6.0 / C# >= 8.0
