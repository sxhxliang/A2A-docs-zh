# 智能体卡片

定义完技能后，即可创建智能体卡片（Agent Card）。

远程智能体需以 JSON 格式发布智能体卡片，该卡片描述了智能体的能力集、技能及认证机制。简而言之，这是智能体的"身份名片"，用于向外界宣告其功能和使用方式。更多细节请参考 [官方文档](documentation?id=agent-card)。

## 代码实现 <!-- {docsify-ignore} -->

### 步骤 1：添加命令行工具支持
安装 Click 库处理命令行参数：
```bash
uv add click
```

### 步骤 2：更新主程序
修改 `src/my-project/__init__.py` 内容如下：

```python
import logging
import click
from dotenv import load_dotenv
import google_a2a
from google_a2a.common.types import AgentSkill, AgentCapabilities, AgentCard

# 配置基础日志
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@click.command()
@click.option("--host", default="localhost", help="服务监听地址")
@click.option("--port", default=10002, type=int, help="服务监听端口")
def main(host, port):
    """主程序入口"""
    # 创建技能实例
    skill = AgentSkill(
        id="my-project-echo-skill",
        name="回声工具",
        description="将输入内容原样返回",
        tags=["回声", "重复器"],
        examples=["我将看到这段话被原样返回"],
        inputModes=["text"],
        outputModes=["text"],
    )
    logger.info("创建技能: %s", skill)

    # 构建智能体能力集
    capabilities = AgentCapabilities()
    
    # 生成智能体卡片
    agent_card = AgentCard(
        name="回声智能体",
        description="输入内容回显代理",
        url=f"http://{host}:{port}/",
        version="0.1.0",
        defaultInputModes=["text"],
        defaultOutputModes=["text"],
        capabilities=capabilities,
        skills=[skill]
    )
    logger.info("智能体卡片信息: %s", agent_card)

if __name__ == "__main__":
    main()
```

## 运行测试 <!-- {docsify-ignore} -->

执行命令验证智能体卡片生成：
```bash
uv run my-project
```

预期输出示例：
```bash
INFO:root:创建技能: id='my-project-回声-skill' name='回声工具' description='将输入内容原样返回' tags=['回声', '重复器'] examples=['我将看到这段话被原样返回'] inputModes=['text'] outputModes=['text']
INFO:root:智能体卡片信息: name='回声智能体' description='输入内容回显代理' url='http://localhost:10002/' provider=None version='0.1.0' documentationUrl=None capabilities=AgentCapabilities(streaming=False, pushNotifications=False, stateTransitionHistory=False) authentication=None defaultInputModes=['text'] defaultOutputModes=['text'] skills=[AgentSkill(id='my-project-回声-skill', name='回声工具', description='将输入内容原样返回', tags=['回声', '重复器'], examples=['我将看到这段话被原样返回'], inputModes=['text'], outputModes=['text'])]
```

<div class="bottom-buttons" style="flex flex-row">
  <a href="#/tutorials/python/4_agent_skills.md" class="back-button">上一步</a>
  <a href="#/tutorials/python/6_start_server.md?id=a2a-server" class="next-button">下一步</a>
</div>