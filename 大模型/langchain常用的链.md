# langchain常用的链

在 **LangChain** 中，创建链是构建各种功能的核心步骤。根据具体需求，LangChain 提供了一些常用的方法和函数来创建不同类型的链。这些方法覆盖了从简单的单一任务链到复杂的多步骤链的构建。以下是一些常用的方法及其功能：

------

## **1. 创建基础链**

### **`LLMChain`**

- **功能**：最基础的链，用于直接将输入传递给 LLM，并返回结果。

- **适用场景**：简单的提示生成任务，例如生成文本、总结内容。

- 构造函数

  ：

  ```python
  from langchain.chains import LLMChain
  from langchain.prompts import PromptTemplate
  
  prompt = PromptTemplate.from_template("What is the capital of {country}?")
  llm_chain = LLMChain(llm=llm, prompt=prompt)
  ```

------

## **2. 创建问答链**

### **`RetrievalQA`**

- **功能**：结合向量数据库检索器（Retriever）和 LLM，进行文档问答。

- **适用场景**：基于上下文（如文档或知识库）的问答任务。

- 构造函数

  ：

  ```python
  from langchain.chains import RetrievalQA
  
  qa_chain = RetrievalQA.from_chain_type(
      llm=llm,
      retriever=retriever,
      chain_type="stuff"
  )
  ```

------

## **3. 创建文档整合链**

文档整合链用于处理和整合多个文档的内容。

### **`create_stuff_documents_chain`**

- **功能**：直接将所有文档拼接，传递给 LLM。
- **适用场景**：文档数量少、内容较短。

### **`create_map_reduce_documents_chain`**

- **功能**：逐一处理每个文档（Map 阶段），然后聚合结果（Reduce 阶段）。
- **适用场景**：文档数量多，或内容较长。

### **`create_refine_documents_chain`**

- **功能**：增量改进答案，逐步整合文档内容。
- **适用场景**：答案需要逐步完善，文档内容之间存在强关联。

------

## **4. 创建对话链**

### **`ConversationChain`**

- **功能**：用于维护对话上下文的基础对话链。

- **适用场景**：需要与用户进行多轮对话的任务。

- 构造函数

  ：

  ```python
  from langchain.chains import ConversationChain
  conversation_chain = ConversationChain(llm=llm)
  ```

### **`create_history_aware_retriever`**

- **功能**：结合聊天历史记录和检索器，生成上下文感知的检索链。
- **适用场景**：多轮对话中的动态问题回答。

------

## **5. 创建工具链（工具调用）**

### **`Tool`**

- **功能**：将外部工具封装成链，可以是计算器、搜索引擎、API 调用等。
- **适用场景**：需要与外部资源交互的任务。

### **`initialize_agent`**

- **功能**：结合多个工具，生成一个可以动态选择工具的智能 Agent。

- **适用场景**：多工具任务，如信息检索、问题回答、API 调用。

- 构造函数

  ：

  ```python
  from langchain.agents import initialize_agent, Tool
  
  tools = [Tool(name="search", func=search_function, description="Search the web")]
  agent = initialize_agent(tools, llm, agent="zero-shot-react-description")
  ```

------

## **6. 自定义链**

### **`SequentialChain`**

- **功能**：按顺序执行多个子链，每个子链的输出可以作为下一个子链的输入。

- **适用场景**：需要逐步处理任务。

- 构造函数

  ：

  ```python
  from langchain.chains import SequentialChain
  
  chain1 = LLMChain(llm=llm, prompt=prompt1)
  chain2 = LLMChain(llm=llm, prompt=prompt2)
  
  sequential_chain = SequentialChain(chains=[chain1, chain2])
  ```

### **`SimpleSequentialChain`**

- **功能**：简化的顺序链，将多个任务按顺序处理。
- **适用场景**：简单的任务链式处理。

### **`Custom Chain`**

- **功能**：通过继承 `Chain` 基类，构建完全自定义的链。
- **适用场景**：需要特殊逻辑或高级功能。

------

## **7. 高级链**

### **`Retrieval-based Generation`**

- **功能**：将检索和生成任务结合，用于复杂的文本生成。
- **适用场景**：需要先从数据库中检索相关信息，再生成答案。

### **`ReAct (Reasoning + Acting)`**

- **功能**：实现推理与行动的交替循环。
- **适用场景**：需要动态决策的任务，如复杂的工具调用。

------

### **总结**

LangChain 提供了从基础链到复杂链的一系列构建工具，满足各种任务需求：

| **类别**   | **代表函数**                        | **适用场景**       |
| ---------- | ----------------------------------- | ------------------ |
| 基础链     | `LLMChain`                          | 直接生成任务       |
| 问答链     | `RetrievalQA`                       | 文档问答任务       |
| 文档整合链 | `create_stuff_documents_chain`      | 小规模文档整合     |
|            | `create_map_reduce_documents_chain` | 大规模文档整合     |
|            | `create_refine_documents_chain`     | 需要逐步改进答案   |
| 对话链     | `ConversationChain`                 | 多轮对话           |
| 工具链     | `initialize_agent`                  | 工具调用、任务分发 |
| 顺序链     | `SequentialChain`                   | 逐步处理的任务链   |

根据具体需求，可以灵活选择和组合这些方法来创建功能强大的链式应用。