# 处理响应

在 FastAPI 中，你可以精细地控制返回给客户端的响应。这包括定义数据结构、设置 HTTP 状态码以及添加自定义标头或 Cookie。本节将介绍管理 API 输出的主要方法。

## 定义响应模型

你可以在任何*路径操作装饰器*中使用 `response_model` 参数来声明用于响应的模型。FastAPI 会使用此 `response_model` 来：

- 将输出数据转换为模型所定义的类型。
- 验证数据。
- 在 OpenAPI 路径操作中为响应添加 JSON Schema。
- 过滤输出数据，仅将模型中定义的字段包含在响应中。

### 示例：过滤响应数据

在这里，`response_model` 被设置为 `Item` 模型。即使函数返回的数据多于 `Item` 中定义的字段，FastAPI 也会对其进行过滤以匹配该模型。

```python
from typing import Any, List, Union

from fastapi import FastAPI
from pydantic import BaseModel

ap = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: List[str] = []


@app.post("/items/", response_model=Item)
async def create_item(item: Item) -> Any:
    return item


@app.get("/items/", response_model=List[Item])
async def read_items() -> Any:
    return [
        {"name": "Portal Gun", "price": 42.0},
        {"name": "Plumbus", "price": 32.0},
    ]
```

在此示例中，`create_item` 将返回该项目，但只会发送 `Item` Pydantic 模型中定义的字段。对于 `read_items`，响应将是一个对象列表，其中每个对象都符合 `Item` 模型。

## 更改状态码

默认情况下，路径操作返回 `200 OK` 状态码。你可以使用装饰器中的 `status_code` 参数来覆盖成功响应的默认状态码。

```python
from fastapi import FastAPI

app = FastAPI()


@app.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}
```

在这里，创建项目现在将返回 `201 Created` 状态码，这是成功创建新资源的标准 HTTP 状态。

## 使用直接的响应对象

对于设置自定义标头或 Cookie 等高级场景，你可以直接返回一个 `Response` 对象。FastAPI 提供了 `JSONResponse` 等多个辅助工具，以简化此过程。

### 自定义标头

要添加自定义标头，你可以创建一个 `JSONResponse` 并向其传递一个包含标头的字典。

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

此端点的响应将包含自定义的 `X-Cat-Dog` 和 `Content-Language` 标头。

### 设置 Cookie

要设置 Cookie，可以创建一个 `JSONResponse` 实例，然后使用其 `set_cookie()` 方法。

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

这样你就可以完全控制 `key`、`value`、`domain`、`path` 等 Cookie 属性。

## 使用 `jsonable_encoder` 处理复杂数据类型

有时需要返回包含非 JSON 兼容类型的数据，例如 `datetime` 对象或 Pydantic 模型。FastAPI 提供了 `jsonable_encoder` 实用工具，可将此类数据转换为与 JSON 兼容的结构。

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

在这里，`jsonable_encoder` 将 `item` 对象（包括其 `timestamp` 字段）转换为一个字典，其中的 `datetime` 会被表示为字符串。然后，这个字典可以安全地作为内容传递给 `JSONResponse`。

## 其他响应类型

FastAPI 提供了多种直接继承自 Starlette 的响应类，以满足不同需求。你可以直接在路径操作中返回其中任何一个类的实例。

| Class               | Description                                                 |
|---------------------|-------------------------------------------------------------|
| `Response`          | 基类，可用于自定义响应。           |
| `JSONResponse`      | 默认响应类，用于 JSON 编码的数据。                         |
| `HTMLResponse`      | 用于返回 HTML 内容。                                 |
| `PlainTextResponse` | 用于返回纯文本。                                   |
| `RedirectResponse`  | 用于发送 HTTP 重定向 (307)。                         |
| `StreamingResponse` | 用于流式传输响应体。                              |
| `FileResponse`      | 用于以流式传输文件作为响应。                       |

对于高性能应用程序，你也可以在安装相应的库（`ujson` 或 `orjson`）后使用 `UJSONResponse` 或 `ORJSONResponse`。

## 后续步骤

你现在已经掌握了控制 API 响应各个方面的工具。要学习如何管理共享逻辑和依赖项，请继续阅读下一节关于[依赖注入](./user-guide-dependency-injection.md)的内容。