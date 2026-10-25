# Qlib 架构笔记

面向**有 Python 经验、第一次读 Qlib** 的工程师：一套从"框架长什么样"到"怎么搭自己的系统"的文档。

- **分析对象**：`F:\code\qlib`，Qlib `v0.9.8`，commit `54355232`（2026-10-08）
- **规模**：`qlib/` 下 11 个子包、约 349 个 `.py`
- **约定**：`路径:行号` 均指该 commit；正文用 【源码事实】/【设计评述】/【我的建议】 区分确定性；代码块标注【示意代码】的为骨架，不是仓库原文
- **语言**：中文正文，术语保留英文原词

## 阅读顺序

| # | 文档 | 一句话 | 建议用时 |
|---|---|---|---|
| 01 | [全景与设计哲学](01-overview.md) | 六层架构、12 个核心抽象、五条设计取舍 | 40 min |
| 02 | [分层与调用链](02-layering.md) | 一条 yaml 从 `qrun` 跑到 artifacts 的完整时序 | 40 min |
| 03 | [数据层](03-data-layer.md) | 二进制列存、五类 Provider、两层缓存、PIT | 50 min |
| 04 | [表达式引擎](04-expression.md) | `$close` 如何被解析成树、如何求值、如何缓存 | 40 min |
| 05 | [数据集层](05-dataset.md) | Loader → Handler → Dataset → TSDataSampler | 40 min |
| 06 | [模型层](06-model.md) | 两个方法的契约、Trainer 家族、滚动训练 | 35 min |
| 07 | [策略与回测](07-strategy-backtest.md) | 决策/执行/撮合/账户四层分离、`shift=1` | 60 min |
| 08 | [工作流](08-workflow.md) | Recorder 与 MLflow、RecordTemp 依赖链 | 35 min |
| 09 | [在线与 RL](09-rl-online.md) | 滚动更新、client/server、RL 环境 | 30 min |
| 10 | [落地篇](10-extend.md) | 10 个扩展点、六步最小系统、反模式 | 45 min |
| 11 | [导读](11-reading-path.md) | 三条阅读路径、调试手法、排错索引、术语表 | 20 min |

**赶时间**：01 → 02 → 07 → 10。
**要动手**：按 11 章的"路径 C"七天计划走。

## 一张图记住它

```
yaml ──qrun──▶ qlib.init(C 注册 Provider) ──▶ Dataset(Loader→Handler→Processor)
                                                   │
                                                   ▼
                                              Model.fit/predict
                                                   │  pred.pkl
                                                   ▼
                    Strategy ──TradeDecision──▶ Executor ──Order──▶ Exchange
                                                                      │
                                              Account/Position ◀──────┘
                                                   │
                                                   ▼
                                        PortfolioMetrics → Recorder(mlruns)
```

## 你可能最关心的三个问题

1. **我的数据怎么进来？** 优先按 `scripts/dump_bin.py` 的格式落盘（`features/{inst}/{field}.{freq}.bin`）；已经有宽表就直接 `StaticDataLoader`。见 03、10 章。
2. **我的模型怎么插进去？** 继承 `Model`，实现 `fit(Dataset)` / `predict(Dataset, segment)`，`fit` 用 `DK_L`、`predict` 用 `DK_I`。见 06 章。
3. **回测结果可信吗？** 逐条核对 07 章 7.7 的八项检查（`shift`、`deal_price`、涨跌停、成本、成交量、整手、停牌估值、基准）。

## 说明

本目录由一次对源码的系统性阅读生成，未改动仓库任何代码。若你升级 Qlib 版本，优先复核三类锚点：`qlib/cli/run.py`、`qlib/backtest/executor.py`、`qlib/workflow/record_temp.py`——这三处变动最频繁，且最容易让文档里的行号漂移。
