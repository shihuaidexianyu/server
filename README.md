# 实验室服务器知识库

定位：使用 MkDocs Material 构建实验室服务器使用、开发与运维文档。
入口：[文档首页](docs/index.md)、[快速开始](docs/快速开始.md)、[文档维护](docs/文档维护.md)。
内容：18篇服务器文档，以及 MkDocs 配置和发布工作流。服务器实际配置与站点发布状态按维护说明核实。

- 在线文档：<https://shihuaidexianyu.github.io/lserver/>
- 文档源文件：[`docs/`](docs/)

## 本地预览

```bash
cd /path/to/qhwiki
uv sync --extra docs    # 在 qhwiki 根准备依赖，与本站 requirements.txt 对齐
cd 服务器
../.venv/bin/python -m mkdocs serve
```

打开 <http://127.0.0.1:8000/> 即可预览。

## 发布

本站工作流配置为推送到 `main` 后严格构建并发布到 GitHub Pages，也支持手动运行。实际发布取决于远端 Pages/Actions 配置和运行结果。维护与隔离构建说明见 [文档维护](docs/文档维护.md)。
