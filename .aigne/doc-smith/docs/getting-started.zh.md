# 入门

本教程将向你展示如何设置 FastAPI 并构建一个简单的 API，内容涵盖从安装到运行应用的整个过程。学完本教程后，你将拥有一个带有交互式文档且可正常工作的 API。

## 安装

首先，你需要安装 FastAPI 和一个 Web 服务器。你只需一条命令即可完成此操作。

<x-card data-title="前提条件" data-icon="lucide:python" data-horizontal="true">
  FastAPI 要求 Python 3.8 或更高版本。你可以在终端中运行 `python --version` 来查看你的版本。
</x-card>

要安装 FastAPI 及其标准依赖项和 Uvicorn 服务器，请运行以下命令：

```console
$ pip install "fastapi[standard]"

---> 100%
```

该命令会安装 FastAPI、用于数据验证的 Pydantic、用于 Web 组件的 Starlette 以及用于为你的应用提供服务的 Uvicorn。

## 创建 API

现在，我们来创建 API 代码。

1.  创建一个名为 `main.py` 的文件。
2.  将以下 Python 代码添加到该文件中：

```python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Union[str, None] = None):
    return {"item_id": item_id, "q": q}
```

这段代码定义了一个包含两个端点的简单 API：

*   `@app.get("/")`: 用于处理对根 URL `/` 的 `GET` 请求。
*   `@app.get("/items/{item_id}")`: 用于处理对 `/items/5` 这类路径的 `GET` 请求。它使用标准的 Python 类型提示来捕获路径参数 `item_id` 和可选的查询参数 `q`。

## 运行 API

在终端中使用 `fastapi` 命令运行开发服务器：

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
INFO:     Started reloader process [2248755] using WatchFiles
INFO:     Started server process [2248757]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

`fastapi dev` 命令会启动一个本地服务器，该服务器在你修改代码时会自动重新加载，非常适合开发环境。

## 检查运行结果

打开浏览器并访问 [http://127.0.0.1:8000/items/5?q=somequery](http://127.0.0.1:8000/items/5?q=somequery)。

你将看到以下 JSON 响应：

```json
{"item_id": 5, "q": "somequery"}
```

你刚刚创建了一个 API，它能接收并验证一个路径参数（`item_id`，整数类型）和一个查询参数（`q`，字符串类型）。

## 交互式 API 文档

FastAPI 最有用的功能之一是其自动生成的文档。

在浏览器中访问 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)。你将看到由 Swagger UI 提供的交互式 API 文档：

![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

FastAPI 还提供了另一种文档界面。访问 [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc) 查看由 ReDoc 生成的文档：

![ReDoc](https://fastapi.tiangolo.com/img/index/index-02-redoc-simple.png)

这些文档页面是根据你代码中的类型提示自动生成的，并允许你直接从浏览器测试 API 端点。

## 后续步骤

你已成功创建并运行了一个基本的 FastAPI 应用。要学习如何处理更复杂的场景，例如在请求体中接收数据或使用依赖注入，请继续阅读用户指南。

<x-card data-title="下一步：用户指南" data-icon="lucide:book-open" data-href="/user-guide/path-parameters" data-cta="开始阅读用户指南">
  通过实际示例探索核心概念，从路径参数开始。
</x-card>