# 快速入门

本指南将引导你创建你的第一个 FastAPI 应用程序。你将学习如何安装 FastAPI 及其所需的依赖项，编写一个简单的“Hello World”应用，并使用开发服务器运行它。

## 安装

首先，你需要安装 FastAPI。建议在[虚拟环境](https://fastapi.tiangolo.com/virtual-environments/)中工作。你可以使用 `[standard]` extras 来安装 FastAPI 和一个生产就绪的 Web 服务器，如 Uvicorn。

```console title="使用标准依赖项安装 FastAPI"
$ pip install "fastapi[standard]"
```

`"fastapi[standard]"` 命令（引号很重要）会安装 FastAPI 以及用于为你的应用程序提供服务的 `uvicorn` 和其他有助于开发和实现高性能的可选依赖项。

## 创建你的第一个 API

创建一个名为 `main.py` 的文件并添加以下代码：

```python main.py icon=logos:python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```

### 代码解析

1.  **`from fastapi import FastAPI`**：此行从 `fastapi` 库中导入 `FastAPI` 类。该类是你 Web 应用程序的核心。

2.  **`app = FastAPI()`**：在这里，你创建了 `FastAPI` 类的一个实例。这个 `app` 变量将是创建所有 API 端点的主要交互点。

3.  **`@app.get("/")`**：这是一个**路径操作装饰器**。它告诉 FastAPI，紧随其后的函数负责处理发往以下目标的请求：
    *   路径 `/`（根 URL）。
    *   使用 `GET` 操作（也称为 HTTP 方法）。

4.  **`async def root(): ...`**：这是你的**路径操作函数**。这是一个标准的 Python 函数，每当 FastAPI 收到对 URL `/` 的 `GET` 请求时，都会调用该函数。你可以将其定义为普通函数（`def root():`）或 `async` 函数（`async def root():`），如果你需要在其中使用 `await`。

5.  **`return {"message": "Hello World"}`**：你可以返回 `dict`、`list` 或其他标准的 Python 数据类型。FastAPI 会自动将此字典转换为 JSON 响应。

## 运行开发服务器

现在，从终端运行你的应用程序：

```console title="运行服务器"
$ fastapi dev main.py
```

`fastapi dev` 命令会找到你的 `app` 对象，并使用 Uvicorn 启动一个本地服务器。它还启用了自动重载功能，因此每当你保存代码更改时，服务器都会自动重启。

你应该会看到类似以下的输出：

```console 终端输出
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
INFO:     Started reloader process [12345] using WatchFiles
INFO:     Started server process [12347]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

此输出告诉你，你的应用程序正在 `http://127.0.0.1:8000` 上运行。

## 检查一下

打开你的 Web 浏览器并访问 `http://127.0.0.1:8000`。

你应该会看到以下 JSON 响应：

```json 响应 icon=mdi:code-json
{
  "message": "Hello World"
}
```

## 交互式 API 文档

FastAPI 最好的功能之一是自动生成交互式 API 文档。你无需执行任何额外操作即可启用它。

只需在浏览器中访问 `http://127.0.0.1:8000/docs` 即可。

你将看到 Swagger UI，它提供了一个丰富的界面，可直接在浏览器中浏览和测试你的 API 端点。

![“快速入门”应用的 Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

FastAPI 还提供了使用 ReDoc 的备选文档样式。你可以通过 `http://127.0.0.1:8000/redoc` 访问它。

## 总结

恭喜！你已成功：

- 安装了 FastAPI 和一个 Web 服务器。
- 创建了一个带单个端点的简单 API。
- 使用带自动重载功能的开发服务器运行了该应用程序。
- 查看了 JSON 响应和自动生成的交互式文档。

## 后续步骤

现在你已经有了一个基本的应用程序在运行，让我们来学习如何处理来自客户端的数据。在下一节中，你将学习**路径和查询参数**。

<x-card data-title="下一步：路径和查询参数" data-icon="lucide:arrow-right-circle" data-href="/tutorials/path-and-query-parameters" data-cta="继续教程">
  学习如何声明作为 URL 路径和查询字符串一部分的参数。
</x-card>