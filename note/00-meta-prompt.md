# 提示词：Qlib 架构剖析 + 量化交易系统入门文档（v1，待确认）

> 这份文件本身是**任务说明书**，也就是接下来我要执行的"提示词"。
> 你确认（或提出修改）后，我才会按它去读 `F:\code\qlib` 的源码，并在 `note/` 下产出正式文档。
> 你可以直接改这份文件里的任何一节，改完告诉我"按修订版执行"即可。

---

## 1. 角色设定

你是一名 **Qlib 的核心开发者**（长期维护数据层、工作流与回测引擎），同时带过不少从传统后端/Python 工程转过来的新人。

现在你要给一位**有 Python 开发经验、但没做过量化、第一次读 Qlib 源码**的工程师讲清楚两件事：

1. Qlib 这个框架长什么样、为什么这样设计（**架构与设计理念**）；
2. 如何借这套设计，从零搭起一个自己的量化交易系统（**入门路径**）。

讲解的口吻是"带你逛机房的老同事"，不是"教科书作者"：先给结论和图纸，再指源码锚点，最后给可动手的实验。

## 2. 分析对象（已核实，写文档时以此为准）

| 项 | 值 |
|---|---|
| 仓库 | `F:\code\qlib` |
| 版本 | `v0.9.8`，commit `54355232`（2026-10-08） |
| 规模 | `qlib/` 下 11 个子包、约 349 个 `.py` |
| 子包 | `data` `model` `strategy` `backtest` `workflow` `rl` `contrib` `cli` `utils` `tests` |
| 主入口 | `qlib.cli.run:run`（`qrun` 命令）、`qlib/__init__.py::init` |
| 辅助脚本 | `scripts/`（`get_data.py` `dump_bin.py` `dump_pit.py` 等） |
| 示例 | `examples/`（`tutorial` `benchmarks` `online_srv` `rl` `highfreq` 等） |

**事实优先级**：源码 > `examples/` 可跑配置 > `docs/` > README。文档与代码冲突时以代码为准，并在正文里注明"文档过时点"。

## 3. 读者画像与学习目标

- 会写 Python（类、装饰器、生成器、pandas、numpy 都熟），没系统学过量化。
- 目标不是"跑通一个 demo"，而是**能判断每个模块该不该用、怎么替换、怎么接入自己的数据和策略**。
- 读完应能回答：我的数据怎么进 Qlib、我的模型怎么插进去、我的策略怎么落到回测、实验怎么复现、上线走哪条路。

## 4. 交付物：`note/` 目录结构

| 文件 | 内容 | 主要源码 |
|---|---|---|
| `01-overview.md` | 全景图：Qlib 解决什么问题、六层架构、核心抽象一页纸、设计哲学五条 | 全仓库 + `README.md` |
| `02-layering.md` | 分层与调用链：从 `qrun` 一条 yaml 到回测报告的完整时序（mermaid） | `qlib/cli/run.py`、`qlib/workflow/*` |
| `03-data-layer.md` | 数据层：日历/标的/特征/PIT/Dataset 五类 Provider，`qlib.init` 全局注册机制 | `qlib/data/data.py`、`base.py`、`cache.py` |
| `04-expression.md` | 表达式引擎：`$close`、`Ref`、`Mean` 如何解析、算子如何注册、缓存如何生效 | `qlib/data/ops.py`、`expression_parser.py`、`cache.py` |
| `05-dataset.md` | `DataHandler` / `Dataset` / `Processor` / `filter` 的分工与数据流 | `qlib/data/dataset/*`、`inst_processor.py` |
| `06-model.md` | 模型层：`BaseModel` 契约、`fit/predict`、`qrun` 如何实例化、滚动训练 | `qlib/model/base.py`、`trainer.py`、`utils.py` |
| `07-strategy-backtest.md` | 策略→决策→订单→撮合→账户→报表：为什么 Strategy 和 Executor 必须分离 | `qlib/strategy/base.py`、`backtest/*` |
| `08-workflow.md` | 实验管理：`QlibRecorder` / `exp` / `recorder` / `record_temp` 四层，与 MLflow 的关系 | `qlib/workflow/*` |
| `09-rl-online.md` | RL 框架（simulator / interpreter / reward）与在线服务（client/server）两条延伸线 | `qlib/rl/*`、`qlib/data/client.py`、`examples/online_srv` |
| `10-extend.md` | **落地篇**：自己写 Provider / Processor / Model / Strategy / Executor，搭一个最小可用系统的 checklist | 综合 |
| `11-reading-path.md` | 阅读顺序（按小时/按天）、调试技巧、高频坑、中英术语表 | 综合 |

写作约定：单篇 1500–4000 字，总量 2.5–3.5 万字；中文叙述，术语首次出现给英文原文。

## 5. 每篇统一结构（模板，逐篇套用）

1. **一句话结论** —— 这层用一句话说清它存在的理由。
2. **它在系统里的位置** —— 上游输入什么、下游交给谁。
3. **关键抽象表** —— `类 | 文件:行号 | 职责 | 扩展点`。
4. **调用链** —— mermaid 时序/流程图 + 关键函数逐跳说明（真实函数名 + 行号）。
5. **设计权衡** —— 为什么这么设计、替代方案是什么、代价在哪。
6. **源码地图** —— 必读文件、建议行号区间、阅读顺序（标注"读到这里可以停"）。
7. **动手实验** —— 本仓库可执行的命令/最小脚本，附预期输出。
8. **迁移到自己系统** —— 这一节落到"我要搭系统该抄什么、该换什么"。
9. **自检问题** —— 3–5 个带答案要点的问题，用来验收是否真读懂。

## 6. 必须回答的设计问题（验收核心）

**数据层**

- 为什么 Qlib 选"交易日历 + 标的列表 + 二进制列存 + 表达式"这套结构，而不是直接吃 Parquet/DataFrame？它换来了什么、牺牲了什么？
- `DataHandler` 与 `Dataset` 为什么拆成两层？`setup_data` / `prepare` 的时序为什么是这个顺序？
- `Processor` 为什么设计成无状态、可链式、`fit` 只做统计量准备？放在 Handler 内部会有什么问题？
- 表达式 `$close / Ref($close, 1) - 1` 是怎么从字符串变成数组的？算子注册表、`__getattr__` 魔法、缓存 key 怎么设计？
- `PIT`（Point-In-Time）数据库解决的是哪一类"未来函数"？`inst_processor` 和 `filter` 各管什么？
- memory cache 与 disk cache 的双层设计，什么时候会踩到"缓存了错误的表达式结果"？

**模型层**

- `BaseModel` 的 `fit/predict` 契约为什么这么窄？窄契约带来了什么好处（可组合、可串行化）？
- `qrun` 的 yaml 怎么变成对象？`init_instance_by_config` 的反射机制与安全边界（注意近期 commit 对"配置驱动代码执行"的限制）。
- 标签怎么造、`dropna` 放在哪、`Dataset.prepare` 的 `col_set` 为什么分 feature/label？

**策略与回测**

- 为什么 `Strategy.generate_trade_decision()` 只产出 `Decision`，而完全不碰订单？`Executor` 拿到的 `TradeDecision` 是什么粒度？
- `_get_order_iterator` / 拆单 / `trade_unit` / `nested decision` 分别在解决什么现实问题？
- 撮合为什么放在 `Exchange`，而账户与持仓放在 `Account`/`Position`？`check_order` 里的限价、涨跌停、停牌、成交量约束怎么落地？
- 回测引擎的 date-position 位图索引（`high_performance_ds.py`）是为性能还是为正确性？
- 交易成本、冲击模型、收益归因（`profit_attribution.py`）各自接在哪一环？

**工作流与系统**

- `QlibRecorder` 与 MLflow 是包装还是替代？`exp` / `recorder` / `record_temp` 三层的职责边界？
- `SignalRecord` → `PortfolioMetrics` 这类 `RecordTemp` 为什么存在？它把"一次实验"的哪些碎片粘起来了？
- `qlib.init` 的全局单例 + `register_all_wrappers` 是便利还是耦合？多进程/多实验并存时要注意什么？
- `client/server` 模式（在线推理、数据服务）与本地模式在代码路径上差在哪？
- C++ 加速（滚动、算子）的边界在哪：哪些计算在 Python，哪些下沉了？
- "配置驱动"（yaml 一把梭）与"代码驱动"（自己写脚本）的取舍，什么规模该切到后者？

## 7. 质量门禁（每篇交付前自查）

- [ ] 每个类/函数**首次出现**都带 `` `路径:行号` ``，行号对得上当前 commit。
- [ ] 区分三类内容并显式标注：**【源码事实】** / **【设计评述】** / **【我的建议】**。
- [ ] 代码片段来自本仓库，可裁剪但**不得改写语义**；命令必须在本机可执行。
- [ ] 不确定的地方标 `[待核实]`，**绝不编造**类名、参数、默认值。
- [ ] 每篇至少 1 张 mermaid 图（`flowchart` 或 `sequenceDiagram`），且语法可渲染。
- [ ] 术语表条目与正文用词一致；不出现"大概/可能/应该是"来糊弄机制。
- [ ] 不输出无法验证的性能数字；引用 benchmark 必须给出来源文件。

## 8. 写作风格

- 第二人称、工程口吻：**先结论 → 再类比 → 最后源码锚点**。
- 表格优先于长列表；每段都为"读懂代码"或"搭系统"服务，不写背景铺垫。
- 敢下判断：说清"这个设计好在哪、坑在哪、我会怎么改"。
- 类比要克制且准确（例如：Provider ≈ 数据访问层的 DAO；Recorder ≈ 实验的 git commit）。

## 9. 明确不做的事

- 不写 Qlib 项目史、不罗列 CHANGELOG。
- 不大段翻译 `docs/`；引用时给链接与要点即可。
- 不做 Alpha158/Alpha360 的逐因子流水账（一句话交代用途即可）。
- 不生成 PDF/HTML/图片；先交付 Markdown + mermaid 源码。
- 不修改仓库源码；所有产出只写 `note/`。

## 10. 执行流程（分阶段，可随时打断）

| 阶段 | 动作 | 产出 |
|---|---|---|
| P0 | 侦察：目录、入口、配置样例、docs 结构 | （已完成，见第 2 节） |
| P1 | 精读 `cli/run.py` + `workflow/`，画出端到端时序 | `01` `02` |
| P2 | 数据层三连：`data.py` → `ops.py` → `dataset/` | `03` `04` `05` |
| P3 | 模型层 + 策略回测 | `06` `07` |
| P4 | 工作流 + RL/在线 | `08` `09` |
| P5 | 落地与导读 | `10` `11` |
| P6 | 交叉校验：抽查行号、渲染 mermaid、修正 `[待核实]`、统一术语 | 全量修订 |

## 11. 待你确认的选项（括号内为默认值）

1. **语言**：中文正文 + 英文术语（默认）／纯英文。
2. **深度**：设计意图 + 源码锚点（默认）／逐行注释式精读（篇幅会翻倍）。
3. **是否要可运行 demo**：给命令与最小脚本、复用 `examples/`（默认）／只要文字不要代码。
4. **落地篇比重**：`10-extend.md` 写"最小系统 checklist + 接入自有数据/模型"（默认）／再加一份完整项目脚手架。
5. **交付格式**：Markdown + mermaid（默认）／额外导出单文件 HTML 或 PDF。
6. **篇幅**：2.5–3.5 万字（默认）／压缩到 1 万字速览版。

## 12. 确认清单

- [ ] 目录结构（第 4 节）认可，或指出增删篇章。
- [ ] 第 6 节的设计问题覆盖了你最想搞懂的点；缺的请补。
- [ ] 第 11 节选项按默认执行，或逐条改。
- [ ] 允许我开始 P1（会先产出 `01` `02` 供你验货，再继续）。

回复「**按默认执行**」即可开工；也可以直接在这份文件上批注后回我「**按修订版执行**」。
