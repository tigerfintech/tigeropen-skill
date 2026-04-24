
# Tiger OpenAPI TypeScript SDK — 期权 / Options Trading

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/quote-option

## 期权操作工作流 / Option Workflow

当用户提到期权时，按以下流程操作 / When user mentions options, follow this workflow:

### 查询期权 / Query Options

1. **查到期日 Get expirations**: `qc.optionExpirations()` → 获取可选到期日列表
2. **查期权链 Get chain**: `qc.optionChain()` → 获取指定到期日的所有合约
3. **查行情 Get quotes**: `qc.optionBriefs()` → 获取期权实时行情和 Greeks

### 港股期权特殊处理 / HK Option Special Handling

- 港股期权标的代码不同于正股 / HK option underlyings differ from stock codes: `00700` → `TCH`（腾讯）
- 使用 `qc.optionSymbols()` 查询港股期权代码映射

---

## 初始化 / Initialize

```typescript
import { createClientConfig } from 'tigeropen';
import { HttpClient } from 'tigeropen/dist/esm/client/http-client.js';
import { QuoteClient } from 'tigeropen/dist/esm/quote/quote-client.js';
import { TradeClient } from 'tigeropen/dist/esm/trade/trade-client.js';

const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});

const httpClient = new HttpClient(config);
const qc = new QuoteClient(httpClient);
const tc = new TradeClient(httpClient, config.account);
```

---

## 期权到期日 / Option Expirations

```typescript
const result = await qc.optionExpirations({
  symbols: ['AAPL'],
  market: 'US',   // 'US' | 'HK'
});
console.log(result);

// 返回字段 / Response fields:
// symbol     - 股票代码
// count      - 到期日数量
// dates      - 到期日数组 (e.g. "2024-06-28")
// timestamps - 到期日时间戳数组（毫秒，纽约时间）
// periodTags - 周期标签: "m"=月期权, "w"=周期权, "q"=季度
```

---

## 期权链 / Option Chain

```typescript
const result = await qc.optionChain({
  symbol: 'AAPL',
  expiry: '2025-08-29',
  market: 'US',
  returnGreekValue: true,   // 返回希腊字母 / Return Greeks
  // 可选筛选 / Optional filters:
  // inTheMoney: true,
  // impliedVolatilityMin: 0.15,
  // impliedVolatilityMax: 0.80,
  // deltaMin: 0.2,
  // deltaMax: 0.8,
  // openInterestMin: 100,
});
console.log(result);

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
// delta/gamma/theta/vega/rho - 希腊字母（需 returnGreekValue=true）
```

---

## 港股期权代码映射 / HK Option Symbol Mapping

```typescript
const result = await qc.optionSymbols({
  market: 'HK',
  lang: 'en_US',   // 'en_US' | 'zh_CN' | 'zh_TW'
});

if (Array.isArray(result)) {
  // 查找腾讯 / Find Tencent (00700 -> TCH.HK)
  const tencent = result.find(s => s.underlyingSymbol === '00700');
  console.log(`Option symbol: ${tencent?.symbol}`);
}

// 返回字段 / Response fields:
// symbol           - 期权 symbol (e.g. "TCH.HK")
// name             - 标的名称
// underlyingSymbol - 正股代码 (e.g. "00700")
```

---

## 期权实时行情 / Option Brief (Real-time Quotes)

```typescript
const result = await qc.optionBriefs({
  market: 'US',
  optionBasic: [
    {
      symbol: 'AAPL',
      right: 'CALL',
      expiry: '2025-08-29',   // yyyy-MM-dd
      strike: '150.0',         // 小数位须和期权链一致
    },
  ],
});
console.log(result);

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

```typescript
const result = await qc.optionDepth({
  market: 'US',
  optionBasic: [
    {
      symbol: 'AAPL',
      right: 'PUT',
      expiry: '2024-06-28',
      strike: '210.0',
    },
  ],
});
console.log(result);

// 返回字段 / Response fields:
// ask[]/bid[] - 卖/买盘挂单列表
//   price    - 委托价
//   volume   - 委托量
//   code     - 交易所代码 (CBOE, PHLX 等)
//   timestamp - 交易所时间
```

---

## 期权逐笔成交 / Option Trade Ticks

```typescript
// 仅支持美股期权 / US market only

const result = await qc.optionTradeTicks({
  optionBasic: [
    {
      symbol: 'AAPL',
      right: 'PUT',
      expiry: '2024-03-08',
      strike: '185.0',
    },
  ],
});
console.log(result);

// 返回字段 / Response fields:
// items[] - 逐笔成交列表
//   price  - 成交价
//   volume - 成交量
//   time   - 成交时间（毫秒）
```

---

## 期权K线 / Option K-line

```typescript
const result = await qc.optionKline({
  market: 'US',
  optionQuery: [
    {
      symbol: 'AAPL',
      right: 'CALL',
      expiry: '2024-06-28',
      strike: '170.0',
      beginTime: '2024-06-26',
      endTime: '2024-06-26 23:59:59',
      period: '1min',   // 'day' | '1min' | '5min' | '30min' | '60min'
      limit: 10,
      sortDir: 'DESC',
    },
  ],
});
console.log(result);

// 返回字段 / Response fields:
// items[] - K线数据点列表
//   open/high/low/close - 开高低收
//   volume              - 成交量
//   time                - 时间戳（毫秒）
//   openInterest        - 持仓量（仅日K线）
```

---

## 期权分时数据 / Option Timeline

```typescript
// 目前仅支持港股期权 / HK market only currently

const result = await qc.optionTimeline({
  market: 'HK',
  optionList: [
    {
      symbol: 'ALB.HK',
      right: 'CALL',
      expiry: 1753878054000,   // 毫秒时间戳
      strike: '117.50',
    },
  ],
});
console.log(result);

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

```typescript
const result = await qc.optionAnalysis({
  symbols: [
    {
      symbol: 'AAPL',
      period: '52week',   // '3year' | '52week' | '26week' | '13week'
      requireVolatilityList: true,   // 返回IV/HV历史时序
    },
  ],
  market: 'US',
});
console.log(result);

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

```typescript
// 查询单个期权的盘中实时指标（Greeks、IV、杠杆等）
const result = await qc.optionIndicator({
  symbol: 'BABA',
  right: 'CALL',
  strike: '205.0',
  expiry: '2019-11-01',   // yyyy-MM-dd
});
console.log(`delta: ${result?.delta}, iv: ${result?.volatility}%`);

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

```typescript
// 买入看涨期权 / Buy call option
const result = await tc.placeOrder({
  account: config.account,
  contract: {
    symbol: 'AAPL',
    secType: 'OPT',
    expiry: '20250829',   // YYYYMMDD
    strike: 150.0,
    right: 'CALL',
    currency: 'USD',
  },
  action: 'BUY',
  orderType: 'LMT',
  limitPrice: 5.0,
  quantity: 1,             // 1张 = 100股
});
console.log(`order id: ${result?.id}, status: ${result?.status}`);
```

---

## 多腿组合策略 / Multi-leg Combo Strategies

```typescript
// 牛市看涨价差 / Bull Call Spread (VERTICAL)
const result = await tc.placeComboOrder({
  account: config.account,
  comboType: 'VERTICAL',
  action: 'BUY',
  orderType: 'LMT',
  limitPrice: 3.0,
  quantity: 1,
  legs: [
    {
      symbol: 'AAPL',
      secType: 'OPT',
      expiry: '20250829',
      strike: 145.0,
      right: 'CALL',
      action: 'BUY',
      ratio: 1,
    },
    {
      symbol: 'AAPL',
      secType: 'OPT',
      expiry: '20250829',
      strike: 155.0,
      right: 'CALL',
      action: 'SELL',
      ratio: 1,
    },
  ],
});
```

### 其他策略示例 / Other Strategy Examples

```typescript
// 跨式策略 / Straddle — comboType: 'STRADDLE'，同行权价 Call+Put
// Iron Condor — comboType: 'CUSTOM'，4腿：Put spread + Call spread
// 备兑策略 / Covered Call — comboType: 'COVERED'
//   Leg1: secType='STK', action='BUY', ratio=100
//   Leg2: secType='OPT', action='SELL', ratio=1
```

### 组合策略类型总览 / Combo Strategy Types

| comboType | 策略 Strategy | 说明 Description |
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

```typescript
const result = await tc.positions({
  account: config.account,
  sec_type: 'OPT',
});

if (Array.isArray(result)) {
  result.forEach(pos => {
    console.log(`${pos.symbol} ${pos.expiry} ${pos.strike} ${pos.right}: `
      + `qty=${pos.positionQty}, pnl=${pos.unrealizedPnl}`);
  });
}

// 期权持仓额外字段 / Option-specific position fields:
// strike - 行权价
// expiry - 到期日
// right  - CALL/PUT
```

---

## 注意事项 / Notes

- 港股期权需先用 `optionSymbols()` 获取代码映射 / HK options require symbol mapping
- 期权每张合约通常代表100股标的 / Each contract = 100 shares
- 期权链返回的 Greeks 为上一交易日收盘值，盘中请用 `optionIndicator()` / Chain Greeks are from previous close
- 行权价小数位须和期权链一致 / Strike decimals must match the option chain
- 期权行情需要期权行情权限 / Option quotes require option quote permission
- 所有方法均为 async，需要 `await`
- 子路径导入需使用 `tigeropen/dist/esm/...` 路径（参见 quickstart.md）
