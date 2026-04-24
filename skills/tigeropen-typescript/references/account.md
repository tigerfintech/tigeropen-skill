
# Tiger OpenAPI TypeScript SDK — 账户管理 / Account Management

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/accounts

## 初始化 / Initialize

```typescript
import { createClientConfig } from 'tigeropen';
import { HttpClient } from 'tigeropen/dist/esm/client/http-client.js';
import { TradeClient } from 'tigeropen/dist/esm/trade/trade-client.js';

const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});

const httpClient = new HttpClient(config);
const tc = new TradeClient(httpClient, config.account);
```

---

## 账户列表 / Account List

```typescript
// 不传 account 返回所有账号（综合、环球、模拟）
// Omit account to return all accounts
const result = await tc.accounts({});
console.log(result);

// 返回字段 / Response fields:
// account     - 账户号（综合5~10位数字，模拟17位，环球以U开头）
// capability  - CASH / RegTMargin / PMGRN
// status      - Funded / Open / Pending / Rejected / Closed
// accountType - STANDARD / GLOBAL / PAPER
```

---

## 账户资产 / Account Assets

### 环球账户 / Global Account

```typescript
const result = await tc.assets({
  account: 'DU000001',
  segment: true,       // 按证券/期货分类
  market_value: true   // 按市场分市值（仅环球账户）
});

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
```

### 综合/模拟账号 / Standard/Paper Account

```typescript
const result = await tc.primeAssets({
  account: '123456',
  base_currency: 'USD',
  consolidated: true   // SEC+FUND聚合显示
});

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
```

---

## 账户持仓 / Account Positions

```typescript
const result = await tc.positions({
  account: '123456',
  sec_type: 'STK',   // STK/OPT/FUT，默认STK
  currency: 'ALL',   // ALL/USD/HKD/CNH
  market: 'ALL'      // ALL/US/HK/CN
});

if (Array.isArray(result)) {
  result.forEach(pos => {
    console.log(`${pos.symbol}: qty=${pos.positionQty}, pnl=${pos.unrealizedPnl}`);
  });
}

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

```typescript
const result = await tc.positions({
  account: '123456',
  sec_type: 'OPT'
});
// 期权持仓额外字段 / Option-specific fields:
// strike - 行权价, expiry - 到期日, right - CALL/PUT
```

---

## 历史资产分析 / Asset Analytics (PnL History)

```typescript
const result = await tc.primeAnalyticsAsset({
  account: '123456',
  start_date: '2024-01-01',
  end_date: '2024-01-31',
  seg_type: 'SEC',   // SEC / FUT
  currency: 'USD'
});

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

```typescript
const result = await tc.estimateTradableQuantity({
  account: '123456',
  symbol: 'AAPL',
  sec_type: 'STK',
  action: 'BUY',
  order_type: 'LMT',
  limit_price: 150.0
});

console.log(`Can buy: ${result?.tradableQuantity} shares`);

// 返回字段 / Response fields:
// tradableQuantity          - 现金可买/卖数量
// financingQuantity         - 融资融券可买/卖数量
// positionQuantity          - 持仓数量
// tradablePositionQuantity  - 持仓可交易数量
```

---

## 资金转账（Segment 间）/ Segment Fund Transfer

### 查询可转出金额 / Query Available Amount

```typescript
const result = await tc.segmentFundAvailable({
  account: '123456',
  from_segment: 'SEC',   // SEC / FUT
  currency: 'USD'
});
// 返回 fromSegment, currency, amount
```

### 发起转账 / Transfer

```typescript
const result = await tc.segmentFundTransfer({
  account: '123456',
  from_segment: 'SEC',
  to_segment: 'FUT',
  currency: 'USD',
  amount: 1000.0
});
// 转账状态 status: NEW / PROC / SUCC / FAIL / CANC
```

---

## 出入金记录 / Deposit & Withdrawal Records

```typescript
const result = await tc.depositWithdraw({
  account: '123456'
});

if (Array.isArray(result)) {
  result.forEach(r => {
    console.log(`${r.typeDesc}: ${r.currency} ${r.amount} on ${r.businessDate}`);
  });
}

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

- 环球账户(Global)用 `assets()`，综合/模拟账户(Standard/Paper)用 `primeAssets()`
- Segment 分类：S=证券, C=期货, F=基金, D=数字货币
- 持仓使用 `positionQty` 字段，旧字段 `position`+`positionScale` 已废弃
- `maintainMargin` 低于 0 时会触发强制平仓
- 机构用户额外传 `secret_key` 字段
- 所有方法均为 async，需要 `await`
- 子路径导入需使用 `tigeropen/dist/esm/...` 路径（参见 quickstart.md）
