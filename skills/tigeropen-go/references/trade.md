# Tiger OpenAPI Go SDK — Trading / 交易

> Go SDK 交易 API 参考 / Trade API Reference
<!-- 当用户提到"下单"、"买入"、"卖出"、"撤单"、"改单"、"持仓"、"资产"、"order"、"trade"时 -->

## 安全规范 / Safety Rules

> ⚠️ **默认使用模拟账户。Default to Paper Trading.**

实盘下单前，**必须执行**以下流程：
1. 与用户确认：标的代码、方向（买/卖）、数量、价格、账户
2. 先调用 `PreviewOrder()` 查看预估佣金和保证金
3. 用户确认后再调用 `PlaceOrder()`
4. 下单后通过 `Orders()` 确认订单状态

---

## 初始化 / Initialize

```go
import (
    "github.com/tigerfintech/openapi-sdks/go/client"
    "github.com/tigerfintech/openapi-sdks/go/config"
    "github.com/tigerfintech/openapi-sdks/go/model"
    "github.com/tigerfintech/openapi-sdks/go/trade"
)

cfg, _ := config.NewClientConfig(
    config.WithPropertiesFile("tiger_openapi_config.properties"),
)
httpClient := client.NewHttpClient(cfg)
tc := trade.NewTradeClient(httpClient, cfg.Account)
```

---

## 下单 / Place Orders
<!-- 当用户提到"下单"、"买入"、"卖出"、"buy"、"sell"、"order"时 -->

### 创建订单 / Create Order

```go
import "github.com/tigerfintech/openapi-sdks/go/model"

// 限价单 / Limit order
order := model.LimitOrder("AAPL", "BUY", 100, 150.0)

// 市价单 / Market order
order := model.MarketOrder("AAPL", "BUY", 100)

// 止损限价单 / Stop-limit order
order := model.StopLimitOrder("AAPL", "SELL", 100, 145.0, 148.0)
// stopPrice=148.0, limitPrice=145.0
```

### 提交下单 / Submit Order

```go
result, err := tc.PlaceOrder(order)
if err != nil {
    log.Fatal(err)
}
// result: {"id": 12345678, "orderId": "...", "status": "PendingSubmit"}
```

### 预览订单 / Preview Order

```go
result, err := tc.PreviewOrder(order)
// 返回预估保证金和佣金信息
// {"initMargin": 15000, "maintMargin": 12000, "minCommission": 1.0, "maxCommission": 1.5, ...}
```

### 修改订单 / Modify Order

```go
order.LimitPrice = 155.0
result, err := tc.ModifyOrder(orderID, order)
```

### 取消订单 / Cancel Order

```go
result, err := tc.CancelOrder(orderID)  // orderID: int64
```

---

## 查询订单 / Query Orders
<!-- 当用户提到"订单"、"委托"、"orders"时 -->

```go
// 所有订单 / All orders
result, err := tc.Orders()

// 待成交订单 / Active (pending) orders
result, err := tc.ActiveOrders()

// 已成交订单 / Filled orders
result, err := tc.FilledOrders()

// 已撤销订单 / Inactive (cancelled) orders
result, err := tc.InactiveOrders()

// 订单成交明细 / Order transactions
result, err := tc.OrderTransactions(orderID)
```

---

## 持仓查询 / Query Positions
<!-- 当用户提到"持仓"、"仓位"、"positions"时 -->

```go
result, err := tc.Positions()
// 返回持仓列表，每个元素包含:
// symbol, secType, quantity, averageCost, marketPrice, marketValue, unrealizedPnl
```

---

## 资产查询 / Query Assets
<!-- 当用户提到"资产"、"资金"、"余额"、"assets"时 -->

```go
// 普通账户资产 / Standard account assets
result, err := tc.Assets()

// 综合账户（Prime）资产 / Prime account assets
result, err := tc.PrimeAssets()
// 返回: netLiquidation, cashBalance, buyingPower, unrealizedPl, initMargin, maintMargin 等
```

---

## 合约查询 / Contract Query

```go
// 单个合约 / Single contract
result, err := tc.Contract("AAPL", "STK")

// 批量合约 / Batch contracts
result, err := tc.Contracts([]string{"AAPL", "TSLA"}, "STK")

// secType 值: "STK"(股票)/"OPT"(期权)/"FUT"(期货)/"CASH"(外汇)
```

---

## model.Order 字段说明 / Order Fields

| 字段 | 类型 | 说明 |
|-----|------|------|
| `Symbol` | string | 标的代码，如 `"AAPL"` |
| `SecType` | string | `"STK"` / `"OPT"` / `"FUT"` |
| `Action` | string | `"BUY"` / `"SELL"` |
| `OrderType` | string | `"LMT"` / `"MKT"` / `"STP"` / `"STP_LMT"` |
| `TotalQuantity` | float64 | 数量 |
| `LimitPrice` | float64 | 限价（LMT/STP_LMT 必填） |
| `AuxPrice` | float64 | 止损价（STP/STP_LMT 必填） |
| `TimeInForce` | string | `"DAY"` / `"GTC"` / `"GTD"` |
| `OutsideRth` | bool | 是否允许盘前盘后交易 |
| `Account` | string | 账户（自动填充） |
| `ID` | int64 | 订单 ID（修改/取消时使用） |

---

## 注意事项 / Notes

- `PlaceOrder()` 返回成功仅表示订单**已提交**，不代表已成交
- 需通过 `Orders()` 或推送回调确认最终成交状态
- 期权下单需先通过 `quote.OptionChain` 获取 identifier，构造合约时设置 `Expiry`/`Strike`/`Right`
- 所有返回值为 `json.RawMessage`，需 `json.Unmarshal` 解析
