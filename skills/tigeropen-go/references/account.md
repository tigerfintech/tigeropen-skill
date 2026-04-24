
# Tiger OpenAPI Go SDK — 账户管理 / Account Management

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/accounts

## 初始化 / Initialize

```go
import (
    "github.com/tigerfintech/openapi-sdks/go/config"
    "github.com/tigerfintech/openapi-sdks/go/client"
    "github.com/tigerfintech/openapi-sdks/go/trade"
)

cfg, err := config.NewClientConfig(
    config.WithPropertiesFile("tiger_openapi_config.properties"),
)
httpClient := client.NewHttpClient(cfg)
tc := trade.NewTradeClient(httpClient, cfg.Account)
```

---

## 账户列表 / Account List

```go
result, err := httpClient.ExecuteRaw("accounts", map[string]interface{}{})
// 不传 account 返回所有账号（综合、环球、模拟）
// Omit account to return all accounts

// 返回字段 / Response fields:
// account     - 账户号（综合5~10位数字，模拟17位，环球以U开头）
// capability  - CASH（现金）/ RegTMargin（保证金）/ PMGRN（组合保证金）
// status      - Funded / Open / Pending / Rejected / Closed
// accountType - STANDARD / GLOBAL / PAPER

var accounts []map[string]interface{}
json.Unmarshal(result, &accounts)
```

---

## 账户资产 / Account Assets

### 环球账户 / Global Account

```go
result, err := tc.Assets(map[string]interface{}{
    "account":      "DU575569",
    "segment":      true,        // 按证券/期货分类
    "market_value": true,        // 按市场分市值（仅环球账户）
})

// 主要返回字段 / Key response fields:
// netLiquidation  - 净清算值
// availableFunds  - 可用资金
// buyingPower     - 购买力
// cashValue       - 现金
// initMarginReq   - 初始保证金要求
// maintMarginReq  - 维持保证金要求
// unrealizedPnl   - 浮动盈亏
// realizedPnl     - 已实现盈亏
// segments        - 按交易品种（S=证券, C=期货）分类
// marketValues    - 按市场（USD/HKD）分类

var assetData map[string]interface{}
json.Unmarshal(result, &assetData)
```

### 综合/模拟账号 / Standard/Paper Account

```go
result, err := tc.PrimeAssets(map[string]interface{}{
    "account":       "123456",
    "base_currency": "USD",
    "consolidated":  true,   // SEC+FUND聚合显示
})

// segments 数组主要字段 / Segment key fields:
// category              - S（证券）/ C（期货）/ F（基金）/ D（数字货币）
// capability            - RegTMargin / Cash
// buyingPower           - 最大购买力（保证金账户日内4倍，隔夜2倍）
// cashAvailableForTrade - 可用资金
// cashBalance           - 现金余额
// netLiquidation        - 净清算值
// initMargin            - 初始保证金
// maintainMargin        - 维持保证金（低于此值会强平）
// unrealizedPL          - 浮动盈亏
// currencyAssets        - 按币种（USD/HKD/SGD/CNH）细分

var data map[string]interface{}
json.Unmarshal(result, &data)
```

---

## 账户持仓 / Account Positions

```go
result, err := tc.Positions(map[string]interface{}{
    "account":  "123456",
    "sec_type": "STK",   // STK/OPT/FUT，默认STK
    "currency": "ALL",   // ALL/USD/HKD/CNH
    "market":   "ALL",   // ALL/US/HK/CN
})

// 主要持仓字段 / Key position fields:
// symbol        - 股票代码
// positionQty   - 持仓数量
// averageCost   - 平均成本（FIFO）
// marketValue   - 市值
// unrealizedPnl - 浮动盈亏
// secType       - 证券类型
// market        - 市场
// currency      - 币种

var positions []map[string]interface{}
json.Unmarshal(result, &positions)
```

### 期权持仓 / Option Positions

```go
result, err := tc.Positions(map[string]interface{}{
    "account":  "123456",
    "sec_type": "OPT",
})
// 期权持仓额外字段 / Option-specific fields:
// strike - 行权价, expiry - 到期日, right - CALL/PUT
```

---

## 历史资产分析 / Asset Analytics (PnL History)

```go
result, err := tc.PrimeAnalyticsAsset(map[string]interface{}{
    "account":    "123456",
    "start_date": "2024-01-01",
    "end_date":   "2024-01-31",
    "seg_type":   "SEC",   // SEC / FUT
    "currency":   "USD",
})

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

var analytics map[string]interface{}
json.Unmarshal(result, &analytics)
```

---

## 最大可交易数量 / Estimate Tradable Quantity

```go
result, err := tc.EstimateTradableQuantity(map[string]interface{}{
    "account":     "123456",
    "symbol":      "AAPL",
    "sec_type":    "STK",
    "action":      "BUY",
    "order_type":  "LMT",
    "limit_price": 150.0,
})

// 返回字段 / Response fields:
// tradableQuantity          - 现金可买/卖数量
// financingQuantity         - 融资融券可买/卖数量
// positionQuantity          - 持仓数量
// tradablePositionQuantity  - 持仓可交易数量

var qty map[string]interface{}
json.Unmarshal(result, &qty)
```

---

## 资金转账（Segment 间）/ Segment Fund Transfer

### 查询可转出金额 / Query Available Amount

```go
result, err := tc.SegmentFundAvailable(map[string]interface{}{
    "account":      "123456",
    "from_segment": "SEC",   // SEC / FUT
    "currency":     "USD",
})
// 返回 fromSegment, currency, amount
```

### 发起转账 / Transfer

```go
result, err := tc.SegmentFundTransfer(map[string]interface{}{
    "account":      "123456",
    "from_segment": "SEC",
    "to_segment":   "FUT",
    "currency":     "USD",
    "amount":       1000.0,
})
// 转账状态 status: NEW / PROC / SUCC / FAIL / CANC
```

---

## 出入金记录 / Deposit & Withdrawal Records

```go
result, err := tc.DepositWithdraw(map[string]interface{}{
    "account": "123456",
})

// 每条记录字段 / Record fields:
// type         - 1(入金) / 3(出金) / 20(出金费用) 等
// typeDesc     - 类型描述
// currency     - 币种
// amount       - 金额
// businessDate - 业务日期

var records []map[string]interface{}
json.Unmarshal(result, &records)
```

---

## 注意事项 / Notes

- 环球账户(Global)用 `Assets()`，综合/模拟账户(Standard/Paper)用 `PrimeAssets()`
- Segment 分类：S=证券, C=期货, F=基金, D=数字货币
- 所有 API 返回 `(json.RawMessage, error)`，需自行 `json.Unmarshal` 解析
- 持仓使用 `positionQty` 字段，旧字段 `position`+`positionScale` 已废弃
- `maintainMargin` 低于 0 时会触发强制平仓
- 机构用户额外传 `"secret_key"` 字段
