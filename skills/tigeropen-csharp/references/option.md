
# Tiger OpenAPI C# SDK — 期权 / Options Trading

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/quote-option

## 期权操作工作流 / Option Workflow

当用户提到期权时，按以下流程操作 / When user mentions options, follow this workflow:

### 查询期权 / Query Options

1. **查到期日 Get expirations**: `QuoteApiService.OPTION_EXPIRATION` → 获取可选到期日列表
2. **查期权链 Get chain**: `QuoteApiService.OPTION_CHAIN` → 获取指定到期日的所有合约
3. **查行情 Get quotes**: `QuoteApiService.OPTION_BRIEF` → 获取期权实时行情和 Greeks

### 港股期权特殊处理 / HK Option Special Handling

- 港股期权标的代码不同于正股 / HK option underlyings differ from stock codes: `00700` → `TCH`（腾讯）
- 使用 `QuoteApiService.OPTION_SYMBOL` 查询港股期权代码映射

---

## 初始化 / Initialize

```csharp
using TigerOpenAPI.Config;
using TigerOpenAPI.Quote;
using TigerOpenAPI.Trade;

TigerConfig config = new TigerConfig()
{
    ConfigFilePath = "path/to/config/dir/"
};
QuoteClient quoteClient = new QuoteClient(config);
TradeClient tradeClient = new TradeClient(config);
```

---

## 期权到期日 / Option Expirations

```csharp
using TigerOpenAPI.Quote.Model;
using TigerOpenAPI.Quote.Response;

var request = new TigerRequest<OptionExpirationResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_EXPIRATION,
    ModelValue = new OptionExpirationModel()
    {
        Symbols = new List<string> { "AAPL" },
        Market = "US"   // US / HK
    }
};
var response = await quoteClient.ExecuteAsync(request);

// response.OptionExpirationItems 为列表
// OptionExpirationItem 字段 / Fields:
// Symbol     - 股票代码
// Count      - 到期日数量
// Dates      - 到期日数组 (e.g. "2024-06-28")
// Timestamps - 到期日时间戳数组（毫秒，纽约时间）
// PeriodTags - 周期标签: "m"=月期权, "w"=周期权, "q"=季度
```

---

## 期权链 / Option Chain

```csharp
using TigerOpenAPI.Quote.Model;
using TigerOpenAPI.Quote.Response;

var request = new TigerRequest<OptionChainResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_CHAIN,
    ModelValue = new OptionChainModel()
    {
        Symbol = "AAPL",
        Expiry = "2025-08-29",
        Market = "US",
        ReturnGreekValue = true,   // 返回希腊字母 / Return Greeks
        // 可选筛选 / Optional filters:
        InTheMoney = true,
        ImpliedVolatilityMin = 0.15,
        ImpliedVolatilityMax = 0.80,
        DeltaMin = 0.2,
        DeltaMax = 0.8,
        OpenInterestMin = 100
    }
};
var response = await quoteClient.ExecuteAsync(request);

// OptionChainItem 字段 / Fields:
// Symbol      - 标的代码
// Expiry      - 到期日（毫秒）
// Items       - List<OptionRealTimeQuoteGroup>
//   .Call     - 看涨期权 OptionRealTimeQuote
//   .Put      - 看跌期权 OptionRealTimeQuote
//
// OptionRealTimeQuote 字段 / Fields:
// Identifier  - 期权完整代码
// Strike      - 行权价
// Right       - CALL / PUT
// AskPrice/BidPrice/LatestPrice
// Volume/OpenInterest
// ImpliedVol  - 隐含波动率
// Delta/Gamma/Theta/Vega/Rho
```

---

## 港股期权代码映射 / HK Option Symbol Mapping

```csharp
var request = new TigerRequest<OptionSymbolResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_SYMBOL,
    ModelValue = new OptionSymbolModel()
    {
        Market = "HK",
        Lang = "en_US"   // en_US / zh_CN / zh_TW
    }
};
var response = await quoteClient.ExecuteAsync(request);

// response.SymbolItems 列表
// OptionSymbolItem 字段 / Fields:
// Symbol           - 期权 symbol (e.g. "TCH.HK")
// Name             - 标的名称
// UnderlyingSymbol - 正股代码 (e.g. "00700")

// 然后使用映射后的代码查询港股期权 / Then use mapped symbol for HK queries
```

---

## 期权实时行情 / Option Brief (Real-time Quotes)

```csharp
using TigerOpenAPI.Quote.Model;
using TigerOpenAPI.Quote.Response;

var models = new List<OptionCommonModel>()
{
    new OptionCommonModel()
    {
        Symbol = "AAPL",
        Right = "CALL",
        Expiry = "2025-08-29",   // yyyy-MM-dd
        Strike = "150.0"          // 小数位须和期权链一致
    }
};

var request = new TigerRequest<OptionBriefResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_BRIEF,
    ModelValue = new OptionBriefModel()
    {
        OptionBasic = models,
        Market = "US"
    }
};
var response = await quoteClient.ExecuteAsync(request);

// OptionBriefItem 字段 / Fields:
// Identifier    - 期权完整代码
// Symbol        - 标的代码
// BidPrice/AskPrice/LatestPrice
// Volume/OpenInterest
// High/Low/Open/PreClose
// Change        - 涨跌额
// MidPrice      - 中间价
// MarkPrice     - 标记价格
// SellingReturn - 卖出年化收益率
```

---

## 期权深度行情 / Option Depth Quotes

```csharp
var models = new List<OptionCommonModel>()
{
    new OptionCommonModel()
    {
        Symbol = "AAPL",
        Right = "PUT",
        Expiry = "2024-06-28",
        Strike = "210.0"
    }
};

var request = new TigerRequest<OptionDepthResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_DEPTH,
    ModelValue = new OptionDepthModel()
    {
        OptionBasic = models,
        Market = "US"
    }
};
var response = await quoteClient.ExecuteAsync(request);

// OptionDepthItem 字段 / Fields:
// Ask/Bid - List<OptionDepthOrderBook>
//   Price   - 委托价
//   Volume  - 委托量
//   Code    - 交易所代码 (CBOE, PHLX 等)
```

---

## 期权逐笔成交 / Option Trade Ticks

```csharp
// 仅支持美股期权 / US market only

var models = new List<OptionCommonModel>()
{
    new OptionCommonModel()
    {
        Symbol = "AAPL",
        Right = "PUT",
        Expiry = "2024-03-08",
        Strike = "185.0"
    }
};

var request = new TigerRequest<OptionTradeTickResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_TRADE_TICK,
    ModelValue = new OptionTradeTickModel()
    {
        OptionBasic = models
    }
};
var response = await quoteClient.ExecuteAsync(request);

// OptionTradeTickItem 字段 / Fields:
// Items - List<TradeTickPoint>
//   Price  - 成交价
//   Volume - 成交量
//   Time   - 成交时间（毫秒）
```

---

## 期权K线 / Option K-line

```csharp
using TigerOpenAPI.Quote.Model;
using TigerOpenAPI.Quote.Response;

var klineModels = new List<OptionKlineModel>()
{
    new OptionKlineModel()
    {
        Symbol = "AAPL",
        Right = "CALL",
        Expiry = "2024-06-28",
        Strike = "170.0",
        BeginTime = "2024-06-26",
        EndTime = "2024-06-26 23:59:59",
        Period = "1min",     // day/1min/5min/30min/60min
        Limit = 10,
        SortDir = "DESC"
    }
};

var request = new TigerRequest<OptionKlineResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_KLINE,
    ModelValue = new OptionKlineQueryModel()
    {
        OptionQuery = klineModels,
        Market = "US"
    }
};
var response = await quoteClient.ExecuteAsync(request);

// OptionKlineItem 字段 / Fields:
// Items - List<OptionKlinePoint>
//   Open/High/Low/Close - 开高低收
//   Volume              - 成交量
//   Time                - 时间戳（毫秒）
//   OpenInterest        - 持仓量（仅日K线）
```

---

## 期权分时数据 / Option Timeline

```csharp
// 目前仅支持港股期权 / HK market only currently

var models = new List<OptionTimelineModel>()
{
    new OptionTimelineModel()
    {
        Symbol = "ALB.HK",
        Right = "CALL",
        Expiry = 1753878054000L,    // 毫秒时间戳
        Strike = "117.50"
    }
};

var request = new TigerRequest<OptionTimelineResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_TIMELINE,
    ModelValue = new OptionTimelineQueryModel()
    {
        OptionTimelineModels = models,
        Market = "HK"
    }
};
var response = await quoteClient.ExecuteAsync(request);

// OptionTimelineItem 字段 / Fields:
// PreClose  - 昨日收盘价
// Minutes   - List<OptionTimelinePoint>
//   Price     - 最新价
//   AvgPrice  - 均价
//   Volume    - 成交量
//   Time      - 时间戳（毫秒）
```

---

## 期权分析 / Option Analysis

```csharp
using TigerOpenAPI.Quote.Model;
using TigerOpenAPI.Quote.Response;

var items = new List<OptionAnalysisModel>()
{
    new OptionAnalysisModel()
    {
        Symbol = "AAPL",
        Period = "52week"    // 3year/52week/26week/13week
    },
    new OptionAnalysisModel()
    {
        Symbol = "TSLA",
        Period = "52week",
        RequireVolatilityList = true   // 返回IV/HV历史时序
    }
};

var request = new TigerRequest<OptionAnalysisResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_ANALYSIS,
    ModelValue = new OptionAnalysisQueryModel()
    {
        Symbols = items,
        Market = "US"
    }
};
var response = await quoteClient.ExecuteAsync(request);

// OptionAnalysisItem 字段 / Fields:
// Symbol           - 标的代码
// ImpliedVol30Days - 30日隐含波动率
// HisVolatility    - 历史波动率（30天）
// IvHisVRatio      - IV/HV 比率
// CallPutRatio     - Call/Put 比率
// ImpliedVolMetric - ImpliedVolMetric 对象
//   Percentile   - IV百分位（0%-100%）
//   Rank         - IV排名（0-1）
//   Period       - 分析周期
// VolatilityList  - List<VolatilityItem> (requireVolatilityList=true)
```

---

## 期权指标计算 / Option Indicator Calculation

```csharp
// 查询单个期权的盘中实时指标（Greeks、IV、杠杆等）
using TigerOpenAPI.Quote.Model;
using TigerOpenAPI.Quote.Response;

var request = new TigerRequest<OptionFundamentalsResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_INDICATOR,
    ModelValue = new OptionFundamentalsModel()
    {
        Symbol = "BABA",
        Right = "CALL",
        Strike = "205.0",
        Expiry = "2019-11-01"   // yyyy-MM-dd
    }
};
var response = await quoteClient.ExecuteAsync(request);

// 返回字段 / Response fields:
// Delta/Gamma/Theta/Vega/Rho - 希腊字母
// InsideValue       - 内在价值
// TimeValue         - 时间价值
// Leverage          - 杠杆率
// OpenInterest      - 未平仓量
// HistoryVolatility - 历史波动率（百分比，如24.38表示24.38%）
// Volatility        - 隐含波动率（百分比）
// PremiumRate       - 溢价率（百分比）
// ProfitRate        - 买入盈利率（百分比）
```

---

## 单腿期权下单 / Single-leg Option Order

```csharp
using TigerOpenAPI.Trade.Model;
using TigerOpenAPI.Trade.Response;

// 买入看涨期权 / Buy call option
var request = new TigerRequest<PlaceOrderResponse>()
{
    ApiMethodName = TradeApiService.PLACE_ORDER,
    ModelValue = new PlaceOrderModel()
    {
        Account = "123456",
        Symbol = "AAPL",
        SecType = "OPT",
        Expiry = "20250829",     // YYYYMMDD
        Strike = 150.0,
        Right = "CALL",
        Currency = "USD",
        Action = "BUY",
        OrderType = "LMT",
        LimitPrice = 5.0,
        Quantity = 1             // 1张 = 100股
    }
};
var response = await tradeClient.ExecuteAsync(request);
```

---

## 多腿组合策略 / Multi-leg Combo Strategies

```csharp
using TigerOpenAPI.Trade.Model;
using TigerOpenAPI.Trade.Response;

// 牛市看涨价差 / Bull Call Spread (VERTICAL)
var legs = new List<ContractLeg>()
{
    new ContractLeg()
    {
        Symbol = "AAPL",
        SecType = "OPT",
        Expiry = "20250829",
        Strike = 145.0,
        Right = "CALL",
        Action = "BUY",
        Ratio = 1
    },
    new ContractLeg()
    {
        Symbol = "AAPL",
        SecType = "OPT",
        Expiry = "20250829",
        Strike = 155.0,
        Right = "CALL",
        Action = "SELL",
        Ratio = 1
    }
};

var request = new TigerRequest<PlaceOrderResponse>()
{
    ApiMethodName = TradeApiService.PLACE_COMBO_ORDER,
    ModelValue = new PlaceComboOrderModel()
    {
        Account = "123456",
        ComboType = "VERTICAL",
        Action = "BUY",
        OrderType = "LMT",
        LimitPrice = 3.0,
        Quantity = 1,
        Legs = legs
    }
};
var response = await tradeClient.ExecuteAsync(request);
```

### 其他常用策略示例 / Other Common Strategy Examples

```csharp
// 跨式策略 / Straddle — 同行权价 Call+Put，BUY
// ComboType = "STRADDLE"

// 日历价差 / Calendar Spread — 近月卖远月买，同行权价
// ComboType = "CALENDAR"

// Iron Condor — 4条腿，使用 CUSTOM
// ComboType = "CUSTOM"

// 备兑策略 / Covered Call — 买股 + 卖 Call
// Leg1: SecType="STK", Action="BUY", Ratio=100
// Leg2: SecType="OPT", Action="SELL"
// ComboType = "COVERED"
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

```csharp
var request = new TigerRequest<PositionsResponse>()
{
    ApiMethodName = TradeApiService.POSITIONS,
    ModelValue = new PositionModel()
    {
        Account = "123456",
        SecType = "OPT"
    }
};
var response = await tradeClient.ExecuteAsync(request);

// 期权持仓额外字段 / Option-specific position fields:
// Strike - 行权价
// Expiry - 到期日
// Right  - CALL/PUT
```

---

## 注意事项 / Notes

- 港股期权需先用 `OPTION_SYMBOL` 获取代码映射 / HK options require symbol mapping
- 期权每张合约通常代表100股标的 / Each contract = 100 shares
- 期权链返回的 Greeks 为上一交易日收盘值，盘中请用 `OPTION_INDICATOR` / Chain Greeks are from previous close; use `OPTION_INDICATOR` for intraday
- 行权价小数位须和期权链一致 / Strike decimals must match the option chain
- 期权行情需要期权行情权限 / Option quotes require option quote permission
- 支持 `Execute()` 同步和 `ExecuteAsync()` 异步两种调用方式
- 机构用户额外传 `SecretKey` 字段
