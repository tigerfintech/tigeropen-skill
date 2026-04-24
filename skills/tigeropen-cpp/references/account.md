
# Tiger OpenAPI C++ SDK — 账户管理 / Account Management

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/accounts

## 初始化 / Initialize

```cpp
#include "tigerapi/client_config.h"
#include "tigerapi/trade_client.h"

using namespace TIGER_API;

ClientConfig config(false, "path/to/config/dir/");
auto trade_client = make_shared<TradeClient>(config);
```

---

## 账户列表 / Account List

API 名称 / API name: `ACCOUNTS`

```cpp
#include "tigerapi/tiger_client.h"

value obj = value::object(true);
// 不传 account 返回所有账号（综合、环球、模拟）
// Omit account to return all accounts (standard, global, paper)
value result = trade_client->post(ACCOUNTS, obj);
ucout << result << endl;

// 返回字段 / Response fields:
// account     - 账户号（综合5~10位数字，模拟17位，环球以U开头）
// capability  - 账户类型：CASH（现金）/ RegTMargin（保证金）/ PMGRN（组合保证金）
// status      - 状态：Funded（已入金）/ Open（已开户）/ Pending（待开户）/ Rejected（被拒）/ Closed（已注销）
// accountType - 分类：STANDARD（综合）/ GLOBAL（环球）/ PAPER（模拟）
```

---

## 账户资产 / Account Assets

### 环球账户 / Global Account (ASSETS)

```cpp
value obj = value::object(true);
obj[U("account")] = value::string(U("DU000001"));
// 可选参数 / Optional params:
// obj[U("segment")] = value::boolean(true);      // 按证券/期货分类
// obj[U("market_value")] = value::boolean(true); // 按市场分市值（仅环球）

value result = trade_client->post(ASSETS, obj);
ucout << result << endl;

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

### 综合/模拟账号 / Standard/Paper Account (PRIME_ASSETS)

```cpp
value obj = value::object(true);
obj[U("account")] = value::string(U("123456"));
obj[U("base_currency")] = value::string(U("USD"));
obj[U("consolidated")] = value::boolean(true);  // SEC+FUND聚合显示

value result = trade_client->post(PRIME_ASSETS, obj);
ucout << result << endl;

// segments 数组主要字段 / Segment key fields:
// category            - S（证券）/ C（期货）/ F（基金）/ D（数字货币）
// capability          - RegTMargin / Cash
// buyingPower         - 最大购买力（保证金账户日内4倍，隔夜2倍）
// cashAvailableForTrade - 可用资金（用于判断能否开仓）
// cashBalance         - 现金余额
// netLiquidation      - 净清算值
// initMargin          - 初始保证金
// maintainMargin      - 维持保证金（低于此值会强平）
// unrealizedPL        - 浮动盈亏
// currencyAssets      - 按币种（USD/HKD/SGD/CNH）细分资产
```

---

## 账户持仓 / Account Positions

```cpp
value obj = value::object(true);
obj[U("account")] = value::string(U("123456"));
// obj[U("sec_type")] = value::string(U("STK"));  // STK/OPT/FUT，默认STK
// obj[U("currency")] = value::string(U("ALL"));  // ALL/USD/HKD/CNH
// obj[U("market")] = value::string(U("ALL"));    // ALL/US/HK/CN

value result = trade_client->post(POSITIONS, obj);

// 返回数组，每条持仓主要字段 / Key position fields:
// symbol        - 股票代码
// positionQty   - 持仓数量
// averageCost   - 平均成本（FIFO）
// marketValue   - 市值
// unrealizedPnl - 浮动盈亏
// secType       - 证券类型（STK/OPT等）
// market        - 市场
// currency      - 币种
```

### 期权持仓 / Option Positions

```cpp
value obj = value::object(true);
obj[U("account")] = value::string(U("123456"));
obj[U("sec_type")] = value::string(U("OPT"));

value result = trade_client->post(POSITIONS, obj);
// 期权持仓额外字段：strike（行权价）, expiry（到期日）, right（CALL/PUT）
```

---

## 历史资产分析 / Asset Analytics (PnL History)

```cpp
value obj = value::object(true);
obj[U("account")] = value::string(U("123456"));
obj[U("start_date")] = value::string(U("2024-01-01"));
obj[U("end_date")] = value::string(U("2024-01-31"));
// obj[U("seg_type")] = value::string(U("SEC"));  // SEC / FUT
// obj[U("currency")] = value::string(U("USD"));

value result = trade_client->post(PRIME_ANALYTICS_ASSET, obj);

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

```cpp
value obj = value::object(true);
obj[U("account")] = value::string(U("123456"));
obj[U("symbol")] = value::string(U("AAPL"));
obj[U("sec_type")] = value::string(U("STK"));
obj[U("action")] = value::string(U("BUY"));
obj[U("order_type")] = value::string(U("LMT"));
obj[U("limit_price")] = value::number(150.0);

value result = trade_client->post(ESTIMATE_TRADABLE_QUANTITY, obj);

// 返回字段 / Response fields:
// tradableQuantity          - 现金可买/卖数量
// financingQuantity         - 融资融券可买/卖数量
// positionQuantity          - 持仓数量
// tradablePositionQuantity  - 持仓可交易数量
```

---

## 资金转账（Segment 间）/ Segment Fund Transfer

### 查询可转出金额 / Query Available Amount

```cpp
value obj = value::object(true);
obj[U("account")] = value::string(U("123456"));
obj[U("from_segment")] = value::string(U("SEC"));  // SEC / FUT
obj[U("currency")] = value::string(U("USD"));

value result = trade_client->post(SEGMENT_FUND_AVAILABLE, obj);
// fromSegment, currency, amount
```

### 发起转账 / Transfer

```cpp
value obj = value::object(true);
obj[U("account")] = value::string(U("123456"));
obj[U("from_segment")] = value::string(U("SEC"));
obj[U("to_segment")] = value::string(U("FUT"));
obj[U("currency")] = value::string(U("USD"));
obj[U("amount")] = value::number(1000.0);

value result = trade_client->post(SEGMENT_FUND_TRANSFER, obj);
// 转账状态 / Transfer status: NEW / PROC / SUCC / FAIL / CANC
```

---

## 出入金记录 / Deposit & Withdrawal Records

```cpp
value obj = value::object(true);
obj[U("account")] = value::string(U("123456"));

value result = trade_client->post(DEPOSIT_WITHDRAW, obj);

// 每条记录字段 / Record fields:
// type         - 1(入金) / 3(出金) / 20(出金费用) 等
// typeDesc     - 类型描述
// currency     - 币种
// amount       - 金额
// businessDate - 业务日期
```

---

## 注意事项 / Notes

- 环球账户(Global)使用 `ASSETS`，综合/模拟账户(Standard/Paper)使用 `PRIME_ASSETS`
- Segment 分类：S=证券, C=期货, F=基金, D=数字货币
- 持仓中 `positionQty` 为正确字段，旧字段 `position`+`positionScale` 已废弃
- 维持保证金 `maintainMargin` < 0 时会被强制平仓
- 机构用户（sub_account / secret_key）可在 obj 中额外传入对应字段
