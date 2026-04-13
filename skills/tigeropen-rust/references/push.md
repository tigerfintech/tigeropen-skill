# Tiger OpenAPI Rust SDK — Real-time Push / 实时推送

> Rust SDK 实时推送 API 参考 / Push API Reference (async)
<!-- 当用户提到"推送"、"实时"、"订阅"、"WebSocket"、"push"、"subscribe"时 -->

## 初始化 / Initialize

```rust
use tigeropen::config::ClientConfig;
use tigeropen::push::PushClient;

let config = ClientConfig::builder()
    .properties_file("tiger_openapi_config.properties")
    .build()?;
let pc = PushClient::new(config, None);
```

---

## 完整示例 / Full Example

```rust
use tigeropen::config::ClientConfig;
use tigeropen::push::{PushClient, Callbacks, SubjectType};
use std::sync::Arc;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = ClientConfig::builder()
        .properties_file("tiger_openapi_config.properties")
        .build()?;

    let pc = PushClient::new(config, None);

    // 设置回调 / Set callbacks
    // ⚠️ 订阅必须在 on_connect 回调中执行 / Subscriptions MUST be placed inside on_connect callback
    let pc_clone = pc.clone();
    pc.set_callbacks(Callbacks {
        on_quote: Some(Arc::new(|data| {
            println!("[行情] {}: {:?} 量: {:?}",
                data.symbol, data.latest_price, data.volume);
        })),
        on_order: Some(Arc::new(|data| {
            println!("[订单] {} 状态: {:?}", data.symbol, data.status);
        })),
        on_asset: Some(Arc::new(|data| {
            println!("[资产] 账户: {} 净资产: {:?}", data.account, data.net_liquidation);
        })),
        on_position: Some(Arc::new(|data| {
            println!("[持仓] {} 数量: {}", data.symbol, data.quantity);
        })),
        on_connect: Some(Arc::new(move || {
            println!("推送连接成功");
            // ⚠️ 在此处执行订阅 / Subscribe here after connection established
            // 重连后订阅不会自动恢复，需在此重新订阅 / Subscriptions NOT auto-restored on reconnect
            pc_clone.add_subscription(SubjectType::Quote, &["AAPL".into(), "TSLA".into()]);
            pc_clone.add_account_sub(SubjectType::Asset);
            pc_clone.add_account_sub(SubjectType::Order);
            pc_clone.add_account_sub(SubjectType::Position);
        })),
        on_disconnect: Some(Arc::new(|| println!("推送连接断开"))),
        on_error: Some(Arc::new(|err| eprintln!("推送错误: {}", err))),
        ..Default::default()
    });

    // 连接 / Connect
    pc.connect().await?;

    // 等待退出 / Wait for Ctrl+C
    tokio::signal::ctrl_c().await?;

    // 退订并断开 / Unsubscribe and disconnect
    pc.remove_subscription(SubjectType::Quote, Some(&["AAPL".into(), "TSLA".into()]));
    pc.disconnect();

    Ok(())
}
```

---

## 回调函数 / Callbacks

| 回调字段 | 触发时机 | 数据类型 |
|---------|---------|---------|
| `on_quote` | 行情更新时 | `QuotePushData` |
| `on_tick` | 逐笔成交时 | `TickPushData` |
| `on_depth` | 深度行情更新时 | `DepthPushData` |
| `on_kline` | K线更新时 | `KlinePushData` |
| `on_option` | 期权行情更新时 | `QuotePushData` |
| `on_future` | 期货行情更新时 | `QuotePushData` |
| `on_asset` | 账户资产变动时 | `AssetPushData` |
| `on_position` | 持仓变动时 | `PositionPushData` |
| `on_order` | 订单状态变化时 | `OrderPushData` |
| `on_transaction` | 成交时 | `TransactionPushData` |
| `on_connect` | WebSocket 连接成功时 | - |
| `on_disconnect` | 连接断开时 | - |
| `on_error` | 发生错误时 | `String` |
| `on_kickout` | 被踢出时 | `String` |

---

## 订阅方法 / Subscribe Methods

```rust
use tigeropen::push::SubjectType;

// 添加行情订阅 / Add quote subscription
pc.add_subscription(SubjectType::Quote, &["AAPL".into(), "TSLA".into()]);
pc.add_subscription(SubjectType::Depth, &["AAPL".into()]);
pc.add_subscription(SubjectType::Tick, &["AAPL".into()]);

// 移除行情订阅 / Remove quote subscription
pc.remove_subscription(SubjectType::Quote, Some(&["TSLA".into()]));

// 移除全部该类型订阅 / Remove all subscriptions for a type
pc.remove_subscription(SubjectType::Quote, None);

// 添加账户级别订阅 / Add account subscriptions
pc.add_account_sub(SubjectType::Asset);
pc.add_account_sub(SubjectType::Order);
pc.add_account_sub(SubjectType::Position);
pc.add_account_sub(SubjectType::Transaction);

// 移除账户订阅 / Remove account subscription
pc.remove_account_sub(&SubjectType::Asset);

// 查询当前订阅状态 / Get current subscriptions
let subs = pc.get_subscriptions();
let account_subs = pc.get_account_subscriptions();

// 断开连接 / Disconnect
pc.disconnect();
```

---

## SubjectType 枚举 / SubjectType Enum

| 值 | 说明 |
|----|------|
| `SubjectType::Quote` | 实时行情 |
| `SubjectType::Tick` | 逐笔成交 |
| `SubjectType::Depth` | 深度行情 |
| `SubjectType::Kline` | K线 |
| `SubjectType::Option` | 期权行情 |
| `SubjectType::Future` | 期货行情 |
| `SubjectType::Asset` | 资产变动 |
| `SubjectType::Position` | 持仓变动 |
| `SubjectType::Order` | 订单状态 |
| `SubjectType::Transaction` | 成交记录 |

---

## QuotePushData 字段 / QuotePushData Fields

| 字段 | 类型 | 说明 |
|------|------|------|
| `symbol` | `String` | 标的代码 |
| `latest_price` | `Option<f64>` | 最新价 |
| `pre_close` | `Option<f64>` | 昨收价 |
| `open` | `Option<f64>` | 开盘价 |
| `high` | `Option<f64>` | 最高价 |
| `low` | `Option<f64>` | 最低价 |
| `volume` | `Option<i64>` | 成交量 |
| `amount` | `Option<f64>` | 成交额 |
| `timestamp` | `Option<i64>` | 时间戳（毫秒） |

---

## AssetPushData 字段 / AssetPushData Fields

| 字段 | 类型 | 说明 |
|------|------|------|
| `account` | `String` | 账户号 |
| `net_liquidation` | `Option<f64>` | 净资产 |
| `equity_with_loan` | `Option<f64>` | 含贷款净值 |
| `cash_balance` | `Option<f64>` | 现金余额 |
| `buying_power` | `Option<f64>` | 购买力 |
| `currency` | `Option<String>` | 计价货币 |

---

## PositionPushData 字段 / PositionPushData Fields

| 字段 | 类型 | 说明 |
|------|------|------|
| `account` | `String` | 账户号 |
| `symbol` | `String` | 标的代码 |
| `sec_type` | `Option<String>` | 证券类型 |
| `quantity` | `i32` | 持仓数量 |
| `average_cost` | `Option<f64>` | 平均成本 |
| `market_price` | `Option<f64>` | 最新价格 |
| `market_value` | `Option<f64>` | 市值 |
| `unrealized_pnl` | `Option<f64>` | 浮动盈亏 |

---

## OrderPushData 字段 / OrderPushData Fields

| 字段 | 类型 | 说明 |
|------|------|------|
| `account` | `String` | 账户号 |
| `id` | `Option<i64>` | 订单 ID |
| `symbol` | `String` | 标的代码 |
| `action` | `Option<String>` | `"BUY"` / `"SELL"` |
| `order_type` | `Option<String>` | 订单类型 |
| `quantity` | `Option<i32>` | 数量 |
| `limit_price` | `Option<f64>` | 限价 |
| `status` | `Option<String>` | 订单状态 |
| `filled` | `Option<i32>` | 已成交数量 |
| `avg_fill_price` | `Option<f64>` | 成交均价 |

---

## 注意事项 / Notes

- 调用 `pc.connect().await?` 后，在 `on_connect` 回调中执行订阅（确保连接成功后才订阅）
- SDK 自动处理断线重连（`auto_reconnect` 默认为 `true`）
- ⚠️ **断线重连后订阅不会自动恢复**，必须在 `on_connect` 回调中重新订阅 / Subscriptions NOT auto-restored after reconnect; re-subscribe in `on_connect`
- 心跳保活由 SDK 自动维护
- 回调函数必须实现 `Send + Sync`，在多线程环境中需使用 `Arc`
- 同一个 `PushClient` 实例只能有一个活跃连接
