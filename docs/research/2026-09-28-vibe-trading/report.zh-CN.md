# Vibe-Trading 研究：金融研究 Agent 的工程设计、交易边界与演示价值

> 研究日期：2026-09-28。上游：HKUDS/Vibe-Trading。固定提交：47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a。
> 方法：一手文档与关键源码的有限范围静态研究，按架构、风险、产品价值并行分析，再独立审查。没有安装或运行 Vibe-Trading，没有执行测试、接入券商账户、下单或测量收益。不是全面安全审计或投资建议。
> GitHub 研究目录：https://github.com/turbo998/Vibe-Trading/tree/main/docs/research/2026-09-28-vibe-trading

# 一句话摘要

**Vibe-Trading 值得研究的是“把金融问题变成可检查的执行过程”，而不是“让大模型自动赚钱”。** 它把自然语言、市场数据、策略生成、回测、验证、报告与持久记忆连接成研究工作台，同时提供另行授权的实验性券商交易分支。[S1][S7]

本研究判断：适合作为金融 Agent 工程参考与隔离环境中的研究演示候选；不能据此认定已经生产就绪、可安全实盘或具有超额收益。

# 核心要点

## 1. 它解决什么问题

- **研究问题到执行产物**：自然语言问题 → 选择技能、工具与数据 → 生成策略或分析 → 回测及适用验证 → 报告、运行记录与后续研究。
- **交易复盘**：Shadow Account 从交易日志提取行为和规则，再比较实际交易与规则化反事实路径。它不是券商账户，也不是必然提升收益的策略。
- **投研协作**：预设投资、量化、宏观、风险等角色，按依赖关系协作生成报告。
- **多入口**：CLI、Web/API、MCP 及消息渠道。入口存在不等于所有渠道均已在目标环境验证。

以上产品流程由 README 描述；关键编排实现另有源码证据。README 对 Monte Carlo、Bootstrap、Walk-Forward 等使用“适用时”的限定，不能说每次研究必然完整执行所有验证。[S1][S2]

## 2. 架构：专业工具、主 Agent 与 Swarm 分层

**入口 → 会话服务 → 主 Agent／Swarm → 工具与数据 → 回测、验证、产物。**

- **界面与服务**：Python 后端、FastAPI、React/TypeScript 前端；会话服务桥接执行状态与流式事件。包元数据声明 Python ≥3.11、MIT、Beta；该提交包版本为 0.1.15。
- **主 Agent**：显式 ReAct 循环，围绕工具注册表、技能加载、上下文压缩、持久记忆及证据校验工作。虽然依赖中有 LangChain/LangGraph，不能因此把所读编排直接称为“LangGraph 驱动”。
- **工具调度**：连续只读调用可并行，写工具串行；MCP 包装器走串行路径。只读是工具声明，不是对其副作用的自动证明。
- **Swarm**：YAML 预设角色、工具白名单、可配置的技能限制和任务依赖；分层 DAG 在同层并发，上游失败会阻断下游。已完成且定义未变的节点可在恢复时复用；重试会清理旧 worker 产物。
- **重要限制**：它不是分布式集群；worker 内部的工具调用逐个执行，取消／线程超时也不等于强制终止正在运行的所有 Python 工具。[S2][S3]

## 3. 最值得借鉴的 Agent 工程设计

**证据要先进入账本，再进入最终答案。** 主 Agent 将工具完整结果在截断前纳入证据账本，核查其规则能够识别的数字、身份和来源。核验失败可有限补查或改写，预算耗尽后删除未证实数字或降级输出；这不是对全部自然语言推理正确性的证明。[S4]

**压缩不能只留下漂亮摘要。** 主循环区分确定性缓存与只读结果重放，保留轨迹并处理工具消息配对；写操作会清空只读重放缓存。值得借鉴的是压缩后的证据恢复与状态失效，而不是注释中的“零信息损失”承诺。[S5]

**失败是产物的一部分。** Swarm 区分失败、未完成和取消，传播上游失败并隔离重试产物。研究流程不应仅因旧指标还在就宣告本轮成功。[S1][S3]

**“自我改进”主要是工作流沉淀。** 所读实现是持久记忆、检索和按需加载的技能；它不构成模型权重学习或收益持续提升的证据。[S5]

## 4. 主 Agent 与 Swarm 的可信度不能混为一谈

主 Agent 有确定性的 grounding 发布校验路径；Swarm worker 使用单独的轻量循环，共享启动时获取的行情，并进行交付启发式检查，例如拒绝计划空壳、mock 标记或未解析工具语法。

**在所读 worker 路径中，没有实例化主 Agent 的 GroundingLedger。** 因而“有多个角色讨论、写出了 report.md”不等于“每个数字均经主 Agent 的证据门验证”。Swarm 共享行情快照也不是持续更新的实时行情。[S3][S4]

演示时应展示来源、时间和实际运行产物，而不是仅展示角色头像与最终共识。

# 交易能力：按 profile 判断，不按券商品牌判断

以下分类来自固定提交的生成能力矩阵及相关说明，**是内置能力声明，不是本次账户联通或下单验证**。矩阵明确不证明撤单、清仓等全部运行时保护覆盖；不包含用户自定义插件。[S6]

- **声明 live 下单且要求 mandate 的具体 profile**：`alpaca-live-trade`、`binance-live-trade`、`etoro-live-trade`、`futu-live-trade`、`mt5-live-trade`、`okx-live-trade`、`robinhood-live-mcp`、`tiger-live-trade`。同一券商另有只读 profile；不能推断其全部市场、订单类型或操作都可用。
- **下单限定 paper、live 为只读**：Longbridge、Dhan、Shoonya、Zerodha、Upbit、KIS。文档说明前五者缺少相应运行时 paper/live 判别条件，KIS 的 live 下单尚未接入。
- **内置下单面只读**：IBKR、Scalable Capital、Trading 212、Toss；环境名称含 live 不代表允许交易。
- **paper 的两种含义**：可能是券商沙盒，也可能是本地模拟。它不自动授予 live 权限；Robinhood 的所列交易路径为 live-only。

源码将模型提议与正式 mandate 提交分开，提交要求明确同意；所读 SDK 下单门禁检查授权、到期、halt、额度、持仓及计数。但这些静态结构不证明所有 API、shell、插件和同权限文件访问路径均无法绕过，也不等于独立的人类身份认证。[S7]

**Halt 不等于立即清仓。** 模型中 flatten_on_halt 默认关闭，风险降低操作有独立处理。额度是交易前约束，不能保证最终成交价格或最大损失。[S7]

# 主要风险与容易误读的地方

## 1. 生成策略会执行代码，不是只读聊天

安全政策明确：回测会在本地执行生成的 Python，并保留联网取市场数据的能力。runner 有环境收窄、尽力创建的临时 HOME 和条件性降权；临时 HOME 创建失败时会继承 HOME，降权不可用时也存在继续运行的回退。临时 HOME 不是文件系统访问隔离。[S8]

**建议**：后续功能验证放入独立低权限环境，不暴露真实交易秘密、客户文件或内部网络服务；关闭 shell 工具不能替代对生成策略执行面的隔离。

## 2. “回测成功”与“策略正确”是不同层次

所读 Runner 的进程 success 来自退出码，产物按存在性收集；这不意味着下游完全没有校验，也不意味着经济假设已经正确。报告引用与哈希可增强追溯，但不能证明原始行情、市场规则和收益结论无误。[S8]

后续需要分别检查：市场引擎、时点数据、复权、币种与单位、佣金和滑点、成交假设、样本外验证、过拟合和幸存者偏差。多 Agent 共识与历史收益不能代替这些检查。

## 3. 密钥代理与逐笔审批不是默认覆盖所有券商

README 的 TAP 模式是默认关闭的 Alpaca 可选路径，不能宣传为默认全券商密钥隔离。文档仍提示批准与超时边界的竞态；源码又存在 Alpaca pending-action 恢复处理，二者是否已完全衔接，本次未验证。**超时报错不能直接解释为订单未送达。**[S9]

## 4. 功能数量与测试目录不是可靠性认证

README 列出大量工具、技能、因子、预设及数据源，但本研究未逐项验收。测试源码中的 fake connector 支持理解设计意图，不证明真实券商接口可用。上游免责声明仍将券商交易能力称为实验性；不能把这句笼统免责声明反向解释为从未有任何读取路径被任何人验证。[S6][S7]

# 已发现的文档与源码差异

- **Swarm 外部 MCP**：README 旧 v1 表称不支持；同一文档“SWARM external MCP tools”章节却描述支持，注册表及 runtime→worker 的配置传递也有代码支撑。采用更具体且有实现支撑的表述：运营者配置可信服务器、工具位于白名单时可用；没有做协议端到端验证。[S10]
- **Copilot 认证顺序**：README 将已存 Copilot CLI 凭据排在 gh 凭据之前；所读实现先尝试显式环境凭据，再取 gh 凭据，均无可用 token 时才交由 SDK 解析存储凭据。部署时应针对锁定 SDK 版本实测，不能照抄文档顺序。[S11]
- **主循环 manifest 覆盖**：文档对技能、工具注册表的哈希描述较宽；所读调用点传入系统提示、工具名与包版本，运行中加载技能另见 trace。因此不宣称 manifest 已独立哈希覆盖所有技能全文及工具 Schema。[S5]

这些是固定版本内的证据冲突，不做“取平均”处理，也不据此宣称其他模块失效。

# 对客户与合作伙伴的价值

## 推荐演示主线：可复核的金融研究，而非收益预测

以下为本研究提出的演示方案，**尚未执行**：

- 使用已授权、冻结的本地历史数据，提出明确标的、日期、规则与基准的研究问题。
- 展示实际数据来源和时间、生成策略、匹配的回测引擎、结果与适用验证。
- 展示 run card、报告、执行记录及数值引用；制造一个缺数据情形，检查是否明确失败而不是编造结果。
- 由另一位评审核对关键指标与输入；全程不接券商，不演示 paper/live 下单。

潜在客户价值是缩短研究交接、增强结果可追溯性；本次没有量化效率提升或 ROI 证据。

## 微软生态：区分已见实现与拟议集成

- **已见实现：GitHub Copilot 模型 provider。** 可选依赖为官方 github-copilot-sdk，工厂返回 CopilotSDKLLM，适配器桥接模型与工具请求。它是 Vibe-Trading 内部的模型后端，不是现成的 Copilot Chat 插件，也不是微软认证的金融产品。订阅政策、配额、认证和工具回传未实测。[S11]
- **仅文档／依赖级证据：Teams。** 文档列有 msteams 接入面，但本次未审阅完整适配器或验证租户权限、发送者身份和投递回执。[S12]
- **部署参考：自托管与 Docker。** 提供前后端同服务部署及持久卷，Compose 默认端口绑定回环地址并声明若干加固配置；本次没有构建镜像。Copilot 是可选 extra，未核验默认镜像是否具备该 SDK。[S12]
- **本研究拟议、未核验原生集成：Azure 平台部署与服务集成。** 可先评估隔离 VM，再决定是否适配 Container Apps／AKS；Entra、Key Vault、存储、Monitor 属于后续身份、秘密、状态及观测集成。需要验证子进程、容器权限、SSE、持久化和多用户隔离，不能仅凭支持自定义模型 URL 就声称 Azure 原生可用。

# 需进一步关注

- **已覆盖**：产品流程、关键 Agent／Swarm 编排、工具权限与证据门、交易 profile 声明、授权及代码执行边界、Copilot 接入方式、演示建议。
- **未执行**：安装、构建、测试、模型调用、数据源连通性、回测结果复现、交易、云部署与性能成本测量。
- **未全面审阅**：全部数据 loader、回测引擎、券商连接器、API 认证链、前端页面和第三方依赖。读取范围与文件 blob SHA 见同目录 evidence.json。
- **阻断生产／实盘结论的证据缺口**：逐连接器运行验证、最小权限隔离、账户与授权身份绑定、故障对账、可信数据及经济假设验证、业务合规评估。

**结论：值得 fork 并作为研究／演示候选继续评估；当前交付是静态研究，不是部署验收或实盘认证。**

# 一手来源与定位

- **S1 定位与研究流程**：README 的 What Is、Shadow Account、Research Workflow。https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/README.md#L330-L395
- S1 研究验证的适用性限定：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/README.md#L523-L543
- **S2 技术与包元数据**：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/pyproject.toml
- S2 工具注册：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/agent/tools.py
- **S3 Swarm 执行和 worker**：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/swarm/runtime.py
- S3 worker：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/swarm/worker.py
- **S4 证据账本**：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/agent/grounding/ledger.py
- S4 Swarm 行情：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/swarm/grounding.py
- **S5 主循环、压缩与发布**：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/agent/loop.py
- S5 技能：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/agent/skills.py
- **S6 连接器声明矩阵**：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/README.md#L423-L495
- **S7 授权提交**：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/live/mandate/commit.py
- S7 SDK 门禁：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/live/sdk_order_gate.py
- S7 halt 默认：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/live/mandate/model.py#L135-L148
- S7 免责声明：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/README.md#L2233-L2239
- **S8 生成代码安全政策**：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/SECURITY.md#L23-L27
- S8 Runner：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/core/runner.py#L571-L709
- **S9 TAP 与竞态限定**：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/README.md#L1836-L1898
- **S10 MCP 支持说明**：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/README.md#L1588-L1590
- S10 冲突旧表：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/README.md#L1949-L1957
- S10 注册实现：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/tools/__init__.py#L310-L391
- **S11 Copilot provider**：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/agent/src/providers/copilot_auth.py
- S11 文档：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/README.md#L953-L977
- **S12 部署声明**：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/docker-compose.yml
- S12 镜像构建：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/Dockerfile
- S12 Teams 声明：https://github.com/HKUDS/Vibe-Trading/blob/47a036d66daa7f86dc1561be4b3f59c6a1f6ec7a/pyproject.toml#L218-L221

# 审查与版本说明

本报告使用固定上游提交，不把未来 main 分支变化混入结论。独立审查记录见同目录 review.md；证据元数据见 evidence.json。报告中的“建议”“拟议”“本研究判断”均非上游承诺，也非已执行结果。GitHub 与飞书发布后的完整性、权限核验与研究事实审查分开记录。
