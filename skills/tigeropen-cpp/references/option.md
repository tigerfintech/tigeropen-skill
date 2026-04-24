
# Tiger OpenAPI C++ SDK — 期权 / Options Trading

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/quote-option

## 期权操作工作流 / Option Workflow

当用户提到期权时，按以下流程操作 / When user mentions options, follow this workflow:

### 查询期权 / Query Options

1. **查到期日 Get expirations**: `OPTION_EXPIRATION` → 获取可选到期日列表
2. **查期权链 Get chain**: `OPTION_CHAIN` → 获取指定到期日的所有合约，可按 Greeks 筛选
3. **查行情 Get quotes**: `OPTION_BRIEF` → 获取期权实时行情和希腊字母

### 港股期权特殊处理 / HK Option Special Handling

- 港股期权标的代码不同于正股 / HK option underlyings differ from stock codes: `00700` → `TCH`（腾讯）
- 使用 `OPTION_SYMBOL` 查询港股期权代码映射

---

## 初始化 / Initialize

```cpp
#include "tigerapi/client_config.h"
#include "tigerapi/trade_client.h"
#include "tigerapi/quote_client.h"

using namespace TIGER_API;

ClientConfig config(false, "path/to/config/dir/");
auto trade_client = make_shared<TradeClient>(config);
auto quote_client = make_shared<QuoteClient>(config);
```

---

## 期权到期日 / Option Expirations

```cpp
value obj = value::object(true);
obj[U("symbols")] = value::array();
obj[U("symbols")][0] = value::string(U("AAPL"));
obj[U("market")] = value::string(U("US"));  // US / HK

value result = quote_client->post(OPTION_EXPIRATION, obj);
ucout << result << endl;

// 返回字段 / Response fields:
// symbol     - 股票代码
// count      - 到期日数量
// dates      - 到期日数组 (e.g. "2024-06-28")
// timestamps - 到期日时间戳数组（毫秒，纽约时间）
// periodTags - 期权周期标签: "m"=月期权, "w"=周期权, "q"=季度
```

---

## 期权链 / Option Chain

```cpp
value obj = value::object(true);
obj[U("symbol")] = value::string(U("AAPL"));
obj[U("expiry")] = value::string(U("2025-08-29"));
obj[U("market")] = value::string(U("US"));

// 可选筛选参数 / Optional filter params:
// obj[U("in_the_money")] = value::boolean(true);
// obj[U("implied_volatility_min")] = value::number(0.15);
// obj[U("implied_volatility_max")] = value::number(0.80);
// obj[U("delta_min")] = value::number(0.2);
// obj[U("delta_max")] = value::number(0.8);
// obj[U("open_interest_min")] = value::number(100);
// obj[U("return_greek_value")] = value::boolean(true); // 返回 Greeks

value result = quote_client->post(OPTION_CHAIN, obj);
ucout << result << endl;

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
// delta        - Delta（需 return_greek_value=true）
// gamma        - Gamma
// theta        - Theta
// vega         - Vega
// rho          - Rho
```

---

## 港股期权代码映射 / HK Option Symbol Mapping

```cpp
// 港股期权标的代码与股票代码不同，需先查询映射
// HK option underlying symbols differ from stock codes

value obj = value::object(true);
obj[U("market")] = value::string(U("HK"));

value result = quote_client->post(OPTION_SYMBOL, obj);
ucout << result << endl;

// 返回字段 / Response fields:
// symbol           - 期权四要素 symbol (e.g. "TCH.HK")
// name             - 标的名称
// underlyingSymbol - 正股代码 (e.g. "00700")

// 然后使用映射后的代码查询港股期权 / Then use mapped symbol:
// obj[U("symbols")][0] = value::string(U("TCH.HK"));
// obj[U("market")] = value::string(U("HK"));
// quote_client->post(OPTION_EXPIRATION, obj);
```

---

## 期权实时行情 / Option Brief (Real-time Quotes)

```cpp
value obj = value::object(true);
obj[U("market")] = value::string(U("US"));

// option_basic 列表（支持批量，最多30条）
value option_basic = value::array();
value item = value::object(true);
item[U("symbol")] = value::string(U("AAPL"));
item[U("right")] = value::string(U("CALL"));
item[U("expiry")] = value::string(U("2025-08-29"));  // yyyy-MM-dd
item[U("strike")] = value::string(U("150.0"));
option_basic[0] = item;
obj[U("option_basic")] = option_basic;

value result = quote_client->post(OPTION_BRIEF, obj);
ucout << result << endl;

// 返回字段 / Response fields:
// identifier    - 期权完整代码
// symbol        - 标的代码
// bidPrice      - 买盘价格
// askPrice      - 卖盘价格
// latestPrice   - 最新价
// volume        - 成交量
// openInterest  - 未平仓量
// high/low/open - 最高/最低/开盘
// impliedVol    - 隐含波动率（通过分析接口获取）
// midPrice      - 中间价
// markPrice     - 标记价格
// sellingReturn - 卖出年化收益率
```

---

## 期权深度行情 / Option Depth Quotes

```cpp
value obj = value::object(true);
obj[U("market")] = value::string(U("US"));

value option_basic = value::array();
value item = value::object(true);
item[U("symbol")] = value::string(U("AAPL"));
item[U("right")] = value::string(U("PUT"));
item[U("expiry")] = value::string(U("2024-06-28"));
item[U("strike")] = value::string(U("210.0"));
option_basic[0] = item;
obj[U("option_basic")] = option_basic;

value result = quote_client->post(OPTION_DEPTH, obj);
ucout << result << endl;

// 返回字段 / Response fields:
// ask[]/bid[] - 卖/买盘挂单列表
//   price    - 委托价
//   volume   - 委托量
//   code     - 交易所代码 (CBOE, PHLX 等)
//   timestamp - 交易所时间
```

---

## 期权逐笔成交 / Option Trade Ticks

```cpp
// 仅支持美股期权 / US market only

value obj = value::object(true);

value option_list = value::array();
value item = value::object(true);
item[U("symbol")] = value::string(U("AAPL"));
item[U("right")] = value::string(U("PUT"));
item[U("expiry")] = value::string(U("2024-03-08"));
item[U("strike")] = value::string(U("185.0"));
option_list[0] = item;
obj[U("option_basic")] = option_list;

value result = quote_client->post(OPTION_TRADE_TICK, obj);
ucout << result << endl;

// 返回字段 / Response fields:
// items[] - 逐笔成交列表
//   price  - 成交价
//   volume - 成交量
//   time   - 成交时间（毫秒）
```

---

## 期权K线 / Option K-line

```cpp
value obj = value::object(true);
obj[U("market")] = value::string(U("US"));

value option_query = value::array();
value item = value::object(true);
item[U("symbol")] = value::string(U("AAPL"));
item[U("right")] = value::string(U("CALL"));
item[U("expiry")] = value::string(U("2024-06-28"));
item[U("strike")] = value::string(U("170.0"));
item[U("begin_time")] = value::string(U("2024-06-26"));
item[U("end_time")] = value::string(U("2024-06-26 23:59:59"));
item[U("period")] = value::string(U("1min"));  // day/1min/5min/30min/60min
option_query[0] = item;
obj[U("option_query")] = option_query;

value result = quote_client->post(OPTION_KLINE, obj);
ucout << result << endl;

// 返回字段 / Response fields:
// items[] - K线数据点列表
//   open/high/low/close - 开高低收
//   volume    - 成交量
//   time      - 时间戳（毫秒）
//   openInterest - 持仓量（仅日K线）
```

---

## 期权分时数据 / Option Timeline

```cpp
// 目前仅支持港股期权 / HK market only currently

value obj = value::object(true);
obj[U("market")] = value::string(U("HK"));

value option_list = value::array();
value item = value::object(true);
item[U("symbol")] = value::string(U("ALB.HK"));
item[U("right")] = value::string(U("CALL"));
item[U("expiry")] = value::number(1753878054000LL);  // 毫秒时间戳
item[U("strike")] = value::string(U("117.50"));
option_list[0] = item;
obj[U("option_list")] = option_list;

value result = quote_client->post(OPTION_TIMELINE, obj);
ucout << result << endl;

// 返回字段 / Response fields:
// preClose    - 昨日收盘价
// minutes[]  - 分时数据点
//   price     - 最新价
//   avgPrice  - 均价
//   volume    - 成交量
//   time      - 时间戳（毫秒）
```

---

## 期权分析 / Option Analysis

```cpp
value obj = value::object(true);

value symbols = value::array();
value sym_item = value::object(true);
sym_item[U("symbol")] = value::string(U("AAPL"));
sym_item[U("period")] = value::string(U("52week"));  // 3year/52week/26week/13week
symbols[0] = sym_item;
obj[U("symbols")] = symbols;
obj[U("market")] = value::string(U("US"));

value result = quote_client->post(OPTION_ANALYSIS, obj);
ucout << result << endl;

// 返回字段 / Response fields:
// symbol          - 标的代码
// impliedVol30Days - 30日隐含波动率
// hisVolatility   - 历史波动率（30天）
// ivHisVRatio     - IV/HV 比率
// callPutRatio    - Call/Put 比率
// impliedVolMetric:
//   percentile  - IV百分位（0%-100%）
//   rank        - IV排名（0-1）
//   period      - 分析周期
```

---

## 期权指标计算 / Option Indicator Calculation

```cpp
// 查询单个期权的盘中实时指标（Greeks、IV、杠杆等）
value obj = value::object(true);
obj[U("symbol")] = value::string(U("BABA"));
obj[U("right")] = value::string(U("CALL"));
obj[U("strike")] = value::string(U("205.0"));
obj[U("expiry")] = value::string(U("2019-11-01"));  // yyyy-MM-dd

value result = quote_client->post(OPTION_INDICATOR, obj);
ucout << result << endl;

// 返回字段 / Response fields:
// delta/gamma/theta/vega/rho  - 希腊字母
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

## 期权合约 / Option Contract

```cpp
// 构造期权合约参数用于下单
// Build option contract parameters for order placement

// 期权代码格式 / Option identifier format:
// 美股 US: "AAPL  250829C00150000"
//   格式: 标的(padding至6位) + YYMMDD + C/P + 行权价*1000(8位)
// 港股 HK: "TCH.HK 230616C00550000"
//   使用映射后的代码 / Use mapped symbol from OPTION_SYMBOL
```

---

## 单腿期权下单 / Single-leg Option Order

```cpp
// 买入看涨期权 / Buy call option
value obj = value::object(true);
obj[U("account")] = value::string(U("123456"));
obj[U("contract")] = value::object(true);
obj[U("contract")][U("symbol")] = value::string(U("AAPL"));
obj[U("contract")][U("sec_type")] = value::string(U("OPT"));
obj[U("contract")][U("expiry")] = value::string(U("20250829"));   // YYYYMMDD
obj[U("contract")][U("strike")] = value::number(150.0);
obj[U("contract")][U("right")] = value::string(U("CALL"));
obj[U("contract")][U("currency")] = value::string(U("USD"));
obj[U("action")] = value::string(U("BUY"));
obj[U("order_type")] = value::string(U("LMT"));
obj[U("limit_price")] = value::number(5.0);
obj[U("quantity")] = value::number(1);  // 1张 = 100股

value result = trade_client->post(PLACE_ORDER, obj);
ucout << result << endl;
```

---

## 多腿组合策略 / Multi-leg Combo Strategies

```cpp
// 牛市看涨价差 / Bull Call Spread (VERTICAL)
value obj = value::object(true);
obj[U("account")] = value::string(U("123456"));
obj[U("combo_type")] = value::string(U("VERTICAL"));
obj[U("action")] = value::string(U("BUY"));
obj[U("order_type")] = value::string(U("LMT"));
obj[U("limit_price")] = value::number(3.0);
obj[U("quantity")] = value::number(1);

value legs = value::array();

value leg1 = value::object(true);
leg1[U("symbol")] = value::string(U("AAPL"));
leg1[U("sec_type")] = value::string(U("OPT"));
leg1[U("expiry")] = value::string(U("20250829"));
leg1[U("strike")] = value::number(145.0);
leg1[U("right")] = value::string(U("CALL"));
leg1[U("action")] = value::string(U("BUY"));
leg1[U("ratio")] = value::number(1);
legs[0] = leg1;

value leg2 = value::object(true);
leg2[U("symbol")] = value::string(U("AAPL"));
leg2[U("sec_type")] = value::string(U("OPT"));
leg2[U("expiry")] = value::string(U("20250829"));
leg2[U("strike")] = value::number(155.0);
leg2[U("right")] = value::string(U("CALL"));
leg2[U("action")] = value::string(U("SELL"));
leg2[U("ratio")] = value::number(1);
legs[1] = leg2;

obj[U("legs")] = legs;
value result = trade_client->post(PLACE_COMBO_ORDER, obj);
ucout << result << endl;
```

### 组合策略类型 / Combo Strategy Types

| ComboType | 策略 Strategy | 说明 Description |
|-----------|--------------|-----------------|
| `VERTICAL` | 垂直价差 | 同到期日不同行权价 Same expiry, different strikes |
| `STRADDLE` | 跨式 | 同行权价同到期日 Call+Put, same strike & expiry |
| `STRANGLE` | 宽跨式 | 不同行权价同到期日 Call+Put, different strikes |
| `CALENDAR` | 日历价差 | 同行权价不同到期日 Same strike, different expiries |
| `DIAGONAL` | 对角线价差 | 不同行权价不同到期日 Different strikes & expiries |
| `COVERED` | 备兑 | 持有股票+卖Call Long stock + short call |
| `PROTECTIVE` | 保护性 | 持有股票+买Put Long stock + long put |
| `SYNTHETIC` | 合成 | 合成多/空头 Synthetic long/short |
| `CUSTOM` | 自定义 | 4条腿组合（Iron Condor等） 4-leg combos |

---

## 查询期权持仓 / Query Option Positions

```cpp
value obj = value::object(true);
obj[U("account")] = value::string(U("123456"));
obj[U("sec_type")] = value::string(U("OPT"));

value result = trade_client->post(POSITIONS, obj);
ucout << result << endl;

// 期权持仓额外字段 / Option-specific position fields:
// strike  - 行权价
// expiry  - 到期日
// right   - CALL/PUT
```

---

## 注意事项 / Notes

- 港股期权需先用 `OPTION_SYMBOL` 获取代码映射 / HK options require symbol mapping
- 期权每张合约通常代表100股标的 / Each contract = 100 shares
- 期权链返回的 Greeks 为上一交易日收盘值 / Chain Greeks are from previous close
- 行权价小数位须和期权链一致 / Strike decimals must match the option chain
- 期权行情需要期权行情权限 / Option quotes require option quote permission
- 机构用户可在 obj 中额外传 `sub_account` / `secret_key` 字段
