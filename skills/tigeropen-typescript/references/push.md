# Tiger OpenAPI TypeScript SDK — Real-time Push / 实时推送

> TypeScript SDK 实时推送 API 参考 / Push API Reference
<!-- 当用户提到"推送"、"实时"、"订阅"、"WebSocket"、"push"、"subscribe"时 -->

## 初始化 / Initialize

```typescript
import { createClientConfig } from 'tigeropen';
import { PushClient } from 'tigeropen/dist/esm/push/push-client.js';

const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});
const pc = new PushClient(config);
```

---

## 完整示例 / Full Example

```typescript
import { createClientConfig } from 'tigeropen';
import { PushClient } from 'tigeropen/dist/esm/push/push-client.js';

const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});

const pc = new PushClient(config);

// 设置回调 / Set callbacks
// ⚠️ 订阅必须在 onConnect 回调中执行 / Subscriptions MUST be placed inside onConnect callback
pc.setCallbacks({
  onQuote: (data) => {
    console.log(`[行情] ${data.symbol} 最新价: ${data.latestPrice} 量: ${data.volume}`);
  },
  onOrder: (data) => {
    console.log(`[订单] ${data.symbol} 状态: ${data.status}`);
  },
  onAsset: (data) => {
    console.log(`[资产] 账户: ${data.account} 净资产: ${data.netLiquidation}`);
  },
  onPosition: (data) => {
    console.log(`[持仓] ${data.symbol} 数量: ${data.quantity}`);
  },
  onConnect: () => {
    console.log('推送连接成功');
    // ⚠️ 在此处执行订阅 / Subscribe here after connection established
    // 重连后订阅不会自动恢复，需在此重新订阅 / Subscriptions NOT auto-restored on reconnect
    pc.subscribeQuote(['AAPL', 'TSLA']);
    pc.subscribeAsset();
    pc.subscribeOrder();
    pc.subscribePosition();
  },
  onDisconnect: () => console.log('推送连接断开'),
  onError: (err) => console.error('推送错误:', err),
});

// 连接 / Connect
await pc.connect();

// 等待退出 / Keep running
process.on('SIGINT', () => {
  pc.unsubscribeQuote(['AAPL', 'TSLA']);
  pc.disconnect();
  process.exit(0);
});
```

---

## 回调函数 / Callbacks

| 回调 Callback | 触发时机 | 数据类型 |
|--------------|---------|---------|
| `onQuote` | 行情更新时 | `QuoteData` |
| `onOrder` | 订单状态变化时 | `OrderData` |
| `onAsset` | 账户资产变动时 | `AssetData` |
| `onPosition` | 持仓变动时 | `PositionData` |
| `onConnect` | WebSocket 连接成功时 | - |
| `onDisconnect` | 连接断开时 | - |
| `onError` | 发生错误时 | `Error` |

---

## 订阅方法 / Subscribe Methods

```typescript
// 行情订阅 / Quote subscription
pc.subscribeQuote(['AAPL', 'TSLA', '00700']);
pc.unsubscribeQuote(['TSLA']);

// 账户推送 / Account push
pc.subscribeAsset();       // 资产变动
pc.subscribeOrder();       // 订单状态
pc.subscribePosition();    // 持仓变动
pc.unsubscribeAsset();
pc.unsubscribeOrder();
pc.unsubscribePosition();

// 断开连接 / Disconnect
pc.disconnect();
```

---

## QuoteData 字段 / QuoteData Fields

| 字段 Field | 说明 |
|-----------|------|
| `symbol` | 标的代码 |
| `latestPrice` | 最新价 |
| `volume` | 成交量 |
| `askPrice` | 卖一价 |
| `bidPrice` | 买一价 |
| `change` | 涨跌额 |
| `changeRatio` | 涨跌幅 |
| `timestamp` | 时间戳（毫秒） |

---

## AssetData 字段 / AssetData Fields

| 字段 Field | 说明 |
|-----------|------|
| `account` | 账户号 |
| `netLiquidation` | 净资产 |
| `cashBalance` | 现金余额 |
| `buyingPower` | 购买力 |
| `currency` | 计价货币 |

---

## PositionData 字段 / PositionData Fields

| 字段 Field | 说明 |
|-----------|------|
| `account` | 账户号 |
| `symbol` | 标的代码 |
| `quantity` | 持仓数量 |
| `averageCost` | 平均成本 |
| `marketPrice` | 最新价格 |
| `marketValue` | 市值 |
| `unrealizedPnl` | 浮动盈亏 |

---

## 注意事项 / Notes

- **在 `onConnect` 回调中执行订阅**，连接成功后才能订阅
- SDK 自动处理断线重连，无需手动重连
- ⚠️ **断线重连后订阅不会自动恢复**，必须在 `onConnect` 回调中重新订阅 / Subscriptions NOT auto-restored after reconnect; re-subscribe in `onConnect`
- 心跳保活由 SDK 自动维护
- 同一个 `PushClient` 实例只能有一个活跃连接
