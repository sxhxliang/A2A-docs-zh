# 添加代理能力

现在我们有了一个基础的A2A服务器运行，让我们添加更多功能。我们将探索A2A如何异步工作和流式传输响应。

## 流式传输 <!-- {docsify-ignore} -->

这允许客户端订阅服务器并接收多个更新而不是单个响应。这对于长时间运行的代理任务或多个Artifact需要流式返回客户端的情况非常有用。参见[流式传输文档](/documentation.md?id=streaming-support)

首先声明我们的代理支持流式传输。打开`src/my_project/__init__.py`并更新AgentCapabilities

```python
# ...
def main(host, port):
  # ...
  capabilities = AgentCapabilities(
    streaming=True
  )
  # ...
```

现在在`src/my_project/task_manager.py`中需要实现`on_send_task_subscribe`

```python
import asyncio
# ...
class MyAgentTaskManager(InMemoryTaskManager):
  # ...
  async def _stream_3_messages(self, request: SendTaskStreamingRequest):
    task_id = request.params.id
    received_text = request.params.message.parts[0].text

    text_messages = ["one", "two", "three"]
    for text in text_messages:
      parts = [
        {
          "type": "text",
          "text": f"{received_text}: {text}",
        }
      ]
      message = Message(role="agent", parts=parts)
      is_last = text == text_messages[-1]
      task_state = TaskState.COMPLETED if is_last else TaskState.WORKING
      task_status = TaskStatus(
        state=task_state,
        message=message
      )
      task_update_event = TaskStatusUpdateEvent(
        id=request.params.id,
        status=task_status,
        final=is_last,
      )
      await self.enqueue_events_for_sse(
        request.params.id,
        task_update_event
      )

  async def on_send_task_subscribe(
    self,
    request: SendTaskStreamingRequest
  ) -> AsyncIterable[SendTaskStreamingResponse] | JSONRPCResponse:
    # 在内存任务管理器中更新任务
    await self.upsert_task(request.params)

    task_id = request.params.id
    # 为该任务创建工作队列
    sse_event_queue = await self.setup_sse_consumer(task_id=task_id)

    # 启动该任务的异步工作
    asyncio.create_task(self._stream_3_messages(request))

    # 通知客户端准备接收流式响应
    return self.dequeue_events_for_sse(
      request_id=request.id,
      task_id=task_id,
      sse_event_queue=sse_event_queue,
    )
```

重启A2A服务器应用更改后重新运行CLI

```bash
$ uv run google-a2a-cli --agent http://localhost:10002
=========  starting a new task ========

What do you want to send to the agent? (:q or quit to exit): Streaming?

"status":{"state":"working","message":{"role":"agent","parts":[{"type":"text","text":"Streaming?: one"}]}
"status":{"state":"working","message":{"role":"agent","parts":[{"type":"text","text":"Streaming?: two"}]}
"status":{"state":"completed","message":{"role":"agent","parts":[{"type":"text","text":"Streaming?: three"}]}

```

有时代理可能需要额外输入。例如代理可能会询问客户端是否要继续重复发送三条消息。这时代理会返回`TaskState.INPUT_REQUIRED`状态，客户端需要重新发送`send_task_streaming`请求（使用相同的`task_id`和`session_id`）并携带新的输入消息。服务端需要更新`on_send_task_subscribe`来处理这种情况。

```python
# ...

class MyAgentTaskManager(InMemoryTaskManager):
  # ...
  async def _stream_3_messages(self, request: SendTaskStreamingRequest):
    # ...
    async for message in messages:
      # ...
      # is_last = message == messages[-1] # 删除这行
      task_state = TaskState.WORKING
      # ...
      task_update_event = TaskStatusUpdateEvent(
        id=request.params.id,
        status=task_status,
        final=False,
      )
      # ...

    ask_message = Message(
      role="agent",
      parts=[
        {
          "type": "text",
          "text": "需要更多消息吗？(Y/N)"
        }
      ]
    )
    task_update_event = TaskStatusUpdateEvent(
      id=request.params.id,
      status=TaskStatus(
        state=TaskState.INPUT_REQUIRED,
        message=ask_message
      ),
      final=True,
    )
    await self.enqueue_events_for_sse(
      request.params.id,
      task_update_event
    )
  # ...
  async def on_send_task_subscribe(
    self,
    request: SendTaskStreamingRequest
  ) -> AsyncIterable[SendTaskStreamingResponse] | JSONRPCResponse:
    task_id = request.params.id
    is_new_task = task_id in self.tasks
    # 更新内存中的任务
    await self.upsert_task(request.params)

    received_text = request.params.message.parts[0].text
    sse_event_queue = await self.setup_sse_consumer(task_id=task_id)
    if not is_new_task and received_text == "N":
      task_update_event = TaskStatusUpdateEvent(
        id=request.params.id,
        status=TaskStatus(
          state=TaskState.COMPLETED,
          message=Message(
            role="agent",
            parts=[
              {
                "type": "text",
                "text": "全部完成！"
              }
            ]
          )
        ),
        final=True,
      )
      await self.enqueue_events_for_sse(
        request.params.id,
        task_update_event,
      )
    else:
      asyncio.create_task(self._stream_3_messages(request))

    return self.dequeue_events_for_sse(
      request_id=request.id,
      task_id=task_id,
      sse_event_queue=sse_event_queue,
    )
```

重启服务器后运行CLI，可以看到任务会持续运行直到回复N

```bash
$ uv run google-a2a-cli --agent http://localhost:10002
=========  启动新任务 ========

What do you want to send to the agent? (:q or quit to exit): Streaming?

"status":{"state":"working","message":{"role":"agent","parts":[{"type":"text","text":"Streaming?: one"}]}
"status":{"state":"working","message":{"role":"agent","parts":[{"type":"text","text":"Streaming?: two"}]}
"status":{"state":"working","message":{"role":"agent","parts":[{"type":"text","text":"Streaming?: three"}]}
"status":{"state":"input-required","message":{"role":"agent","parts":[{"type":"text","text":"需要更多消息吗？(Y/N)"}]}

What do you want to send to the agent? (:q or quit to exit): N

"status":{"state":"completed","message":{"role":"agent","parts":[{"type":"text","text":"全部完成！"}]}

```

恭喜！现在您拥有了一个能够异步执行任务并在需要时请求用户输入的代理。

## 其他能力 <!-- {docsify-ignore} -->

如果您有兴趣，可以查看[文档](/documentation.md?id=sample-methods-and-json-responses)了解A2A代理的其他能力。接下来我们将学习如何通过本地LLM为A2A添加AI功能。


<div class="bottom-buttons" style="flex flex-row">
  <a href="#/tutorials/python/7_interact_with_server.md" class="back-button">返回</a>
  <a href="#/tutorials/python/9_ollama_agent.md?id=using-a-local-ollama-model" class="next-button">下一步</a>
</div>