# 创建项目

首先使用 `uv` 初始化项目。添加 `--package` 参数以便未来扩展测试或发布：
```bash
uv init --package my-project
cd my-project
```

## 配置虚拟环境 <!-- {docsify-ignore} -->

为项目创建独立虚拟环境（只需执行一次）：
```bash
uv venv .venv
```

在终端中激活虚拟环境（后续每次新开终端需执行）：
```bash
source .venv/bin/activate
```

若使用 VS Code，需设置 Python 解释器：
1. 按 `Ctrl-Shift-P` 打开命令面板
2. 选择 `Python: 选择解释器`
3. 选择 `my-project` 项目下的 `.venv/bin/python (Python 3.12.3)`

此时项目结构应如下所示：
```bash
tree .
.
├── pyproject.toml
├── README.md
├── src
│   └── my-project
│       ├── __init__.py
```

## 添加 Google A2A 库 <!-- {docsify-ignore} -->

由于官方 [PR#169](https://github.com/google/A2A/pull/169) 尚未合并，我们暂时从分支获取：
```bash
uv add git+https://github.com/djsamseng/A2A#subdirectory=samples/python --branch prefixPythonPackage
```

或使用 Google 官方仓库（需调整导入语句）：
```bash
uv add git+https://github.com/google/A2A#subdirectory=samples/python
```

使用官方仓库时需修改导入方式：
```diff
- import google_a2a.common
+ import common
```

## 初始化项目结构 <!-- {docsify-ignore} -->

创建核心功能文件：
```bash
touch src/my_project/agent.py       # 智能体实现
touch src/my_project/task_manager.py # 任务管理
```

## 环境验证 <!-- {docsify-ignore} -->

运行测试命令验证配置：
```bash
uv run my-project
```

成功时将显示：
```bash
Hello from my-project!
```

<div class="bottom-buttons" style="flex flex-row">
  <a href="#/tutorials/python/2_setup.md" class="back-button">上一步</a>
  <a href="#/tutorials/python/4_agent_skills.md?id=agent-skills" class="next-button">下一步</a>
</div>