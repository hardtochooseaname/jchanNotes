# **OpenAI Function Calling 机制详解**

OpenAI 的 **Function Calling**（函数调用）机制，允许大模型**不仅仅生成文本**，还可以**识别何时调用外部工具（函数）、如何调用，并以结构化 JSON 方式返回调用请求**。（最早在 `gpt-4-0613` 和 `gpt-3.5-turbo-0613` 版本中引入）

## **1. 什么是Function Calling机制**

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

## **2. Function Calling 机制的工作流程**

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



## **3. OpenAI Function Calling 相关参数**

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

> 对于通常在创建大模型或者agent对象时，都可以通过某个参数来控制如何调用工具，不同大模型的api使用的具体参数可能不一样；
>
> 而且对于支持工具调用的大模型来说，通常都会默认优先调用工具来回答用户问题。



# Function Calling 与 Agent

问：大模型通过Function Calling机制可以调用工具，Agent也可以调用工具，两者有什么区别？

答：以最经典的ReAct Agent为例，简单来说，Function Calling 是一种“单步”的工具调用机制，而 ReAct Agent 是一种可以多次调用工具、带有推理能力的智能体。



## **0. ReAct Agent vs. Function Calling**

| **特性**         | **Function Calling** 💡                           | **ReAct Agent** 🤖                |
| ---------------- | ------------------------------------------------ | -------------------------------- |
| **调用方式**     | **一次性调用工具**，由大模型直接返回工具调用指令 | **可以多轮调用工具**，并进行推理 |
| **工具调用决策** | 仅根据当前输入决定是否调用工具                   | 结合历史交互，动态决定调用工具   |
| **是否有循环**   | ❌ **无循环**，每次调用都是独立的                 | ✅ **有循环**，可以多轮调用工具   |
| **适合的任务**   | **简单任务**，比如查天气、翻译、计算             | **复杂任务**，需要多步推理和决策 |
| **底层实现方式** | 依赖 **tool_calls** 机制                         | 采用 **ReAct（推理+行动）** 框架 |



## 1. Function Calling（大模型直接调用工具）

### ✅ **原理**

Function Calling 让大模型**直接输出 JSON 格式的工具调用指令**，然后应用程序解析指令并执行工具。

- 适用于**单步任务**，比如：
  - 查天气、计算数学公式
  - 获取股票数据
  - 翻译文本
- **每次调用都是独立的**，不涉及循环。

### ✅ **示例**

假设用户输入：

```
"现在纽约的天气如何？"
```

如果大模型支持 Function Calling，它会直接返回：

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

然后你的代码解析 `tool_calls` 并调用 `get_weather` 工具返回天气数据，最终返回给用户。



## **2. ReAct Agent（带有推理能力的智能体）**

### ✅ **原理**

- **ReAct**（**Reasoning + Acting**）是一种大模型的 **“思考-行动” 交互框架**。
- **它可以多轮调用工具**，在工具调用结果的基础上**继续思考，直到得到最终答案**。
- 适用于复杂任务，比如：
  - **需要多轮工具调用**的任务（比如先查天气，再推荐衣服）
  - **涉及推理**的任务（比如数学推理、数据分析）

### ✅ **ReAct Agent 的工作流程**

1. 用户输入

   ```
   "我明天去纽约，需要带伞吗？"
   ```

2. 大模型思考

   （使用 ReAct 方式）

   ```
   "我需要先查询纽约明天的天气，然后决定是否需要带伞。"
   ```

3. 调用 `get_weather` 工具

   ```json
   {
     "tool_calls": [
       {
         "name": "get_weather",
         "arguments": {
           "location": "New York",
           "date": "tomorrow"
         }
       }
     ]
   }
   ```

4. 获取天气结果

   （比如：大雨）

   ```
   "纽约明天下雨"
   ```

5. 继续思考

   （结合上下文）

   ```
   "如果明天下雨，应该带伞。"
   ```

6. 最终回答

   ```
   "明天纽约会下雨，建议你带伞。"
   ```

### ✅ **ReAct 的优势**

✔ **能推理**：不会盲目调用工具，而是根据情况决定是否调用工具。
 ✔ **能多轮调用工具**：如果一次工具调用不够，它会继续调用。
 ✔ **更接近人类思考方式**：它会先思考，然后再执行行动。



## **3. 关系 & 何时用哪种？**

| **场景**                         | **推荐使用**     |
| -------------------------------- | ---------------- |
| 只需**一次工具调用**即可完成任务 | Function Calling |
| 任务需要**多轮推理和工具调用**   | ReAct Agent      |
| 工具调用后**还需进一步处理数据** | ReAct Agent      |
| 工具调用的逻辑**简单且固定**     | Function Calling |
| 工具调用**需要动态决策**         | ReAct Agent      |







# 一些常见名词的解释

### **短期推理（Short-Term Reasoning）**

短期推理指的是**在较短的上下文窗口内进行的逻辑推理和决策**，不依赖长期记忆或者跨多轮对话的长期推理能力。

也就是说，用户问题是原封不动给agent回答，不需要结合历史记录或者长期记忆，对用户问题进行补充完善或其他加工处理后，再给agent回答。任何需要

 💡 **简单来说**：它适用于**当前对话上下文**可以完全包含所有决策信息的情况。

🌟 **短期推理的特点**：

- 依赖**当前的上下文**进行推理
- 适用于一次性计算、逻辑推理、信息整合

### **动态决策（Dynamic Decision-Making）**

动态决策指的是Agent **不会** 事先规划好所有工具调用的步骤。而是**执行一步，看结果，再决定下一步怎么做**。它不同于固定的流水线，而是会根据执行情况选择不同的路径。
