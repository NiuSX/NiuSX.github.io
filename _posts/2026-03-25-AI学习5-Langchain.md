---
title: "AI学习(五）- Agent框架之Langchain(未完待续）"
date: 2026-03-25
categories:
  - AI
excerpt: ""
---


# 1. 参考文档
LangChain 模型接口可参考官方文档：https://reference.langchain.com/python/langchain_core/language_models/

LangChain 提示词官方文档参考：https://reference.langchain.com/python/langchain_core/prompts/

LangChain 输出解析器可参考文档：https://reference.langchain.com/python/langchain_core/output_parsers/

langChain 链式调用可参考文档：https://reference.langchain.com/python/langchain_core/runnables/

langchain 记忆存储相关内容，可参考文档：https://docs.langchain.com/oss/python/langchain/short-term-memory

LangChain内置工具列表：https://docs.langchain.com/oss/python/integrations/tools

langchain 内置第三方搜索库:https://python.langchain.com/docs/integrations/tools/#search

**Ollama Embedding **可参考文档：https://python.langchain.com/api_reference/community/embeddings/langchain_community.embeddings.ollama.OllamaEmbeddings.html


# 2.Model大模型接口

LangChain 模型接口可参考官方文档：https://reference.langchain.com/python/langchain_core/language_models/

## Model的分类

LangChain中将大语言模型分为以下几种，我们主要使用的是聊天模型：

| 模型类型                    | 输入形式                                                     | 输出形式                            | 主要特点                                                     | 典型适用场景                                                 |
| --------------------------- | ------------------------------------------------------------ | ----------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| LLM（Large Language Model） | 纯文本字符串                                                 | 文本字符串                          | 1. 最基础的文本生成模型 2. 无上下文记忆 3. 高速、轻量        | 1. 单轮问答 2. 摘要生成 3. 文本改写/扩写 4. 指令执行（Instruct 模型） |
| ChatModel（聊天模型）       | 消息列表（`List[BaseMessage]`）包括 `HumanMessage`, `SystemMessage`, `AIMessage`等 | 聊天消息对象（`AIMessage` ）        | 1. 面向对话场景 2. 支持多轮上下文 3. 更贴近人类对话逻辑      | 1. 智能助手 2. 客服机器人 3. 多轮推理任务 4. LangChain Agent 工具调用 |
| Embeddings（文本向量模型）  | 文本字符串或列表（`str`或 `List[str]`）                      | 向量（`List[float]` 或 `ndarray` ） | 1. 将文本转化为语义向量 2. 可用于相似度搜索 3. 通常不生成文本 | 1. 文本检索增强（RAG） 2. 知识库问答 3. 聚类 / 分类 / 推荐系统 |



## Model继承体系

在 LangChain 的类结构中，顶层基类是 `BaseLanguageModel`，用于定义模型的通用接口。它分为两支：`BaseChatModel` 和 `BaseLLM`

接入聊天模型时需继承 `BaseChatModel`，如常用的 `ChatOpenAI`；而文本生成模型则继承 `BaseLLM`，如 `OpenAI`。

## 创建环境变量

创建.env 文件内容如下：

```txt
DEEPSEEK_BASE_URL=xxx
DEEPSEEK_API_KEY=xxx
MODEL=xxx
```

加载环境变量:

```python
import os
from dotenv import load_dotenv 
load_dotenv(override=True)
deepseek_api_key = os.getenv("DEEPSEEK_API_KEY")
```

## Chat Model 主要参数

| 参数名      | 参数含义                                                     |
| ----------- | ------------------------------------------------------------ |
| model       | 指定使用的大语言模型名称（如 `"gpt-4"`、`"gpt-3.5-turbo"` 等） |
| temperature | 温度，温度越高，输出内容越随机；温度越低，输出内容越确定     |
| timeout     | 请求超时时间                                                 |
| max_tokens  | 生成内容的最大token数                                        |
| stop        | 模型在生成时遇到这些“停止词”将立刻停止生成，常用于控制输出的边界。 |
| max_retries | 最大重试请求次数                                             |
| api_key     | 大模型供应商提供的API秘钥                                    |
| base_url    | 大模型供应商API 请求地址                                     |

**接入大模型：**

不同平台接口兼容不同：

| 组件             | 接口格式       | 说明                                       |
| :--------------- | :------------- | :----------------------------------------- |
| **OpenAI**       | 标准格式       | 很多平台都兼容这种格式                     |
| **DeepSeek官方** | 兼容OpenAI格式 | DeepSeek官方API本身就是兼容OpenAI格式的    |
| **百炼平台**     | 兼容OpenAI格式 | 百炼为了开发者方便，也实现了OpenAI兼容接口 |
| **ChatDeepSeek** | DeepSeek专用   | 可能有一些DeepSeek特定的实现细节           |

使用bailian平台:

```python
from langchain_openai import ChatOpenAI
model = ChatOpenAI(
    model=MODEL,
    temperature=0,
    max_tokens=None,
    timeout=None,
    max_retries=2,
    base_url=BAILIAN_BASE_URL,
    api_key=BAILIAN_API_KEY,
)
print(model.invoke("什么是LangChain?"))
```

使用deepseek厂商专用：

```python
from langchain_deepseek import ChatDeepSeek
model = ChatDeepSeek(
    model="deepseek-chat",
    temperature=0,
    max_tokens=None,
    timeout=None,
    max_retries=2,
    base_url=DEEPSEEK_BASE_URL,
    api_key=DEEPSEEK_API_KEY,
)
print(model.invoke("什么是LangChain?"))
```

**注意:**同一模型在不同平台名称可能不同

设置本地部署的模型：

```python
from langchain_ollama import ChatOllama
# 设置本地模型，不使用深度思考
model = ChatOllama(base_url="http://localhost:11434", model="qwen3:14b", reasoning=False)
```



## Message组件

调用模型后返回了一条AI消息，在LangChain中，消息有几种不同的类型。所有消息都有 `type` 、 `content` 、 `response_metadata` 等属性。

- **HumanMessage**：人类消息，type为"user"，表示来自用户输入。比如“实现 一个快速排序方法”。 
- **AIMessage**： AI 消息，type为"ai"，这可以是文本，也可以是调用工具的请求。
-  **SystemMessage**：系统消息，type为"system"，告诉大模型当前的背景是什么，应该如何做，并不是所有模型提供商都支持这个消息类型 
- **ToolMessage/FunctionMessage**：工具消息，type为"tool"，用于函数调用结果的消息类型 
- **ChatMessage**：可以自定义角色的通用消息类型。 


| 属性名            | 属性作用                                                     |
| ----------------- | ------------------------------------------------------------ |
| type              | 描述了是哪个类型的消息，包含类型有"user"、“ai”、“system” 和 “tool” |
| content           | 通常是字符串，有些情况下可能是字典列表，这个字典列表用于大模型的多模态输出。 |
| name              | 用来区分当消息类型相同，对消息进行区分，但不是所有模型都支持这一功能。 |
| response_metadata | AI消息才会包含的属性，大语言模型的响应中附加元数据，根据不同模型会有不同，如可能会包含本次 token 使用量等信息。 |
| tool_calls        | AI消息才会包含的属性，当大语言模型决定调用工具时，在 `AIMessage` 中就会包含这个属性，可以通过 `.tool_calls` 属性进行获取该属性返回一个 `ToolCall` 列表，每个 `ToolCall` 是一个字典，包含以下字段：`name` :应调用的工具名`args` : 调用工具的参数`id` : 工具调用的唯一标识 ID |



## 模型调用方法

### 对话模型

```python
# 构建消息列表
messages = [SystemMessage(content="你叫小亮，是一个乐于助人的人工助手"),
            HumanMessage(content="你是谁")
            ]
# 同步调用大模型
response = model.invoke(messages)
# 打印结果
print(response.content)
print(type(response))
```

### 非流式输出(invoke)

同步调用，结果一次性全部输出，这是Langchain与LLM交互时的默认行为，是最简单、最稳定的语言模型调用方式。当用户发出请求后，系统在后台等待模型生成完整响应，然后一次性将全部结果返回。在大多数问答、摘要、信息抽取类任务中，非流式输出提供了结构清晰、逻辑完整的结果，适合快速集成和部署

```python
response = model.invoke(messages)
```

### 流式输出(stream)

一种更具交互感的模型输出方式，用户不再需要等待完整答案，而是能看到模型逐个token 地实时返回内容。  适合构建强调“实时反馈”的应用，如聊天机器人、写作助手等。

```python
'''流式输出'''
response = model.stream(messages)
for chunk in response:
    print(chunk.content, end="",flush=True) # 刷新缓冲区 (无换行符，缓冲区未刷新，内容可能不会立即显示)
print("\n")
print(type(response))
```

### 批量调用(batch)

ngChain 支持 **批量调用（Batch Inference）**，也就是**一次性向模型提交多个输入并并行处理**，从而显著提升吞吐量。

```python
# 批量调用大模型
response = model.batch(batch_input)
for q, r in zip(questions, response):
    print(f"问题：{q}\n回答：{r}\n")
```



### 异步调用(ainvoke)

LangChain 提供 `ainvoke()` 异步调用接口，用于在 异步环境（async/await） 中高效并行地执行模型推理。
它的核心作用是：让你同时调用多个模型请求而不阻塞主线程 —— 特别适合大批量请求或 Web 服务场景（如 FastAPI）。

```python
async def main():
    # 异步调用一条请求
    response = await model.ainvoke("解释一下LangChain是什么")
    print(response)
    
# 运行异步程序的入口点
asyncio.run(main())
```

# 3.PromptTemplate提示词模板

LangChain 提示词官方文档参考：https://reference.langchain.com/python/langchain_core/prompts/

## 提示词模板分类

- `PromptTemplate`：文本生成模型提示词模板，用字符串拼接变量生成提示词
- `ChatPromptTemplate`：聊天模型提示词模板，适用于如 `gpt-3.5-turbo`、`gpt-4` 等聊天模型
- `HumanMessagePromptTemplate`：人类消息提示词模板
- `SystemMessagePromptTemplate`：系统消息提示词模板
- `FewShotPromptTemplate`：少样本学习提示词模板， 构建一个 Prompt，其中包含多个 示例，可以 自动将这些示例格式化并插入到主 Prompt 中 。
- `PipelinePrompt`：管道提示词模板，用于把几个提示词组合在一起使用。

## 模板类继承关系

在 LangChain 的类结构中，顶层基类是 `BasePromptTemplate`，用于定义Prompt 模板系统必须实现的核心方法，而`StringPromptTemplate` 和 `BaseChatPromptTemplate`两个子类分别继承。

**接入聊天模型时需继承`BaseChatPromptTemplate`；而文本生成模型则继承`StringPromptTemplate` 。**

## 文本提示词模板

`PromptTemplate` 针对文本生成模型的提示词模板，也是LangChain提供的最基础的模板，通过格式化字符串生成提示词，在执行invoke时将变量格式化到提示词模板中

主要参数：

- template：定义提示词模板的字符串，其中包含文本和变量占位符（如{name}） ；

- input_variables： 列表，指定了模板中使用的变量名称，在调用模板时被替换；

- partial_variables：字典，用于定义模板中一些固定的变量名。这些值不需要再每次调用时被替换。

函数介绍：

format()：给input_variables变量赋值，并返回提示词。利用format() 进行格式化时就一定要赋值，否则会报错。当在template中未设置input_variables，则会自动忽略。

### 创建提示词

**使用构造方法**

```python
# 创建一个PromptTemplate对象，用于生成格式化的提示词模板
# 该模板包含两个变量：role（角色）和question（问题）
template = PromptTemplate(template="你是一个专业的{role}工程师，请回答我的问题给出回答，我的问题是：{question}",
                        input_variables=['role', 'question'])

# 使用模板格式化具体的提示词内容
# 将role替换为"python开发"，question替换为"冒泡排序怎么写？"
prompt = template.format(role="python开发",question="冒泡排序怎么写？")

# 输出格式化后的提示词内容
print(prompt)
```

**调用from_template(常用)**

```python
# 创建一个PromptTemplate对象，用于生成格式化的提示词模板
# 模板包含两个占位符：{role}表示角色，{question}表示问题
template = PromptTemplate.from_template("你是一个专业的{role}工程师，请回答我的问题给出回答，我的问题是：{question}")

# 使用指定的角色和问题参数来格式化模板，生成最终的提示词字符串
# role: 工程师角色描述
# question: 具体的技术问题
prompt = template.format(role="python开发",question="冒泡排序怎么写？")

# 输出生成的提示词
print(prompt)
```

###  部分提示词模板

```python
# 创建一个包含时间变量的模板，时间变量使用partial_variables预设为当前时间
# 然后格式化问题生成最终提示词
template1 = PromptTemplate.from_template("现在时间是：{time},请对我的问题给出答案，我的问题是：{question}",
partial_variables={"time": datetime.now().strftime("%Y-%m-%d %H:%M:%S")})
prompt1 = template1.format(question="今天是几号？")
print(prompt1)

# 创建一个包含时间变量的模板，通过partial方法预设时间变量为当前时间
# 然后格式化问题生成最终提示词
template2 = PromptTemplate.from_template("现在时间是：{time},请对我的问题给出答案，我的问题是：{question}")
partial = template2.partial(time=datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
prompt2 = partial.format(question="今天是几号？")
print(prompt2)
```

### 组合提示词模板

```python
# 创建一个PromptTemplate模板，用于生成介绍某个主题的提示词
# 该模板包含两个占位符：topic（主题）和length（字数限制）
template1 = PromptTemplate.from_template("请用一句话介绍{topic}，要求通俗易懂\n") + "内容不超过{length}个字"
# 使用format方法填充模板中的占位符，生成具体的提示词
prompt1 = template1.format(topic="LangChain", length=20)
print(prompt1)

# 分别创建两个独立的PromptTemplate模板
prompt_a = PromptTemplate.from_template("请用一句话介绍{topic}，要求通俗易懂\n")
prompt_b = PromptTemplate.from_template("内容不超过{length}个字")
# 将两个模板进行拼接组合
prompt_all = prompt_a + prompt_b
# 填充组合后模板的占位符，生成最终的提示词
prompt2 = prompt_all.format(topic="LangChain", length=20)
print(prompt2)
```

### 提示词方法

**方法**

- invoke：格式化提示词模板为PromptValue  
- format：格式化提示词模板为字符串  
- partial：格式化提示词模板为一个新的提示词模板，可以继续进行格式化  ,在部分提示词模板下用

## 对话提示词模板

### 创建提示词

**构造方法**

```python
# 创建聊天提示模板，包含系统角色设定、用户询问和AI回答的对话历史
# 以及用户当前输入的占位符
prompt_template = ChatPromptTemplate([
    ("system", "你是一个AI助手，你的名字是{name}"),
    ("human", "你能做什么事"),
    ("ai", "我可以陪你聊天，讲笑话，写代码"),
    ("human", "{user_input}"),
])

# 使用指定的参数格式化提示模板，生成最终的提示字符串
# name: AI助手的名称
# user_input: 用户的当前输入
prompt = prompt_template.format(name="小张", user_input="你可以做什么")
print(prompt)
```

**调用from_message(常用)**

```python
# 创建聊天提示模板，包含系统角色设定和用户问题格式
# 系统消息定义了AI的角色，人类消息定义了问题的输入格式
chat_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个{role}，请回答我提出的问题"),
    ("human", "请回答:{question}")
])

# 使用指定的角色和问题参数填充模板，生成具体的提示内容
# role: 指定AI扮演的角色
# question: 用户提出的具体问题
prompt_value = chat_prompt.invoke({"role": "python开发工程师", "question": "冒泡排序怎么写"})

# 输出生成的提示内容
print(prompt_value.to_string())
```

### 提示词方法

- format_messages

  作用：将模板变量替换后，直接生成 消息列表（`List[BaseMessage]`），一般包含：`SystemMessage``HumanMessage``AIMessage`

  常用场景：用于手动查看或调试 Prompt 的最终“消息结构”，或者自己拼接进 Chain

- format_prompt

  作用：生成一个 `PromptValue` 对象，这是一种抽象层次更高的封装。

  - 对于 `PromptTemplate`（单纯文本），返回 `StringPromptValue`
  - 对于 `ChatPromptTemplate`（对话模板），返回 `ChatPromptValue`

  `PromptValue` 有两个常用方法：

  - `.to_string()` → 转成文本
  - `.to_messages()` → 转成消息列表（同上）

  返回值：`PromptValue` 对象

### 实例化参数类型

- str类型 列表参数格式是str类型（不推荐），因为默认都是HumanMessage。

- dict类型

- message类型 `System/Human/AIMessage` 是 `langchain` 中用于构建不同角色的一个类。它通常用于创建聊天消息的一部分，特别是当你构建一个多轮对话的 prompt 模板时，区分系统、AI、和人类消息。

- BaseChatPromptTemplate 类型
  使用 BaseChatPromptTemplate，可以理解为ChatPromptTemplate里嵌套了ChatPromptTemplate
  
- BaseMessagePromptTemplate 类型

  LangChain提供不同类型的MessagePromptTemplate。最常用的是SystemMessagePromptTemplate 、HumanMessagePromptTemplate 和AIMessagePromptTemplate ，分别创建系统消息、人工消息和AI消息，它们是ChatMessagePromptTemplate的特定角色子类。
  
  
  
  **基本概念：**
  

HumanMessagePromptTemplate，专用于生成用户消息（HumanMessage） 的模板类，是ChatMessagePromptTemplate的特定角色子类。

- 本质：预定义了 role=“human” 的 MessagePromptTemplate，且无需无需手动指定角色
- 模板化：支持使用变量占位符，可以在运行时填充具体值
- 格式化：能够将模板与输入变量结合生成最终的聊天消息
- 输出类型：生成 HumanMessage 对象（ content + role=“human” ）
- 设计目的 ：简化用户输入消息的模板化构造，避免重复定义角色
  

SystemMessagePromptTemplate、AIMessagePromptTemplate：类似于上面，不再赘述

ChatMessagePromptTemplate，用于构建聊天消息的模板。它允许你创建可重用的消息模板，这些模板可以动态地插入变量值来生成最终的聊天消息

- 角色指定：可以为每条消息指定角色（如 “system”、“human”、“ai”） 等，角色灵活。
- 模板化：支持使用变量占位符，可以在运行时填充具体值
- 格式化：能够将模板与输入变量结合生成最终的聊天消息
  
  
  
## 少量样本提示词模板

**方法**

- FewShotPromptTemplate  

- FewShotChatMessagePromptTemplate  
- Example selectors

  ### FewShotPromptTemplate  

- 构建一个 Prompt，其中包含多个 示例（examples）；
- 自动将这些示例格式化并插入到主 Prompt 中；
- 实现 Few-Shot Prompting 方式，以增强大模型在特定任务（如分类、问答、翻译等）上的表现。

它通常由以下几部分构成：

1. `examples`：少量的人工示例（dict 列表）；
2. `example_prompt`：如何格式化每个示例（使用 `PromptTemplate`）；
3. `prefix`：示例之前的文字说明（可选）；
4. `suffix`：用户真正的问题模板；
5. `input_variables`：最终 suffix 中需要传入的变量。

### FewShotChatMessagePromptTemplate

特点：

- 自动将示例格式化为聊天消息（ HumanMessage / AIMessage 等）
- 输出结构化聊天消息（ List[BaseMessage] ）
- 保留对话轮次结构

### Example selectors

在实际开发中，我们可以根据当前输入，使用示例选择器，从大量候选示例中选取最相关的示例子集。

使用的好处：避免盲目传递所有示例，减少 token 消耗的同时，还可以提升输出效果。

示例选择策略：语义相似选择、长度选择、最大边际相关示例选择等

- 语义相似选择：通过余弦相似度等度量方式评估语义相关性，选择与输入问题最相似的 k 个示例。
- 长度选择：根据输入文本的长度，从候选示例中筛选出长度最匹配的示例。增强模型对文本结构的理解。比语义相似度计算更轻量，适合对响应速度要求高的场景。
- 最大边际相关示例选择：优先选择与输入问题语义相似的示例；同时，通过惩罚机制避免返回同质化的内容。

## 消息占位符提示词模板

如果我们不确定消息何时生成，也不确定要插入几条消息，比如在提示词中添加聊天历史记忆这种场景，可以在ChatPromptTemplate添加`MessagesPlaceholder`占位符，在调用invoke时，在占位符处插入消息。

### 显式使用MessagesPlaceholder


```python
# 构建一个 ChatPromptTemplate，包含多种消息类型：
prompt = ChatPromptTemplate.from_messages([
    # 插入 memory 占位符，用于填充历史对话记录（如多轮对话上下文）
    MessagesPlaceholder("memory"),

    # 添加一条系统消息，设定 AI 的角色或行为准则
    SystemMessage("你是一个资深的Python应用开发工程师，请认真回答我提出的Python相关的问题"),

    # 添加一条用户问题消息，用变量 {question} 表示
    ("human", "{question}")
])

# 调用 prompt.invoke 来格式化整个 Prompt 模板
# 传入的参数中：
# - memory：是一组历史消息，表示之前的对话内容（多轮上下文）
# - question：是当前用户的问题
prompt_value = prompt.invoke({
    "memory": [
        # 用户第一轮说的话
        HumanMessage("我的名字叫亮仔，是一名程序员"),
        # AI 第一轮的回应
        AIMessage("好的，亮仔你好")
    ],
    # 当前问题：结合上下文，测试模型是否记住了用户名字
    "question": "请问我的名字叫什么？"
})

# 打印生成的完整 prompt 文本，格式化后的聊天记录
print(prompt_value.to_string())
```



### 隐式使用MessagesPlaceholder

`"placeholder"` 是 `("placeholder", "{memory}")` 的简写语法，等价于 `MessagesPlaceholder("memory")`。

```python
# 使用 ChatPromptTemplate 构建一个多角色对话提示模板
prompt = ChatPromptTemplate.from_messages([
    # 占位符，用于插入对话“记忆”内容，即之前的聊天记录（历史上下文）
    ("placeholder", "{memory}"),

    # 系统消息，用于设定 AI 的角色 —— 是一个资深的 Python 应用开发工程师
    SystemMessage("你是一个资深的Python应用开发工程师，请认真回答我提出的Python相关的问题"),

    # 用户当前提问，使用变量 {question} 进行动态填充
    ("human", "{question}")
])

# 使用 invoke 方法传入上下文变量，生成格式化后的对话 prompt 内容
prompt_value = prompt.invoke({
    # memory：是之前的对话上下文，会被插入到 {memory} 的位置
    "memory": [
        # 用户第一轮对话
        HumanMessage("我的名字叫亮仔，是一名程序员"),
        # AI 第一轮回答
        AIMessage("好的，亮仔你好")
    ],

    # 当前的问题，将替换模板中的 {question}
    "question": "请问我的名字叫什么？"
})
# 使用 .to_string() 将格式化后的对话链转换成纯文本字符串，方便查看输出
print(prompt_value.to_string())

```

## 提示词模板仓库

LangChain Hub ：https://smith.langchain.com/hub  一个公共的 prompt仓库。专门存放 LangChain 的 Prompt、Chains、Tools 等。我们可以在 hub 中搜索通用的提示词模板并使用。代码如下：

```python
from langchain import hub

prompt = hub.pull("hwchase17/openai-tools-agent")

# 查看结构（Langchain PromptTemplate 的 repr）
print(prompt)

# 或者访问具体字段
print(prompt.messages)
```














