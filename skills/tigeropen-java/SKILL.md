---
name: tigeropen-java
description: |
  Tiger Brokers OpenAPI Java SDK — complete skills for AI coding tools. Covers SDK setup, market data queries, stock/futures/options trading, real-time push subscriptions, account management, and strategy examples. Use when building Java trading applications, querying market data, placing orders, or integrating Tiger Brokers API.
  老虎证券 OpenAPI Java SDK 完整技能集。涵盖 SDK 配置、行情查询、股票/期货/期权交易、实时推送订阅、账户管理、策略示例。适用于构建 Java 交易应用、查询行情数据、下单交易、或集成老虎 API。
license: Apache-2.0
compatibility: Requires Java JDK 1.7+, Maven or Gradle, and a Tiger Brokers developer account
metadata:
  author: tigerbrokers
  version: "2.0.3"
  language: zh_CN, en_US
---

# Tiger Open API Java SDK

> 老虎量化开放平台 Java SDK 完整技能集 / Complete AI skill set for Tiger Brokers OpenAPI (Java)

- Docs: https://docs.itigerup.com/docs/prepare-java
- GitHub: https://github.com/tigerfintech/openapi-java-sdk
- SDK: Maven `com.tigerbrokers:openapi-java-sdk` | Java 1.7+

## Reference Guides

This skill is organized into focused reference files. Load the relevant guide based on your task:

- **[Quickstart](references/quickstart.md)** — SDK install (Maven/Gradle), authentication, client setup, basic examples, object reference, strategy examples
- **[Market Data](references/quote.md)** — Stock/futures/fund/crypto/warrant quotes, K-lines, depth, ticks, scanner, fundamentals, corporate actions
- **[Trading](references/trade.md)** — Place orders (market/limit/stop/algo), order management, contracts, assets, positions, fund transfers
- **[Options](references/option.md)** — Option expiration dates, chains, Greeks, K-line, tick data
- **[Real-time Push](references/push.md)** — Subscribe to quote/depth/tick/K-line/order/position/asset changes via WebSocketClient
- **[Account](references/account.md)** — Account queries, assets, positions, analytics, P&L

## Quick Start

```java
import com.tigerbrokers.stock.openapi.client.config.ClientConfig;
import com.tigerbrokers.stock.openapi.client.https.client.TigerHttpClient;
import com.tigerbrokers.stock.openapi.client.https.request.quote.QuoteMarketRequest;
import com.tigerbrokers.stock.openapi.client.https.response.quote.QuoteMarketResponse;

// 1. Configure / 配置
ClientConfig clientConfig = ClientConfig.DEFAULT_CONFIG;
clientConfig.tigerId = "your_tiger_id";
clientConfig.defaultAccount = "your_account";
clientConfig.privateKey = "your_private_key_content";

// 2. Create client / 创建客户端
TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(clientConfig);

// 3. Query market status / 查询市场状态
QuoteMarketResponse response = client.execute(QuoteMarketRequest.newRequest(Market.US));
```

## When to Use Each Reference

| Task | Reference |
|------|-----------|
| First time setup, SDK install, auth config | [quickstart.md](references/quickstart.md) |
| Get stock/option/future quotes, K-lines, screener | [quote.md](references/quote.md) |
| Place/modify/cancel orders, check positions/assets | [trade.md](references/trade.md) |
| Option chains, Greeks, expiration dates | [option.md](references/option.md) |
| Real-time streaming data via WebSocket | [push.md](references/push.md) |
| Account info, assets, positions, P&L analytics | [account.md](references/account.md) |
