# 处理响应

在构建 API 时，您可以精细地控制返回给客户端的内容。FastAPI 允许您定义响应的结构、更改默认的 HTTP 状态码以及设置自定义的标头或 Cookie。这可以确保您的 API 是可预测、文档完善且其行为完全符合客户端的预期。

本节将介绍管理 API 输出的主要方法。

## 使用 `response_model` 参数

控制响应最常用的方法是在*路径操作装饰器*中声明一个 `response_model`。这个模型（通常是 Pydantic 模型）有多种用途：

- **数据过滤**：它能确保返回的数据符合模型的结构。返回对象中任何未在 `response_model` 中定义的数据都将被排除。
- **数据校验**：它会校验输出数据。如果返回对象的类型不正确（例如，期望的是 `int`，但实际是 `float`），FastAPI 将会引发错误。
- **文档**：它会将响应结构添加到 API 的 OpenAPI 文档中，让用户清楚地知道应该期望得到什么数据。

### 单个项目的响应模型

以下是如何为一个创建项目的端点声明 `response_model`。尽管函数接收和返回的是同一个 `item` 对象，但 `response_model` 保证了输出会匹配 `Item` 模型的结构。

```python title="main.py" icon=logos:python
from typing import Any, List, Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: List[str] = []


@app.post("/items/", response_model=Item)
async def create_item(item: Item) -> Any:
    return item
```

FastAPI 将使用这个 `response_model` 来过滤、校验和记录输出。

### 项目列表的响应模型

您也可以在 `response_model` 中使用 Python `typing` 模块中的类型提示，例如 `List`。

```python title="main.py" icon=logos:python
@app.get("/items/", response_model=List[Item])
async def read_items() -> Any:
    return [
        {"name": "Portal Gun", "price": 42.0},
        {"name": "Plumbus", "price": 32.0},
    ]
```

在这种情况下，FastAPI 将确保响应是一个 JSON 数组，其中每个对象都符合 `Item` 模型。

## 更改状态码

默认情况下，成功的响应使用 `200 OK` 状态码。您可以通过向*路径操作装饰器*添加 `status_code` 参数来轻松覆盖此行为。这对于创建端点尤其有用，因为 `201 Created` 状态码更为合适。

```python title="main.py" icon=logos:python
from fastapi import FastAPI

app = FastAPI()


@app.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}
```

现在，向 `/items/` 发送成功的 POST 请求将返回 `201 Created` 状态码。

## 设置自定义标头和 Cookie

为了实现更高级的控制，例如设置自定义标头或 Cookie，您可以直接返回一个 `Response` 对象。FastAPI 提供了多种 `Response` 子类，其中 `JSONResponse` 是 API 中最常用的。

### 自定义标头

要向响应中添加自定义标头，可以创建一个 `JSONResponse` 实例并将标头作为字典传递。

```python title="main.py" icon=logos:python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()


@app.get("/headers/")
def get_headers():
    content = {"message": "Hello World"}
    headers = {"X-Cat-Dog": "alone in the world", "Content-Language": "en-US"}
    return JSONResponse(content=content, headers=headers)
```

客户端现在将收到自定义的 `X-Cat-Dog` 和 `Content-Language` 标头。

### 设置 Cookie

同样，您可以通过创建一个 `JSONResponse` 对象并使用其 `set_cookie` 方法来设置 Cookie。

```python title="main.py" icon=logos:python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()


@app.post("/cookie/")
def create_cookie():
    content = {"message": "Come to the dark side, we have cookies"}
    response = JSONResponse(content=content)
    response.set_cookie(key="fakesession", value="fake-cookie-session-value")
    return response
```

## 直接返回响应

返回 `Response` 对象可以让您获得完全的控制权。当您需要序列化非 JSON 原生支持的数据类型（例如 `datetime` 对象）时，这也很有用。为此，FastAPI 提供了一个 `jsonable_encoder` 实用工具。

```python title="main.py" icon=logos:python
from datetime import datetime
from typing import Union

from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from fastapi.responses import JSONResponse
from pydantic import BaseModel


class Item(BaseModel):
    title: str
    timestamp: datetime
    description: Union[str, None] = None


app = FastAPI()


@app.put("/items/{id}")
def update_item(id: str, item: Item):
    json_compatible_item_data = jsonable_encoder(item)
    return JSONResponse(content=json_compatible_item_data)
```

在这里，`jsonable_encoder` 会在将 `Item` 模型中的 `datetime` 对象传递给 `JSONResponse` 之前，将其转换为适合 JSON 的字符串格式。

## 更多响应类型

FastAPI 基于 Starlette 构建，为不同的使用场景提供了一系列响应类。您可以直接从 `fastapi.responses` 导入它们。

<x-cards data-columns="3">
  <x-card data-title="JSONResponse" data-icon="lucide:code-json">JSON 数据的默认响应类型。支持高性能编码器。</x-card>
  <x-card data-title="HTMLResponse" data-icon="lucide:code">用于直接向浏览器返回 HTML 内容。</x-card>
  <x-card data-title="PlainTextResponse" data-icon="lucide:file-text">用于发送纯文本响应。</x-card>
  <x-card data-title="RedirectResponse" data-icon="lucide:corner-up-right">发出 HTTP 重定向，指向不同的 URL。</x-card>
  <x-card data-title="StreamingResponse" data-icon="lucide:workflow">以流式传输响应体，适用于大文件或实时数据。</x-card>
  <x-card data-title="FileResponse" data-icon="lucide:file">从磁盘以流式传输文件作为响应。</x-card>
</x-cards>

更多详细信息，请参阅 [Responses API 参考](./api-reference-responses.md)。

借助这些工具，您可以精确控制 API 响应的各个方面。接下来，您将学习一个用于管理依赖和共享逻辑的强大系统。

---

接下来，让我们探讨如何使用 [依赖注入](./user-guide-dependency-injection.md) 来组织代码和处理依赖。