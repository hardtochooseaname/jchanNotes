# **OpenAI Function Calling 机制详解**

OpenAI 的 **Function Calling**（函数调用）机制，最早在 `gpt-4-0613` 和 `gpt-3.5-turbo-0613` 版本中引入，允许大模型**不仅仅生成文本**，还可以**识别何时调用外部工具（函数）、如何调用，并以结构化 JSON 方式返回调用请求**。

## **🔹 为什么需要 Function Calling？**

在 Function Calling 出现前，LLM 只能通过**普通文本交互**来回答问题，例如：

```plaintext
User: 现在纽约的天气怎么样？
Assistant: 你可以访问 weather.com 查询天气。
```

但是这样不够智能，用户希望 **LLM 直接调用天气 API 获取实时天气**。

有了 Function Calling，模型可以返回：

```json
{
  "tool_calls": [
    {
      "name": "get_weather",
      "arguments": {
        "location": "New York"
      }
    }
  ]
}
```

然后应用代码可以**解析 JSON 并调用 `get_weather` 函数**，再把结果返回给用户。

------

## **🔹 Function Calling 机制的工作流程**

使用 OpenAI 的 Function Calling，一般按照以下 **3 个步骤** 进行：

### **✅ 1. 定义工具**

你需要用 JSON Schema 定义可以调用的工具：

```python
from openai import OpenAI

client = OpenAI(api_key="your_api_key")

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "获取某个城市的实时天气",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "要查询天气的城市"
                    }
                },
                "required": ["location"]
            }
        }
    }
]
```

> **解释**：
>
> - `name` 是函数名（模型会根据名字判断何时调用）。
> - `description` 让 LLM 知道该工具的作用。
> - `parameters` 定义 JSON Schema，让 LLM 知道需要哪些参数。

------

### **✅ 2. 让 LLM 选择是否调用工具**

然后我们向 LLM 发送用户问题，并让它决定是否调用工具：

```python
response = client.chat.completions.create(
    model="gpt-4-turbo",
    messages=[{"role": "user", "content": "现在纽约的天气如何？"}],
    tools=tools
)
```

如果 LLM 认为应该调用 `get_weather`，它不会直接回答，而是返回：

```json
{
  "tool_calls": [
    {
      "name": "get_weather",
      "arguments": {
        "location": "New York"
      }
    }
  ]
}
```

------

### **✅ 3. 解析工具调用 & 执行函数**

当 `tool_calls` 不为空时，应用程序就可以**解析 JSON，调用实际的 `get_weather` API**：

```python
import json

def get_weather(location):
    return f"{location} 的天气是晴天，气温 25°C"

# 解析 LLM 返回的工具调用
tool_call = response.tool_calls[0]
tool_name = tool_call["name"]
tool_args = json.loads(tool_call["arguments"])

# 执行工具
if tool_name == "get_weather":
    result = get_weather(**tool_args)
    print("结果:", result)
```

最终，应用程序会返回：

```plaintext
纽约的天气是晴天，气温 25°C
```

------

## **🔹 OpenAI Function Calling 相关参数**

在调用 OpenAI API 时，你可以通过 `tool_choice` 控制模型是否调用工具：

```python
response = client.chat.completions.create(
    model="gpt-4-turbo",
    messages=[{"role": "user", "content": "现在纽约的天气如何？"}],
    tools=tools,
    tool_choice="auto"  # 让 LLM 自动决定是否调用工具
)
```

`tool_choice` 可选：

- `"auto"`：**默认值**，模型自行判断是否调用工具（推荐）。
- `"none"`：模型**不会调用工具**，仅仅返回文本回答。
- `{"name": "get_weather"}`：强制**调用指定工具**。

------

## **🔹 Function Calling 模式下，模型如何决定是否调用工具？**

大模型**默认**会根据以下情况决定是否调用工具：

1. **任务是否超出 LLM 直接回答的能力**（如实时数据、数据库查询、计算等）。
2. **工具描述是否匹配用户问题**（模型会根据工具 `description` 进行匹配）。
3. **历史聊天记录是否暗示需要调用工具**（如用户多次询问天气）。

但是，你**可以通过 `tool_choice="none"` 强制禁用工具调用**，或者使用提示词影响 LLM 的决策：

```plaintext
"你必须自己回答，而不能调用工具。"
```

------

## **🔹 是否所有支持 Function Calling 的模型，都会优先考虑工具调用？**

> **不一定！** **不同模型 & 不同框架的工具调用方式不同**，有些**默认优先考虑工具调用**，有些则不会。

### **✅ 1. OpenAI `gpt-4-turbo` (Function Calling 模式)**

- **默认**情况下，模型会**自行决定**是否调用工具。
- **如果工具非常匹配问题，它会优先调用工具，而不是自己生成答案**。

### **✅ 2. LangChain `AgentType.OPENAI_FUNCTIONS`**

在 LangChain 中，`AgentType.OPENAI_FUNCTIONS` **优先考虑**工具调用：

```python
agent = initialize_agent(
    tools=[weather_tool],
    llm=llm,
    agent=AgentType.OPENAI_FUNCTIONS
)
```

这会强制模型**始终考虑工具调用**，尽量不自己生成文本。

### **✅ 3. LangGraph**

LangGraph 允许你手动控制工具调用，**默认不会强制工具调用**，而是让 LLM 自行决定。你可以：

- 在 `tool_choice="auto"` 时，模型会根据问题**决定是否调用工具**。
- 也可以手动处理 LLM 返回的 `tool_calls`，然后执行工具。

### **✅ 4. 其他支持工具调用的模型**

目前，除了 OpenAI 的 `gpt-4-turbo`，**Anthropic Claude、Mistral 等模型也开始支持 Function Calling**，但：

- Claude 3 的 Function Calling **需要明确的提示词**才能调用工具。
- Mistral-7B、Gemini 1.5 也在尝试支持类似的功能，但并不默认优先调用工具。

> **总结**：
>
> - **OpenAI `gpt-4-turbo`（API）** 在 Function Calling 模式下**优先考虑**调用工具。
> - **LangChain `AgentType.OPENAI_FUNCTIONS`** 会让 LLM **尽可能调用工具**，而不是自己生成答案。
> - **LangGraph 允许灵活控制**，默认不会强制工具调用。

------

## **🔹 总结**

✅ **Function Calling 让 LLM 不仅能回答问题，还能调用 API**，提供更强大的智能应用能力。
 ✅ **OpenAI `gpt-4-turbo` 默认会自行决定是否调用工具**，但可以通过 `tool_choice` 控制。
 ✅ **LangChain `AgentType.OPENAI_FUNCTIONS` 强制 LLM 优先考虑工具调用**。
 ✅ **LangGraph 允许更灵活的工具调用逻辑**，默认不会强制工具调用。

如果你想深入研究 **LangChain / LangGraph** 结合 Function Calling 的高级用法，可以继续问！🚀