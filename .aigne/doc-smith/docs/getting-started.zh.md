# 快速入门

本教程将引导你逐步安装 FastAPI 并创建你的第一个应用程序。你将从零开始构建一个简单而完整的 API。

## 安装

首先，你需要安装 FastAPI。此过程也会安装必要的依赖项，包括一个 Web 服务器。

<x-card data-title="前提条件" data-icon="lucide:python" data-horizontal="true">
  FastAPI 要求 Python 3.8 或更高版本。你可以在终端中运行 `python --version` 来验证你的安装情况。
</x-card>

要安装 FastAPI 及其标准依赖项（包括 Uvicorn 服务器），请在终端中运行以下命令：

```console
$ pip install "fastapi[standard]"

---> 100%
```

这条命令会安装 FastAPI、用于数据验证的 Pydantic 以及用于为你的应用程序提供服务的 Uvicorn。

## 创建应用

现在，我们来创建你的第一个 API。

1.  创建一个名为 `main.py` 的文件。
2.  向其中添加以下代码：

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "q": q}
```

这段代码定义了一个包含两个端点的简单 API：
*   `@app.get("/")`：处理对根 URL `/` 的 `GET` 请求。
*   `@app.get("/items/{item_id}")`：处理对 `/items/5` 等路径的 `GET` 请求。它会捕获路径参数 `item_id` 和一个可选的查询参数 `q`。

## 运行应用

在终端中运行开发服务器：

```console
$ fastapi dev main.py

 ╭────────── FastAPI CLI - Development mode ───────────╮
 │                                                     │
 │  Serving at: http://127.0.0.1:8000                  │
 │                                                     │
 │  API docs: http://127.0.0.1:8000/docs               │
 │                                                     │
 │  Running in development mode, for production use:   │
 │                                                     │
 │  fastapi run                                        │
 │                                                     │
 ╰─────────────────────────────────────────────────────╯

INFO:     Will watch for changes in these directories: ['.']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [12345] using StatReload
INFO:     Started server process [12347]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

`fastapi dev` 命令会启动一个本地服务器，当你修改代码时，服务器会自动重新加载，这非常适合开发。

## 检查结果

打开浏览器并访问 [http://127.0.0.1:8000/items/5?q=somequery](http://127.0.0.1:8000/items/5?q=somequery)。

你将看到以下 JSON 响应：

```json
{"item_id":5,"q":"somequery"}
```

你刚刚创建并运行了你的第一个 API，该 API 能够验证路径和查询参数。

## 交互式 API 文档

FastAPI 的一个主要特性是其自动生成的文档。

在浏览器中访问 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)。你将看到由 Swagger UI 提供的交互式 API 文档：

![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

FastAPI 还提供了另一种文档界面。访问 [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc) 查看由 ReDoc 生成的文档：

![ReDoc](https://fastapi.tiangolo.com/img/index/index-02-redoc-simple.png)

这些文档页面是根据你的代码自动生成的，可用于直接在浏览器中测试你的 API 端点。

## 后续步骤

你已成功创建并运行了一个基本的 FastAPI 应用程序。要了解如何处理更复杂的场景（例如在请求体中接收数据），请继续阅读我们用户指南的下一部分。

<x-card data-title="下一步：用户指南" data-icon="lucide:book-open" data-href="/user-guide/path-parameters" data-cta="开始阅读用户指南">
  通过实际示例探索核心概念，从路径参数开始。
</x-card>