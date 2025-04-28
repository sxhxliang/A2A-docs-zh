# 设置您的环境


## 所需准备 <!-- {docsify-ignore} -->

- 代码编辑器（如 Visual Studio Code）
- 命令行工具（如 Linux 的 Terminal、Mac 的 iTerm 或 VS Code 内置终端）

## Python 环境 <!-- {docsify-ignore} -->

我们将使用 [uv](https://docs.astral.sh/uv/getting-started/installation/) 作为包管理工具并配置项目环境。

使用的 A2A 库要求 `python >= 3.12`（[uv 可自动安装](https://docs.astral.sh/uv/guides/install-python/) Python 版本）。本教程将使用 Python 3.12。

## 环境检查 <!-- {docsify-ignore} -->

运行以下命令验证环境是否就绪：
```bash
echo 'import sys; print(sys.version)' | uv run -
```

若看到类似以下输出，说明环境已就绪：
```bash
3.12.3 (main, Feb  4 2025, 14:48:35) [GCC 13.3.0]
```

<div class="bottom-buttons" style="flex flex-row">
  <a href="#/tutorials/python/1_introduction.md" class="back-button">上一步</a>
  <a href="#/tutorials/python/3_create_a_project.md?id=creating-a-project" class="next-button">下一步</a>
</div>