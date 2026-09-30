# Event Engine Codemap

**Last Updated:** 2026-09-30
**Total Files:** 2
**Total Lines:** 153

## Entry Points

- `vnpy/event/engine.py` — `EventEngine`、`Event`、`EVENT_TIMER`
- `vnpy/event/__init__.py` — 导出上述三者

## Architecture

```
event/
└── engine.py    # 145 行，整个包就这一个实现文件
```

## Key Modules

| File | Lines | 职责 |
|------|-------|------|
| `vnpy/event/engine.py` | 145 | `Event`(16)：type+data 载荷；`EventEngine`(33)：`_queue` + 分发线程 + `_timer` 秒级定时线程 |

## Data Flow

```
put(event) ──▶ queue ──▶ 分发线程取出 ──▶ 按 event.type 精确回调
                                   └────▶ general 回调（全部事件）
timer 线程 ──每秒──▶ Event(EVENT_TIMER) ──▶ put()
```

- `register(type, handler)` / `unregister`：精确类型订阅
- `register_general(handler)`：无差别订阅全部事件
- `start()` 启动分发 + 定时两个线程；`stop()` 停止并 join

## 设计要点

- **零内部依赖**：整个包只用标准库（`queue/threading/time`），是 vnpy 的底座
- 事件类型名带点号（如 `"eTick."`）由上层约定用于前缀子事件匹配，本包不做前缀解析
- `interval: int = 1`：定时器周期（秒）

## External Dependencies

无（纯标准库）。
