# Tiger OpenAPI TypeScript SDK — Market Data / 行情查询

> TypeScript SDK 行情 API 参考 / Quote API Reference
<!-- 当用户提到"行情"、"报价"、"K线"、"价格"、"深度"、"quote"、"kline"、"price"时 -->

## 初始化 / Initialize

```typescript
import { createClientConfig } from 'tigeropen';
import { HttpClient } from 'tigeropen/client/http-client';
import { QuoteClient } from 'tigeropen/quote/quote-client';

const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});
const httpClient = new HttpClient(config);
const qc = new QuoteClient(httpClient);
```

---

## 市场状态 / Market State

```typescript
// market: 'US' / 'HK' / 'CN' / 'SG'
const data = await qc.getMarketState('US');
// 返回 [{market, status, openTime, closeTime, ...}]
```

---

## 实时报价 / Real-time Quotes
<!-- 当用户提到"实时报价"、"最新价"、"real-time"时 -->

```typescript
const data = await qc.getBrief(['AAPL', 'TSLA']);
// 每个元素包含: symbol, latestPrice, askPrice, bidPrice, volume, change, changeRatio 等
```

---

## K 线 / Kline
<!-- 当用户提到"K线"、"kline"、"bar"、"日线"时 -->

```typescript
// period: 'day'/'week'/'month'/'year'/'1min'/'3min'/'5min'/'10min'/'15min'/'30min'/'60min'
const data = await qc.getKline('AAPL', 'day');
// 返回数组，每个元素: time, open, high, low, close, volume
```

---

## 分时 / Timeline

```typescript
const data = await qc.getTimeline(['AAPL', 'TSLA']);
// 返回分时数据: time, price, volume, avgPrice
```

---

## 深度行情 / Quote Depth
<!-- 当用户提到"买卖盘"、"深度"、"depth"时 -->

```typescript
const data = await qc.getQuoteDepth('AAPL');
// 返回 {asks: [{price, volume},...], bids: [{price, volume},...]}
```

---

## 逐笔成交 / Trade Ticks

```typescript
const data = await qc.getTradeTick(['AAPL']);
// 返回逐笔: time, price, volume, direction
```

---

## 期权到期日 / Option Expirations

```typescript
const data = await qc.getOptionExpiration('AAPL');
// 返回到期日列表: dates, timestamps, periodTag ("m"=月度/"w"=周度)
```

---

## 期权链 / Option Chain

```typescript
const data = await qc.getOptionChain('AAPL', '2025-08-29');
// 返回 call/put 合约: identifier, strike, bid/ask, volume, openInterest, impliedVol, delta, gamma 等
```

---

## 期权报价 / Option Brief

```typescript
const data = await qc.getOptionBrief(['AAPL  250829C00150000']);
// 期权代码格式: 标的(6位右空格填充) + YYMMDD + C/P + 行权价*1000(8位)
// 返回: identifier, strike, putCall, latestPrice, bid/ask, impliedVol, delta, gamma, theta, vega 等
```

---

## 期货 / Futures

```typescript
// 获取交易所列表
const exchanges = await qc.getFutureExchange();

// 获取期货合约
const contracts = await qc.getFutureContracts('CME');

// 期货实时报价
const data = await qc.getFutureRealTimeQuote(['CL2509']);
```

---

## 直接调用 API / Raw API Call

当以上方法不满足需求时：

```typescript
import { HttpClient } from 'tigeropen/client/http-client';

const resp = await httpClient.execute(
  'quote_real_time',
  JSON.stringify({ symbols: ['AAPL', 'TSLA'] })
);
```

---

## 注意事项 / Notes

- 所有方法返回 `Promise`，需 `await`
- 行情数据需要对应市场的行情权限
- 期权行情需要单独开通期权行情权限
- 港股期权代码格式：`TCH.HK 230616C00550000`（与美股不同）
