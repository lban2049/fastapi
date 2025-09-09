# 快速入门

本教程将指导你使用 FastAPI 创建一个简单的 API，从安装到运行应用程序。学完本教程后，你将拥有一个功能齐全且带有交互式文档的 API。

## 安装

首先，你需要安装 FastAPI 和一个 ASGI 服务器，如 Uvicorn。

<x-card data-title="先决条件" data-icon="lucide:python" data-horizontal="true">
  FastAPI 要求 Python 3.8 或更高版本。你可以在终端中运行 `python --version` 来检查你的版本。
</x-card>

要安装 FastAPI 及其标准依赖项（包括 Uvicorn 服务器），请运行以下命令：

```console
$ pip install "fastapi[standard]"

---> 100%
```

这条命令将安装 FastAPI、用于数据验证的 Pydantic、用于底层 Web 功能的 Starlette 以及用于为应用程序提供服务的 Uvicorn。

## 创建你的第一个 API

现在，我们来编写 API 的代码。

1.  创建一个名为 `main.py` 的文件。
2.  向其中添加以下 Python 代码：

```python main.py icon=logos:python
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

*   位于根 URL `/` 的 `GET` 端点。
*   位于 `/items/{item_id}` 的 `GET` 端点，它接受一个整数 `item_id` 作为路径参数和一个可选的字符串 `q` 作为查询参数。

## 运行开发服务器

代码准备就绪后，从终端运行开发服务器：

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

`fastapi dev` 命令会启动一个启用了自动重新加载的本地服务器，非常适合开发。

## 检查

打开浏览器并访问 [http://127.0.0.1:8000/items/5?q=somequery](http://127.0.0.1:8000/items/5?q=somequery)。

你将看到以下 JSON 响应：

```json
{"item_id": 5, "q": "somequery"}
```

恭喜！你刚刚创建了一个能够接收并验证路径参数（整数 `item_id`）和查询参数（字符串 `q`）的 API。

## 交互式 API 文档

FastAPI 的最佳特性之一是其自动生成的文档。

在浏览器中访问 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)。你将看到由 Swagger UI 提供的交互式 API 文档：

![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

FastAPI 还提供了另一种文档界面。访问 [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc) 查看由 ReDoc 生成的文档：

![ReDoc](https://fastapi.tiangolo.com/img/index/index-02-redoc-simple.png)

这些文档页面是根据代码中的类型提示自动生成的，并允许你直接在浏览器中测试 API 端点。

## 升级你的 API

现在，我们来增强 API 以处理请求体。修改 `main.py`，加入一个使用 Pydantic 模型接收数据的 `PUT` 请求。

```python main.py icon=logos:python
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

你的开发服务器将自动重新加载。现在，刷新位于 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 的交互式文档。文档将会更新，包含新的 `PUT` 端点：

![更新后的 Swagger UI](https://fastapi.tiangolo.com/img/index/index-03-swagger-02.png)

点击“Try it out”按钮，填写参数并直接在浏览器中与 API 交互：

![Swagger UI 交互](https://fastapi.tiangolo.com/img/index/index-04-swagger-03.png)

点击“Execute”后，UI 会将请求发送到你的 API 并显示结果：

![Swagger UI 结果](https://fastapi.tiangolo.com/img/index/index-05-swagger-04.png)

## 总结

通过使用标准 Python 声明类型，你可以获得：

*   **编辑器支持**：自动补全和类型检查。
*   **数据验证**：自动验证传入数据并提供清晰的错误提示。
*   **数据转换**：将网络数据转换为 Python 类型。
*   **自动文档**：交互式文档界面。

FastAPI 负责处理验证、转换和文档，让你能够专注于应用程序的逻辑。

## 后续步骤

你已经成功创建、运行和升级了一个 FastAPI 应用程序。要了解如何处理更复杂的场景，例如验证约束和依赖注入，请继续阅读用户指南。

<x-card data-title="下一步：用户指南" data-icon="lucide:book-open" data-href="/user-guide/path-parameters" data-cta="探索路径参数">
  通过实际示例深入了解核心概念，从如何处理路径参数开始。
</x-card>