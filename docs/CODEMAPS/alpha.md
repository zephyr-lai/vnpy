# Alpha Codemap

**Last Updated:** 2026-09-30
**Total Files:** 25
**Total Lines:** ~4680
**Note:** 4.4.0 新增模块 — 多因子/ML 研究框架，**不依赖** `vnpy.trader`，是独立研究栈。

## Entry Points

- `vnpy/alpha/lab.py` — `AlphaLab`：研究工作流入口（数据下载→加工→IC→训练→回测）
- `vnpy/alpha/strategy/strategies/equity_demo_strategy.py` — 示例策略
- `tests/test_alpha101.py` / `tests/alpha/test_dataproxy.py` — 本仓仅有的测试

## Architecture

```
alpha/
├── lab.py               # AlphaLab：一站式研究编排（480 行）
├── logger.py            # 研究日志
├── dataset/             # 因子表达式与数据处理
│   ├── template.py      # AlphaDataset 模板（train/valid/test 划分）
│   ├── ts_function.py   # 时序算子（329 行）
│   ├── cs_function.py   # 截面算子
│   ├── ta_function.py   # 技术指标算子
│   ├── math_function.py # 基础数学算子（167 行）
│   ├── processor.py     # CSZScore/RobustZScore/Fillna 等处理器
│   ├── utility.py       # 数据加载/对齐工具（285 行）
│   └── datasets/
│       ├── alpha_101.py # WorldQuant Alpha101 因子集（330 行）
│       └── alpha_158.py # Alpha158 因子集（qlib 风格，130 行）
├── model/
│   ├── template.py      # AlphaModel 模板
│   └── models/
│       ├── mlp_model.py # PyTorch MLP（683 行）
│       ├── lgb_model.py # LightGBM（170 行）
│       └── lasso_model.py # Lasso 回归（139 行）
└── strategy/
    ├── template.py      # AlphaStrategy 模板（205 行）
    ├── backtesting.py   # 向量化回测引擎（944 行，本包最大文件）
    └── strategies/
        └── equity_demo_strategy.py
```

## Key Modules

| File | Lines | 职责 |
|------|-------|------|
| `vnpy/alpha/strategy/backtesting.py` | 944 | `AlphaBacktester`：按 IC 加权组合预测，向量化模拟调仓与绩效 |
| `vnpy/alpha/model/models/mlp_model.py` | 683 | `MlpModel`：PyTorch 多层感知机（训练/预测/保存） |
| `vnpy/alpha/lab.py` | 480 | `AlphaLab`：数据下载→表达式加工→划分→因子 IC 分析→模型训练→回测 全流程编排 |
| `vnpy/alpha/dataset/datasets/alpha_101.py` | 330 | Alpha101 因子表达式集 |
| `vnpy/alpha/dataset/ts_function.py` | 329 | 时序因子算子库 |
| `vnpy/alpha/dataset/template.py` | 305 | `AlphaDataset`：polars 惰性表达式 + 数据集划分 |

## Data Flow

```
数据源 ──▶ AlphaLab.download_bar_data / download_component_data
      ──▶ AlphaDataset(表达式算子加工, polars 惰性计算)
      ──▶ train/valid/test 划分 ──▶ AlphaModel.fit（LGB/MLP/Lasso）
      ──▶ 预测 IC/IR 评估 ──▶ AlphaBacktester 组合回测 ──▶ 绩效报告
```

## External Dependencies

`polars`（惰性 DataFrame）、`lightgbm`、`torch`、`scipy`、`plotly`（报告）、`alphalens-reloaded`（IC 分析）— 均在 `pyproject.toml` 的 `alpha` extra 中：`pip install vnpy[alpha]`。

## Testing

本仓仅有的两个测试都在这里：`tests/test_alpha101.py`（因子表达式正确性）、`tests/alpha/test_dataproxy.py`。
