
# Tiger OpenAPI C# SDK — 账户管理 / Account Management

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/accounts

## 初始化 / Initialize

```csharp
using TigerOpenAPI.Config;
using TigerOpenAPI.Trade;

TigerConfig config = new TigerConfig()
{
    ConfigFilePath = "path/to/config/dir/"
};
TradeClient tradeClient = new TradeClient(config);
```

---

## 账户列表 / Account List

```csharp
using TigerOpenAPI.Trade;
using TigerOpenAPI.Trade.Model;
using TigerOpenAPI.Trade.Response;

var request = new TigerRequest<AccountsResponse>()
{
    ApiMethodName = TradeApiService.ACCOUNTS,
    ModelValue = new AccountModel()
    // 不传 Account 字段返回所有账号（综合、环球、模拟）
    // Omit Account field to return all accounts
};
var response = await tradeClient.ExecuteAsync(request);

// 返回字段 / Response fields:
// account     - 账户号（综合5~10位数字，模拟17位，环球以U开头）
// capability  - CASH（现金）/ RegTMargin（保证金）/ PMGRN（组合保证金）
// status      - Funded / Open / Pending / Rejected / Closed
// accountType - STANDARD / GLOBAL / PAPER
```

---

## 账户资产 / Account Assets

### 环球账户 / Global Account

```csharp
using TigerOpenAPI.Trade.Model;
using TigerOpenAPI.Trade.Response;

var request = new TigerRequest<AssetsResponse>()
{
    ApiMethodName = TradeApiService.ASSETS,
    ModelValue = new AssetModel()
    {
        Account = "DU000001",
        Segment = true,       // 按证券/期货分类
        MarketValue = true    // 按市场分市值（仅环球账户）
    }
};
var response = await tradeClient.ExecuteAsync(request);

// 主要返回字段 / Key response fields:
// netLiquidation  - 净清算值
// availableFunds  - 可用资金
// buyingPower     - 购买力
// cashValue       - 现金
// initMarginReq   - 初始保证金要求
// maintMarginReq  - 维持保证金要求
// unrealizedPnl   - 浮动盈亏
// realizedPnl     - 已实现盈亏
// segments        - 按交易品种（S=证券, C=期货）分类资产
// marketValues    - 按市场（USD/HKD）分类资产
```

### 综合/模拟账号 / Standard/Paper Account

```csharp
using TigerOpenAPI.Trade.Model;
using TigerOpenAPI.Trade.Response;

var request = new TigerRequest<PrimeAssetResponse>()
{
    ApiMethodName = TradeApiService.PRIME_ASSETS,
    ModelValue = new PrimeAssetModel()
    {
        Account = "123456",
        BaseCurrency = "USD",
        Consolidated = true   // SEC+FUND聚合显示
    }
};
var response = await tradeClient.ExecuteAsync(request);

// segments 数组主要字段 / Segment key fields:
// category              - S（证券）/ C（期货）/ F（基金）/ D（数字货币）
// capability            - RegTMargin / Cash
// buyingPower           - 最大购买力（保证金账户日内4倍，隔夜2倍）
// cashAvailableForTrade - 可用资金（用于判断能否开仓）
// cashBalance           - 现金余额
// netLiquidation        - 净清算值
// initMargin            - 初始保证金
// maintainMargin        - 维持保证金（低于此值会强平）
// unrealizedPL          - 浮动盈亏
// currencyAssets        - 按币种（USD/HKD/SGD/CNH）细分资产
```

---

## 账户持仓 / Account Positions

```csharp
using TigerOpenAPI.Trade.Model;
using TigerOpenAPI.Trade.Response;

var request = new TigerRequest<PositionsResponse>()
{
    ApiMethodName = TradeApiService.POSITIONS,
    ModelValue = new PositionModel()
    {
        Account = "123456",
        SecType = "STK",    // STK/OPT/FUT，默认STK
        Currency = "ALL",   // ALL/USD/HKD/CNH
        Market = "ALL"      // ALL/US/HK/CN
    }
};
var response = await tradeClient.ExecuteAsync(request);

// 主要持仓字段 / Key position fields:
// symbol        - 股票代码
// positionQty   - 持仓数量
// averageCost   - 平均成本（FIFO）
// marketValue   - 市值
// unrealizedPnl - 浮动盈亏
// secType       - 证券类型
// market        - 市场
// currency      - 币种
```

### 期权持仓 / Option Positions

```csharp
var request = new TigerRequest<PositionsResponse>()
{
    ApiMethodName = TradeApiService.POSITIONS,
    ModelValue = new PositionModel()
    {
        Account = "123456",
        SecType = "OPT"
    }
};
// 期权持仓额外字段 / Option-specific fields:
// strike - 行权价, expiry - 到期日, right - CALL/PUT
```

---

## 历史资产分析 / Asset Analytics (PnL History)

```csharp
using TigerOpenAPI.Trade.Model;
using TigerOpenAPI.Trade.Response;

var request = new TigerRequest<PrimeAnalyticsAssetResponse>()
{
    ApiMethodName = TradeApiService.PRIME_ANALYTICS_ASSET,
    ModelValue = new PrimeAnalyticsAssetModel()
    {
        Account = "123456",
        StartDate = "2024-01-01",
        EndDate = "2024-01-31",
        SegType = "SEC",   // SEC / FUT
        Currency = "USD"
    }
};
var response = await tradeClient.ExecuteAsync(request);

// summary 字段 / Summary fields:
// pnl                - 盈亏金额
// pnlPercentage      - 收益率
// annualizedReturn   - 年化收益率

// history 数组每项字段 / History item fields:
// date               - 日期时间戳（毫秒）
// asset              - 总资产
// pnl                - 当日盈亏
// cashBalance        - 现金余额
// grossPositionValue - 持仓市值
// deposit            - 入金
// withdrawal         - 出金
```

---

## 最大可交易数量 / Estimate Tradable Quantity

```csharp
var request = new TigerRequest<EstimateTradableQuantityResponse>()
{
    ApiMethodName = TradeApiService.ESTIMATE_TRADABLE_QUANTITY,
    ModelValue = new EstimateTradableQuantityModel()
    {
        Account = "123456",
        Symbol = "AAPL",
        SecType = "STK",
        Action = "BUY",
        OrderType = "LMT",
        LimitPrice = 150.0
    }
};
var response = await tradeClient.ExecuteAsync(request);

// 返回字段 / Response fields:
// tradableQuantity          - 现金可买/卖数量
// financingQuantity         - 融资融券可买/卖数量
// positionQuantity          - 持仓数量
// tradablePositionQuantity  - 持仓可交易数量
```

---

## 资金转账（Segment 间）/ Segment Fund Transfer

### 查询可转出金额 / Query Available Amount

```csharp
var request = new TigerRequest<SegmentFundAvailableResponse>()
{
    ApiMethodName = TradeApiService.SEGMENT_FUND_AVAILABLE,
    ModelValue = new SegmentFundAvailableModel()
    {
        Account = "123456",
        FromSegment = "SEC",  // SEC / FUT
        Currency = "USD"
    }
};
var response = await tradeClient.ExecuteAsync(request);
// fromSegment, currency, amount
```

### 发起转账 / Transfer

```csharp
var request = new TigerRequest<SegmentFundResponse>()
{
    ApiMethodName = TradeApiService.SEGMENT_FUND_TRANSFER,
    ModelValue = new SegmentFundTransferModel()
    {
        Account = "123456",
        FromSegment = "SEC",
        ToSegment = "FUT",
        Currency = "USD",
        Amount = 1000.0
    }
};
var response = await tradeClient.ExecuteAsync(request);
// 转账状态 status: NEW / PROC / SUCC / FAIL / CANC
```

### 取消转账 / Cancel Transfer

```csharp
var request = new TigerRequest<SegmentFundResponse>()
{
    ApiMethodName = TradeApiService.SEGMENT_FUND_CANCEL,
    ModelValue = new SegmentFundCancelModel()
    {
        Account = "123456",
        Id = 30300805635506176L
    }
};
var response = await tradeClient.ExecuteAsync(request);
```

---

## 出入金记录 / Deposit & Withdrawal Records

```csharp
var request = new TigerRequest<DepositWithdrawResponse>()
{
    ApiMethodName = TradeApiService.DEPOSIT_WITHDRAW,
    ModelValue = new DepositWithdrawModel()
    {
        Account = "123456",
        Lang = "en_US"
    }
};
var response = await tradeClient.ExecuteAsync(request);

// 每条记录字段 / Record fields:
// type         - 1(入金) / 3(出金) / 20(出金费用) 等
// typeDesc     - 类型描述
// currency     - 币种
// amount       - 金额
// businessDate - 业务日期
// completedStatus - 是否完成
```

---

## 注意事项 / Notes

- 环球账户(Global)用 `ASSETS`，综合/模拟账户(Standard/Paper)用 `PRIME_ASSETS`
- Segment 分类：S=证券, C=期货, F=基金, D=数字货币
- 持仓使用 `positionQty` 字段，旧字段 `position`+`positionScale` 已废弃
- `maintainMargin` 降至 0 以下时会触发强制平仓
- 机构用户额外传 `SecretKey` 字段
- 支持 `Execute()` 同步和 `ExecuteAsync()` 异步两种调用方式
