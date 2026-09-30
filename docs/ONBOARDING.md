# Onboarding Guide: vnpy (VeighNa 核心框架)

> 生成于 2026-09-30，基于 vnpy 4.4.0（`fa5206fe`）。
> 配套文档：[架构图](architecture/vnpy-architecture.html) · [Codemaps](CODEMAPS/INDEX.md)

## Overview

vnpy 是 VeighNa 量化交易框架的**核心包**：提供事件驱动引擎、主引擎、交易网关抽象、OMS（订单管理系统）与 GUI 基础组件。
具体的交易接口（CTP 等）与交易应用（CTA 策略等）**不在本仓内**，而是以独立 pip 包分布在 VeighNa 生态中（`vnpy_ctp`、`vnpy_ctastrategy` …），通过本仓定义的 `BaseGateway` / `BaseApp` 契约挂载。

## Tech Stack

| 层 | 技术 | 版本 |
|-------|-----------|---------|
| 语言 | Python（Typed，含 `py.typed`） | >=3.10（支持 3.13） |
| GUI | PySide6 + pyqtgraph + qdarkstyle | 6.8.x |
| 数据 | numpy / pandas / ta-lib | — |
| 策略优化 | deap（遗传算法）+ multiprocessing | — |
| 进程间通信 | pyzmq（`vnpy.rpc`） | 26.x |
| 可视化 | plotly（alpha 模块） | 6.x |
| 构建 | hatchling + babel（i18n 编译钩子） | — |
| 质量 | ruff（B/E/F/UP/W）+ mypy（disallow_untyped_defs） | — |

## Architecture

事件驱动 + 插件化，核心契约三件套：

```
                    ┌────────────────────────────┐
                    │         MainEngine         │  vnpy/trader/engine.py
                    │  (网关/引擎/应用的组装中枢)   │
                    └──────┬──────────────┬──────┘
              add_gateway  │              │  add_app / add_engine
                    ┌──────▼──────┐  ┌────▼─────────────┐
                    │ BaseGateway │  │ OmsEngine 等引擎  │
                    │ (生态包实现)  │  │ (缓存 order/trade │
                    └──────┬──────┘  │  /position/account)│
              on_tick/on_order/on_trade   └────▲─────────┘
                    ┌──────▼──────┐           │ register(EVENT_*)
                    │ EventEngine │───────────┘  vnpy/event/engine.py
                    │ (队列+定时器) │
                    └─────────────┘
```

- **EventEngine**（`vnpy/event/engine.py`）：全局事件总线。`put()` 入队，后台线程分发，`register(type, handler)` 订阅；内置 1 秒定时器事件（`EVENT_TIMER`）驱动轮询逻辑。
- **MainEngine**（`vnpy/trader/engine.py:81`）：中枢。持有全部 gateway/engine/app；`subscribe` / `send_order` / `cancel_order` 按名字路由到对应网关。
- **BaseGateway**（`vnpy/trader/gateway.py:33`）：柜台/交易所抽象（ABC）。生态包实现 `connect/subscribe/send_order/close`，收到回报后调 `on_tick/on_order/on_trade` 推入事件总线。
- **OmsEngine**（`vnpy/trader/engine.py:360`）：订阅所有 `EVENT_*`，维护全量订单/成交/持仓/资金/合约缓存，供查询。
- **BaseApp**（`vnpy/trader/app.py`）：应用插件契约（`app_name` + `engine_class` + `widget_name`），如 `vnpy_ctastrategy`。MainEngine 启动其 engine_class，GUI 菜单挂其 widget。

## Key Entry Points

- **事件总线**：`vnpy/event/engine.py` — `EventEngine`、`Event`、`EVENT_TIMER`
- **引擎中枢**：`vnpy/trader/engine.py` — `MainEngine`、`OmsEngine`、`LogEngine`、`EmailEngine`、`WechatEngine`
- **网关契约**：`vnpy/trader/gateway.py` — `BaseGateway`（含 REST 轮询/WS 模板思路）
- **应用契约**：`vnpy/trader/app.py` — `BaseApp`
- **数据模型**：`vnpy/trader/object.py` — `TickData/BarData/OrderData/TradeData/PositionData/AccountData/ContractData` + `SubscribeRequest/OrderRequest/CancelRequest/HistoryRequest`
- **枚举常量**：`vnpy/trader/constant.py` — `Direction/Offset/Status/OrderType/Exchange`
- **事件名**：`vnpy/trader/event.py` — `EVENT_TICK = "eTick."`（**带点号**，前缀匹配支持 `eTick.IF2112` 子事件）
- **工具箱**：`vnpy/trader/utility.py`（1281 行，最大文件）— `BarGenerator`（K线合成）、`ArrayManager`（指标缓存）
- **GUI 入口**：`vnpy/trader/ui/mainwindow.py` — VeighNa Trader 主窗口
- **无 UI 启动示例**：`examples/no_ui/run.py` — 事件引擎 + 主引擎 + 网关的最小装配

## Directory Map

```
vnpy/                    # Python 包（与仓同名）
├── trader/              # 核心：引擎、网关契约、数据对象、常量、UI、i18n locale
│   └── ui/              # PySide6 组件（mainwindow/widget/qt）
├── event/               # EventEngine（仅 engine.py 一个文件）
├── chart/               # pyqtgraph K线图组件（widget/item/manager）
├── rpc/                 # ZeroMQ RPC（server/client，用于分布式部署）
└── alpha/               # 4.4.0 新增：多因子研究（lab/dataset/model/strategy）
examples/                # 各场景启动脚本（no_ui / veighna_trader / cta_backtesting …）
tests/                   # 仅 alpha 相关（test_alpha101 / test_dataproxy）
docs/                    # Sphinx 文档（community/elite/fusion）
```

## Event Flow（一笔 CTA 委托的生命周期）

1. 策略（生态包 `vnpy_ctastrategy`）调用其引擎 → `MainEngine.send_order(OrderRequest, gateway_name)`
2. MainEngine 查找网关 → `BaseGateway.send_order(req)` 发往柜台
3. 柜台回报 → 网关调 `on_order(OrderData)` / `on_trade(TradeData)` → `EventEngine.put()`
4. `OmsEngine` 收到 `EVENT_ORDER/EVENT_TRADE` 更新缓存；CtaEngine 收到后回调策略 `on_order/on_trade`
5. 若 UI 打开，对应 widget 监听同一事件刷新表格
6. 开平仓的净持仓转换由 `vnpy/trader/converter.py`（`OffsetConverter`）在引擎层完成

## Conventions

- **命名**：模块 snake_case、类 PascalCase；事件常量 `EVENT_X = "eX."`（点号 + 前缀 = 子事件机制）
- **类型**：全部公开 API 带注解；mypy 强制 `disallow_untyped_defs`
- **i18n**：用户可见字符串一律 `tr("...")` 包裹，`.po` 位于 `vnpy/trader/locale/`，由 hatch 构建钩子编译
- **Git**：commit 前缀 `[Add] / [Mod] / [Del] / [Fix]`
- **测试**：核心包测试覆盖极薄（仅 alpha），生态组件各自负责测试 —— 改核心契约时注意无回归网

## Common Tasks

```bash
# 安装（开发模式）
pip install -e .[dev]          # 或 ./install.sh
# Lint / 格式
ruff check vnpy && ruff format --check vnpy
# 类型检查
mypy vnpy                      # 需先装 pandas-stubs 等 dev 依赖
# 构建 wheel
python -m build
# 跑测试
pytest tests/
```

## Where to Look

| 我想… | 看… |
|--------------|-----------|
| 接入新交易所/柜台 | 实现 `BaseGateway` 子类（建议独立生态包），`on_*` 推事件 |
| 写一个交易应用 | 实现 `BaseApp` + `engine_class`，参考 `examples/notebook_trading/` |
| 理解委托/持仓流转 | `trader/object.py` + `trader/converter.py` + `trader/engine.py:OmsEngine` |
| K线合成、技术指标 | `trader/utility.py`（`BarGenerator` / `ArrayManager`） |
| 分布式/远程交易 | `vnpy/rpc/`（ZeroMQ 封装）+ `examples/simple_rpc/` |
| 多因子研究（新） | `vnpy/alpha/` + `tests/alpha/` |
