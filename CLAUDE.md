# vnpy — Claude Code 项目指南

> 本文件由 ecc codebase-onboarding 生成（2026-09-30），基于 vnpy 4.4.0。

## 项目定位

VeighNa 量化交易框架核心包：事件引擎 + 主引擎 + 网关/应用插件契约。
交易接口（CTP 等）和应用（CTA 等）在**独立生态包**中，本仓只定义契约。

## Tech Stack

- Python >=3.10，完整类型注解（`py.typed`），mypy `disallow_untyped_defs`
- PySide6（GUI）、pyqtgraph（K线）、numpy/pandas/ta-lib、pyzmq（RPC）、deap（优化）
- 构建：hatchling + babel；Lint：ruff（B/E/F/UP/W，忽略 E501）

## 核心契约（不得随意改动签名）

| 契约 | 位置 |
|------|------|
| `EventEngine`：`put/register/start`，`EVENT_TIMER` 秒级定时 | `vnpy/event/engine.py` |
| `MainEngine`：`add_gateway/add_app/add_engine`，订单路由 | `vnpy/trader/engine.py` |
| `BaseGateway`（ABC）：`connect/subscribe/send_order/close` + `on_tick/on_order/on_trade` | `vnpy/trader/gateway.py` |
| `BaseApp`（ABC）：`app_name/app_module/engine_class/widget_name` | `vnpy/trader/app.py` |
| 数据对象：`TickData/BarData/OrderData/TradeData/PositionData/AccountData/ContractData` | `vnpy/trader/object.py` |

改动以上任一签名 = 生态包（vnpy_ctp、vnpy_ctastrategy 等数百个）破坏性变更。

## 事件流

网关回报 → `gateway.on_*()` → `EventEngine.put()` → 各引擎/widget `register` 回调。
事件名带点号（`EVENT_TICK = "eTick."`），前缀匹配实现子事件（如 `eTick.IF2112`）。

## 代码风格

- 模块 snake_case、类 PascalCase；事件常量 `EVENT_X = "eX."`
- 用户可见字符串必须 `tr()` 包裹（babel i18n，locale 在 `vnpy/trader/locale/`）
- 引擎类继承 `BaseEngine`，由 `MainEngine.add_engine()` 挂载

## 命令

```bash
pip install -e .[dev]   # 安装
ruff check vnpy         # lint
mypy vnpy               # 类型检查
pytest tests/           # 测试（覆盖极薄：仅 alpha 模块）
python -m build         # 构建
```

## 注意事项

- `vnpy/trader/utility.py`（BarGenerator/ArrayManager）是最大模块，改动需同步 `chart/manager.py`
- `tests/` 几乎为空：改核心契约无回归网，需手动验证 examples/
- commit 前缀约定：`[Add] / [Mod] / [Del] / [Fix]`
