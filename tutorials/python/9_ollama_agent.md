# 使用本地Ollama模型

现在进入激动人心的部分——为我们的A2A服务器添加AI功能。

本教程将指导您设置本地Ollama模型并将其集成到A2A服务器。您也可以选择使用Google的Agent Development Kit (ADK)，可以在GitHub查看[示例项目](https://github.com/google/A2A/tree/main/samples/python/agents)。

## 要求 <!-- {docsify-ignore} -->

需要安装`ollama`、`langchain`并下载支持MCP工具的ollama模型（为后续教程准备）：

1. 下载[ollama](https://ollama.com/download)
2. 运行ollama服务端
```bash
# 注意：如果ollama已在运行，可能会报错：
# Error: listen tcp 127.0.0.1:11434: bind: address already in use
# Linux系统可通过systemctl stop ollama停止服务
ollama serve
```
3. 从[模型列表](https://ollama.com/search)下载模型。推荐使用`qwq`（支持`tools`工具，需24GB显存）
```bash
ollama pull qwq
```
4. 安装依赖库
```bash
uv add langchain langchain-ollama langgraph
```

完成ollama配置后，开始集成到A2A服务器。

## 将Ollama集成到A2A服务器 <!-- {docsify-ignore} -->

首先修改`src/my_project/__init__.py`

```python
# ...

@click.command()
@click.option("--host", default="localhost")
@click.option("--port", default=10002)
@click.option("--ollama-host", default="http://127.0.0.1:11434")
@click.option("--ollama-model", default=None)
def main(host, port, ollama_host, ollama_model):
  # ...
  capabilities = AgentCapabilities(
    streaming=False # 流式传输功能留给读者自行实现
  )
  # ...
  task_manager = MyAgentTaskManager(
    ollama_host=ollama_host,
    ollama_model=ollama_mode,
  )
  # ..
```

在`src/my_project/agent.py`中添加AI功能

```python
from langchain_ollama import ChatOllama
from langgraph.prebuilt import create_react_agent
from langgraph.graph.graph import CompiledGraph

def create_ollama_agent(ollama_base_url: str, ollama_model: str):
  ollama_chat_llm = ChatOllama(
    base_url=ollama_base_url,
    model=ollama_model,
    temperature=0.2
  )
  agent = create_react_agent(ollama_chat_llm, tools=[])
  return agent

async def run_ollama(ollama_agent: CompiledGraph, prompt: str):
  agent_response = await ollama_agent.ainvoke(
    {"messages": prompt }
  )
  message = agent_response["messages"][-1].content
  return str(message)
```

最后在`src/my_project/task_manager.py`调用ollama

```python
# ...
from my_project.agent import create_ollama_agent, run_ollama

class MyAgentTaskManager(InMemoryTaskManager):
  def __init__(
    self,
    ollama_host: str,
    ollama_model: typing.Union[None, str]
  ):
    super().__init__()
    if ollama_model is not None:
      self.ollama_agent = create_ollama_agent(
        ollama_base_url=ollama_host,
        ollama_model=ollama_model
      )
    else:
      self.ollama_agent = None

  async def on_send_task(self, request: SendTaskRequest) -> SendTaskResponse:
    # ...
    received_text = request.params.message.parts[0].text
    response_text = f"on_send_task received: {received_text}"
    if self.ollama_agent is not None:
      response_text = await run_ollama(ollama_agent=self.ollama_agent, prompt=received_text)

    task = await self._update_task(
      task_id=task_id,
      task_state=TaskState.COMPLETED,
      response_text=response_text
    )

    # 返回响应
    return SendTaskResponse(id=request.id, result=task)

  # ...
```

## 测试运行

重新启动A2A服务器（将`qwq`替换为您下载的模型）：

```bash
uv run my-project --ollama-host http://127.0.0.1:11434 --ollama-model qwq
```

运行CLI客户端：

```bash
uv run google-a2a-cli --agent http://localhost:10002
```

注意：大型模型加载需要时间，若CLI超时可稍后重试。成功运行后将看到类似输出：

```bash
=========  启动新任务 ========

What do you want to send to the agent? (:q or quit to exit): hey

"message":{"role":"agent","parts":[{"type":"text","text":"<think>\n用户输入\"hey\"，需要友好回应。建议询问今日需求，保持开放态度。确保语气积极亲切。最终回复：\"您好！今天有什么可以帮您的？😊\"\n</think>\n\n您好！今天有什么可以帮您的？😊"}]}
```

恭喜！您的A2A服务器已成功集成AI模型！

<div class="bottom-buttons" style="flex flex-row">
  <a href="#/tutorials/python/8_agent_capabilities.md" class="back-button">返回</a>
  <a href="#/tutorials/python/10_next_steps.md?id=next-steps" class="next-button">下一步</a>
</div>