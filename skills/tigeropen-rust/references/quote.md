# Tiger OpenAPI Rust SDK — Market Data / 行情查询

> Rust SDK 行情 API 参考 / Quote API Reference (async)
<!-- 当用户提到"行情"、"报价"、"K线"、"价格"、"深度"、"quote"、"kline"、"price"时 -->

## 初始化 / Initialize

```rust
use tigeropen::config::ClientConfig;
use tigeropen::client::HttpClient;
use tigeropen::quote::QuoteClient;

let config = ClientConfig::builder()
    .properties_file("tiger_openapi_config.properties")
    .build()?;
let http_client = HttpClient::new(config)?;
let qc = QuoteClient::new(&http_client);
```

---

## 市场状态 / Market State

```rust
// market: "US" / "HK" / "CN" / "SG"
let data = qc.market_state("US").await?;
// 返回 Option<Value>，解析示例:
// [{"market":"US","status":"Trading","openTime":"09:30","closeTime":"16:00",...}]
```

---

## 实时报价 / Real-time Quotes
<!-- 当用户提到"实时报价"、"最新价"、"real-time"时 -->

```rust
let data = qc.quote_real_time(&["AAPL", "TSLA"]).await?;
// 每个元素包含: latestPrice, askPrice, bidPrice, volume, change, changeRatio 等
```

---

## K 线 / Kline
<!-- 当用户提到"K线"、"kline"、"bar"、"日线"时 -->

```rust
// period: "day"/"week"/"month"/"year"/"1min"/"3min"/"5min"/"10min"/"15min"/"30min"/"60min"
let data = qc.kline("AAPL", "day").await?;
// 返回数组，每个元素: time, open, high, low, close, volume
```

---

## 分时 / Timeline

```rust
let data = qc.timeline(&["AAPL", "TSLA"]).await?;
// 返回分时数据: time, price, volume, avgPrice
```

---

## 深度行情 / Quote Depth
<!-- 当用户提到"买卖盘"、"深度"、"depth"时 -->

```rust
let data = qc.quote_depth("AAPL").await?;
// 返回 asks/bids 数组，每层: price, volume
```

---

## 逐笔成交 / Trade Ticks

```rust
let data = qc.trade_tick(&["AAPL"]).await?;
// 返回逐笔: time, price, volume, direction
```

---

## 期权到期日 / Option Expirations

```rust
let data = qc.option_expiration("AAPL").await?;
// 返回到期日列表: dates, timestamps, period_tag ("m"=月度/"w"=周度)
```

---

## 期权链 / Option Chain

```rust
let data = qc.option_chain("AAPL", "2025-08-29").await?;
// 返回 call/put 合约: identifier, strike, bid/ask, volume, openInterest, impliedVol, delta, gamma 等
```

---

## 期权报价 / Option Brief

```rust
let data = qc.option_brief(&["AAPL  250829C00150000"]).await?;
// 期权代码格式: 标的(6位右空格填充) + YYMMDD + C/P + 行权价*1000(8位)
// 返回: identifier, strike, put_call, latestPrice, bid/ask, impliedVol, delta, gamma, theta, vega 等
```

---

## 期权 K 线 / Option Kline

```rust
let data = qc.option_kline("AAPL  250829C00150000", "day").await?;
```

---

## 期货 / Futures

```rust
// 获取交易所列表
let exchanges = qc.future_exchange().await?;

// 获取期货合约列表
let contracts = qc.future_contracts("CME").await?;

// 期货实时报价
let data = qc.future_real_time_quote(&["CL2509"]).await?;

// 期货 K 线
let data = qc.future_kline("CL2509", "day").await?;
```

---

## 资金流向 / Capital Flow

```rust
let data = qc.capital_flow("AAPL").await?;
let data = qc.capital_distribution("AAPL").await?;
```

---

## 选股器 / Market Scanner

```rust
use serde_json::json;

let data = qc.market_scanner(json!({
    "market": "US",
    "scanCode": "TOP_VOLUME",  // 成交量排行
})).await?;
```

---

## 直接调用 API / Raw API Call

当以上方法不满足需求时：

```rust
use serde_json::json;

let result = http_client.execute_raw("quote_real_time", json!({
    "symbols": ["AAPL", "TSLA"],
})).await?;
```

---

## 解析返回值 / Parse Response

所有方法返回 `Result<Option<serde_json::Value>, TigerError>`，需手动解析：

```rust
use serde_json::Value;

if let Some(data) = qc.quote_real_time(&["AAPL"]).await? {
    // 作为数组解析
    if let Value::Array(items) = &data {
        for item in items {
            let symbol = item["symbol"].as_str().unwrap_or("");
            let price = item["latestPrice"].as_f64().unwrap_or(0.0);
            println!("{}: {:.2}", symbol, price);
        }
    }
}
```

---

## 注意事项 / Notes

- 所有方法为 `async`，必须在 tokio 运行时中 `.await`
- 返回 `Ok(None)` 表示请求成功但无数据（如市场关闭时）
- 行情数据需要对应市场的行情权限
- 期权行情需要单独开通期权行情权限
- 港股期权代码格式：`TCH.HK 230616C00550000`（与美股不同）
