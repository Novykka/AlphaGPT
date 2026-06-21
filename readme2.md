# AlphaGPT

**Automatic Formula Discovery & Quantitative Trading System for Solana Memecoins**

**Solana Meme 币全自动因子发现与量化交易系统**

*Not a price predictor — a formula generator that writes, tests, and trades its own alpha strategies.*

*不是价格预测器——而是一个能自主编写、测试并交易 Alpha 策略的公式生成器。*

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.10%2B-blue)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.0%2B-ee4c2c)](https://pytorch.org/)
[![Solana](https://img.shields.io/badge/Solana-live%20trading-9945FF)](https://solana.com/)

---

## Overview / 概述

AlphaGPT is an end-to-end automated quantitative trading system that **discovers trading formulas using a Transformer model**, backtests them against historical market data, and executes the best strategies live on the Solana blockchain via Jupiter aggregator.

AlphaGPT 是一套端到端的自动化量化交易系统：**用 Transformer 模型自动发现交易公式**，在历史行情数据上回测验证，并将最优策略通过 Jupiter 聚合器在 Solana 区块链上实盘执行。

The core insight: instead of training a model to predict prices directly, we train it to **generate mathematical formulas** (factor expressions) that produce profitable trading signals. A stack-based VM interprets these formulas against market features, a backtest engine scores them by risk-adjusted returns, and reinforcement learning (policy gradient) steers the generator toward ever-better strategies.

核心思路：不直接预测价格，而是训练模型**生成数学公式**（因子表达式），由公式产生可盈利的交易信号。基于栈的虚拟机将公式解释执行于市场特征之上，回测引擎以风险调整收益进行评分，强化学习（策略梯度）驱动公式生成器不断进化。

```text
┌──────────────┐    ┌──────────────────┐    ┌───────────────────┐    ┌──────────────┐
│  Data        │    │  Model Core      │    │  Strategy         │    │  Execution   │
│  Pipeline    │───▶│  (AlphaGPT)      │───▶│  Manager          │───▶│  (Jupiter)   │
│  数据管线     │    │  模型核心         │    │  策略管理          │    │  交易执行     │
│              │    │                  │    │                   │    │              │
│  Birdeye ─┐  │    │  Transformer ─┐  │    │  Live Scoring ─┐  │    │  Quote ─┐    │
│  DexScrn ─┤  │    │  StackVM     ─┤  │    │  Risk Engine   ─┤  │    │  Swap   ─┤  │
│  Postgres─┘  │    │  Backtest    ─┘  │    │  Portfolio     ─┘  │    │  RPC    ─┘  │
└──────────────┘    └──────────────────┘    └───────────────────┘    └──────────────┘
                                                                           │
                                                                   ┌───────────────┐
                                                                   │  Dashboard    │
                                                                   │  看板          │
                                                                   │  (Streamlit)  │
                                                                   └───────────────┘
```

## Key Features / 核心特性

### 🧠 Formula Discovery Engine / 公式发现引擎

- **Transformer-based generator** — generates trading formulas as token sequences (Polish/Reverse-Polish notation)
- **基于 Transformer 的生成器** — 以 token 序列（波兰/逆波兰表示法）的形式生成交易公式
- **StackVM interpreter** — executes formulas against batched feature tensors with 12 mathematical operators
- **StackVM 解释器** — 以 12 种数学算子对批量化特征张量执行公式
- **RL policy gradient training** — REINFORCE with Sortino ratio as reward, plus slippage, turnover, and drawdown penalties
- **强化学习策略梯度训练** — 以 Sortino 比率作为奖励的 REINFORCE 算法，附加滑点、换手率和回撤惩罚
- **Automatic formula evolution** — iteratively discovers, scores, refines, and saves the best strategy to JSON
- **公式自动进化** — 迭代式发现、评分、精炼，将最优策略保存为 JSON

### ⚡ Modern Transformer Architecture / 现代 Transformer 架构

- Looped Transformer layers (recurrent refinement within each layer)
- 循环 Transformer 层（每层内部循环精炼）
- RMSNorm (Root Mean Square Layer Normalization)
- RMSNorm（均方根归一化）
- QK-Norm for attention stability
- QK-Norm 注意力稳定性机制
- SwiGLU activation in feed-forward networks
- 前馈网络中的 SwiGLU 激活函数
- Multi-Task Pooling Head (MTPHead)
- 多任务池化头（MTPHead）

### 🔬 LoRD Regularization / LoRD 正则化

- **Low-Rank Decay** — Newton-Schulz iteration pushes attention parameters toward lower-rank structures
- **低秩衰减** — Newton-Schulz 迭代将注意力参数推向低秩结构
- Improves generalization and produces cleaner algorithmic attention patterns
- 提升泛化能力，产生更清晰的算法化注意力模式
- Validated on modular addition grokking experiments (see `lord/`)
- 已在模加法 grokking 实验中得到验证（参见 `lord/`）

### 📊 Market Data Pipeline / 行情数据管线

- Multi-source data ingestion (Birdeye primary, DexScreener fallback)
- 多源数据接入（Birdeye 为主，DexScreener 为备）
- Discovers trending tokens, filters by liquidity and FDV
- 发现热门代币，按流动性和完全稀释估值（FDV）进行筛选
- 1-minute OHLCV data for up to 500 tokens stored in PostgreSQL/TimescaleDB
- 最多 500 个代币的 1 分钟级 OHLCV 数据，存入 PostgreSQL/TimescaleDB
- Async concurrency for high-throughput API calls
- 异步并发实现高吞吐 API 调用

### 💱 Live Trading Execution / 实盘交易执行

- Full **Solana** integration via **Jupiter Aggregator v6**
- 通过 **Jupiter 聚合器 v6** 完整接入 Solana 链
- SOL → Token buy and Token → SOL sell with configurable slippage
- SOL → 代币买入、代币 → SOL 卖出，滑点可配置
- Priority fee support (High tier by default)
- 优先费支持（默认 High 档位）
- Private key loading: base58 string or JSON byte array
- 私钥加载：支持 base58 字符串或 JSON 字节数组

### 🛡️ Risk Management / 风控管理

- **Pre-trade safety check** — verifies sell quote obtainability (honeypot detection)
- **交易前安全检查** — 验证能否获取卖出报价（蜜罐检测）
- **Liquidity floor** — minimum $5K liquidity requirement
- **流动性门槛** — 最低 $5K 流动性要求
- **Stop-loss** — default -5% hard stop
- **止损** — 默认 -5% 硬止损
- **Trailing stop** — activates at +5%, triggers at -3% from peak
- **移动止盈止损** — 浮盈 +5% 激活，从高点回撤 -3% 触发
- **Take-profit** — partial sell at +10%, leaves moonbag to ride
- **止盈** — +10% 部分卖出，保留底仓吃趋势
- **Position caps** — maximum 3 concurrent open positions
- **仓位上限** — 最多同时持有 3 个仓位

### 📈 Real-time Dashboard / 实时看板

- Streamlit web UI with Plotly interactive charts
- Streamlit Web 界面 + Plotly 交互图表
- Portfolio overview, PnL tracking, market scanner
- 投资组合总览、盈亏追踪、市场扫描
- Emergency stop button
- 紧急停止按钮
- Auto-refresh every 30 seconds
- 每 30 秒自动刷新

## Tech Stack / 技术栈

| Layer 层 | Technology 技术 |
| --- | --- |
| **Language** 语言 | Python 3.10+ (recommended 3.11 推荐 3.11) |
| **Deep Learning** 深度学习 | PyTorch 2.0+ |
| **Data Processing** 数据处理 | Pandas, NumPy |
| **Database** 数据库 | PostgreSQL / TimescaleDB (asyncpg, SQLAlchemy) |
| **Async Runtime** 异步运行时 | asyncio, aiohttp |
| **Blockchain** 区块链 | solana-py 0.30+, solders 0.18+, Jupiter API v6 |
| **Data APIs** 数据接口 | Birdeye API, DexScreener API |
| **Dashboard** 看板 | Streamlit, Plotly |
| **Logging** 日志 | Loguru |

## Project Structure / 项目结构

```text
AlphaGPT/
├── data_pipeline/            # Data ingestion & storage / 数据接入与存储
│   ├── providers/            # Birdeye & DexScreener API clients / API 客户端
│   ├── db_manager.py         # PostgreSQL/TimescaleDB schema & operations / 数据库模式与操作
│   ├── data_manager.py       # Orchestrator: discover → filter → fetch → store / 调度：发现→筛选→抓取→入库
│   ├── processor.py          # Data cleaning & basic feature engineering / 数据清洗与基础特征工程
│   └── run_pipeline.py       # Entry point / 入口
│
├── model_core/               # Formula discovery engine / 公式发现引擎
│   ├── alphagpt.py           # Transformer model + LoRD regularization / Transformer 模型 + LoRD 正则化
│   ├── ops.py                # 12 mathematical operators (ADD, SUB, MUL, DIV, ...) / 12 种数学算子
│   ├── vm.py                 # Stack-based VM for formula execution / 基于栈的公式执行虚拟机
│   ├── backtest.py           # Sortino-ratio backtest with cost modeling / Sortino 比率回测 + 成本建模
│   ├── factors.py            # 6 basic + 12 advanced feature engineers / 6 基础 + 12 高级因子工程
│   ├── data_loader.py        # PostgreSQL → tensor pipeline / PostgreSQL → 张量管线
│   └── engine.py             # RL training loop / 强化学习训练循环
│
├── strategy_manager/         # Live trading orchestration / 实盘交易调度
│   ├── runner.py             # Main loop: sync → score → trade → monitor / 主循环：同步→评分→交易→监控
│   ├── portfolio.py          # Position tracking & PnL / 持仓跟踪与盈亏
│   └── risk.py               # Risk checks & position sizing / 风险检查与仓位管理
│
├── execution/                # Solana blockchain integration / Solana 链集成
│   ├── jupiter.py            # Jupiter aggregator: quote & swap / Jupiter 聚合器：报价与兑换
│   ├── rpc_handler.py        # Solana RPC client / Solana RPC 客户端
│   └── trader.py             # High-level buy/sell interface / 高层买卖接口
│
├── dashboard/                # Streamlit monitoring UI / Streamlit 监控界面
│   ├── app.py                # Web dashboard / Web 看板
│   ├── data_service.py       # Portfolio & market data for UI / 为 UI 提供持仓与行情数据
│   └── visualizer.py         # Plotly charts / Plotly 图表
│
├── lord/                     # LoRD regularization experiments / LoRD 正则化实验
│   └── experiment.py         # Modular addition grokking studies / 模加法 grokking 研究
│
├── times.py                  # Standalone A-share ETF backtester / 独立 A 股 ETF 回测脚本
├── paper/                    # Research materials / 研究材料
├── assets/                   # Images & charts / 图片与图表
├── requirements.txt          # Core dependencies / 核心依赖
├── requirements-optional.txt # Optional: matplotlib, seaborn, tushare / 可选依赖
└── LICENSE                   # Apache 2.0
```

## Operators / 算子

The StackVM supports 12 operators that form the building blocks of discovered formulas:

StackVM 支持 12 种算子，构成发现公式的基本构件：

| Operator 算子 | Description 描述 |
| --- | --- |
| `ADD` | Addition / 加法 |
| `SUB` | Subtraction / 减法 |
| `MUL` | Multiplication / 乘法 |
| `DIV` | Division (safe, with epsilon) / 除法（安全，含 epsilon） |
| `NEG` | Negation / 取负 |
| `ABS` | Absolute value / 绝对值 |
| `SIGN` | Sign function / 符号函数 |
| `GATE` | Gated selection: `condition > 0 ? x : y` / 门控选择 |
| `JUMP` | Extreme jump detection (z-score > 3) / 极端跳变检测 |
| `DECAY` | Weighted decay: `t + 0.8·lag₁ + 0.6·lag₂` / 加权衰减 |
| `DELAY1` | 1-period lag / 滞后 1 期 |
| `MAX3` | Max of current, lag₁, and lag₂ / 当前、滞后1期、滞后2期取最大值 |

## Features (Factors) / 特征（因子）

### Basic Factors (6) / 基础因子（6 个）

| Factor 因子 | Description 描述 |
| --- | --- |
| `ret` | Log returns / 对数收益 |
| `liq_score` | Liquidity / FDV health score / 流动性/FDV 健康度 |
| `pressure` | Buy/sell imbalance pressure / 买卖力量不平衡 |
| `fomo` | Volume acceleration (FOMO indicator) / 成交量加速度（FOMO 指标） |
| `dev` | Price deviation from moving average / 价格偏离均值 |
| `log_vol` | Log trading volume / 对数成交量 |

### Advanced Factors (12) / 高级因子（12 个）

The basic 6 plus: / 基础 6 个加：

| Factor 因子 | Description 描述 |
| --- | --- |
| `vol_cluster` | Volatility clustering / 波动率聚集 |
| `momentum_rev` | Momentum reversal signal / 动量反转信号 |
| `rel_strength` | Relative strength (RSI-like) / 相对强弱（RSI 类） |
| `hl_range` | High-low price amplitude / 高低价振幅 |
| `close_pos` | Close position within period range / 收盘在区间位置 |
| `vol_trend` | Volume trend indicator / 成交量趋势 |

## Getting Started / 快速开始

### Prerequisites / 环境要求

- **Python 3.10+** (3.11 recommended / 推荐 3.11)
- **PostgreSQL** (or TimescaleDB) instance / 实例
- **Birdeye API key** / API 密钥 ([birdeye.so](https://birdeye.so))
- **Solana RPC endpoint** / RPC 节点 (e.g., QuickNode, Helius)
- **Solana wallet** with private key / 带私钥的钱包 (for live trading / 用于实盘交易)

### Installation / 安装

```bash
# Clone the repository / 克隆仓库
git clone https://github.com/imbue-bit/AlphaGPT.git
cd AlphaGPT

# Install core dependencies / 安装核心依赖
pip install -r requirements.txt

# (Optional) Install dependencies for experimental scripts / （可选）安装实验脚本依赖
pip install -r requirements-optional.txt
```

### Configuration / 配置

Create a `.env` file in the project root: / 在项目根目录创建 `.env` 文件：

```bash
# Database / 数据库
DB_USER=postgres
DB_PASSWORD=your_password
DB_HOST=localhost
DB_PORT=5432
DB_NAME=crypto_quant

# Birdeye API / Birdeye 接口
BIRDEYE_API_KEY=your_api_key_here

# Solana RPC / Solana RPC 节点
QUICKNODE_RPC_URL=https://your-rpc-endpoint

# Wallet (base58 private key or JSON byte array) / 钱包（base58 私钥或 JSON 字节数组）
SOLANA_PRIVATE_KEY=your_private_key_here
```

Tunable parameters are available in each module's `config.py`: / 各模块可调参数见其 `config.py`：

- [data_pipeline/config.py](data_pipeline/config.py) — `MIN_LIQUIDITY_USD`, `MIN_FDV`, `TIMEFRAME`, `HISTORY_DAYS`
- [model_core/config.py](model_core/config.py) — `BATCH_SIZE`, `TRAIN_STEPS`, `MAX_FORMULA_LEN`
- [strategy_manager/config.py](strategy_manager/config.py) — `MAX_OPEN_POSITIONS`, `ENTRY_AMOUNT_SOL`, `STOP_LOSS_PCT`

### Usage / 使用

Each component runs independently: / 各组件可独立运行：

```bash
# 1. Data Pipeline — fetch and store market data / 数据管线——抓取并存储行情数据
python -m data_pipeline.run_pipeline

# 2. Model Training — discover trading formulas / 模型训练——发现交易公式
python -m model_core.engine
# → produces best_meme_strategy.json and training_history.json / 生成最优策略和训练历史

# 3. Live Trading — execute the best formula / 实盘交易——执行最优公式
python -m strategy_manager.runner

# 4. Dashboard — monitor in real-time / 看板——实时监控
streamlit run dashboard/app.py

# 5. Research — run LoRD experiments / 研究——运行 LoRD 实验
python lord/experiment.py --mode mechanism
python lord/experiment.py --mode phase_diagram
```

## How It Works / 工作原理

### Phase 1: Formula Mining (Offline) / 第一阶段：因子挖掘（离线）

1. **Data Loading** — Market data is loaded from PostgreSQL into tensors. Feature engineers compute basic and advanced factors.
   **数据加载** — 从 PostgreSQL 将行情数据加载为张量。特征工程计算基础与高级因子。
2. **Formula Generation** — The Transformer model autoregressively generates token sequences. Each token is either a **feature** (data input) or an **operator** (mathematical function).
   **公式生成** — Transformer 模型自回归地生成 token 序列。每个 token 要么是**特征**（数据输入），要么是**算子**（数学函数）。
3. **VM Execution** — The StackVM interprets each formula against the full feature tensor batch, producing a signal time series for every token.
   **VM 执行** — StackVM 对全量特征张量批次逐一解释执行公式，为每个代币生成信号时间序列。
4. **Backtesting** — The backtest engine simulates trading: entry/exit at signal thresholds, accounting for slippage, turnover, and drawdown. The Sortino ratio is the primary score.
   **回测** — 回测引擎模拟交易：在信号阈值触发买卖，计入滑点、换手和回撤。Sortino 比率作为主要评分指标。
5. **Policy Gradient** — REINFORCE updates the model parameters: formulas that score well get reinforced, poor performers get suppressed.
   **策略梯度** — REINFORCE 更新模型参数：高分公式被强化，低分公式被抑制。
6. **Iteration** — Steps 2–5 repeat for `TRAIN_STEPS` iterations. The best formula is saved to `best_meme_strategy.json`.
   **迭代** — 重复步骤 2–5 共 `TRAIN_STEPS` 轮。最优公式保存至 `best_meme_strategy.json`。

### Phase 2: Live Scoring / 第二阶段：实时打分

1. The saved formula is loaded from `best_meme_strategy.json`.
   从 `best_meme_strategy.json` 加载已保存的公式。
2. Current market data for the top N tokens is fetched and featurized.
   获取 Top N 代币的当前行情数据并进行特征化。
3. The StackVM executes the formula against live data, producing a score for each token.
   StackVM 对实时数据执行公式，为每个代币产出一个分数。

### Phase 3: Trade Execution / 第三阶段：交易执行

1. Tokens scoring above `BUY_THRESHOLD` are evaluated by the risk engine.
   评分超过 `BUY_THRESHOLD` 的代币交由风控引擎评估。
2. Pre-trade checks: liquidity floor, honeypot detection, position count limits.
   交易前检查：流动性门槛、蜜罐检测、仓位数量限制。
3. Buy orders are executed via Jupiter aggregator on Solana.
   通过 Jupiter 聚合器在 Solana 上执行买入订单。
4. Positions are monitored on a 60-second cycle for exit conditions: stop-loss, trailing stop, take-profit, or score drop below `SELL_THRESHOLD`.
   以 60 秒周期监控持仓退出条件：止损、移动止盈止损、止盈、或分数跌破 `SELL_THRESHOLD`。

## Design Philosophy / 设计哲学

The system maintains a clean separation between **strategy research** and **trade execution**:

系统在**策略研究**与**交易执行**之间保持清晰的分离：

- `model_core/` is purely research — it knows nothing about Solana, live trading, or portfolio management. It consumes data, produces formulas.
  `model_core/` 纯研究——对 Solana、实盘交易或持仓管理一无所知。消费数据，生产公式。
- `strategy_manager/` is purely execution — it consumes a formula JSON, produces trades. It knows nothing about Transformers or training.
  `strategy_manager/` 纯执行——消费公式 JSON，产出交易。对 Transformer 或训练一无所知。
- `best_meme_strategy.json` is the contract between the two layers.
  `best_meme_strategy.json` 是两层之间的合约。

This means you can: / 这意味着你可以：

- Train formulas on any data source (swap out `data_pipeline/`)
- 在任何数据源上训练公式（替换 `data_pipeline/`）
- Execute formulas on any venue (swap out `execution/`)
- 在任何交易场所执行公式（替换 `execution/`）
- Experiment with the model architecture without touching execution code
- 在不触及执行代码的情况下实验模型架构
- Run backtests independently of the live system
- 独立于实盘系统运行回测

## Complementary Research / 相关研究

For those trading in crypto markets: / 加密市场交易者另可参考：

> **Defense in Predatory Markets: A Differential Game Framework for AMM Liquidity via Uniswap V4 Hooks** — see [github.com/imbue-bit/no_JIT](https://github.com/imbue-bit/no_JIT)
>
> **防御掠夺性市场：基于 Uniswap V4 Hooks 的 AMM 流动性微分博弈框架** — 详见 [github.com/imbue-bit/no_JIT](https://github.com/imbue-bit/no_JIT)
>
> A game-theoretic approach to liquidity provision on Uniswap V4. Contact: imbue2025@outlook.com
>
> 一种基于博弈论的 Uniswap V4 流动性提供方法。联系：imbue2025@outlook.com

## ⚠️ Disclaimer / 免责声明

**This software is for research and educational purposes only.** Cryptocurrency trading involves substantial risk of loss and is not suitable for all investors. Past performance of discovered strategies does not guarantee future results.

**本软件仅用于研究和教育目的。** 加密货币交易涉及重大亏损风险，并不适合所有投资者。所发现策略的历史表现不保证未来结果。

- The system may contain bugs that could result in financial loss.
- 系统可能存在导致财务损失的缺陷。
- Live trading with this codebase should only be done with capital you can afford to lose.
- 使用本代码库进行实盘交易，应仅使用可承受全部亏损的资金。
- The authors assume no responsibility for any financial losses incurred through the use of this software.
- 作者对因使用本软件而产生的任何财务损失不承担任何责任。
- Always test thoroughly on devnet/testnet before deploying with real funds.
- 在使用真实资金部署前，请务必在 devnet/testnet 上充分测试。

## Community / 社群

For discussion and community, join the QQ group: **1076893473**

交流与社群，请加入 QQ 群：**1076893473**

> **Note:** GitHub Issues are not a check-in or social tool. Issues created solely for "checking in" or "leaving a mark" will be removed.
>
> **注意：** GitHub Issues 并非打卡或社交工具。仅用于「打卡」「留名」的 Issue 将被直接删除。

## License / 许可证

This project is licensed under the Apache License 2.0 — see [LICENSE](LICENSE) for details.

本项目基于 Apache License 2.0 许可——详见 [LICENSE](LICENSE)。

## Star History / Star 历史

[![Star History Chart](https://api.star-history.com/svg?repos=imbue-bit/AlphaGPT&type=date&legend=top-left)](https://www.star-history.com/#imbue-bit/AlphaGPT&type=date&legend=top-left)
