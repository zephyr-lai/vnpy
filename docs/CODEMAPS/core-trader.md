# Core Trader Codemap

**Last Updated:** 2026-09-30
**Total Files:** 22
**Total Lines:** ~6491

## Entry Points

> Python 包无 index/main 约定；以下为对外契约入口。

- `vnpy/trader/engine.py` — `MainEngine`（一切装配的起点）
- `vnpy/trader/gateway.py` — `BaseGateway`（网关 ABC，生态包继承）
- `vnpy/trader/app.py` — `BaseApp`（应用 ABC，生态包继承）
- `examples/no_ui/run.py` — 最小装配示例

## Architecture

```
trader/
├── engine.py        # MainEngine + LogEngine + OmsEngine + EmailEngine + WechatEngine
├── gateway.py       # BaseGateway ABC
├── app.py           # BaseApp ABC
├── object.py        # 数据对象 + 请求对象
├── constant.py      # Direction/Offset/Status/OrderType/Exchange 等枚举
├── event.py         # EVENT_* 事件名常量（"eXxx." 带点号前缀）
├── converter.py     # OffsetConverter：开平仓 ↔ 净持仓转换
├── database.py      # BaseDatabase ABC + get_database()
├── datafeed.py      # BaseDatafeed ABC + get_datafeed()
├── utility.py       # BarGenerator / ArrayManager（K线合成与指标缓存）
├── optimize.py      # 优化器（多进程暴力/遗传算法）
├── logger.py        # loguru 封装
├── setting.py       # 全局 SETTINGS dict
├── wechat.py        # 企业微信/Server 酱通知
└── ui/              # PySide6
    ├── mainwindow.py  # VeighNa Trader 主窗口（菜单挂载 App widget）
    ├── widget.py      # 全部基础表格/监控组件（1710 行，最大文件）
    └── qt.py          # QApplication 封装与全局字体/主题
```

## Key Modules

| File | Lines | 职责 |
|------|-------|------|
| `vnpy/trader/ui/widget.py` | 1710 | 全部 Qt 监控组件（Tick/Order/Trade/Position/Account 表格等） |
| `vnpy/trader/utility.py` | 1281 | `BarGenerator`（Tick→1min→Xmin→小时/日线合成）、`ArrayManager`（环形缓冲指标计算） |
| `vnpy/trader/engine.py` | 838 | `MainEngine`(81) / `LogEngine`(325) / `OmsEngine`(360) / `EmailEngine`(590) / `WechatEngine`(657) |
| `vnpy/trader/object.py` | 427 | 数据对象：`TickData`(30) `BarData`(88) `OrderData`(112) `TradeData`(154) `PositionData`(179) `AccountData`(201) `ContractData`(233)；请求：`SubscribeRequest`(307) `OrderRequest`(321) `CancelRequest`(359) `HistoryRequest`(374) |
| `vnpy/trader/converter.py` | 402 | `OffsetConverter`：上期所平今/平昨等交易所级开平仓差异处理 |
| `vnpy/trader/gateway.py` | 272 | `BaseGateway`(33)：`on_tick/on_order/on_trade/on_position/on_account` 推事件 |
| `vnpy/trader/optimize.py` | 250 | `run_bf_optimization` / `run_ga_optimization`（multiprocessing + deap） |
| `vnpy/trader/constant.py` | 160 | `Direction/Offset/Status/OrderType/Exchange` 枚举 |
| `vnpy/trader/database.py` | 159 | `BaseDatabase` ABC + `get_database()`（默认 `database_sqlite` 生态包） |
| `vnpy/trader/datafeed.py` | 68 | `BaseDatafeed` ABC（行情下载适配） |

## Data Flow

```
生态包 Gateway ──on_tick/on_order──▶ EventEngine.put ──▶ OmsEngine 缓存
      ▲                                                        │
      │ subscribe/send_order/cancel_order (MainEngine 按名路由)   │ EVENT_* 事件
      │                                                        ▼
      └────────────────────────────────────── App 引擎 / UI widget 回调
```

## External Dependencies

`PySide6`（ui/）、`numpy`+`ta-lib`（utility.py 指标）、`deap`（optimize.py）、`loguru`（logger.py）、`requests`+`qrcode`（wechat.py）
