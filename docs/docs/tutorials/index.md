---
hide:
  - toc
---

# 教程

欢迎来到 LangGraph 教程！这些笔记本通过构建各种语言代理和应用程序来介绍 LangGraph。

## 快速入门

通过一个全面的快速入门学习 LangGraph 的基础知识，您将在其中从头开始构建一个代理。

- [快速入门](introduction.ipynb)

## 用例

从为特定场景设计并实现常见设计模式的图的示例实现中学习。

#### 聊天机器人

- [客户支持](customer-support/customer-support.ipynb): 构建一个客户支持聊天机器人来管理航班、酒店预订、租车和其他任务
- [根据用户需求生成提示](chatbots/information-gather-prompting.ipynb): 构建一个信息收集聊天机器人
- [代码助手](code_assistant/langgraph_code_assistant.ipynb): 构建一个代码分析和生成助手

#### 多代理系统

- [协作](multi_agent/multi-agent-collaboration.ipynb): 使两个代理能够协作完成一项任务
- [监督](multi_agent/agent_supervisor.ipynb): 使用 LLM 来协调和委派给各个代理
- [层次结构团队](multi_agent/hierarchical_agent_teams.ipynb): 协调嵌套的代理团队来解决问题

#### RAG

- [自适应 RAG](rag/langgraph_adaptive_rag.ipynb)
    - [使用本地 LLM 的自适应 RAG](rag/langgraph_adaptive_rag_local.ipynb)
- [代理 RAG](rag/langgraph_agentic_rag.ipynb)
- [纠正 RAG](rag/langgraph_crag.ipynb)
    - [使用本地 LLM 的纠正 RAG](rag/langgraph_crag_local.ipynb)
- [自 RAG](rag/langgraph_self_rag.ipynb)
    - [使用本地 LLM 的自 RAG](rag/langgraph_self_rag_local.ipynb)
- [SQL 代理](sql-agent.ipynb)

#### 规划代理

- [计划和执行](plan-and-execute/plan-and-execute.ipynb): 实现一个基本的计划和执行代理
- [无观察推理](rewoo/rewoo.ipynb): 通过将观察结果保存为变量来减少重新规划
- [LLM 编译器](llm-compiler/LLMCompiler.ipynb): 从规划器流式传输和急切地执行任务的 DAG

#### 反思和批判 

- [基本反思](reflection/reflection.ipynb): 提示代理反思和修改其输出
- [反思](reflexion/reflexion.ipynb): 批判缺失和多余的细节以指导下一步
- [语言代理树搜索](lats/lats.ipynb): 使用反思和奖励来驱动对代理的树搜索
- [自我发现代理](self-discover/self-discover.ipynb): 分析一个了解自身能力的代理

#### 评估

- [基于代理的](chatbot-simulation-evaluation/agent-simulation-evaluation.ipynb): 通过模拟的用户交互来评估聊天机器人
- [在 LangSmith 中](chatbot-simulation-evaluation/langsmith-agent-simulation-evaluation.ipynb): 在 LangSmith 中使用对话数据集来评估聊天机器人

#### 实验

- [网络研究 (STORM)](storm/storm.ipynb): 通过研究和多角度问答生成类似维基百科的文章
- [TNT-LLM](tnt-llm/tnt-llm.ipynb): 构建用户意图的丰富、可解释的分类，使用微软为其必应 Copilot 应用程序开发的分类系统。
- [网页导航](web-navigation/web_voyager.ipynb): 构建一个可以导航和与网站交互的代理
- [竞技编程](usaco/usaco.ipynb): 构建一个具有少量“情节记忆”和人机交互协作的代理来解决来自美国计算奥林匹克竞赛的问题；改编自 Shi、Tang、Narasimhan 和 Yao 的论文 ["语言模型能否解决奥林匹克编程？"](https://arxiv.org/abs/2404.10952v1)。
- [复杂数据提取](extraction/retries.ipynb): 构建一个可以使用函数调用来执行复杂提取任务的代理