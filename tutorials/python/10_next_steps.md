# 后续步骤

恭喜！您已掌握运行集成AI模型的A2A服务器的基础知识。以下是一些进阶方向建议：

* **连接AI模型与[MCP工具]**
  * 前置步骤：[创建MCP服务器](https://modelcontextprotocol.io/quickstart/server)
  * 集成方法：在`create_react_agent(ollama_chat_llm, tools=[])`调用中[集成MCP工具](https://github.com/langchain-ai/langchain-mcp-adapters?tab=readme-ov-file#client)
  ```python
  # 示例代码片段
  from langchain_mcp_adapters import MCPToolkit
  mcp_tools = MCPToolkit(base_url="http://localhost:8080").get_tools()
  agent = create_react_agent(ollama_chat_llm, tools=mcp_tools)
  ```

* **开发定制化代理**
  * 使用[Google Agent Development Kit](https://developers.google.com/agent-development-kit)构建专属代理
  * 参考官方[示例项目库](https://github.com/google/A2A/tree/main/samples/python/agents)获取灵感

* **深入技术文档**
  * 📚 研读[A2A技术文档](/documentation)掌握协议能力
  * 📝 查看A2A协议[JSON规范](/specification)理解数据结构

* **核心概念精研**
  * [A2A与MCP的协同](/topics/a2a_and_mcp.md) - 理解协议间的互补关系
  * [代理发现机制](/topics/agent_discovery.md) - 实现分布式服务定位
  * [企业级功能](/topics/enterprise_ready.md) - 了解安全审计与SLA保障
  * [推送通知](/topics/push_notifications.md) - 构建实时交互系统

* **扩展实践**
  * 添加性能监控（推荐使用Prometheus+Grafana）
  * 实现多模型热切换功能
  * 加入JWT身份验证层
  ```python
  # 简易认证示例
  from fastapi import Depends, HTTPException
  from fastapi.security import HTTPBearer
  security = HTTPBearer()
  async def verify_token(token: str = Depends(security)):
      if token != "VALID_TOKEN":
          raise HTTPException(status_code=403)
  ```

* **社区资源**
  * 加入[A2A开发者论坛](https://github.com/google/A2A/discussions)交流经验
  * 参与[月度代码挑战赛](https://a2a-hackathons.io)赢取奖励

<div class="bottom-buttons" style="flex flex-row">
  <a href="#/tutorials/python/9_ollama_agent.md" class="back-button">返回</a>
  <span class="fill-space"></span>
</div>