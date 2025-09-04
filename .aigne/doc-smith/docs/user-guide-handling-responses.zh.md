# 处理响应

构建 API 时，你可以精细地控制返回给客户端的内容。FastAPI 允许你定义响应的结构、更改默认的 HTTP 状态码以及设置自定义的标头或 Cookie。这能确保你的 API 是可预测、文档完善且其行为完全符合客户端的预期。

本节将介绍管理 API 输出的主要方法。

## 使用 `response_model` 参数

控制响应最常用的方法是在*路径操作装饰器*中声明一个 `response_model`。该模型（通常是 Pydantic 模型）有以下几个用途：

- **数据过滤**：确保返回的数据符合模型的结构。返回对象中任何未在 `response_model` 中定义的数据都将被排除。
- **数据校验**：校验输出数据。如果返回对象的类型不正确（例如，期望是 `int` 类型，但实际是 `float` 类型），FastAPI 将会引发错误。
- **文档**：将响应模型添加到 API 的 OpenAPI 文档中，让用户清楚地知道应该预期什么样的数据。

### 单个项目的响应模型

下面展示了如何为一个创建项目的端点声明 `response_model`。尽管函数接收并返回的是同一个 `item` 对象，但 `response_model` 能保证输出与 `Item` 模型的结构相匹配。

```python
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

你也可以在 `response_model` 中使用 Python 的 `typing` 模块中的类型提示，例如 `List`。

```python
@app.get("/items/", response_model=List[Item])
asyncdiff --git a/docs/user-guide/handling-responses.md b/docs/user-guide/handling-responses.md
index b557262..e31885b 100644
--- a/docs/user-guide/handling-responses.md
+++ b/docs/user-guide/handling-responses.md
@@ -53,7 +53,7 @@
 
 You can also use type hints from Python's `typing` module, like `List`, in the `response_model`.
 
-```python
+```python
 @app.get("/items/", response_model=List[Item])
 async def read_items() -> Any:
     return [
         {"name": "Portal Gun", "price": 42.0},
         {"name": "Plumbus", "price": 32.0},
     ]
 ```

在这种情况下，FastAPI 将确保响应是一个 JSON 数组，其中每个对象都符合 `Item` 模型。

## 更改状态码

默认情况下，成功的响应使用 `200 OK` 状态码。你可以通过向*路径操作装饰器*添加 `status_code` 参数来轻松覆盖此默认值。这对于创建操作的端点尤其有用，因为 `201 Created` 状态码更为合适。

```python
from fastapi import FastAPI

app = FastAPI()


@app.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}

```

现在，向 `/items/` 发送一个成功的 POST 请求将返回 `201 Created` 状态码。

## 设置自定义标头和 Cookie

要实现更高级的控制，例如设置自定义标头或 Cookie，你可以直接返回一个 `Response` 对象。FastAPI 提供了多个 `Response` 子类，其中 `JSONResponse` 是 API 中最常用的。

### 自定义标头

要向响应中添加自定义标头，可以创建一个 `JSONResponse` 实例，并将标头作为字典传递。

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()


@app.get("/headers/")
def get_headers():
    content = {"message": "Hello World"}
    headers = {"X-Cat-Dog": "alone in the world", "Content-Language": "en-US"}
    return JSONResponse(content=content, headers=headers)
```

现在，客户端将收到自定义的 `X-Cat-Dog` 和 `Content-Language` 标头。

### 设置 Cookie

同样，你可以通过创建一个 `JSONResponse` 对象并使用其 `set_cookie` 方法来设置 Cookie。

```python
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

直接返回 `Response` 对象可以让你获得完全的控制权。当你需要序列化非 JSON 原生数据类型（例如 `datetime` 对象）时，这也很有用。FastAPI 为此提供了一个 `jsonable_encoder` 工具。

```python
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

在这里，`jsonable_encoder` 在将 `Item` 模型中的 `datetime` 对象传递给 `JSONResponse` 之前，会将其转换为适合 JSON 的字符串格式。

## 更多响应类型

FastAPI 基于 Starlette 构建，为不同的使用场景提供了一系列响应类。你可以直接从 `fastapi.responses` 导入它们。

<x-cards data-columns="3">
  <x-card data-title="JSONResponse" data-icon="lucide:code-json">用于 JSON 数据的默认响应类型。支持高性能编码器。</x-card>
  <x-card data-title="HTMLResponse" data-icon="lucide:code">用于直接向浏览器返回 HTML 内容。</x-card>
  <x-card data-title="PlainTextResponse" data-icon="lucide:file-text">用于发送纯文本响应。</x-card>
  <x-card data-title="RedirectResponse" data-icon="lucide:corner-up-right">向另一个 URL 发出 HTTP 重定向。</x-card>
  <x-card data-title="StreamingResponse" data-icon="lucide:workflow">流式传输响应体，适用于大文件或实时数据。</x-card>
  <x-card data-title="FileResponse" data-icon="lucide:file">将磁盘中的文件作为响应进行流式传输。</x-card>
</x-cards>

更多详细信息，请参阅 [响应的 API 参考](./api-reference-responses.md)。

借助这些工具，你可以精确控制 API 响应的各个方面。接下来，你将学习一个用于管理依赖和共享逻辑的强大系统。

---

接下来，让我们通过 [依赖注入](./user-guide-dependency-injection.md) 来探索如何组织代码和处理依赖关系。