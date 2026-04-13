# Tiger OpenAPI Rust SDK — Quickstart

> Rust SDK 快速入门 / Quick Start for Rust SDK
> GitHub: https://github.com/tigerfintech/openapi-sdks

## 安装 / Installation

在 `Cargo.toml` 中添加依赖：

```toml
[dependencies]
tigeropen = "0.1"
tokio = { version = "1", features = ["full"] }
serde_json = "1"
```

要求 / Requirements: Rust 1.70+，tokio async runtime

---

## 配置 / Configuration

### 方式一：代码直接设置 / Method 1: Builder pattern

```rust
use tigeropen::config::ClientConfig;

let config = ClientConfig::builder()
    .tiger_id("your_tiger_id")
    .private_key("your_rsa_private_key")  // PEM 格式私钥字符串
    .account("your_account")
    .build()?;
```

### 方式二：从 properties 文件加载 / Method 2: Properties file

```rust
let config = ClientConfig::builder()
    .properties_file("tiger_openapi_config.properties")
    .build()?;
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

> 优先级：环境变量 > Builder 设置 > properties 文件

### 配置项说明 / Config Options

| Builder 方法 | 说明 | 必填 |
|-------------|------|------|
| `.tiger_id(id)` | 开发者 ID | ✅ |
| `.private_key(key)` | RSA 私钥 PEM 字符串 | ✅ |
| `.account(account)` | 交易账户号 | - |
| `.properties_file(path)` | .properties 文件路径 | - |
| `.license(license)` | 牌照类型（如 `"TBNZ"`） | - |
| `.language(Language::EnUs)` | `Language::ZhCn` / `Language::EnUs` | - |
| `.timezone(tz)` | 时区字符串 | - |
| `.timeout(Duration::from_secs(30))` | 请求超时（默认 15s） | - |
| `.sandbox_debug(true)` | 使用沙箱环境（模拟账户） | - |

---

## 客户端创建 / Create Clients

```rust
use tigeropen::config::ClientConfig;
use tigeropen::client::http_client::HttpClient;
use tigeropen::quote::QuoteClient;
use tigeropen::trade::TradeClient;
use tigeropen::push::PushClient;

let config = ClientConfig::builder()
    .tiger_id("your_tiger_id")
    .private_key("your_rsa_private_key")
    .account("your_account")
    .build()?;

let http_client = HttpClient::new(config.clone())?;

let qc = QuoteClient::new(&http_client);           // 行情客户端
let tc = TradeClient::new(&http_client, &config.account); // 交易客户端
let pc = PushClient::new(config, None);            // 推送客户端
```

---

## 通用 API 调用 / Generic API Call

当 SDK 未封装某个 API 时，直接调用：

```rust
use serde_json::json;

let result = http_client.execute_raw(
    "market_state",
    json!({"market": "US"})
).await?;

println!("{}", serde_json::to_string_pretty(&result)?);
```

---

## 错误处理 / Error Handling

```rust
use tigeropen::error::TigerError;

match qc.quote_real_time(&["AAPL"]).await {
    Ok(Some(data)) => {
        // data 是 serde_json::Value，需手动解析
        println!("{}", serde_json::to_string_pretty(&data)?);
    }
    Ok(None) => println!("无数据"),
    Err(TigerError::Api(msg)) => eprintln!("API 错误: {}", msg),
    Err(TigerError::Network(e)) => eprintln!("网络错误: {}", e),
    Err(TigerError::Config(msg)) => eprintln!("配置错误: {}", msg),
    Err(e) => eprintln!("其他错误: {}", e),
}
```

---

## 完整示例 / Full Example

```rust
use tigeropen::config::ClientConfig;
use tigeropen::client::http_client::HttpClient;
use tigeropen::quote::QuoteClient;
use tigeropen::trade::TradeClient;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = ClientConfig::builder()
        .tiger_id("your_tiger_id")
        .private_key("your_rsa_private_key")
        .account("your_account")
        .build()?;

    let http_client = HttpClient::new(config.clone())?;

    // 查询实时行情
    let qc = QuoteClient::new(&http_client);
    if let Some(data) = qc.quote_real_time(&["AAPL", "TSLA"]).await? {
        println!("行情: {}", serde_json::to_string_pretty(&data)?);
    }

    // 查询订单
    let tc = TradeClient::new(&http_client, &config.account);
    if let Some(orders) = tc.orders().await? {
        println!("订单: {}", serde_json::to_string_pretty(&orders)?);
    }

    Ok(())
}
```

---

## 前置条件 / Prerequisites

1. 老虎证券账户 + 开发者 API 权限：https://developer.itigerup.com/
2. 准备好 `tiger_id`、RSA 私钥、`account`
3. 行情数据需要对应的行情权限；期权行情需要期权行情权限

---

## FAQ

**Q: 私钥如何生成?**
参考官方文档 https://docs.itigerup.com/docs/prepare，使用 RSA-2048 生成密钥对，上传公钥到开发者后台。

**Q: 返回值格式?**
所有 API 返回 `Result<Option<serde_json::Value>, TigerError>`，需手动用 `serde_json` 解析。

**Q: 模拟账户和实盘账户区别?**
模拟账户在开发者后台申请，`.sandbox_debug(true)` 连接沙箱环境；实盘账户直接使用生产域名。

**Q: `async` 运行时要求?**
必须在 tokio 运行时中使用，通常在 `main` 函数加 `#[tokio::main]`。
