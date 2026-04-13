# Tiger OpenAPI Rust SDK — Trading / 交易

> Rust SDK 交易 API 参考 / Trade API Reference (async)
<!-- 当用户提到"下单"、"买入"、"卖出"、"撤单"、"改单"、"持仓"、"资产"、"order"、"trade"时 -->

## 安全规范 / Safety Rules

> ⚠️ **默认使用模拟账户。Default to Paper Trading.**

实盘下单前，**必须执行**以下流程：
1. 与用户确认：标的代码、方向（买/卖）、数量、价格、账户
2. 先调用 `preview_order()` 查看预估佣金和保证金
3. 用户确认后再调用 `place_order()`
4. 下单后通过 `orders()` 确认订单状态

---

## 初始化 / Initialize

```rust
use tigeropen::config::ClientConfig;
use tigeropen::client::HttpClient;
use tigeropen::trade::TradeClient;

let config = ClientConfig::builder()
    .properties_file("tiger_openapi_config.properties")
    .build()?;
let http_client = HttpClient::new(config.clone())?;
let tc = TradeClient::new(&http_client, &config.account);
```

---

## 下单 / Place Orders
<!-- 当用户提到"下单"、"买入"、"卖出"、"buy"、"sell"、"order"时 -->

### 构造订单 / Build Order

```rust
use serde_json::json;

// 限价单 / Limit order
let order = json!({
    "symbol": "AAPL",
    "secType": "STK",
    "action": "BUY",
    "orderType": "LMT",
    "totalQuantity": 100,
    "limitPrice": 150.0,
});

// 市价单 / Market order
let order = json!({
    "symbol": "AAPL",
    "secType": "STK",
    "action": "BUY",
    "orderType": "MKT",
    "totalQuantity": 100,
});

// 止损限价单 / Stop-limit order
let order = json!({
    "symbol": "AAPL",
    "secType": "STK",
    "action": "SELL",
    "orderType": "STP_LMT",
    "totalQuantity": 100,
    "limitPrice": 145.0,   // 限价
    "auxPrice": 148.0,     // 触发价
});

// 盘后交易 / After-hours trading
let order = json!({
    "symbol": "AAPL",
    "secType": "STK",
    "action": "BUY",
    "orderType": "LMT",
    "totalQuantity": 100,
    "limitPrice": 150.0,
    "outsideRth": true,    // 允许盘前盘后
    "timeInForce": "GTC",
});
```

### 预览订单 / Preview Order

```rust
let result = tc.preview_order(order.clone()).await?;
// 返回预估保证金和佣金:
// {"initMargin": 15000, "maintMargin": 12000, "minCommission": 1.0, "maxCommission": 1.5, ...}
```

### 提交下单 / Submit Order

```rust
let result = tc.place_order(order).await?;
// 返回: {"id": 12345678, "orderId": "...", "status": "PendingSubmit"}
```

### 修改订单 / Modify Order

```rust
let mut modified = order.clone();
modified["limitPrice"] = json!(155.0);
let result = tc.modify_order(order_id, modified).await?;  // order_id: i64
```

### 取消订单 / Cancel Order

```rust
let result = tc.cancel_order(order_id).await?;  // order_id: i64
```

---

## 查询订单 / Query Orders
<!-- 当用户提到"订单"、"委托"、"orders"时 -->

```rust
// 所有订单 / All orders
let result = tc.orders().await?;

// 待成交订单 / Active (pending) orders
let result = tc.active_orders().await?;

// 已成交订单 / Filled orders
let result = tc.filled_orders().await?;

// 已撤销订单 / Inactive (cancelled) orders
let result = tc.inactive_orders().await?;

// 订单成交明细 / Order transactions
let result = tc.order_transactions(order_id).await?;  // order_id: i64
```

---

## 持仓查询 / Query Positions
<!-- 当用户提到"持仓"、"仓位"、"positions"时 -->

```rust
let result = tc.positions().await?;
// 返回持仓列表，每个元素包含:
// symbol, secType, quantity, averageCost, marketPrice, marketValue, unrealizedPnl
```

---

## 资产查询 / Query Assets
<!-- 当用户提到"资产"、"资金"、"余额"、"assets"时 -->

```rust
// 普通账户资产 / Standard account assets
let result = tc.assets().await?;

// 综合账户（Prime）资产 / Prime account assets
let result = tc.prime_assets().await?;
// 返回: netLiquidation, cashBalance, buyingPower, unrealizedPl, initMargin, maintMargin 等
```

---

## 合约查询 / Contract Query

```rust
// 单个合约 / Single contract
let result = tc.contract("AAPL", "STK").await?;

// 批量合约 / Batch contracts
let result = tc.contracts(&["AAPL", "TSLA"], "STK").await?;

// secType 值: "STK"(股票)/"OPT"(期权)/"FUT"(期货)/"CASH"(外汇)
```

---

## 订单字段说明 / Order Fields

| 字段 | 类型 | 说明 |
|-----|------|------|
| `symbol` | String | 标的代码，如 `"AAPL"` |
| `secType` | String | `"STK"` / `"OPT"` / `"FUT"` |
| `action` | String | `"BUY"` / `"SELL"` |
| `orderType` | String | `"LMT"` / `"MKT"` / `"STP"` / `"STP_LMT"` |
| `totalQuantity` | f64 | 数量 |
| `limitPrice` | f64 | 限价（LMT/STP_LMT 必填） |
| `auxPrice` | f64 | 止损价（STP/STP_LMT 必填） |
| `timeInForce` | String | `"DAY"` / `"GTC"` / `"GTD"` |
| `outsideRth` | bool | 是否允许盘前盘后交易 |

---

## 解析返回值 / Parse Response

```rust
use serde_json::Value;

if let Some(orders) = tc.orders().await? {
    if let Value::Array(list) = &orders {
        for order in list {
            let symbol = order["symbol"].as_str().unwrap_or("");
            let status = order["status"].as_str().unwrap_or("");
            println!("{}: {}", symbol, status);
        }
    }
}
```

---

## 注意事项 / Notes

- `place_order()` 返回成功仅表示订单**已提交**，不代表已成交
- 需通过 `orders()` 或推送回调确认最终成交状态
- 期权下单需先通过 `quote.option_chain()` 获取 identifier，构造合约时设置 `expiry`/`strike`/`right`
- 所有方法为 `async`，必须 `.await`
