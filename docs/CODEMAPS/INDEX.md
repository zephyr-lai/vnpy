# vnpy Codemap Index

**Last Updated:** 2026-09-30
**Based on:** vnpy 4.4.0 (`fa5206fe`)
**Generator:** ecc codemap 约定（Python 适配版 — 原版 TS/Web 分区正则不适用本仓）

## Areas

| Area | 描述 | Files | Lines | Doc |
|------|------|-------|-------|-----|
| Core Trader | 引擎中枢、网关/应用契约、数据对象、GUI | 22 | ~6491 | [core-trader.md](core-trader.md) |
| Event Engine | 全局事件总线 | 2 | 153 | [event-engine.md](event-engine.md) |
| Chart | pyqtgraph K 线图组件 | 6 | ~1133 | [chart.md](chart.md) |
| RPC | ZeroMQ 进程间 RPC | 4 | ~327 | [rpc.md](rpc.md) |
| Alpha | 4.4.0 新增多因子研究框架 | 25 | ~4680 | [alpha.md](alpha.md) |

> 行数为 `wc -l` 实测（不含 locale/资源文件）。总计约 12,784 行 Python。

## Package Dependency Flow

```
                  ┌───────────────┐
                  │    trader     │  (core)
                  └──┬────┬────┬──┘
        imports      │    │    │
        ┌────────────┘    │    └─────────────┐
┌───────▼──────┐  ┌───────▼───────┐  ┌──────▼───────┐
│    event     │  │     chart     │  │     rpc      │
│ (事件总线,无依赖)│  │ (依赖 trader  │  │ (仅 pyzmq)   │
└──────────────┘  │  object/常量)  │  └──────────────┘
                  └───────────────┘
                  ┌───────────────┐
                  │     alpha     │  (独立研究框架, 不依赖 trader)
                  └───────────────┘
```

`vnpy/event` 是唯一无内部依赖的底座包；`vnpy/alpha` 是独立的研究栈（自带的回测/模型/数据集）。

## Related Docs

- [Onboarding Guide](../ONBOARDING.md)
- [架构图](../architecture/vnpy-architecture.html)
