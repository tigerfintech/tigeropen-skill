# Tiger OpenAPI Go SDK — Real-time Push / 实时推送

> Go SDK 实时推送 API 参考 / Push API Reference
<!-- 当用户提到"推送"、"实时"、"订阅"、"WebSocket"、"push"、"subscribe"时 -->

## 初始化 / Initialize

```go
import (
    "github.com/tigerfintech/openapi-sdks/go/config"
    "github.com/tigerfintech/openapi-sdks/go/push"
)

cfg, _ := config.NewClientConfig(
    config.WithPropertiesFile("tiger_openapi_config.properties"),
)
pc := push.NewPushClient(cfg)
```

---

## 完整示例 / Full Example

```go
package main

import (
    "fmt"
    "os"
    "os/signal"
    "syscall"

    "github.com/tigerfintech/openapi-sdks/go/config"
    "github.com/tigerfintech/openapi-sdks/go/push"
)

func main() {
    cfg, _ := config.NewClientConfig(
        config.WithPropertiesFile("tiger_openapi_config.properties"),
    )

    pc := push.NewPushClient(cfg)

    // 设置回调 / Set callbacks
    // ⚠️ 订阅必须在 OnConnect 回调中执行 / Subscriptions MUST be placed inside OnConnect callback
    pc.SetCallbacks(push.Callbacks{
        OnQuote: func(data *push.QuoteData) {
            fmt.Printf("[行情] %s 最新价: %.2f 成交量: %d\n",
                data.Symbol, data.LatestPrice, data.Volume)
        },
        OnOrder: func(data *push.OrderData) {
            fmt.Printf("[订单] %s 状态: %s\n", data.Symbol, data.Status)
        },
        OnAsset: func(data *push.AssetData) {
            fmt.Printf("[资产] 净资产: %.2f\n", data.NetLiquidation)
        },
        OnPosition: func(data *push.PositionData) {
            fmt.Printf("[持仓] %s 数量: %d\n", data.Symbol, data.PositionQty)
        },
        OnConnect: func() {
            fmt.Println("推送连接成功")
            // ⚠️ 在此处执行订阅 / Subscribe here after connection established
            // 重连后订阅不会自动恢复，需在此重新订阅 / Subscriptions NOT auto-restored on reconnect
            pc.SubscribeQuote([]string{"AAPL", "TSLA"})
            pc.SubscribeAsset("")    // 资产变动
            pc.SubscribeOrder("")    // 订单状态变动
            pc.SubscribePosition("") // 持仓变动
        },
        OnDisconnect: func() { fmt.Println("推送连接断开") },
        OnError:      func(err error) { fmt.Printf("推送错误: %v\n", err) },
    })

    // 连接 / Connect
    if err := pc.Connect(); err != nil {
        panic(err)
    }
    defer pc.Disconnect()

    // 等待退出 / Wait for signal
    sig := make(chan os.Signal, 1)
    signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)
    <-sig

    // 退订 / Unsubscribe
    pc.UnsubscribeQuote([]string{"AAPL", "TSLA"})
}
```

---

## 回调函数 / Callbacks

| 回调 Callback | 触发时机 | 数据类型 |
|--------------|---------|---------|
| `OnQuote` | 行情更新时 | `*QuoteData` |
| `OnOrder` | 订单状态变化时 | `*OrderData` |
| `OnAsset` | 账户资产变动时 | `*AssetData` |
| `OnPosition` | 持仓变动时 | `*PositionData` |
| `OnConnect` | WebSocket 连接成功时 | - |
| `OnDisconnect` | 连接断开时 | - |
| `OnError` | 发生错误时 | `error` |

---

## 订阅方法 / Subscribe Methods

```go
// 行情订阅 / Quote subscription
pc.SubscribeQuote([]string{"AAPL", "TSLA", "00700"})
pc.UnsubscribeQuote([]string{"TSLA"})

// 账户推送 / Account push (account="" uses default account)
pc.SubscribeAsset(account)
pc.SubscribeOrder(account)
pc.SubscribePosition(account)
pc.UnsubscribeAsset(account)
pc.UnsubscribeOrder(account)
pc.UnsubscribePosition(account)

// 断开连接 / Disconnect
pc.Disconnect()
```

---

## QuoteData 字段 / QuoteData Fields

| 字段 Field | 说明 |
|-----------|------|
| `Symbol` | 标的代码 |
| `LatestPrice` | 最新价 |
| `Volume` | 成交量 |
| `AskPrice` | 卖一价 |
| `BidPrice` | 买一价 |
| `Change` | 涨跌额 |
| `ChangeRatio` | 涨跌幅 |

## AssetData 字段 / AssetData Fields

| 字段 Field | 说明 |
|-----------|------|
| `Account` | 账户号 |
| `NetLiquidation` | 净资产 |
| `CashBalance` | 现金余额 |
| `BuyingPower` | 购买力 |
| `AvailableFunds` | 可用资金 |
| `InitMarginReq` | 初始保证金 |
| `MaintMarginReq` | 维持保证金 |

---

## 注意事项 / Notes

- **在 `OnConnect` 回调中执行订阅**，连接成功后才能订阅
- SDK 自动处理断线重连，无需手动重连
- ⚠️ **断线重连后订阅不会自动恢复**，必须在 `OnConnect` 回调中重新订阅 / Subscriptions NOT auto-restored after reconnect; re-subscribe in `OnConnect`
- 心跳保活由 SDK 自动维护
- 同一个 `PushClient` 实例只能有一个活跃连接
