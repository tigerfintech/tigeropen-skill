# Tiger OpenAPI Go SDK — Market Data / 行情查询

> Go SDK 行情 API 参考 / Quote API Reference
<!-- 当用户提到"行情"、"报价"、"K线"、"价格"、"深度"、"quote"、"kline"、"price"时 -->

## 初始化 / Initialize

```go
import (
    "github.com/tigerfintech/openapi-sdks/go/client"
    "github.com/tigerfintech/openapi-sdks/go/config"
    "github.com/tigerfintech/openapi-sdks/go/quote"
)

cfg, _ := config.NewClientConfig(
    config.WithPropertiesFile("tiger_openapi_config.properties"),
)
httpClient := client.NewHttpClient(cfg)
qc := quote.NewQuoteClient(httpClient)
```

---

## 市场状态 / Market State

```go
// market: "US" / "HK" / "CN" / "SG"
result, err := qc.MarketState("US")
// 返回 json.RawMessage，解析示例:
// [{"market":"US","status":"Trading","openTime":"09:30","closeTime":"16:00",...}]
```

---

## 实时报价 / Real-time Quotes
<!-- 当用户提到"实时报价"、"最新价"、"real-time"时 -->

```go
result, err := qc.QuoteRealTime([]string{"AAPL", "TSLA"})
// 返回 json.RawMessage
// 每个元素包含: symbol, latestPrice, askPrice, bidPrice, volume, change, changeRatio 等
```

---

## K 线 / Kline
<!-- 当用户提到"K线"、"kline"、"bar"、"日线"时 -->

```go
// period: "day"/"week"/"month"/"year"/"1min"/"3min"/"5min"/"10min"/"15min"/"30min"/"60min"
result, err := qc.Kline("AAPL", "day")
// 返回数组，每个元素: time, open, high, low, close, volume
```

---

## 分时 / Timeline

```go
result, err := qc.Timeline([]string{"AAPL", "TSLA"})
// 返回分时数据: time, price, volume, avgPrice
```

---

## 深度行情 / Quote Depth
<!-- 当用户提到"买卖盘"、"深度"、"depth"时 -->

```go
result, err := qc.QuoteDepth("AAPL")
// 返回 asks/bids 数组，每层: price, volume
```

---

## 逐笔成交 / Trade Ticks

```go
result, err := qc.TradeTick([]string{"AAPL"})
// 返回逐笔: time, price, volume, direction
```

---

## 期权到期日 / Option Expirations

```go
result, err := qc.OptionExpiration("AAPL")
// 返回到期日列表: dates, timestamps, period_tag ("m"=月度/"w"=周度)
```

---

## 期权链 / Option Chain

```go
result, err := qc.OptionChain("AAPL", "2025-08-29")
// 返回 call/put 合约: identifier, strike, bid/ask, volume, openInterest, impliedVol, delta, gamma 等
```

---

## 期权报价 / Option Brief

```go
result, err := qc.OptionBrief([]string{"AAPL  250829C00150000"})
// 期权代码格式: 标的(6位右空格填充) + YYMMDD + C/P + 行权价*1000(8位)
// 返回: identifier, strike, put_call, latestPrice, bid/ask, impliedVol, delta, gamma, theta, vega 等
```

---

## 期货交易所 / Future Exchange

```go
result, err := qc.FutureExchange()
// 返回交易所列表
```

---

## 直接调用 API / Raw API Call

当以上方法不满足需求时，直接使用 method 名调用：

```go
httpClient := client.NewHttpClient(cfg)
result, err := httpClient.ExecuteRaw("quote_real_time", map[string]interface{}{
    "symbols": []string{"AAPL", "TSLA"},
})
```

---

## 注意事项 / Notes

- 所有返回值为 `json.RawMessage`，需 `json.Unmarshal` 解析
- 行情数据需要对应市场的行情权限
- 期权行情需要单独开通期权行情权限
- 港股期权代码格式：`TCH.HK 230616C00550000`（与美股不同）
