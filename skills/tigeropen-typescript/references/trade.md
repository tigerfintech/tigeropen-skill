# Tiger OpenAPI TypeScript SDK — Trading / 交易

> TypeScript SDK 交易 API 参考 / Trade API Reference
<!-- 当用户提到"下单"、"买入"、"卖出"、"撤单"、"改单"、"持仓"、"资产"、"order"、"trade"时 -->

## 安全规范 / Safety Rules

> ⚠️ **默认使用模拟账户。Default to Paper Trading.**

实盘下单前，**必须执行**以下流程：
1. 与用户确认：标的代码、方向（买/卖）、数量、价格、账户
2. 先调用 `previewOrder()` 查看预估佣金和保证金
3. 用户确认后再调用 `placeOrder()`
4. 下单后通过 `getOrders()` 确认订单状态

---

## 初始化 / Initialize

```typescript
import { createClientConfig } from 'tigeropen';
import { HttpClient } from 'tigeropen/client/http-client';
import { TradeClient } from 'tigeropen/trade/trade-client';

const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});
const httpClient = new HttpClient(config);
const tc = new TradeClient(httpClient, config.account);
```

---

## 下单 / Place Orders
<!-- 当用户提到"下单"、"买入"、"卖出"、"buy"、"sell"、"order"时 -->

### 构造订单 / Build Order

```typescript
// 限价单 / Limit order
const order = {
  symbol: 'AAPL',
  secType: 'STK',
  action: 'BUY',
  orderType: 'LMT',
  totalQuantity: 100,
  limitPrice: 150.0,
  timeInForce: 'DAY',
};

// 市价单 / Market order
const order = {
  symbol: 'AAPL',
  secType: 'STK',
  action: 'BUY',
  orderType: 'MKT',
  totalQuantity: 100,
};

// 止损限价单 / Stop-limit order
const order = {
  symbol: 'AAPL',
  secType: 'STK',
  action: 'SELL',
  orderType: 'STP_LMT',
  totalQuantity: 100,
  limitPrice: 145.0,   // 限价
  auxPrice: 148.0,     // 触发价
};

// 盘后交易 / After-hours trading
const order = {
  symbol: 'AAPL',
  secType: 'STK',
  action: 'BUY',
  orderType: 'LMT',
  totalQuantity: 100,
  limitPrice: 150.0,
  outsideRth: true,   // 允许盘前盘后
  timeInForce: 'GTC',
};
```

### 预览订单 / Preview Order

```typescript
const preview = await tc.previewOrder(order);
// 返回预估保证金和佣金信息
```

### 提交下单 / Submit Order

```typescript
const result = await tc.placeOrder(order);
// 返回 {id, orderId, status: 'PendingSubmit', ...}
```

### 修改订单 / Modify Order

```typescript
const modified = { ...order, limitPrice: 155.0 };
await tc.modifyOrder(orderId, modified);  // orderId: number
```

### 取消订单 / Cancel Order

```typescript
await tc.cancelOrder(orderId);  // orderId: number
```

---

## 查询订单 / Query Orders
<!-- 当用户提到"订单"、"委托"、"orders"时 -->

```typescript
// 所有订单 / All orders
const orders = await tc.getOrders();

// 待成交订单 / Active (pending) orders
const active = await tc.getActiveOrders();

// 已成交订单 / Filled orders
const filled = await tc.getFilledOrders();

// 已撤销订单 / Inactive (cancelled) orders
const inactive = await tc.getInactiveOrders();

// 订单成交明细 / Order transactions
const transactions = await tc.getOrderTransactions(orderId);
```

---

## 持仓查询 / Query Positions
<!-- 当用户提到"持仓"、"仓位"、"positions"时 -->

```typescript
const positions = await tc.positions();
// ⚠️ 返回 JSON 字符串，需手动解析 / Returns JSON string, parse manually:
// const items = JSON.parse(positions).items;
// 每个元素包含 / each item has:
// symbol, secType, positionQty, averageCost, marketPrice/latestPrice, marketValue, unrealizedPnl
```

---

## 资产查询 / Query Assets
<!-- 当用户提到"资产"、"资金"、"余额"、"assets"时 -->

```typescript
// 普通账户资产 / Standard account assets
const assets = await tc.assets();
// ⚠️ 返回 JSON 字符串，需手动解析 / Returns JSON string, parse manually

// 综合账户（Prime）资产 / Prime account assets
const prime = await tc.primeAssets();
// 返回: netLiquidation, cashBalance, buyingPower, unrealizedPl, initMargin, maintMargin 等
```

---

## 合约查询 / Contract Query

```typescript
// 单个合约
const contract = await tc.contract('AAPL', 'STK');

// 批量合约
const contracts = await tc.contracts(['AAPL', 'TSLA'], 'STK');

// secType: 'STK'(股票)/'OPT'(期权)/'FUT'(期货)/'CASH'(外汇)
```

---

## 订单字段说明 / Order Fields

| 字段 | 类型 | 说明 |
|-----|------|------|
| `symbol` | string | 标的代码，如 `'AAPL'` |
| `secType` | string | `'STK'` / `'OPT'` / `'FUT'` |
| `action` | string | `'BUY'` / `'SELL'` |
| `orderType` | string | `'LMT'` / `'MKT'` / `'STP'` / `'STP_LMT'` |
| `totalQuantity` | number | 数量 |
| `limitPrice` | number | 限价（LMT/STP_LMT 必填） |
| `auxPrice` | number | 止损价（STP/STP_LMT 必填） |
| `timeInForce` | string | `'DAY'` / `'GTC'` / `'GTD'` |
| `outsideRth` | boolean | 是否允许盘前盘后交易 |

---

## 注意事项 / Notes

- `placeOrder()` 返回成功仅表示订单**已提交**，不代表已成交
- 需通过 `getOrders()` 或推送回调确认最终成交状态
- 期权下单需先通过 `qc.getOptionChain()` 获取 identifier，并设置 `expiry`/`strike`/`right` 字段
