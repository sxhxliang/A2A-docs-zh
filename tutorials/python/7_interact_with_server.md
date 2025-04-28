# 与 A2A 服务器交互

我们将通过两种方式与智能体服务交互：使用 Google-A2A 官方 CLI 工具，以及自定义客户端实现。

## 使用官方 CLI 工具 <!-- {docsify-ignore} -->

确保服务已运行：
```bash
# 终端窗口1（保持运行）
$ uv run my-project
INFO:     服务进程已启动 [20538]
INFO:     等待应用初始化...
INFO:     应用初始化完成
INFO:     Uvicorn 运行于 http://localhost:10002 (CTRL+C 退出)
```

新开终端执行 CLI 工具：
```bash
source .venv/bin/activate
uv run google-a2a-cli --agent http://localhost:10002
```

> **注意**：若未通过 [PR#169](https://github.com/google/A2A/pull/169) 安装库，需手动操作：
> 1. 克隆仓库 `git clone https://github.com/google/A2A`
> 2. 进入目录 `cd A2A/samples/python`
> 3. 直接运行 `python cli.py --agent http://localhost:10002`

输入消息进行测试：
```bash
=========  新任务创建 ========

请输入要发送的内容（输入 :q 或 quit 退出）: 你好！
```

成功响应示例：
```bash
"message":{"role":"agent","parts":[{"type":"text","text":"on_send_task received: 你好！"}]}
```

输入 `:q` 退出交互

<div class="bottom-buttons" style="flex flex-row">
  <a href="#/tutorials/python/6_start_server.md" class="back-button">上一步</a>
  <a href="#/tutorials/python/8_agent_capabilities.md?id=adding-agent-capabilities" class="next-button">下一步</a>
</div>