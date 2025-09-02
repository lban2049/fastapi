# 快速入门

本教程将指导你创建并运行你的第一个 FastAPI 应用。你将学习如何安装 FastAPI、编写一个简单的 API、在本地运行它，并探索其自动文档功能。

## 1. 安装 FastAPI

首先，你需要安装 FastAPI 和一个用于运行它的服务器。我们建议使用虚拟环境来管理你项目的依赖项。

使用 `pip` 安装 FastAPI 及其标准依赖项，其中包括 Uvicorn 服务器：

```console
$ pip install "fastapi[standard]"

---> 100%
```

**注意：**`"fastapi[standard]"` 两边的引号很重要，以确保该命令在所有终端中都能正常工作。

## 2. 创建代码

创建一个名为 `main.py` 的文件，并添加以下代码：

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

*   `@app.get("/")`: 根路径 `/` 上的一个 `GET` 操作，返回一个简单的 JSON 消息。
*   `@app.get("/items/{item_id}")`: 一个 `GET` 操作，它接受一个路径参数 `item_id`（必须是整数）和一个可选的查询参数 `q`（可以是字符串）。

## 3. 运行应用

在终端中使用 `fastapi` 命令运行服务器：

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

INFO:     Will watch for changes in these directories: ['/home/user/code/awesomeapp']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [2248755] using WatchFiles
INFO:     Started server process [2248757]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

`fastapi dev` 命令会启动一个本地开发服务器，当你对代码进行更改时，它会自动重新加载。

## 4. 查看效果

打开你的 Web 浏览器并访问 [http://127.0.0.1:8000/items/5?q=somequery](http://127.0.0.1:8000/items/5?q=somequery)。

你将看到以下 JSON 响应：

```json
{"item_id": 5, "q": "somequery"}
```

你已成功创建一个 API，它能接收路径参数和查询参数，验证其类型，并返回 JSON 响应。

## 5. 探索交互式文档

FastAPI 会为你的 API 自动生成交互式文档。

访问 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看 Swagger UI 文档：

![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

还有一个由 ReDoc 提供的备选文档界面。你可以通过 [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc) 访问它：

![ReDoc](https://fastapi.tiangolo.com/img/index/index-02-redoc-simple.png)

## 6. 添加请求体

现在，我们来修改 `main.py` 文件，以处理客户端在请求体中发送的数据。你可以使用 Pydantic 模型来定义请求体的结构。

用以下代码更新 `main.py`：

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    price: float
    is_offer: Union[bool, None] = None


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Union[str, None] = None):
    return {"item_id": item_id, "q": q}


@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item):
    return {"item_name": item.name, "item_id": item_id}
```

以下是更改的内容：
- 我们从 `pydantic` 导入了 `BaseModel`。
- 我们创建了一个继承自 `BaseModel` 的 `Item` 类，以定义请求体的模式。
- 我们添加了一个新的路径操作 `@app.put("/items/{item_id}")`，它接受 `PUT` 请求，并期望一个与 `Item` 模型匹配的请求体。

## 7. 查看自动更新

由于服务器会自动重新加载，你的更改已经生效。刷新文档页面 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)。交互式文档将会更新，以包含新的 `PUT` 端点：

![Swagger UI Updated](https://fastapi.tiangolo.com/img/index/index-03-swagger-02.png)

你可以使用“Try it out”按钮来填写参数，并直接在浏览器中与你的新端点进行交互。

## 回顾与后续步骤

恭喜！你已成功创建一个 FastAPI 应用，它能处理路径参数、查询参数和请求体，并配有自动生成的交互式文档。

既然你已了解基础知识，就可以深入学习 FastAPI 的核心概念了。

<x-cards data-columns="3">
  <x-card data-title="路径参数" data-icon="lucide:milestone" data-href="/user-guide/path-parameters">
    了解更多关于声明和验证路径参数的信息。
  </x-card>
  <x-card data-title="查询参数" data-icon="lucide:list-filter" data-href="/user-guide/query-parameters">
    了解如何为查询参数定义和添加更多验证。
  </x-card>
  <x-card data-title="请求体" data-icon="lucide:file-code-2" data-href="/user-guide/request-body">
    探索如何使用 Pydantic 模型接收和验证复杂的数据结构。
  </x-card>
</x-cards>