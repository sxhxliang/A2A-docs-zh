# 智能体技能

智能体技能指智能体具备的功能集合。以下是我们回声智能体的技能示例：

```ts
{
  id: "my-project-echo-skill"
  name: "Echo工具",
  description: "将输入内容原样返回",
  tags: ["Echo", "重复器"],
  examples: ["我将看到这段话被原样返回"],
  inputModes: ["text"],
  outputModes: ["text"]
}
```

该结构符合 [智能体卡片](documentation?id=representation) 的技能定义规范：

```ts
{
  id: string;        // 技能唯一标识符
  name: string;      // 人类可读的技能名称
  description: string; // 功能描述（用于客户端或人工理解）
  tags: string[];    // 能力分类标签（如"烹饪","客服"等）
  examples?: string[]; // 典型使用场景示例
  inputModes?: string[]; // 支持的输入格式（MIME类型）
  outputModes?: string[]; // 支持的输出格式（MIME类型）
}
```

## 代码实现 <!-- {docsify-ignore} -->

在 `src/my-project/__init__.py` 文件中添加以下代码：

```python
import google_a2a
from google_a2a.common.types import AgentSkill

def main():
  skill = AgentSkill(
    id="my-project-echo-skill",
    name="回声工具",
    description="将输入内容原样返回",
    tags=["回声", "重复器"],
    examples=["我将看到这段话被原样返回"],
    inputModes=["text"],
    outputModes=["text"],
  )
  print(skill)

if __name__ == "__main__":
  main()
```

## 运行测试 <!-- {docsify-ignore} -->

执行命令验证功能：
```bash
uv run my-project
```

预期输出示例：
```bash
id='my-project-echo-skill' name='Echo工具' description='将输入内容原样返回' tags=['Echo', '重复器'] examples=['我将看到这段话被原样返回'] inputModes=['text'] outputModes=['text']
```

<div class="bottom-buttons" style="flex flex-row">
  <a href="#/tutorials/python/3_create_a_project.md" class="back-button">上一步</a>
  <a href="#/tutorials/python/5_add_agent_card.md?id=agent-card" class="next-button">下一步</a>
</div>