
# Tiger OpenAPI Go SDK — 期权 / Options Trading

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/quote-option

## 期权操作工作流 / Option Workflow

当用户提到期权时，按以下流程操作 / When user mentions options, follow this workflow:

### 查询期权 / Query Options

1. **查到期日 Get expirations**: `qc.OptionExpirations()` → 获取可选到期日列表
2. **查期权链 Get chain**: `qc.OptionChain()` → 获取指定到期日的所有合约
3. **查行情 Get quotes**: `qc.OptionBriefs()` → 获取期权实时行情和 Greeks

### 港股期权特殊处理 / HK Option Special Handling

- 港股期权标的代码不同于正股 / HK option underlyings differ from stock codes: `00700` → `TCH`（腾讯）
- 使用 `qc.OptionSymbols()` 查询港股期权代码映射

---

## 初始化 / Initialize

```go
import (
    "github.com/tigerfintech/openapi-sdks/go/config"
    "github.com/tigerfintech/openapi-sdks/go/client"
    "github.com/tigerfintech/openapi-sdks/go/quote"
    "github.com/tigerfintech/openapi-sdks/go/trade"
)

cfg, err := config.NewClientConfig(
    config.WithPropertiesFile("tiger_openapi_config.properties"),
)
httpClient := client.NewHttpClient(cfg)
qc := quote.NewQuoteClient(httpClient)
tc := trade.NewTradeClient(httpClient, cfg.Account)
```

---

## 期权到期日 / Option Expirations

```go
result, err := qc.OptionExpirations(map[string]interface{}{
    "symbols": []string{"AAPL"},
    "market":  "US",   // US / HK
})

var expirations []map[string]interface{}
json.Unmarshal(result, &expirations)

// 返回字段 / Response fields:
// symbol     - 股票代码
// count      - 到期日数量
// dates      - 到期日数组 (e.g. "2024-06-28")
// timestamps - 到期日时间戳数组（毫秒，纽约时间）
// periodTags - 周期标签: "m"=月期权, "w"=周期权, "q"=季度
```

---

## 期权链 / Option Chain

```go
result, err := qc.OptionChain(map[string]interface{}{
    "symbol":  "AAPL",
    "expiry":  "2025-08-29",
    "market":  "US",
    "return_greek_value": true,   // 返回希腊字母 / Return Greeks
    // 可选筛选 / Optional filters:
    // "in_the_money": true,
    // "implied_volatility_min": 0.15,
    // "implied_volatility_max": 0.80,
    // "delta_min": 0.2,
    // "delta_max": 0.8,
    // "open_interest_min": 100,
})

var chainItems []map[string]interface{}
json.Unmarshal(result, &chainItems)

// 返回字段 / Response fields (items[].call / items[].put):
// identifier   - 期权完整代码 (e.g. "AAPL  250829C00150000")
// strike       - 行权价
// right        - "CALL" / "PUT"
// askPrice     - 卖价
// bidPrice     - 买价
// latestPrice  - 最新价
// volume       - 成交量
// openInterest - 持仓量
// impliedVol   - 隐含波动率
// delta/gamma/theta/vega/rho - 希腊字母（需 return_greek_value=true）
```

---

## 港股期权代码映射 / HK Option Symbol Mapping

```go
result, err := qc.OptionSymbols(map[string]interface{}{
    "market": "HK",
    "lang":   "en_US",   // en_US / zh_CN / zh_TW
})

var symbols []map[string]interface{}
json.Unmarshal(result, &symbols)

// 返回字段 / Response fields:
// symbol           - 期权 symbol (e.g. "TCH.HK")
// name             - 标的名称
// underlyingSymbol - 正股代码 (e.g. "00700")

// 然后使用映射后的代码查询港股期权 / Then use mapped symbol for HK queries
```

---

## 期权实时行情 / Option Brief (Real-time Quotes)

```go
result, err := qc.OptionBriefs(map[string]interface{}{
    "market": "US",
    "option_basic": []map[string]interface{}{
        {
            "symbol": "AAPL",
            "right":  "CALL",
            "expiry": "2025-08-29",   // yyyy-MM-dd
            "strike": "150.0",         // 小数位须和期权链一致
        },
    },
})

var briefs []map[string]interface{}
json.Unmarshal(result, &briefs)

// 返回字段 / Response fields:
// identifier    - 期权完整代码
// symbol        - 标的代码
// bidPrice/askPrice/latestPrice
// volume/openInterest
// high/low/open/preClose
// change        - 涨跌额
// midPrice      - 中间价
// markPrice     - 标记价格
// sellingReturn - 卖出年化收益率
```

---

## 期权深度行情 / Option Depth Quotes

```go
result, err := qc.OptionDepth(map[string]interface{}{
    "market": "US",
    "option_basic": []map[string]interface{}{
        {
            "symbol": "AAPL",
            "right":  "PUT",
            "expiry": "2024-06-28",
            "strike": "210.0",
        },
    },
})

var depth []map[string]interface{}
json.Unmarshal(result, &depth)

// 返回字段 / Response fields:
// ask[]/bid[] - 卖/买盘挂单列表
//   price    - 委托价
//   volume   - 委托量
//   code     - 交易所代码 (CBOE, PHLX 等)
//   timestamp - 交易所时间
```

---

## 期权逐笔成交 / Option Trade Ticks

```go
// 仅支持美股期权 / US market only

result, err := qc.OptionTradeTicks(map[string]interface{}{
    "option_basic": []map[string]interface{}{
        {
            "symbol": "AAPL",
            "right":  "PUT",
            "expiry": "2024-03-08",
            "strike": "185.0",
        },
    },
})

var ticks []map[string]interface{}
json.Unmarshal(result, &ticks)

// 返回字段 / Response fields:
// items[] - 逐笔成交列表
//   price  - 成交价
//   volume - 成交量
//   time   - 成交时间（毫秒）
```

---

## 期权K线 / Option K-line

```go
result, err := qc.OptionKline(map[string]interface{}{
    "market": "US",
    "option_query": []map[string]interface{}{
        {
            "symbol":     "AAPL",
            "right":      "CALL",
            "expiry":     "2024-06-28",
            "strike":     "170.0",
            "begin_time": "2024-06-26",
            "end_time":   "2024-06-26 23:59:59",
            "period":     "1min",   // day/1min/5min/30min/60min
            "limit":      10,
            "sort_dir":   "DESC",
        },
    },
})

var klines []map[string]interface{}
json.Unmarshal(result, &klines)

// 返回字段 / Response fields:
// items[] - K线数据点列表
//   open/high/low/close - 开高低收
//   volume              - 成交量
//   time                - 时间戳（毫秒）
//   openInterest        - 持仓量（仅日K线）
```

---

## 期权分时数据 / Option Timeline

```go
// 目前仅支持港股期权 / HK market only currently

result, err := qc.OptionTimeline(map[string]interface{}{
    "market": "HK",
    "option_list": []map[string]interface{}{
        {
            "symbol": "ALB.HK",
            "right":  "CALL",
            "expiry": int64(1753878054000),   // 毫秒时间戳
            "strike": "117.50",
        },
    },
})

var timeline []map[string]interface{}
json.Unmarshal(result, &timeline)

// 返回字段 / Response fields:
// preClose   - 昨日收盘价
// minutes[]  - 分时数据点
//   price    - 最新价
//   avgPrice - 均价
//   volume   - 成交量
//   time     - 时间戳（毫秒）
```

---

## 期权分析 / Option Analysis

```go
result, err := qc.OptionAnalysis(map[string]interface{}{
    "symbols": []map[string]interface{}{
        {
            "symbol":                "AAPL",
            "period":                "52week",   // 3year/52week/26week/13week
            "requireVolatilityList": true,        // 返回IV/HV历史时序
        },
    },
    "market": "US",
})

var analysis []map[string]interface{}
json.Unmarshal(result, &analysis)

// 返回字段 / Response fields:
// symbol           - 标的代码
// impliedVol30Days - 30日隐含波动率
// hisVolatility    - 历史波动率（30天）
// ivHisVRatio      - IV/HV 比率
// callPutRatio     - Call/Put 比率
// impliedVolMetric:
//   percentile  - IV百分位（0%-100%）
//   rank        - IV排名（0-1）
//   period      - 分析周期
```

---

## 期权指标计算 / Option Indicator Calculation

```go
result, err := qc.OptionIndicator(map[string]interface{}{
    "symbol": "BABA",
    "right":  "CALL",
    "strike": "205.0",
    "expiry": "2019-11-01",   // yyyy-MM-dd
})

var indicator map[string]interface{}
json.Unmarshal(result, &indicator)

// 返回字段 / Response fields:
// delta/gamma/theta/vega/rho - 希腊字母
// insideValue      - 内在价值
// timeValue        - 时间价值
// leverage         - 杠杆率
// openInterest     - 未平仓量
// historyVolatility - 历史波动率（百分比，如24.38表示24.38%）
// volatility       - 隐含波动率（百分比）
// premiumRate      - 溢价率（百分比）
// profitRate       - 买入盈利率（百分比）
```

---

## 单腿期权下单 / Single-leg Option Order

```go
// 买入看涨期权 / Buy call option
result, err := tc.PlaceOrder(map[string]interface{}{
    "account": cfg.Account,
    "contract": map[string]interface{}{
        "symbol":   "AAPL",
        "sec_type": "OPT",
        "expiry":   "20250829",   // YYYYMMDD
        "strike":   150.0,
        "right":    "CALL",
        "currency": "USD",
    },
    "action":      "BUY",
    "order_type":  "LMT",
    "limit_price": 5.0,
    "quantity":    1,             // 1张 = 100股
})

var order map[string]interface{}
json.Unmarshal(result, &order)
```

---

## 多腿组合策略 / Multi-leg Combo Strategies

```go
// 牛市看涨价差 / Bull Call Spread (VERTICAL)
result, err := tc.PlaceComboOrder(map[string]interface{}{
    "account":     cfg.Account,
    "combo_type":  "VERTICAL",
    "action":      "BUY",
    "order_type":  "LMT",
    "limit_price": 3.0,
    "quantity":    1,
    "legs": []map[string]interface{}{
        {
            "symbol":   "AAPL",
            "sec_type": "OPT",
            "expiry":   "20250829",
            "strike":   145.0,
            "right":    "CALL",
            "action":   "BUY",
            "ratio":    1,
        },
        {
            "symbol":   "AAPL",
            "sec_type": "OPT",
            "expiry":   "20250829",
            "strike":   155.0,
            "right":    "CALL",
            "action":   "SELL",
            "ratio":    1,
        },
    },
})
```

### 其他策略示例 / Other Strategy Examples

```go
// 跨式策略 / Straddle
// combo_type: "STRADDLE"，同行权价 Call+Put，action: "BUY"

// Iron Condor（4腿，使用 CUSTOM）
// combo_type: "CUSTOM"，4条腿：Put spread + Call spread

// 备兑策略 / Covered Call
// combo_type: "COVERED"
// Leg1: sec_type="STK", action="BUY", ratio=100
// Leg2: sec_type="OPT", action="SELL", ratio=1
```

### 组合策略类型总览 / Combo Strategy Types

| ComboType | 策略 Strategy | 说明 Description |
|-----------|--------------|-----------------|
| `VERTICAL` | 垂直价差 | 同到期日不同行权价 Same expiry, different strikes |
| `STRADDLE` | 跨式 | 同行权价同到期日 Call+Put, same strike & expiry |
| `STRANGLE` | 宽跨式 | 不同行权价同到期日 Different strikes, same expiry |
| `CALENDAR` | 日历价差 | 同行权价不同到期日 Same strike, different expiries |
| `DIAGONAL` | 对角线价差 | 不同行权价不同到期日 Different strikes & expiries |
| `COVERED` | 备兑 | 持有股票+卖Call Long stock + short call |
| `PROTECTIVE` | 保护性 | 持有股票+买Put Long stock + long put |
| `SYNTHETIC` | 合成 | 合成多/空头 Synthetic long/short |
| `CUSTOM` | 自定义 | 4条腿组合（Iron Condor等） |

---

## 查询期权持仓 / Query Option Positions

```go
result, err := tc.Positions(map[string]interface{}{
    "account":  cfg.Account,
    "sec_type": "OPT",
})

var positions []map[string]interface{}
json.Unmarshal(result, &positions)

// 期权持仓额外字段 / Option-specific position fields:
// strike - 行权价
// expiry - 到期日
// right  - CALL/PUT
```

---

## 注意事项 / Notes

- 所有 API 返回 `(json.RawMessage, error)`，需自行 `json.Unmarshal` 解析
- 港股期权需先用 `OptionSymbols()` 获取代码映射 / HK options require symbol mapping
- 期权每张合约通常代表100股标的 / Each contract = 100 shares
- 期权链返回的 Greeks 为上一交易日收盘值，盘中请用 `OptionIndicator()` / Chain Greeks are from previous close
- 行权价小数位须和期权链一致 / Strike decimals must match the option chain
- 期权行情需要期权行情权限 / Option quotes require option quote permission
- 机构用户额外传 `"secret_key"` 字段
