# 快速入门

## 环境准备

1. 准备deepseek api key
可以去官网申请， https://platform.deepseek.com/usage

2. 安装langgraph

    ```bash
    pip install -U langgraph "langchain[deepseek]"
    ```

## 创建Agent

```

import os
from langchain_deepseek import ChatDeepSeek
from langgraph.prebuilt import create_react_agent

os.environ["DEEPSEEK_API_KEY"] = "替换成自己的key"
model = ChatDeepSeek(
    model="deepseek-chat",
    temperature=0,
    max_tokens=None,
    timeout=None,
    max_retries=2,
)

def get_weather(city: str) -> str:  
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

agent = create_react_agent(
    model=model,
    tools=[get_weather],
    prompt="You are a helpful assistant" 
    
)

# Run the agent
response = agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather in sf"}]}
)


# 查看每条消息的详细信息
for i, msg in enumerate(response["messages"]):
    print(f"\n--- 消息 {i} ---")
    print("类名:", msg.__class__.__name__)
    
    # 查看消息的属性v
    if hasattr(msg, 'content'):
        print("内容:", msg.content)
    
    if hasattr(msg, 'role'):
        print("角色:", msg.role)
    
    if hasattr(msg, 'name'):
        print("名称:", msg.name)
```

输出为:
```
--- 消息 0 ---
类名: HumanMessage
内容: what is the weather in sf
名称: None

--- 消息 1 ---
类名: AIMessage
内容: I'll get the weather information for San Francisco for you.
名称: None

--- 消息 2 ---
类名: ToolMessage
内容: It's always sunny in San Francisco!
名称: get_weather

--- 消息 3 ---
类名: AIMessage
内容: The weather in San Francisco is sunny! According to the weather service, it's always sunny in San Francisco.
名称: None
```

从消息里可以看到完整的交互流程，包括用户输入、模型回复、工具调用和最终回复。

## 提示词

提示词控制了Agent的行为，包括模型的回复和工具的调用。 支持2种提示词设置方式.

### 静态提示词

字符串内容将被解析为系统提示词直接使用。 上面例子就是使用的静态提示词。

### 动态提示词

```

import os
from langchain_deepseek import ChatDeepSeek
from langgraph.prebuilt import create_react_agent
from langchain_core.messages import AnyMessage
from langchain_core.runnables import RunnableConfig
from langgraph.prebuilt.chat_agent_executor import AgentState

os.environ["DEEPSEEK_API_KEY"] = "替换成自己的key"
model = ChatDeepSeek(
    model="deepseek-chat",
    temperature=0,
    max_tokens=None,
    timeout=None,
    max_retries=2,
)

def get_weather(city: str) -> str:  
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

def prompt(state: AgentState, config: RunnableConfig) -> list[AnyMessage]:  
    user_name = config["configurable"].get("user_name")
    system_msg = f"You are a helpful assistant. Address the user as {user_name}."
    return [{"role": "system", "content": system_msg}] + state["messages"]

agent = create_react_agent(
    model=model,
    tools=[get_weather],
    prompt=prompt
    
)

# Run the agent
response = agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather in sf"}]},
     config={"configurable": {"user_name": "John Smith"}}
)


# 查看每条消息的详细信息
for i, msg in enumerate(response["messages"]):
    print(f"\n--- 消息 {i} ---")
    print("类名:", msg.__class__.__name__)
    
    # 查看消息的属性v
    if hasattr(msg, 'content'):
        print("内容:", msg.content)
    
    if hasattr(msg, 'role'):
        print("角色:", msg.role)
    
    if hasattr(msg, 'name'):
        print("名称:", msg.name)
```

输出:
```
--- 消息 0 ---
类名: HumanMessage
内容: what is the weather in sf
名称: None

--- 消息 1 ---
类名: AIMessage
内容: I'll get the weather information for San Francisco for you, John Smith.
名称: None

--- 消息 2 ---
类名: ToolMessage
内容: It's always sunny in San Francisco!
名称: get_weather

--- 消息 3 ---
类名: AIMessage
内容: The weather in San Francisco is sunny! It's always sunny there according to the weather service. Enjoy the beautiful weather, John Smith!
名称: None
```

动态提示词，不仅可以传入变量，而且提示词是可以动态变化的。 可以把prompt函数里的` state["messages"]`打印出来，可以看到调用了2次：
```
第一次: [HumanMessage(content='what is the weather in sf', additional_kwargs={}, response_metadata={}, id='ac0fd4b1-dd9f-4de1-8d83-7821c745200d')]
第二次: [HumanMessage(content='what is the weather in sf', additional_kwargs={}, response_metadata={}, id='ac0fd4b1-dd9f-4de1-8d83-7821c745200d'), AIMessage(content="I'll get the weather information for San Francisco for you, John Smith.", additional_kwargs={'tool_calls': [{'id': 'call_0_1b8f5d42-2bcc-4817-a1b2-bd78d59e07fd', 'function': {'arguments': '{"city": "San Francisco"}', 'name': 'get_weather'}, 'type': 'function', 'index': 0}], 'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 30, 'prompt_tokens': 165, 'total_tokens': 195, 'completion_tokens_details': None, 'prompt_tokens_details': {'audio_tokens': None, 'cached_tokens': 128}, 'prompt_cache_hit_tokens': 128, 'prompt_cache_miss_tokens': 37}, 'model_name': 'deepseek-chat', 'system_fingerprint': 'fp_feb633d1f5_prod0820_fp8_kvcache', 'id': '0d1ec6aa-7d9d-4a9e-9b55-c23172fe5-c235-c23172fe5-c23172fef70b'5-c23172fef70b', 'service_tier': None, 'finis5-c23172fe5-c23172fe5-c235-c23172fef70b'5-c23172fef70b', 'service_tier': None, 'finish_reason': 'tool_calls', 'logprobs': None}, id='run--46a9372e-d4c7-4145-a29a-6fad4d0f5622-0', tool_calls=[{'name': 'get_weather', 'args': {'city': 'San Francisco'}, 'id': 'call_0_1b8f5d42-2bcc-4817-a1b2-bd78d59e07fd', 'type': 'tool_call'}], usage_metadata={'input_tokens': 165, 'output_tokens': 30, 'total_tokens': 195, 'input_token_details': {'cache_read': 128}, 'output_token_details': {}}), ToolMessage(content="It's always sunny in San Francisco!", name='get_weather', id='8e52c54d-6955-49e7-a1b8-73d60e4c6ba6', tool_call_id='call_0_1b8f5d42-2bcc-4817-a1b2-bd78d59e07fd')]
```