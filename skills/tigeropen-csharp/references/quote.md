# Tiger OpenAPI C# SDK — Market Data / 行情查询

> C# SDK 行情 API 参考 / Quote API Reference
<!-- 当用户提到"行情"、"报价"、"K线"、"价格"、"深度"、"quote"、"kline"、"price"时 -->

## 初始化 / Initialize

```csharp
using TigerOpenAPI.Config;
using TigerOpenAPI.Quote;
using TigerOpenAPI.Common.Enum;

TigerConfig config = new TigerConfig()
{
    ConfigFilePath = "path/to/config/",
    Language = Language.en_US,
};
QuoteClient quoteClient = new QuoteClient(config);
```

所有行情 API 均通过 `TigerRequest<TResponse>` 模式调用：

```csharp
var request = new TigerRequest<TResponse>()
{
    ApiMethodName = QuoteApiService.XXX,  // API 名称常量
    ModelValue = new XxxModel() { ... }   // 请求参数
};
var response = await quoteClient.ExecuteAsync(request);
```

---

## 市场状态 / Market State

```csharp
using TigerOpenAPI.Quote.Model;

var request = new TigerRequest<MarketStateResponse>()
{
    ApiMethodName = QuoteApiService.MARKET_STATE,
    ModelValue = new MarketStateModel() { Market = "US" }
    // market: "US" / "HK" / "CN" / "SG"
};
var response = await quoteClient.ExecuteAsync(request);
```

---

## 实时报价 / Real-time Quotes
<!-- 当用户提到"实时报价"、"最新价"、"real-time"时 -->

```csharp
var request = new TigerRequest<QuoteRealTimeResponse>()
{
    ApiMethodName = QuoteApiService.QUOTE_REAL_TIME,
    ModelValue = new QuoteRealTimeModel()
    {
        Symbols = new List<string> { "AAPL", "TSLA", "00700" }
    }
};
var response = await quoteClient.ExecuteAsync(request);

if (response?.Data != null)
{
    foreach (var q in response.Data)
    {
        Console.WriteLine($"{q.Symbol}: latest={q.LatestPrice}, " +
                          $"change={q.Change}, changeRatio={q.ChangeRatio}");
    }
}
```

`BRIEF`（简版报价，含盘前盘后）：

```csharp
var request = new TigerRequest<QuoteBriefResponse>()
{
    ApiMethodName = QuoteApiService.BRIEF,
    ModelValue = new QuoteBriefModel()
    {
        Symbols = new List<string> { "AAPL" },
        IncludeHourTrading = true,  // 包含盘前盘后
        IncludeAskBid = true        // 包含买卖盘
    }
};
```

---

## K 线 / Kline
<!-- 当用户提到"K线"、"kline"、"bar"、"日线"时 -->

```csharp
var request = new TigerRequest<KlineResponse>()
{
    ApiMethodName = QuoteApiService.KLINE,
    ModelValue = new KlineModel()
    {
        Symbols = new List<string> { "AAPL" },
        Period = "day",    // day/week/month/year/1min/3min/5min/15min/30min/60min
        BeginTime = -1,    // -1 = 最新
        EndTime = -1,
        Right = "br",      // br=前复权 / nr=不复权
        Limit = 251
    }
};
var response = await quoteClient.ExecuteAsync(request);

if (response?.Data != null)
{
    foreach (var bar in response.Data)
    {
        Console.WriteLine($"time={bar.Time}, open={bar.Open}, " +
                          $"high={bar.High}, low={bar.Low}, close={bar.Close}, vol={bar.Volume}");
    }
}
```

---

## 分时 / Timeline

```csharp
// 当日分时 / Current day timeline
var request = new TigerRequest<TimelineResponse>()
{
    ApiMethodName = QuoteApiService.TIMELINE,
    ModelValue = new TimelineModel()
    {
        Symbols = new List<string> { "AAPL" },
        IncludeHourTrading = false
    }
};

// 历史分时 / Historical timeline
var histRequest = new TigerRequest<HistoryTimelineResponse>()
{
    ApiMethodName = QuoteApiService.HISTORY_TIMELINE,
    ModelValue = new HistoryTimelineModel()
    {
        Symbols = new List<string> { "AAPL" },
        Date = "2024-01-15"  // yyyy-MM-dd
    }
};
```

---

## 深度行情 / Quote Depth
<!-- 当用户提到"买卖盘"、"深度"、"depth"时 -->

```csharp
var request = new TigerRequest<QuoteDepthResponse>()
{
    ApiMethodName = QuoteApiService.QUOTE_DEPTH,
    ModelValue = new QuoteDepthModel()
    {
        Symbols = new List<string> { "AAPL" },
        Market = "US"   // "US" / "HK"
    }
};
var response = await quoteClient.ExecuteAsync(request);
// 返回 asks/bids，各含 price, volume 的数组
```

---

## 逐笔成交 / Trade Ticks

```csharp
var request = new TigerRequest<TradeTickResponse>()
{
    ApiMethodName = QuoteApiService.TRADE_TICK,
    ModelValue = new TradeTickModel()
    {
        Symbols = new List<string> { "AAPL" },
        BeginIndex = -1,   // -1 = 最新
        EndIndex = -1,
        Limit = 100
    }
};
```

---

## 期权到期日 / Option Expirations

```csharp
var request = new TigerRequest<OptionExpirationResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_EXPIRATION,
    ModelValue = new OptionExpirationModel()
    {
        Symbols = new List<string> { "AAPL" }
    }
};
var response = await quoteClient.ExecuteAsync(request);
// 返回 dates（字符串日期列表）, timestamps, periodTag ("m"=月度/"w"=周度)
```

---

## 期权链 / Option Chain

```csharp
var request = new TigerRequest<OptionChainResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_CHAIN,
    ModelValue = new OptionChainModel()
    {
        Symbol = "AAPL",
        Expiry = "2025-08-29",  // yyyy-MM-dd 或毫秒时间戳
        // 可选筛选
        // Market = "US"
    }
};
var response = await quoteClient.ExecuteAsync(request);
// 每个合约含: identifier, strike, right(CALL/PUT), bid, ask, volume,
//            openInterest, impliedVol, delta, gamma 等
```

---

## 期权报价 / Option Brief

```csharp
// identifier 格式: "SYMBOL  YYMMDD[C/P]STRIKE*1000(8位)"
// 示例: "AAPL  250829C00150000"
var request = new TigerRequest<OptionBriefResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_BRIEF,
    ModelValue = new OptionBriefModel()
    {
        Identifiers = new List<string>
        {
            "AAPL  250829C00150000",
            "AAPL  250829P00150000"
        }
    }
};
```

---

## 期货 / Futures

```csharp
// 期货实时行情 / Future real-time quotes
var request = new TigerRequest<FutureRealTimeQuoteResponse>()
{
    ApiMethodName = QuoteApiService.FUTURE_REAL_TIME_QUOTE,
    ModelValue = new FutureRealTimeQuoteModel()
    {
        Symbols = new List<string> { "CL2509" }
    }
};

// 期货 K 线 / Future kline
var klineRequest = new TigerRequest<FutureKlineResponse>()
{
    ApiMethodName = QuoteApiService.FUTURE_KLINE,
    ModelValue = new FutureKlineModel()
    {
        Symbols = new List<string> { "CL2509" },
        Period = "day",
        Limit = 100
    }
};
```

---

## 资金流向 / Capital Flow

```csharp
var request = new TigerRequest<CapitalFlowResponse>()
{
    ApiMethodName = QuoteApiService.CAPITAL_FLOW,
    ModelValue = new CapitalFlowModel()
    {
        Symbol = "AAPL",
        Market = "US",
        Period = "day"  // day/week/month/intraday
    }
};

// 资金分布 / Capital distribution
var distRequest = new TigerRequest<CapitalDistributionResponse>()
{
    ApiMethodName = QuoteApiService.CAPITAL_DISTRIBUTION,
    ModelValue = new CapitalDistributionModel()
    {
        Symbol = "AAPL",
        Market = "US"
    }
};
```

---

## QuoteApiService 常量速查 / Constants

| 常量 Constant | API 说明 |
|--------------|---------|
| `MARKET_STATE` | 市场状态 |
| `BRIEF` | 实时报价（含盘前后） |
| `QUOTE_REAL_TIME` | 实时报价 |
| `KLINE` | K 线 |
| `TIMELINE` | 当日分时 |
| `HISTORY_TIMELINE` | 历史分时 |
| `TRADE_TICK` | 逐笔成交 |
| `QUOTE_DEPTH` | 深度行情 |
| `OPTION_EXPIRATION` | 期权到期日 |
| `OPTION_CHAIN` | 期权链 |
| `OPTION_BRIEF` | 期权报价 |
| `FUTURE_REAL_TIME_QUOTE` | 期货实时行情 |
| `FUTURE_KLINE` | 期货 K 线 |
| `CAPITAL_FLOW` | 资金流向 |
| `CAPITAL_DISTRIBUTION` | 资金分布 |

---

## 注意事项 / Notes

- 所有方法支持同步 `Execute()` 和异步 `ExecuteAsync()`
- `ConfigFilePath` 是目录路径（以 `/` 结尾），不是文件全路径
- 行情数据需要对应市场的行情权限
- 港股期权代码格式与美股不同：`TCH.HK 20241230 410.00 CALL`
- 期权 identifier 格式（美股）：`"AAPL  250829C00150000"`（注意两个空格）
