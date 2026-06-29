# Agent Architecture Design — Exam

## 1. Workflow vs Agent（⭐）

请解释 workflow 和 agent 的区别。给出两个更适合 workflow 的场景，以及两个更适合 agent 的场景。

## 2. 架构分层（⭐）

一个生产级 Agent 系统至少应该包含哪些层？请列出 5 层以上，并说明每层解决什么问题。

## 3. 工具调用安全（⭐⭐）

你要设计一个“自动帮用户提交报销单”的 Agent。它需要读取发票、填写表单、提交审批。请说明哪些工具可以自动执行，哪些工具必须加入 human-in-the-loop，为什么？

## 4. 控制流选择（⭐⭐）

某客服机器人需要处理：

- 查询订单状态
- 解释退款政策
- 判断是否能退款
- 发起退款
- 遇到异常转人工

你会用固定 workflow、single-agent、multi-agent，还是混合架构？请画出简化流程。

## 5. 上下文与记忆（⭐⭐）

请区分 working context、retrieved context、long-term memory、execution state。为什么不能把所有历史信息都塞进模型上下文？

## 6. 可观测性与评测（⭐⭐⭐）

一个 Agent 在线上成功率从 85% 掉到 70%。你会先看哪些 trace 和指标？如何判断问题来自 prompt、检索、工具、模型还是权限？

## 7. 框架选型（⭐⭐⭐）

请比较 LangGraph、OpenAI Agents SDK、Qwen-Agent、Coze/扣子、AutoGen 的适用场景。面向阿里/腾讯/字节/AI 独角兽的生产系统，你会怎么选型？

## 8. 反模式识别（⭐⭐⭐）

下面这个设计有什么问题？如何改？

```text
用户输入 -> LLM 决定调用任意内部 API -> API 直接执行 -> LLM 总结结果
```

要求至少指出 5 个风险，并给出改进后的架构。

## 来源

- Anthropic, Building effective agents: https://www.anthropic.com/engineering/building-effective-agents
- OpenAI Agents SDK documentation: https://openai.github.io/openai-agents-python/
- LangGraph documentation: https://docs.langchain.com/oss/python/langgraph/overview
- Qwen-Agent GitHub: https://github.com/QwenLM/Qwen-Agent
- OWASP Top 10 for LLM Applications: https://owasp.org/www-project-top-10-for-large-language-model-applications/
