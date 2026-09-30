# Chart Codemap

**Last Updated:** 2026-09-30
**Total Files:** 6
**Total Lines:** ~1133

## Entry Points

- `vnpy/chart/__init__.py` — 导出 `ChartWidget`、`BarManager`、`CandleItem` 等
- `examples/candle_chart/` — 独立运行示例

## Architecture

```
chart/
├── widget.py   # ChartWidget：pyqtgraph PlotWidget 封装（十字光标/缩放/双图区）
├── item.py     # CandleItem / VolumeItem 等绘图 Item
├── manager.py  # BarManager：K线数据缓存
├── axis.py     # DatetimeAxis（索引→时间轴标签）
└── base.py     # 图表相关协议/类型
```

## Key Modules

| File | Lines | 职责 |
|------|-------|------|
| `vnpy/chart/widget.py` | 556 | `ChartWidget`：主图/副图、鼠标交互、动态加载历史区间 |
| `vnpy/chart/item.py` | 333 | `CandleItem`(蜡烛) / `VolumeItem`(成交量) 绘制 |
| `vnpy/chart/manager.py` | 170 | `BarManager`：内存中 BarData 有序缓存，供 item 查询 |
| `vnpy/chart/axis.py` | 44 | 索引轴 → 日期时间标签映射 |

## Data Flow

```
历史K线(数据库/datafeed) ──▶ BarManager.load_history ──▶ CandleItem.updateData
实时Tick ──▶ trader/utility.BarGenerator 合成 ──▶ BarManager.update_bar ──▶ 增量重绘
```

## External Dependencies

`pyqtgraph`（绘制）、`PySide6`（Qt 绑定）；依赖 `vnpy.trader.object.BarData` 与 `vnpy.trader.utility.BarGenerator`。
