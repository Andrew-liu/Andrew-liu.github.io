---
title: 我用 50 行代码重新理解了 Agent：核心不是框架，是 messages + tools + loop
date: 2026-07-27 12:00:00
tags: [Agent, Tool Calling, OpenAI SDK, mini-agent]
---

> 项目：<https://github.com/Andrew-liu/mini-agent>

## 01｜Agent 没有想象中那么神秘

前几天看了李博杰老师的《深入理解 AI Agent》，了解了`Agent = LLM + 上下文 + 工具`这一核心概念。但是对Agent到底怎么运行的还是很好奇，于是今天花了点时间写了一个`mini agent`，核心函数只有50行左右。

项目链接在这里：<https://github.com/Andrew-liu/mini-agent>

它不接 LangChain，不接 AutoGen，也没有一堆抽象类。核心就是 Python + OpenAI SDK + Tool Calling。完整的 `agent.py` 因为加了 CLI、日志、兼容接口处理和安全提示，大概 200 多行；但如果只看 Agent 的最小闭环，其实 50 行左右就能讲清楚。

我写这个项目时最大的感受是：Agent 不是一个新物种。它就是让模型在回答前，有机会先调用几个外部函数。模型负责判断“要不要用工具、用哪个工具、参数是什么”，Python 负责真正执行工具，然后把结果塞回上下文里，让模型继续回答。

这件事拆开以后，只有四块：`messages`对应上下文、`tools schema`和`tool dispatcher`对应工具、`loop`。

![](/image/01-小黑拆解Agent外壳.png)

<!-- more -->

## 02｜50 行 Agent 的骨架

压缩到最小，一个 Agent 大概长这样：

```python
messages = [
    {"role": "system", "content": SYSTEM_PROMPT},
    {"role": "user", "content": question},
]

for step in range(max_steps):
    resp = client.chat.completions.create(
        model=model,
        messages=messages,
        tools=TOOL_SCHEMAS,
        tool_choice="auto",
    )

    msg = resp.choices[0].message
    messages.append({
        "role": "assistant",
        "content": msg.content,
        "tool_calls": msg.tool_calls,
    })

    if not msg.tool_calls:
        return msg.content

    for call in msg.tool_calls:
        name = call.function.name
        args = json.loads(call.function.arguments)
        result = TOOL_FUNCTIONS[name](**args)

        messages.append({
            "role": "tool",
            "tool_call_id": call.id,
            "content": str(result),
        })

raise RuntimeError("Agent did not finish")
```

这段代码里最关键的不是 `for` 循环，而是 `messages.append()` 的顺序。

模型第一次看到的是：system 规则 + user 问题。它如果觉得自己能直接回答，就返回普通 assistant 文本。它如果觉得需要工具，就返回一个 assistant 消息，里面带着 `tool_calls`。Python 执行工具以后，再追加一条 `role="tool"` 的消息。下一轮模型再看到完整上下文，才知道工具返回了什么。

所以 Agent 的“记忆”不是玄学，它就是一个dict。

![alt text](/image/02-小黑修补messages日志.png)

## 03｜messages 是 Agent 的黑匣子记录

很多人第一次写 Tool Calling，容易把 `messages` 当成聊天记录。这个理解对一半。

它确实是聊天记录，但在 Agent 里，它更像一份执行日志：谁提出了任务，模型做了什么决定，工具返回了什么观察结果，最后模型基于这些信息给出答案。

一个完整的工具调用链路通常是这样：

```text
system: 你是助手，需要时可以使用工具
user: 请计算 (123 + 456) * 2
assistant: 我要调用 calculate，参数是 {"expression":"(123 + 456) * 2"}
tool: 1158
assistant: 结果是 1158
```

注意这里的第三行。模型不是直接算出答案，而是先生成一个 `assistant` 消息，里面带 `tool_calls`。这条消息也必须放回 `messages`。很多人会漏掉这一步，只把 tool 结果塞回去，结果上下文就断了：模型看到了工具结果，却看不到自己刚才请求过哪个工具。

`tool_call_id` 也是同样的道理。一次 assistant 响应里可能有多个工具调用，比如同时查时间、算表达式、搜网页。每个工具结果都要带上对应的 `tool_call_id`，告诉模型：这条结果对应刚才哪一次调用。

## 04｜system、user、assistant、tool 分别在干什么

我更喜欢把这四种 role 理解成四种不同的“上下文来源”。

`system` 是运行规则。它适合放角色、边界、工具使用原则。`mini-agent` 里的 system prompt 很短：你是一个 helpful assistant；工具能提高准确性时就用；拿到工具结果后清楚回答用户。我没有写一大堆人格设定，因为这个项目的目的不是调教文风，而是让工具调用链路尽量干净。

`user` 是任务输入。用户问了什么、要求是什么、附带了哪些约束，都在这里。对单轮 CLI Agent 来说，初始 `messages` 里通常只有一条 user 消息。

`assistant` 是模型的行动记录。它可能是一段最终回答，也可能是一次工具调用请求。Tool Calling 里最容易误解的点就在这里：调用工具这件事，本身也是 assistant 的输出。模型没有真的执行函数，它只是说：“我想调用这个函数，参数是这些。”

`tool` 是外部世界返回的观察结果。它不是用户说的话，也不是模型说的话，而是 Python 函数执行完之后给模型看的结果。比如当前时间、计算结果、搜索结果、数据库查询结果，都应该作为 tool 消息回填。

把这四种角色分清楚，Agent 的上下文就清楚了。system 定规则，user 出题，assistant 决策，tool 回报现实。

## 05｜tools 不是函数本身，而是给模型看的说明书

`mini-agent` 里有三个工具：

- `get_current_time()`：返回当前本地时间
- `calculate(expression)`：计算基础算术表达式
- `web_search(query)`：用 `ddgs` 做网页搜索，最多返回 5 条

但模型看不到 Python 函数源码。它看到的是 `TOOL_SCHEMAS`，也就是一组 JSON Schema。

比如 `calculate` 对模型来说大概是这样：

```json
{
  "type": "function",
  "function": {
    "name": "calculate",
    "description": "Calculate a basic arithmetic expression.",
    "parameters": {
      "type": "object",
      "properties": {
        "expression": {
          "type": "string",
          "description": "Arithmetic expression, for example: (15 + 3) * 2"
        }
      },
      "required": ["expression"],
      "additionalProperties": false
    }
  }
}
```

这就是工具设计里很容易被低估的地方。

你给模型看的不是“能力”，而是能力的说明书。工具名、description、参数名、参数描述，都会影响模型怎么调用它。如果工具描述很模糊，模型就会乱填参数；如果参数结构太松，模型就会塞一些你没打算支持的字段。

所以我一般会把工具拆成两层：

```text
TOOL_SCHEMAS：给模型看的工具说明
TOOL_FUNCTIONS：本地真正执行的 Python 函数表
```

模型输出工具名以后，Python 再从 `TOOL_FUNCTIONS` 里找真实函数：

```python
TOOL_FUNCTIONS = {
    "get_current_time": get_current_time,
    "calculate": calculate,
    "web_search": web_search,
}
```

这个dict就是模型和本地能力之间的桥。

![alt text](/image/03-小黑提交工具调用.png)

## 06｜tool call 的本质：模型写参数，程序跑函数

Tool Calling 听起来像模型拥有了工具，其实更准确地说，是模型拥有了“申请调用工具”的能力。

流程是这样的：

1. 你把 `tools=TOOL_SCHEMAS` 发给模型。
2. 模型决定是否调用工具。
3. 如果要调用，它返回 `tool_calls`，里面有工具名和参数 JSON 字符串。
4. Python 解析参数，找到本地函数，执行。
5. Python 把执行结果作为 `role="tool"` 消息回填。
6. 模型基于工具结果继续回答。

这里有个边界很重要：工具是 Python 执行的，不是模型执行的。

所以权限、安全、超时、沙箱、错误处理，都不能指望模型自己做好。模型只是在生成一次函数调用请求。真正决定能不能执行、怎么执行、错了怎么办的人，是你写的 dispatcher。

`mini-agent` 里有一个 `execute_tool()`：它会解析 JSON 参数，检查参数是不是 object，根据工具名查函数，执行函数。如果出错，它不会直接把程序炸掉，而是返回一段 `Tool error: ...` 字符串。这样模型下一轮仍然能看到错误，并尝试修正。

教学项目里这已经够用了。生产环境里还要继续加：超时、重试、权限、参数校验、日志脱敏、危险工具确认。

## 07｜提示词不要替代码背锅

我以前也犯过一个错：Agent 行为不稳定，就想往 system prompt 里继续塞规则。

后来发现，有些问题不该靠提示词解决。比如工具参数是否合规，应该靠 JSON Schema 和参数校验。工具能不能访问文件，应该靠权限系统。循环会不会无限跑，应该靠 `max_steps`。敏感信息会不会进日志，应该靠日志策略。

提示词适合管策略，不适合管边界。

`mini-agent` 的 system prompt 很短，但它抓住了两件事：第一，需要时使用工具；第二，拿到工具结果后清楚回答用户。这就够了。剩下的事情交给代码：`tool_choice="auto"` 让模型自己决定是否调用工具，`max_steps=8` 防止无限循环，`TOOL_SCHEMAS` 限制工具参数，`TOOL_FUNCTIONS` 控制真实可执行能力。

如果我要给这个项目加一条更实用的提示词，我会写得很具体：

```text
Use tools for current facts, calculations, and web information.
Do not guess tool results. If a tool returns an error, explain the limitation or try a safer alternative.
```

这类提示词是在告诉模型“什么时候该用工具”。但“工具能干什么、能干到什么程度”，还是应该由 schema 和代码决定。

## 08｜为什么我觉得这个最小项目值得看

现在很多 Agent 框架把事情包得很好，但也容易把初学者绕晕。你看到的是 memory、planner、executor、graph、workflow、multi-agent，一层套一层，反而忘了最底层那条链路。

这也是我写 `mini-agent` 的原因。先别急着上框架，先把这条链路跑明白：

```text
messages 记录上下文
schema 告诉模型有哪些工具
tool_calls 表示模型想调用什么
dispatcher 执行真实函数
tool 消息把结果回填给模型
loop 让模型继续推理直到回答完成
```

理解这条链路以后，再看 LangChain、AutoGen、各种 Agent Runtime，会轻松很多。它们当然更完整，但底层逃不开这几个动作。差别主要在工程能力：上下文压缩、持久化、重试、并行工具、权限、监控、评估、成本控制。

Agent 的复杂度，很多时候不是来自模型，而是来自上下文治理。

![alt text](/image/04-小黑锁住代码边界.png)

## 09｜最后

如果你想真正理解 Agent，我建议先不要从框架开始。

先写一个 50 行版本：一个 `messages` 列表，一个 `tools` schema，一个工具函数字典，一个循环。让模型调用一次计算器，再调用一次搜索，再把结果回填，最后回答用户。

当你亲手看到 `assistant -> tool -> assistant` 这条消息链跑起来，很多概念就突然落地了。

Agent 不是“模型变聪明了”。

更准确地说，是我们给模型接上了一套可以被记录、被约束、被回放的行动链路。

项目放在这里：<https://github.com/Andrew-liu/mini-agent>

---

## 参考来源

1. `mini-agent` GitHub 仓库：<https://github.com/Andrew-liu/mini-agent>
2. `agent.py` 源码：<https://raw.githubusercontent.com/Andrew-liu/mini-agent/master/agent.py>
3. `README.md`：<https://raw.githubusercontent.com/Andrew-liu/mini-agent/master/README.md>
4. OpenAI Tool Calling / Chat Completions 风格接口：项目源码中基于 `client.chat.completions.create(...)` 使用
