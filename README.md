# 实验室服务器知识库

本仓库使用 MkDocs Material 构建实验室服务器使用、开发与运维文档。

- 在线文档：<https://shihuaidexianyu.github.io/lserver/>
- 文档源文件：[`docs/`](docs/)

## 本地预览

```bash
python -m pip install -r requirements.txt
mkdocs serve
```

打开 <http://127.0.0.1:8000/> 即可预览。

## 发布

推送到 `main` 分支后，GitHub Actions 会严格构建站点并自动发布到 GitHub Pages。也可以在仓库的 Actions 页面手动运行部署工作流。
