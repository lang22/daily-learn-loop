# Agent Architecture Design — Interview Reference Answers

评分标准依据：本文件综合 Anthropic、OpenAI Agents SDK、LangGraph、Qwen-Agent、Coze、AutoGen 与 OWASP LLM Top 10，总结生产 Agent 架构面试中的高频评估点。每题标注主要依据来源。[来源: Anthropic Building Effective Agents][来源: OpenAI Agents SDK][来源: LangGraph Overview][来源: Qwen-Agent][来源: Coze Docs][来源: Microsoft AutoGen][来源: OWASP LLM Top 10]

## 1. 架构总览

题目原文：

你要从 0 到 1 设计一个企业内部知识助手，它可以查文档、查工单、调用内部系统创建任务。请给出整体架构。

追问：

- 哪些部分是 workflow，哪些部分是 agent？
- 工具层怎么设计？
- 如何做权限隔离？
- 如何避免 prompt injection？

满分回答要点：

- 先定义边界：只读问答、工单查询、任务创建属于不同风险等级。
- 架构包含 gateway、router、RAG、agent runtime、tool gateway、permission service、human approval、trace/eval。
- 文档问答用 RAG workflow；复杂工单排查用 single-agent；创建任务走 prepare/confirm/commit。
- 工具层必须有 schema、权限、幂等、审计、超时和重试。
- Prompt injection 防护要把外部文档内容当 data，不当 instruction。

示范回答：

我会用混合架构。入口层做身份、租户、quota 和输入过滤；router 判断是普通问答、工单查询还是需要执行动作。文档问答走 RAG workflow，工单排查可以进入单 Agent 循环。所有内部系统调用都经过 tool gateway，gateway 做 schema validation、RBAC、审计、限流和幂等。创建任务这类有副作用的动作拆成 prepare_task、user_confirm、commit_task。全链路记录 trace，用离线 golden tasks 和线上指标评估。

常见错误：

- 只说“用 LangChain 接知识库”。
- 忽略权限和副作用。
- 没有说明 workflow 和 agent 边界。

主要依据：Anthropic 的 workflow/agent patterns、OpenAI Agents SDK 的 tools/guardrails/tracing、OWASP 风险分类。[来源: Anthropic Building Effective Agents][来源: OpenAI Agents SDK][来源: OWASP LLM Top 10]

## 2. Workflow vs Agent

题目原文：

什么时候你会拒绝使用 Agent，而选择固定 workflow？请结合真实业务场景说明。

追问：

- 如果业务方坚持“要更智能”，你如何解释 trade-off？
- 什么时候可以从 workflow 演进到 agent？

满分回答要点：

- 步骤固定、风险高、合规强时选 workflow。
- 路径不确定、需要探索和根据反馈调整时选 agent。
- 可以先 workflow，积累失败样本和评测集，再局部 agent 化。

示范回答：

退款、报销、审批、支付这类场景我不会一上来用自主 Agent。它们的流程固定，错误代价高，审计要求强。更好的方案是 workflow 控制主路径，LLM 只负责分类、解释和草拟。等我们有足够 trace 和失败样本，确认某些异常处理确实需要动态探索，再把局部节点改成 agent。

常见错误：

- 认为 agent 一定比 workflow 高级。
- 不讨论失败代价。

主要依据：Anthropic 关于 workflows、agents 和复杂度取舍的建议。[来源: Anthropic Building Effective Agents]

## 3. 工具调用与副作用

题目原文：

设计一个 Agent 帮销售自动更新 CRM、生成邮件、安排会议。哪些操作可以自动执行，哪些需要用户确认？

追问：

- 如何保证工具幂等？
- 如何处理工具超时和部分成功？
- 如何审计一次工具调用链？

满分回答要点：

- 低风险动作可自动：读取 CRM、草拟邮件、查空闲时间。
- 高风险动作需确认：写 CRM、发邮件、创建外部会议邀请。
- 幂等 key、事务状态、补偿操作、审计日志是关键。

示范回答：

读 CRM 和草拟邮件可以自动。更新客户阶段、发送邮件、创建会议邀请需要用户确认。每个工具调用都带 request_id 作为幂等 key，工具返回结构化状态：prepared、committed、failed、partial。超时后先查状态再决定是否重试，避免重复发送。审计记录 agent run id、用户、工具名、参数摘要、权限判断、结果和错误。

常见错误：

- 只考虑 function calling，不考虑业务事务。
- 失败后盲目重试。

主要依据：OWASP 对 excessive agency 和不安全工具调用风险的分类。[来源: OWASP LLM Top 10]

## 4. 状态与记忆

题目原文：

一个长任务 Agent 需要连续运行 30 分钟，中间可能等待用户确认，也可能工具失败重试。你如何设计状态管理？

追问：

- 状态存在数据库、Redis 还是向量库？
- 长期记忆和 execution state 有什么区别？
- 如何恢复中断任务？

满分回答要点：

- execution state 适合持久化数据库或 durable runtime，不适合向量库。
- Redis 可做短期缓存，但不能作为唯一事实源。
- long-term memory 是用户/项目长期事实，execution state 是任务进度。
- 中断恢复依赖 run id、checkpoint、状态机节点、工具调用结果。

示范回答：

我会把 execution state 存在数据库或支持 checkpoint 的 runtime 里，记录 run id、当前节点、已完成步骤、工具调用结果、预算、错误次数和等待中的 human approval。Redis 只做缓存。向量库用于检索语义记忆，不用于保存任务进度。恢复时根据 checkpoint 回到上一个安全节点，已 commit 的工具不重复执行。

常见错误：

- 把 memory 全部等同于向量库。
- 没有区分任务状态和长期偏好。

主要依据：LangGraph 对 durable execution、memory、human-in-the-loop 的官方定位。[来源: LangGraph Overview]

## 5. 框架选型

题目原文：

现在有 LangGraph、OpenAI Agents SDK、Qwen-Agent、Coze、AutoGen。你会如何为一个面向国内企业客户的 Agent 产品选型？

追问：

- 如果必须私有化部署呢？
- 如果目标是快速验证业务呢？
- 如果目标是复杂长任务可靠执行呢？

满分回答要点：

- 国内企业客户要考虑模型生态、私有化、合规、团队熟悉度。
- 快速验证可用 Coze。
- 复杂长任务可靠执行可用 LangGraph。
- Qwen 生态可评估 Qwen-Agent。
- OpenAI Agents SDK 适合 OpenAI-first。
- AutoGen 不应默认用于生产主链路。

示范回答：

如果是国内企业客户，我会先看私有化和模型供应链。如果需要快速验证业务，可以用 Coze 类平台做 POC。如果要长任务、可恢复、可审计，我会倾向 LangGraph 或自研状态机。如果模型主要用 Qwen，Qwen-Agent 的 MCP、RAG、code interpreter 集成值得评估。AutoGen 更适合多 Agent 探索，不会默认放到核心生产链路。

常见错误：

- 只按流行度选框架。
- 不考虑部署和审计。

主要依据：各框架/平台官方文档能力边界与生态定位。[来源: LangGraph Overview][来源: OpenAI Agents SDK][来源: Qwen-Agent][来源: Coze Docs][来源: Microsoft AutoGen]

## 6. 评测与线上问题定位

题目原文：

你的 Agent 上线后用户投诉“经常答非所问，有时乱调用工具”。你会怎么建立评测和观测体系？

追问：

- 需要哪些离线测试集？
- trace 里必须记录什么？
- 如何做回归测试？

满分回答要点：

- 建 golden tasks，覆盖问答、工具调用、安全、异常。
- Trace 记录模型输入输出、检索结果、工具调用、状态转移、权限判断、成本延迟。
- 回归测试在 prompt、模型、工具 schema、检索索引变化时运行。

示范回答：

我会先把用户投诉样本沉淀成 golden tasks，标注期望工具、参数、最终结果和禁止行为。线上 trace 必须能还原每一步：router 决策、检索结果、模型消息、工具参数、权限判断、错误、token 和 latency。然后按失败类型分桶：检索错、工具参数错、权限拒绝、模型幻觉、业务规则缺失。每次改 prompt、模型或工具 schema 都跑回归。

常见错误：

- 只看最终回答，不看中间步骤。
- 没有失败分桶。

主要依据：OpenAI Agents SDK tracing/guardrails、LangGraph production runtime，以及 OWASP 风险分类。[来源: OpenAI Agents SDK][来源: LangGraph Overview][来源: OWASP LLM Top 10]

## 7. 成本与延迟

题目原文：

一个 Agent 平均要调用 5 次模型、3 次检索、2 次工具，P95 延迟超过 20 秒。你如何优化？

追问：

- 哪些步骤可以并行？
- 如何做模型路由？
- 如何设置 budget 和 early stop？

满分回答要点：

- 减少模型调用轮数，固定路径 workflow 化。
- 并行检索和独立工具。
- 模型路由：简单任务小模型，复杂规划大模型。
- 缓存、上下文裁剪、early stop、预算限制。
- 流式返回和后台继续执行改善体验。

示范回答：

我会先看 trace，找出最耗时节点。能固定的步骤改成 workflow，减少 agent loop。检索和互不依赖的工具并行。分类、改写、格式化用小模型，复杂规划用大模型。设置 max steps、token budget 和 early stop；对稳定 prompt 和检索结果做缓存；压缩上下文，避免每轮塞全量历史。

常见错误：

- 只说换更快模型。
- 不看 P95 和节点级 latency。

主要依据：Anthropic 关于优先简单 workflow 的建议，以及 OpenAI Agents SDK tracing 用于定位运行时成本/延迟的能力。[来源: Anthropic Building Effective Agents][来源: OpenAI Agents SDK]

## 8. Multi-Agent 取舍

题目原文：

业务方提出“我们做一个多个专家 Agent 协作的系统，一个负责检索，一个负责规划，一个负责执行，一个负责评审”。你如何评价这个方案？

追问：

- multi-agent 的收益是什么？
- 风险是什么？
- 什么情况下你会真的采用？

满分回答要点：

- 收益：职责分离、并行、专家化、评审机制。
- 风险：成本高、延迟高、错误传播、调试困难、责任边界模糊。
- 只有在任务天然可分解且评测证明收益超过成本时采用。

示范回答：

我不会因为听起来高级就上 multi-agent。先用单 Agent 或 graph 节点实现职责分离。如果检索、规划、执行、评审确实可以并行或需要不同模型/权限，再考虑拆成多个 agent。上线前要用任务成功率、成本、延迟和安全指标证明 multi-agent 比单 Agent 好。

常见错误：

- 把每个函数都包装成 agent。
- 没有评估协作失败和成本。

主要依据：Anthropic orchestrator-workers pattern 与 AutoGen multi-agent framework 定位。[来源: Anthropic Building Effective Agents][来源: Microsoft AutoGen]

## 来源

- Anthropic, Building effective agents: https://www.anthropic.com/engineering/building-effective-agents
- OpenAI Agents SDK documentation: https://openai.github.io/openai-agents-python/
- LangGraph documentation: https://docs.langchain.com/oss/python/langgraph/overview
- Microsoft AutoGen documentation: https://microsoft.github.io/autogen/
- Qwen-Agent GitHub: https://github.com/QwenLM/Qwen-Agent
- Coze documentation: https://www.coze.cn/open/docs/guides/welcome
- OWASP Top 10 for LLM Applications: https://owasp.org/www-project-top-10-for-large-language-model-applications/
