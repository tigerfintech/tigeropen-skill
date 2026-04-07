# Tiger OpenAPI Skill for AI Coding Tools

[中文](#中文) | [English](#english)

> 遵循 [Agent Skills](https://agentskills.io) 开放标准，支持 Claude Code、Cursor、Gemini CLI、GitHub Copilot、VS Code、OpenAI Codex、OpenClaw 等 40+ AI 工具。

> ⚠️ 使用本插件前请阅读 [风险提示与免责声明](./DISCLAIMER.md)

---

## 中文

### 简介

本仓库包含老虎证券 OpenAPI 的完整 AI 技能插件，支持 **Python SDK** 和 **Java SDK**。导入后，AI 编码工具（Claude Code、Cursor 等）可直接通过自然语言帮你调用老虎 SDK 完成行情查询、下单交易、账户管理等操作。

### 安装

```bash
npx skills add tigerfintech/tigeropen-skill
```

一条命令即可完成安装。CLI 会自动检测你本机已安装的 AI 编码工具，并引导你选择安装范围（项目级 / 全局）和需要的技能（Python / Java / 全部）。

```bash
# 查看可用技能
npx skills add tigerfintech/tigeropen-skill --list

# 只安装 Python SDK 技能
npx skills add tigerfintech/tigeropen-skill --skill tigeropen

# 只安装 Java SDK 技能
npx skills add tigerfintech/tigeropen-skill --skill tigeropen-java

# 安装到指定工具
npx skills add tigerfintech/tigeropen-skill -a claude-code

# 全局安装（所有项目可用）
npx skills add tigerfintech/tigeropen-skill -g

# 安装全部技能到所有工具
npx skills add tigerfintech/tigeropen-skill --all
```

<details>
<summary>其他安装方式</summary>

#### 通过 --plugin-dir 本地加载（Claude Code）

```bash
git clone https://github.com/tigerfintech/tigeropen-skill.git ~/tigeropen-skill
claude --plugin-dir ~/tigeropen-skill
```

#### 手动复制到 skills 目录

```bash
# 全局生效
git clone https://github.com/tigerfintech/tigeropen-skill.git /tmp/tigeropen-skill
cp -r /tmp/tigeropen-skill/skills/tigeropen ~/.claude/skills/       # Python
cp -r /tmp/tigeropen-skill/skills/tigeropen-java ~/.claude/skills/  # Java
rm -rf /tmp/tigeropen-skill

# 仅当前项目
mkdir -p .claude/skills
cp -r /tmp/tigeropen-skill/skills/tigeropen .claude/skills/
cp -r /tmp/tigeropen-skill/skills/tigeropen-java .claude/skills/
```

</details>

### 包含的技能

#### tigeropen — Python SDK

| 模块 | 内容 |
|------|------|
| [quickstart](skills/tigeropen/references/quickstart.md) | SDK 安装、配置、客户端创建、枚举/对象参考、错误码、FAQ |
| [quote](skills/tigeropen/references/quote.md) | 股票/期权/期货/基金/数字货币行情、K线、深度、选股器、基本面 |
| [trade](skills/tigeropen/references/trade.md) | 下单(市价/限价/止损/算法单)、订单管理、账户资产、持仓、资金划转 |
| [option](skills/tigeropen/references/option.md) | 期权链、Greeks、单腿/多腿组合策略、期权计算工具 |
| [push](skills/tigeropen/references/push.md) | 实时推送(行情/深度/逐笔/K线/订单/持仓/资产变动) |
| [cli](skills/tigeropen/references/cli.md) | CLI 命令行工具：配置管理、行情查询、交易操作、账户查看、实时推送 |
| [mcp](skills/tigeropen/references/mcp.md) | MCP Server 配置，集成 Cursor/Claude Code/Trae |

#### tigeropen-java — Java SDK

| 模块 | 内容 |
|------|------|
| [quickstart](skills/tigeropen-java/references/quickstart.md) | SDK 安装(Maven/Gradle)、配置、客户端创建、对象参考、策略示例 |
| [quote](skills/tigeropen-java/references/quote.md) | 股票/期货/基金/数字货币/窝轮行情、K线、深度、选股器、基本面 |
| [trade](skills/tigeropen-java/references/trade.md) | 下单(市价/限价/止损/算法单)、订单管理、合约、持仓划转 |
| [option](skills/tigeropen-java/references/option.md) | 期权到期日、期权链、Greeks、K线、逐笔数据 |
| [push](skills/tigeropen-java/references/push.md) | 实时推送(行情/深度/逐笔/K线/订单/持仓/资产变动) |
| [account](skills/tigeropen-java/references/account.md) | 账户查询、资产、持仓、盈亏分析 |

### 使用示例

安装后，直接用自然语言操作：

```
> 帮我查询 AAPL 和 TSLA 的实时行情

> 用限价单买入 100 股 AAPL，价格 150

> 获取 AAPL 近 30 天的日K线数据并画图

> 查询我的账户资产和当前持仓

> 获取 AAPL 下个月到期的期权链，筛选 delta 在 0.3-0.7 的

> 用 Java SDK 写一个动量策略
```

### 前置条件

1. **Python SDK**: `pip install tigeropen` (Python 3.8+)
2. **Java SDK**: Maven 依赖 `com.tigerbrokers:openapi-java-sdk` (JDK 1.7+)
3. 老虎证券账户和 API 权限（[开发者页面](https://developer.itigerup.com/)）
4. 准备好 `tiger_id`、私钥文件和 `account`

### 仓库结构

```
tigeropen-skill/
├── .claude-plugin/
│   └── plugin.json              # 插件清单
├── skills/
│   ├── tigeropen/               # Python SDK 技能
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── quickstart.md
│   │       ├── quote.md
│   │       ├── trade.md
│   │       ├── option.md
│   │       ├── push.md
│   │       ├── cli.md
│   │       └── mcp.md
│   └── tigeropen-java/          # Java SDK 技能
│       ├── SKILL.md
│       └── references/
│           ├── quickstart.md
│           ├── quote.md
│           ├── trade.md
│           ├── option.md
│           ├── push.md
│           └── account.md
└── README.md
```

---

## English

### Introduction

This repository contains complete AI skill plugins for the Tiger Brokers OpenAPI, supporting both **Python SDK** and **Java SDK**. Once imported, AI coding tools (Claude Code, Cursor, etc.) can help you query market data, place orders, manage accounts, and more through natural language.

> ⚠️ Please read the [Risk Disclosure and Disclaimer](./DISCLAIMER.md) before using this plugin.

### Install

```bash
npx skills add tigerfintech/tigeropen-skill
```

One command is all you need. The CLI auto-detects your installed AI coding tools and guides you through scope selection (project / global) and which skills to install (Python / Java / all).

```bash
# List available skills
npx skills add tigerfintech/tigeropen-skill --list

# Install Python SDK skill only
npx skills add tigerfintech/tigeropen-skill --skill tigeropen

# Install Java SDK skill only
npx skills add tigerfintech/tigeropen-skill --skill tigeropen-java

# Install for a specific agent
npx skills add tigerfintech/tigeropen-skill -a claude-code

# Install globally (available across all projects)
npx skills add tigerfintech/tigeropen-skill -g

# Install all skills to all agents
npx skills add tigerfintech/tigeropen-skill --all
```

<details>
<summary>Alternative installation methods</summary>

#### Load locally via --plugin-dir (Claude Code)

```bash
git clone https://github.com/tigerfintech/tigeropen-skill.git ~/tigeropen-skill
claude --plugin-dir ~/tigeropen-skill
```

#### Manual copy to skills directory

```bash
# Global (all projects)
git clone https://github.com/tigerfintech/tigeropen-skill.git /tmp/tigeropen-skill
cp -r /tmp/tigeropen-skill/skills/tigeropen ~/.claude/skills/       # Python
cp -r /tmp/tigeropen-skill/skills/tigeropen-java ~/.claude/skills/  # Java
rm -rf /tmp/tigeropen-skill

# Project-specific
mkdir -p .claude/skills
cp -r /tmp/tigeropen-skill/skills/tigeropen .claude/skills/
cp -r /tmp/tigeropen-skill/skills/tigeropen-java .claude/skills/
```

</details>

### Included Skills

#### tigeropen — Python SDK

| Module | Coverage |
|--------|----------|
| [quickstart](skills/tigeropen/references/quickstart.md) | SDK setup, config, client creation, enums/objects reference, error codes, FAQ |
| [quote](skills/tigeropen/references/quote.md) | Stock/option/future/fund/crypto quotes, K-lines, depth, scanner, fundamentals |
| [trade](skills/tigeropen/references/trade.md) | Orders (market/limit/stop/algo), order management, assets, positions, fund transfer |
| [option](skills/tigeropen/references/option.md) | Option chains, Greeks, single-leg/multi-leg combo strategies, calculator tools |
| [push](skills/tigeropen/references/push.md) | Real-time push (quotes/depth/ticks/K-line/order/position/asset changes) |
| [cli](skills/tigeropen/references/cli.md) | CLI tool: config management, market data, trading, account info, real-time push |
| [mcp](skills/tigeropen/references/mcp.md) | MCP Server setup for Cursor/Claude Code/Trae integration |

#### tigeropen-java — Java SDK

| Module | Coverage |
|--------|----------|
| [quickstart](skills/tigeropen-java/references/quickstart.md) | SDK setup (Maven/Gradle), config, client creation, object reference, strategy examples |
| [quote](skills/tigeropen-java/references/quote.md) | Stock/future/fund/crypto/warrant quotes, K-lines, depth, scanner, fundamentals |
| [trade](skills/tigeropen-java/references/trade.md) | Orders (market/limit/stop/algo), order management, contracts, position transfer |
| [option](skills/tigeropen-java/references/option.md) | Option expirations, chains, Greeks, K-line, tick data |
| [push](skills/tigeropen-java/references/push.md) | Real-time push (quotes/depth/ticks/K-line/order/position/asset changes) |
| [account](skills/tigeropen-java/references/account.md) | Account queries, assets, positions, P&L analytics |

### Usage Examples

After installing, use natural language:

```
> Get real-time quotes for AAPL and TSLA

> Buy 100 shares of AAPL with a limit order at $150

> Fetch AAPL daily K-line data for the last 30 days and plot it

> Show my account assets and current positions

> Get AAPL option chain expiring next month, filter delta 0.3-0.7

> Write a momentum strategy using the Java SDK
```

### Prerequisites

1. **Python SDK**: `pip install tigeropen` (Python 3.8+)
2. **Java SDK**: Maven dependency `com.tigerbrokers:openapi-java-sdk` (JDK 1.7+)
3. Tiger Brokers account with API access ([Developer Page](https://developer.itigerup.com/))
4. Have your `tiger_id`, private key file, and `account` ready

### Repository Structure

```
tigeropen-skill/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── skills/
│   ├── tigeropen/               # Python SDK skill
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── quickstart.md
│   │       ├── quote.md
│   │       ├── trade.md
│   │       ├── option.md
│   │       ├── push.md
│   │       ├── cli.md
│   │       └── mcp.md
│   └── tigeropen-java/          # Java SDK skill
│       ├── SKILL.md
│       └── references/
│           ├── quickstart.md
│           ├── quote.md
│           ├── trade.md
│           ├── option.md
│           ├── push.md
│           └── account.md
└── README.md
```
