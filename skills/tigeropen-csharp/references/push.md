# Tiger OpenAPI C# SDK — Real-time Push / 实时推送

> C# SDK 实时推送 API 参考 / Push API Reference
<!-- 当用户提到"推送"、"实时"、"订阅"、"WebSocket"、"push"、"subscribe"时 -->

## 初始化 / Initialize

```csharp
using TigerOpenAPI.Config;
using TigerOpenAPI.Push;
using TigerOpenAPI.Push.Model;

TigerConfig config = new TigerConfig()
{
    ConfigFilePath = "path/to/config/",
    IsSslSocket = true  // 推荐启用 SSL
};

// PushClient 是单例，使用工厂方法 / Singleton, use factory method
PushClient pushClient = PushClient.GetInstance()
    .Config(config)
    .ApiComposeCallback(new MyApiComposeCallback());  // 注册回调实现

pushClient.Connect();
```

---

## 实现回调接口 / Implement Callback Interface

用户需要实现 `IApiComposeCallback` 接口（继承自 `ISubscribeApiCallback`）：

```csharp
using TigerOpenAPI.Push;
using TigerOpenAPI.Push.Model;

public class MyApiComposeCallback : IApiComposeCallback
{
    // ===== 连接生命周期 / Connection lifecycle =====

    public void ConnectionAck()
    {
        Console.WriteLine("Connected!");
        // 在此处执行订阅 / Subscribe here after connection
    }

    public void ConnectionClosed()
    {
        Console.WriteLine("Disconnected.");
    }

    public void ConnectionKickout(AckModel response)
    {
        Console.WriteLine($"Kicked out: {response?.Message}");
    }

    public void HearBeat()
    {
        // 心跳（可留空）
    }

    public void ServerHeartBeatTimeOut(string errorMsg)
    {
        Console.WriteLine($"Heartbeat timeout: {errorMsg}");
    }

    public void Error(AckModel response)
    {
        Console.WriteLine($"Server error: {response?.Message}");
    }

    // ===== 行情数据回调 / Market data callbacks =====

    public void QuoteChange(QuoteBasicData data)
    {
        Console.WriteLine($"Quote: {data.Symbol} price={data.LatestPrice} " +
                          $"volume={data.Volume}");
    }

    public void OptionChange(QuoteBasicData data)
    {
        Console.WriteLine($"Option: {data.Symbol} price={data.LatestPrice}");
    }

    public void FutureChange(QuoteBasicData data)
    {
        Console.WriteLine($"Future: {data.Symbol} price={data.LatestPrice}");
    }

    public void TradeTickChange(TradeTick data)
    {
        Console.WriteLine($"Tick: {data.Symbol} price={data.Price} vol={data.Volume}");
    }

    public void FullTickChange(TickData data)
    {
        Console.WriteLine($"FullTick: {data.Symbol}");
    }

    public void KlineChange(KlineData data)
    {
        Console.WriteLine($"Kline: {data.Symbol} close={data.Close}");
    }

    public void DepthQuoteChange(QuoteDepthData data)
    {
        Console.WriteLine($"Depth: {data.Symbol}");
    }

    // ===== 账户数据回调 / Account data callbacks =====

    public void OrderStatusChange(OrderStatusData data)
    {
        Console.WriteLine($"Order: id={data.Id} status={data.Status} " +
                          $"filled={data.FilledQuantity}");
    }

    public void PositionChange(PositionData data)
    {
        Console.WriteLine($"Position: {data.Symbol} qty={data.PositionQty}");
    }

    public void AssetChange(AssetData data)
    {
        Console.WriteLine($"Asset: cash={data.CashBalance} " +
                          $"net={data.NetLiquidation}");
    }

    // ===== 其他回调 / Other callbacks =====
    public void StockTopPush(object data) { }
    public void OptionTopPush(object data) { }
}
```

---

## 完整示例 / Full Example

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using TigerOpenAPI.Config;
using TigerOpenAPI.Push;
using TigerOpenAPI.Push.Enum;

class Program
{
    static void Main(string[] args)
    {
        TigerConfig config = new TigerConfig()
        {
            ConfigFilePath = "path/to/config/",
        };

        string account = config.DefaultAccount;

        // 1. 创建回调实例（在 ConnectionAck 中执行订阅）
        var callback = new DefaultApiComposeCallback(account);

        // 2. 配置并连接 / Configure and connect
        PushClient client = PushClient.GetInstance()
            .Config(config)
            .ApiComposeCallback(callback);
        client.Connect();

        // 3. 等待信号 / Wait for Ctrl+C
        Console.CancelKeyPress += (s, e) => { e.Cancel = true; };
        Thread.Sleep(Timeout.Infinite);

        // 4. 断开连接 / Disconnect
        client.Disconnect();
    }
}

// 在 ConnectionAck 中执行订阅
class DefaultApiComposeCallback : MyApiComposeCallback
{
    private readonly string _account;
    private PushClient _client;

    public DefaultApiComposeCallback(string account)
    {
        _account = account;
        _client = PushClient.GetInstance();
    }

    public override void ConnectionAck()
    {
        Console.WriteLine("Connected! Subscribing...");
        // 行情订阅 / Quote subscriptions
        _client.SubscribeQuote(new HashSet<string> { "AAPL", "TSLA" });

        // 账户订阅 / Account subscriptions
        _client.Subscribe(Subject.Asset);
        _client.Subscribe(Subject.Order);
        _client.Subscribe(Subject.Position);
    }
}
```

---

## 订阅方法 / Subscribe Methods

### 行情订阅 / Quote Subscription

```csharp
// 股票行情 / Stock quotes
pushClient.SubscribeQuote(new HashSet<string> { "AAPL", "TSLA", "00700" });
pushClient.CancelSubscribeQuote();           // 取消所有行情订阅

// 深度行情 / Quote depth
pushClient.SubscribeDepthQuote(new HashSet<string> { "AAPL" });
pushClient.CancelSubscribeDepthQuote();

// K 线 / Kline
pushClient.SubscribeKline(new HashSet<string> { "AAPL" });
pushClient.CancelSubscribeKline();

// 逐笔成交 / Trade tick
pushClient.SubscribeTradeTick(new HashSet<string> { "AAPL" });
pushClient.CancelSubscribeTradeTick();

// 期权行情 / Option quotes
pushClient.SubscribeOption(new HashSet<string> { "AAPL  250829C00150000" });
pushClient.CancelSubscribeOption();

// 期货行情 / Future quotes
pushClient.SubscribeFuture(new HashSet<string> { "CL2509" });
pushClient.CancelSubscribeFuture();
```

### 账户推送 / Account Push

```csharp
using TigerOpenAPI.Push.Enum;

// 资产变动 / Asset changes
pushClient.Subscribe(Subject.Asset);
pushClient.CancelSubscribe(Subject.Asset);

// 订单状态 / Order status
pushClient.Subscribe(Subject.Order);
pushClient.CancelSubscribe(Subject.Order);

// 持仓变动 / Position changes
pushClient.Subscribe(Subject.Position);
pushClient.CancelSubscribe(Subject.Position);
```

---

## 推送数据字段 / Push Data Fields

### QuoteBasicData（股票/期权/期货行情）

| 字段 Field | 类型 | 说明 |
|-----------|------|------|
| `Symbol` | string | 标的代码 |
| `LatestPrice` | double | 最新价 |
| `Volume` | long | 成交量 |
| `BidPrice` | double | 买一价 |
| `AskPrice` | double | 卖一价 |
| `Change` | double | 涨跌额 |
| `ChangeRatio` | double | 涨跌幅 |
| `Timestamp` | long | 时间戳（毫秒） |

### AssetData（资产）

| 字段 Field | 类型 | 说明 |
|-----------|------|------|
| `Account` | string | 账户号 |
| `NetLiquidation` | double | 净资产 |
| `CashBalance` | double | 现金余额 |
| `BuyingPower` | double | 购买力 |
| `AvailableFunds` | double | 可用资金 |

### OrderStatusData（订单）

| 字段 Field | 类型 | 说明 |
|-----------|------|------|
| `Id` | long | 订单 ID |
| `Symbol` | string | 标的代码 |
| `Status` | string | 订单状态 |
| `AvgFillPrice` | double | 平均成交价 |
| `FilledQuantity` | int | 已成交数量 |
| `TotalQuantity` | int | 总数量 |

### PositionData（持仓）

| 字段 Field | 类型 | 说明 |
|-----------|------|------|
| `Account` | string | 账户号 |
| `Symbol` | string | 标的代码 |
| `PositionQty` | int | 持仓数量 |
| `AverageCost` | double | 平均成本 |
| `MarketPrice` | double | 最新价格 |
| `MarketValue` | double | 市值 |
| `UnrealizedPnl` | double | 浮动盈亏 |

---

## IApiComposeCallback 方法速查 / Interface Methods

| 方法 Method | 触发时机 | 参数类型 |
|------------|---------|---------|
| `ConnectionAck()` | 连接成功时（在此订阅） | — |
| `ConnectionClosed()` | 连接断开时 | — |
| `ConnectionKickout()` | 被踢出时 | `AckModel` |
| `HearBeat()` | 心跳 | — |
| `ServerHeartBeatTimeOut()` | 心跳超时 | `string` |
| `Error()` | 服务端错误 | `AckModel` |
| `QuoteChange()` | 股票行情更新 | `QuoteBasicData` |
| `OptionChange()` | 期权行情更新 | `QuoteBasicData` |
| `FutureChange()` | 期货行情更新 | `QuoteBasicData` |
| `TradeTickChange()` | 逐笔成交（简化） | `TradeTick` |
| `FullTickChange()` | 逐笔成交（完整） | `TickData` |
| `KlineChange()` | K 线更新 | `KlineData` |
| `DepthQuoteChange()` | 深度行情更新 | `QuoteDepthData` |
| `OrderStatusChange()` | 订单状态变化 | `OrderStatusData` |
| `PositionChange()` | 持仓变动 | `PositionData` |
| `AssetChange()` | 账户资产变动 | `AssetData` |

---

## 注意事项 / Notes

- **在 `ConnectionAck()` 回调中执行订阅**，连接成功后才能订阅
- `Connect()` 是非阻塞的，连接成功后触发 `ConnectionAck`
- SDK 不自动重连；若需重连，在 `ConnectionClosed()` 中自行调用 `Connect()`
- 账户推送使用 `Subject` 枚举（`Subject.Asset/Order/Position`），不需要传账户号
- 行情订阅使用 `ISet<string>` 传入标的代码集合
- 多次调用 `GetInstance()` 返回同一个单例对象
