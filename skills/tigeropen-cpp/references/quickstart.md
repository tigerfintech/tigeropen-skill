# Tiger OpenAPI C++ SDK — Quickstart / 快速入门

> C++ SDK 快速入门 / Quick Start for C++ SDK
> GitHub: https://github.com/tigerfintech/openapi-cpp-sdk

## 依赖 / Dependencies

| 依赖 Dependency | 版本 Version | 说明 |
|----------------|-------------|------|
| C++ | 17+ | 必须 |
| CMake | 3.15+ | 构建系统 |
| Boost | 1.86 | system/thread/log/chrono/filesystem |
| cpprestsdk | latest | HTTP client（Microsoft REST SDK） |
| Protobuf | v25.1 | 推送消息序列化 |
| OpenSSL | latest | TLS 支持 |

---

## 安装依赖 / Install Dependencies

### macOS

```bash
# Homebrew 安装 / Install via Homebrew
brew install boost cpprestsdk protobuf openssl

# 克隆 SDK
git clone https://github.com/tigerfintech/openapi-cpp-sdk.git
cd openapi-cpp-sdk
```

### Linux (Ubuntu/Debian)

```bash
sudo apt-get install -y libboost-all-dev libcpprest-dev libprotobuf-dev protobuf-compiler libssl-dev
```

---

## 构建 / Build

```bash
mkdir build && cd build

# macOS
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j4

# Linux
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)

# Windows (vcpkg)
cmake .. -DCMAKE_TOOLCHAIN_FILE=$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake
cmake --build . --config Release
```

构建产物：`output/` 目录下包含 SDK 库文件和头文件。

---

## 配置 / Configuration

### 配置文件 / Config File

在项目目录创建 `tiger_openapi_config.properties`：

```properties
tiger_id=your_tiger_id
private_key=-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA...
-----END RSA PRIVATE KEY-----
account=your_account_number
```

### 代码加载配置 / Load Config in Code

```cpp
#include "tigerapi/client_config.h"

using namespace TIGER_API;

// 从配置文件目录加载（第一个参数 false = 生产环境）
ClientConfig config(false, "path/to/config/dir/");

// 沙箱环境 / Sandbox
ClientConfig sandbox_config(true, "path/to/config/dir/");
```

`ClientConfig` 构造参数：
- 第一个参数（bool）：`true` = 沙箱，`false` = 生产
- 第二个参数（string_t）：配置文件目录路径

---

## 客户端创建 / Create Clients

```cpp
#include "tigerapi/client_config.h"
#include "tigerapi/quote_client.h"
#include "tigerapi/trade_client.h"
#include "tigerapi/push_client.h"

using namespace std;
using namespace web;
using namespace web::json;
using namespace TIGER_API;

ClientConfig config(false, "demo/openapi_cpp_test/");

// 行情客户端 / Quote client
auto quote_client = make_shared<QuoteClient>(config);

// 交易客户端 / Trade client
auto trade_client = make_shared<TradeClient>(config);

// 推送客户端 / Push client
shared_ptr<IPushClient> push_client = IPushClient::create_push_client(config);
```

---

## 字符串与 JSON / Strings and JSON

C++ SDK 使用 cpprestsdk 的 `web::json::value`，跨平台字符串用 `utility::string_t`：

```cpp
#include "cpprest/details/basic_types.h"
using namespace web::json;

// U() 宏：将字符串字面量转为 string_t（跨平台）
utility::string_t symbol = U("AAPL");

// 构建 JSON 数组 / Build array
value symbols = value::array();
symbols[0] = value::string(U("AAPL"));
symbols[1] = value::string(U("TSLA"));

// 输出：ucout 替代 std::cout
ucout << symbols << std::endl;
```

---

## 直接调用 API / Raw API Call

当 SDK 未封装某个 API 时，可直接使用 `TigerClient::post()`：

```cpp
#include "tigerapi/tiger_client.h"

// 直接用 TigerClient（QuoteClient/TradeClient 均继承自它）
auto client = make_shared<TigerClient>(config);

value obj = value::object(true);
obj[U("market")] = value::string(U("US"));

// 第一个参数为 API 名称常量，在 enums.h 中定义
value result = client->post(MARKET_STATE, obj);
ucout << result << endl;
```

常用 API 名称常量（在 `tigerapi/enums.h` 中）：

| 常量 | API |
|------|-----|
| `MARKET_STATE` | 市场状态 |
| `BRIEF` | 实时报价 |
| `KLINE` | K 线 |
| `POSITIONS` | 持仓 |
| `ORDERS` | 订单 |
| `ASSETS` | 资产 |

---

## 工程集成 / CMakeLists.txt Example

在自己的项目中使用已编译好的 SDK（`output/Mac/sdk/Release/`）：

```cmake
cmake_minimum_required(VERSION 3.15)
project(my_tiger_app CXX)
set(CMAKE_CXX_STANDARD 17)

find_package(Boost REQUIRED COMPONENTS system thread log chrono filesystem)
find_package(cpprestsdk CONFIG REQUIRED)
find_package(Protobuf REQUIRED)
find_package(OpenSSL REQUIRED)

# SDK 头文件和库路径
set(TIGERAPI_DIR /path/to/openapi-cpp-sdk/output/Mac/sdk/Release)
include_directories(${TIGERAPI_DIR}/include)
link_directories(${TIGERAPI_DIR}/lib)

add_executable(my_app main.cpp)
target_link_libraries(my_app
    tigerapi
    Boost::system Boost::thread Boost::log Boost::chrono Boost::filesystem
    cpprestsdk::cpprest
    ${Protobuf_LIBRARIES}
    OpenSSL::SSL OpenSSL::Crypto
)
```

---

## 错误处理 / Error Handling

```cpp
try {
    value symbols = value::array();
    symbols[0] = value::string(U("AAPL"));
    value result = quote_client->get_brief(symbols);
    ucout << result << endl;
} catch (const std::exception& e) {
    std::cerr << "Error: " << e.what() << std::endl;
} catch (...) {
    std::cerr << "Unknown error occurred" << std::endl;
}
```

---

## 前置条件 / Prerequisites

1. 老虎证券账户 + 开发者 API 权限：https://developer.itigerup.com/
2. 准备好 `tiger_id`、RSA 私钥（2048位）、账户号
3. 行情数据需要对应市场的行情权限
