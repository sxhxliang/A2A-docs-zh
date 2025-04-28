# A2A 服务器搭建指南

现在让我们启动智能体服务！我们将使用 Google-A2A 的 `A2AServer` 类（底层基于 [uvicorn](https://www.uvicorn.org/) 实现），请注意 Google-A2A 仍处于开发阶段，未来实现可能调整。

## 任务管理器实现 <!-- {docsify-ignore} -->

在启动服务前，需实现任务管理器处理请求。我们继承 `InMemoryTaskManager` 并实现两个核心方法：

```python
from typing import AsyncIterable

import google_a2a
from google_a2a.common.server.task_manager import InMemoryTaskManager
from google_a2a.common.types import (
  Artifact,
  JSONRPCResponse,
  Message,
  SendTaskRequest,
  SendTaskResponse,
  SendTaskStreamingRequest,
  SendTaskStreamingResponse,
  Task,
  TaskState,
  TaskStatus,
  TaskStatusUpdateEvent,
)

class 智能体任务管理器(InMemoryTaskManager):
  def __init__(self):
    super().__init__()

  async def 处理任务请求(self, request: SendTaskRequest) -> SendTaskResponse:
    """处理同步任务请求"""
    await self.更新任务库(request.params)  # 存储任务到内存数据库

    task_id = request.params.id
    # 实现回声逻辑
    输入文本 = request.params.message.parts[0].text
    任务实例 = await self._更新任务状态(
      task_id=task_id,
      新状态=TaskState.已完成,
      响应文本=f"收到请求: {输入文本}"
    )

    return SendTaskResponse(id=request.id, result=任务实例)

  async def 处理订阅请求(
    self,
    request: SendTaskStreamingRequest
  ) -> AsyncIterable[SendTaskStreamingResponse] | JSONRPCResponse:
    """处理流式订阅请求（暂不实现）"""
    pass

  async def _更新任务状态(
    self,
    task_id: str,
    新状态: TaskState,
    响应文本: str,
  ) -> Task:
    """内部方法更新任务状态"""
    任务 = self.任务库[task_id]
    响应内容 = [{"type": "text", "text": 响应文本}]
    
    任务.状态 = TaskStatus(
      state=新状态,
      message=Message(role="agent", parts=响应内容)
    )
    任务.产出物 = [Artifact(parts=响应内容)]
    
    return 任务
```

将代码保存至 `src/my_project/task_manager.py`

## 服务端配置 <!-- {docsify-ignore} -->

在 `src/my-project/__init__.py` 中集成服务：

```python
# ...
from google_a2a.common.server import A2AServer
from my_project.task_manager import 智能体任务管理器

@click.command()
@click.option("--host", default="localhost", help="监听地址")
@click.option("--port", default=10002, type=int, help="监听端口")
def 主程序(host, port):
    # ... [之前智能体卡片代码]
    
    任务处理器 = 智能体任务管理器()
    服务实例 = A2AServer(
        agent_card=智能体卡片,
        task_manager=任务处理器,
        host=host,
        port=port,
    )
    服务实例.启动()
```

## 运行验证 <!-- {docsify-ignore} -->

启动服务：
```bash
uv run my-project
```

正常启动后将显示：
```bash
INFO:     服务进程已启动 [进程ID]
INFO:     等待应用初始化...
INFO:     应用初始化完成
INFO:     Uvicorn 运行于 http://localhost:10002 (CTRL+C 退出)
```

恭喜！您的 A2A 服务已就绪！

<div class="bottom-buttons" style="flex flex-row">
  <a href="#/tutorials/python/5_add_agent_card.md" class="back-button">上一步</a>
  <a href="#/tutorials/python/7_interact_with_server.md?id=interacting-with-your-a2a-server" class="next-button">下一步</a>
</div>