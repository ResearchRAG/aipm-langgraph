# 🦜🕸️LangGraph

![版本](https://img.shields.io/pypi/v/langgraph)
[![下载量](https://static.pepy.tech/badge/langgraph/month)](https://pepy.tech/project/langgraph)
[![开放问题](https://img.shields.io/github/issues-raw/langchain-ai/langgraph)](https://github.com/langchain-ai/langgraph/issues)
[![文档](https://img.shields.io/badge/docs-latest-blue)](https://researchrag.github.io/aipm-langgraph/)

⚡ 使用图构建语言代理 ⚡

> [!注意]
> 想要了解 JS 版本？点击 [这里](https://github.com/langchain-ai/langgraphjs) ([JS 文档](https://langchain-ai.github.io/langgraphjs/)).

## 概述

[LangGraph](https://researchrag.github.io/aipm-langgraph/) 是一个用于构建具有状态的、多角色的 LLM 应用程序的库，用于创建代理和多代理工作流程。与其他 LLM 框架相比，它提供了以下核心优势：循环、可控性和持久性。LangGraph 允许您定义包含循环的流程，这对于大多数代理架构至关重要，使其有别于基于 DAG 的解决方案。作为一个非常底层的框架，它提供了对应用程序的流程和状态的精细控制，这对于创建可靠的代理至关重要。此外，LangGraph 包含内置的持久性，支持高级的人机交互和记忆功能。

LangGraph 受 [Pregel](https://research.google/pubs/pub37252/) 和 [Apache Beam](https://beam.apache.org/) 的启发。公共接口借鉴了 [NetworkX](https://networkx.org/documentation/latest/)。LangGraph 由 LangChain Inc（LangChain 的创建者）构建，但可以在没有 LangChain 的情况下使用。

### 主要特性

- **循环和分支**: 在您的应用程序中实现循环和条件语句。
- **持久性**: 在图中的每个步骤执行后自动保存状态。随时暂停和恢复图执行，以支持错误恢复、人机交互工作流程、时间旅行等等。
- **人机交互**: 中断图执行以批准或编辑代理计划的下一个操作。
- **流支持**: 随着每个节点（包括令牌流）生成输出而流式传输输出。
- **与 LangChain 集成**: LangGraph 与 [LangChain](https://github.com/langchain-ai/langchain/) 和 [LangSmith](https://docs.smith.langchain.com/) 无缝集成（但不需要它们）。


## 安装

```shell
pip install -U langgraph
```

## 示例

LangGraph 的核心概念之一是状态。每次图执行都会创建一个状态，该状态在图中的节点执行时传递，并且每个节点在执行后使用其返回值更新此内部状态。图更新其内部状态的方式由所选图的类型或自定义函数定义。

让我们看一个使用搜索工具的简单代理示例。

```shell
pip install langchain-anthropic
```

```shell
export ANTHROPIC_API_KEY=sk-...
```

可选地，我们可以设置 [LangSmith](https://docs.smith.langchain.com/) 以获得最佳的可观察性。

```shell
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY=lsv2_sk_...
```

```python
from typing import Annotated, Literal, TypedDict

from langchain_core.messages import HumanMessage
from langchain_anthropic import ChatAnthropic
from langchain_core.tools import tool
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import END, START, StateGraph, MessagesState
from langgraph.prebuilt import ToolNode


# 定义代理要使用的工具
@tool
def search(query: str):
    """调用搜索工具."""
    # 这是一个占位符，但不要告诉 LLM...
    if "sf" in query.lower() or "san francisco" in query.lower():
        return "温度为 60 度，有雾。"
    return "温度为 90 度，阳光明媚。"


tools = [search]

tool_node = ToolNode(tools)

model = ChatAnthropic(model="claude-3-5-sonnet-20240620", temperature=0).bind_tools(tools)

# 定义函数来确定是否继续
def should_continue(state: MessagesState) -> Literal["tools", END]:
    messages = state['messages']
    last_message = messages[-1]
    # 如果 LLM 进行工具调用，则路由到 "tools" 节点
    if last_message.tool_calls:
        return "tools"
    # 否则，我们停止（回复用户）
    return END


# 定义调用模型的函数
def call_model(state: MessagesState):
    messages = state['messages']
    response = model.invoke(messages)
    # 我们返回一个列表，因为这将被添加到现有列表中
    return {"messages": [response]}


# 定义一个新的图
workflow = StateGraph(MessagesState)

# 定义我们将循环遍历的两个节点
workflow.add_node("agent", call_model)
workflow.add_node("tools", tool_node)

# 将入口点设置为 `agent`
# 这意味着此节点是第一个被调用的节点
workflow.add_edge(START, "agent")

# 我们现在添加一个条件边
workflow.add_conditional_edges(
    # 首先，我们定义起始节点。我们使用 `agent`。
    # 这意味着这些是在调用 `agent` 节点后采取的边。
    "agent",
    # 接下来，我们传入一个函数，该函数将确定下一个被调用的节点。
    should_continue,
)

# 我们现在从 `tools` 添加一条普通边到 `agent`。
# 这意味着在调用 `tools` 后，下一个被调用的节点是 `agent` 节点。
workflow.add_edge("tools", 'agent')

# 初始化内存以在图运行之间持久化状态
checkpointer = MemorySaver()

# 最后，我们编译它！
# 这将其编译成一个 LangChain Runnable，
# 意味着您可以像使用任何其他可运行程序一样使用它。
# 请注意，我们（可选地）在编译图时传递内存
app = workflow.compile(checkpointer=checkpointer)

# 使用 Runnable
final_state = app.invoke(
    {"messages": [HumanMessage(content="旧金山的 weather 情况如何？")]},
    config={"configurable": {"thread_id": 42}}
)
final_state["messages"][-1].content
```

```
"根据搜索结果，我可以告诉你旧金山的当前天气是：

温度：60 华氏度
条件：有雾

旧金山以其微气候和频繁的雾气而闻名，尤其是在夏季。60°F（约 15.5°C）的温度对这座城市来说相当典型，这座城市全年气温都比较温和。雾气，当地人称之为“卡尔雾”，是旧金山天气的特征，尤其是在早晨和晚上。

您想了解有关旧金山或其他任何地方天气的其他信息吗？"
```

现在，当我们传递相同的 `"thread_id"` 时，对话上下文将通过保存的状态（即存储的消息列表）保留下来。

```python
final_state = app.invoke(
    {"messages": [HumanMessage(content="纽约怎么样？")]},
    config={"configurable": {"thread_id": 42}}
)
final_state["messages"][-1].content
```

```
"根据搜索结果，我可以告诉你纽约市目前的 weather 是：

温度：90 华氏度（约 32.2 摄氏度）
条件：阳光明媚

这种 weather 与我们在旧金山看到的 weather 相差很大。纽约目前正在经历更高的气温。以下是一些需要注意的要点：

1. 90°F 的温度非常热，是纽约市夏季 weather 的典型情况。
2. 阳光明媚的条件表明天空晴朗，这对户外活动来说很好，但也意味着由于阳光直射，可能会感觉更热。
3. 纽约的这种 weather 通常伴随着高湿度，这会让体感温度比实际温度更高。

有趣的是，旧金山温和多雾的 weather 与纽约炎热阳光明媚的 weather 形成鲜明对比。这种差异说明了即使在同一天，美国不同地区的 weather 也可能差异很大。

您想了解有关纽约或其他任何地方天气的其他信息吗？"
```

### 分步分解

1. <details>
    <summary>初始化模型和工具。</summary>

    - 我们使用 `ChatAnthropic` 作为我们的 LLM。**注意：**我们需要确保模型知道它可以使用这些工具。我们可以通过将 LangChain 工具转换为 OpenAI 工具调用格式来实现，使用 `.bind_tools()` 方法。
    - 我们定义了要使用的工具 - 在我们的例子中是搜索工具。创建自己的工具非常容易 - 请参阅有关如何执行此操作的文档 [这里](https://python.langchain.com/docs/modules/agents/tools/custom_tools)。
   </details>

2. <details>
    <summary>使用状态初始化图。</summary>

    - 我们通过传递状态模式（在我们的例子中是 `MessagesState`）来初始化图 (`StateGraph`)
    - `MessagesState` 是一个预构建的状态模式，它有一个属性 - LangChain `Message` 对象的列表，以及将每个节点的更新合并到状态中的逻辑。
   </details>

3. <details>
    <summary>定义图节点。</summary>

    我们需要两个主要节点：

      - `agent` 节点：负责决定采取哪些（如果有）操作。
      - `tools` 节点调用工具：如果代理决定采取操作，则此节点将执行该操作。
   </details>

4. <details>
    <summary>定义入口点和图边。</summary>

      首先，我们需要设置图执行的入口点 - `agent` 节点。

      然后我们定义一条普通边和一条条件边。条件边意味着目的地取决于图的状态 (`MessageState`) 的内容。在我们的例子中，目的地直到代理 (LLM) 决定才为人所知。

      - 条件边：在调用代理后，我们应该：
        - a. 如果代理说要采取行动，则运行工具，或者
        - b. 如果代理没有要求运行工具，则完成（回复用户）
      - 普通边：在调用工具后，图应该始终返回到代理以决定下一步做什么。
   </details>

5. <details>
    <summary>编译图。</summary>

    - 当我们编译图时，我们将其转换为 LangChain [Runnable](https://python.langchain.com/v0.2/docs/concepts/#runnable-interface)，它会自动启用使用您的输入调用 `.invoke()`、`.stream()` 和 `.batch()`。
    - 我们还可以选择传递 checkpointer 对象来持久化图运行之间的状态，并启用内存、人机交互工作流程、时间旅行等等。在我们的例子中，我们使用 `MemorySaver` - 一个简单的内存中 checkpointer。
    </details>

6. <details>
   <summary>执行图。</summary>

    1. LangGraph 将输入消息添加到内部状态，然后将状态传递给入口点节点 `"agent"`。
    2. `"agent"` 节点执行，调用聊天模型。
    3. 聊天模型返回一个 `AIMessage`。LangGraph 将其添加到状态中。
    4. 图循环执行以下步骤，直到 `AIMessage` 上没有更多 `tool_calls`：

        - 如果 `AIMessage` 有 `tool_calls`，则 `"tools"` 节点执行
        - `"agent"` 节点再次执行并返回 `AIMessage`

    5. 执行进程进入特殊 `END` 值并输出最终状态。
    因此，我们得到一个包含所有聊天消息的列表作为输出。
   </details>


## 文档

* [教程](https://researchrag.github.io/aipm-langgraph/tutorials/): 通过指导示例学习使用 LangGraph 进行构建。
* [操作指南](https://researchrag.github.io/aipm-langgraph/how-tos/): 在 LangGraph 中完成特定的事情，从流式传输到添加内存和持久性，再到常见的设计模式（分支、子图等），如果您想复制和运行特定的代码片段，这些都是您需要去的地方。
* [概念指南](https://researchrag.github.io/aipm-langgraph/concepts/): 深入解释 LangGraph 背后的关键概念和原则，例如节点、边、状态等等。
* [API 参考](https://researchrag.github.io/aipm-langgraph/reference/graphs/): 查看重要的类和方法，使用图和检查点 API 的简单示例，更高级别的预构建组件等等。
* [云 (beta)](https://researchrag.github.io/aipm-langgraph/cloud/): 只需单击一下，即可将 LangGraph 应用程序部署到 LangGraph Cloud。

## 贡献

有关如何贡献的更多信息，请参见 [这里](https://github.com/researchrag/aipm-langgraph/blob/main/CONTRIBUTING.md)。
