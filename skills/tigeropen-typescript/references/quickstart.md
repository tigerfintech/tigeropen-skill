# Tiger OpenAPI TypeScript SDK — Quickstart

> TypeScript SDK 快速入门 / Quick Start for TypeScript SDK
> npm: https://www.npmjs.com/package/tigeropen

## 安装 / Installation

```bash
npm install tigeropen
# 或 / or
yarn add tigeropen
# 或 / or
pnpm add tigeropen
```

要求 / Requirements: Node.js 16+，支持 ESM 和 CommonJS

---

## 配置 / Configuration

### 方式一：代码直接设置 / Method 1: Direct config

```typescript
import { createClientConfig } from 'tigeropen';

const config = createClientConfig({
  tigerId: 'your_tiger_id',
  privateKey: 'your_rsa_private_key',   // PEM 格式私钥字符串
  account: 'your_account',
});
```

### 方式二：从 properties 文件加载 / Method 2: Properties file

```typescript
const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});
```

配置文件格式 / Config file format:
```properties
tiger_id=your_tiger_id
private_key=your_rsa_private_key
account=your_account
```

### 方式三：环境变量 / Method 3: Environment variables

```bash
export TIGEROPEN_TIGER_ID=your_tiger_id
export TIGEROPEN_PRIVATE_KEY=your_rsa_private_key
export TIGEROPEN_ACCOUNT=your_account
```

> 优先级：环境变量 > 代码设置 > properties 文件

### 配置项说明 / Config Options

| 配置项 | 说明 | 必填 | 默认值 |
|--------|------|------|--------|
| `tigerId` | 开发者 ID | ✅ | - |
| `privateKey` | RSA 私钥 PEM 字符串 | ✅ | - |
| `account` | 交易账户号 | - | - |
| `propertiesFilePath` | .properties 文件路径 | - | - |
| `language` | `zh_CN` / `en_US` | - | `zh_CN` |
| `timeout` | 请求超时（秒） | - | 15 |
| `sandboxDebug` | 使用沙箱环境 | - | `false` |

---

## 客户端创建 / Create Clients

```typescript
// ⚠️ 包只导出根路径，子路径导入需指定 dist 路径 / Package only exports root, sub-path imports need dist paths
import { createClientConfig } from 'tigeropen';
// ESM sub-path imports (tigeropen@0.1.0 doesn't expose sub-path exports):
import { HttpClient } from 'tigeropen/dist/esm/client/http-client.js';
import { QuoteClient } from 'tigeropen/dist/esm/quote/quote-client.js';
import { TradeClient } from 'tigeropen/dist/esm/trade/trade-client.js';
import { PushClient } from 'tigeropen/dist/esm/push/push-client.js';

const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});

const httpClient = new HttpClient(config);

const qc = new QuoteClient(httpClient);                 // 行情客户端
const tc = new TradeClient(httpClient, config.account); // 交易客户端
const pc = new PushClient(config);                      // 推送客户端
```

> ⚠️ **注意 / Note**: `tigeropen@0.1.0` 包 `exports` 字段仅暴露根路径 `.`，`tigeropen/client/http-client` 等子路径会报 `ERR_PACKAGE_PATH_NOT_EXPORTED` 错误。需直接使用 `tigeropen/dist/esm/...` 路径。CJS 环境同理使用 `tigeropen/dist/cjs/...`。

---

## ESM 和 CommonJS / ESM and CommonJS

```typescript
// ESM (see note above about sub-path exports)
import { createClientConfig } from 'tigeropen';
import { QuoteClient } from 'tigeropen/dist/esm/quote/quote-client.js';

// CommonJS
const { createClientConfig } = require('tigeropen');
const { QuoteClient } = require('tigeropen/dist/cjs/quote/quote-client.js');
```

---

## 通用 API 调用 / Generic API Call

当 SDK 未封装某个 API 时：

```typescript
const resp = await httpClient.execute(
  'market_state',
  JSON.stringify({ market: 'US' })
);
console.log(resp);
```

---

## 错误处理 / Error Handling

```typescript
try {
  const data = await qc.getBrief(['AAPL']);
  console.log(data);
} catch (error) {
  if (error instanceof Error) {
    console.error('API 错误:', error.message);
  }
}
```

---

## 前置条件 / Prerequisites

1. 老虎证券账户 + 开发者 API 权限：https://developer.itigerup.com/
2. 准备好 `tigerId`、RSA 私钥、`account`
3. 行情数据需要对应的行情权限；期权行情需要期权行情权限

---

## FAQ

**Q: 私钥如何生成?**
参考官方文档 https://docs.itigerup.com/docs/prepare，使用 RSA-2048 生成密钥对，上传公钥到开发者后台。

**Q: 模拟账户和实盘账户区别?**
模拟账户在开发者后台申请，`sandboxDebug: true` 连接沙箱环境；实盘账户直接使用生产域名。
