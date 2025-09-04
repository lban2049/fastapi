# 请求体

当需要从客户端（如浏览器）发送数据到 API 时，会将其作为**请求体**发送。

请求体是客户端发送给 API 的数据。**响应体**是 API 发送给客户端的数据。

API 几乎总是需要发送响应体，但客户端不一定总是需要发送请求体。

要声明请求体，可以使用 Pydantic 模型，并利用其所有功能和优点。

## 创建数据模型

首先，需要从 `pydantic` 导入 `BaseModel`。

然后，将数据模型声明为继承自 `BaseModel` 的类。为所有属性使用标准的 Python 类型。

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    return item
```

当模型属性有默认值时，它就不是必需的。否则，它就是必需的。使用 `None` 使其变为可选。

例如，在上面的模型中，`name` 和 `price` 是必需的，而 `description` 和 `tax` 是可选的。

## 将其声明为参数

要将其添加到*路径操作*中，可以像声明路径和查询参数一样声明它：

```python
@app.post("/items/")
async def create_item(item: Item):
    return item
```

……并将其类型声明为你创建的模型 `Item`。

仅通过该 Python 类型声明，**FastAPI** 将会：

*   以 JSON 格式读取请求体。
*   转换相应的类型（如果需要）。
*   验证数据。如果数据无效，它将返回一个清晰明了的错误，指出不正确数据的确切位置和描述。
*   在参数 `item` 中提供接收到的数据。
*   为模型生成 JSON Schema 定义，如果合理，也可以在项目的其他任何地方使用它们。
*   这些模式将成为生成的 OpenAPI 模式的一部分，并被自动文档 UI 使用。

## 使用模型

在函数内部，可以直接访问模型对象的所有属性：

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    item_dict = item.dict()
    if item.tax is not None:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```

## 请求体 + 路径参数

可以同时声明路径参数和请求体。**FastAPI** 会识别出与路径参数匹配的函数参数应从路径中获取，而已声明为 Pydantic 模型的函数参数应从请求体中获取。

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.dict()}
```

## 请求体 + 路径 + 查询参数

也可以同时声明**请求体**、**路径**和**查询**参数。

**FastAPI** 会识别它们中的每一个，并从正确的位置获取数据。

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, q: Union[str, None] = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result
```

函数参数将按以下方式被识别：

*   如果参数也在**路径**中声明，它将被用作路径参数。
*   如果参数是**单一类型**（如 `int`、`float`、`str`、`bool` 等），它将被解释为**查询**参数。
*   如果参数被声明为 **Pydantic 模型**类型，它将被解释为请求**体**。

## 混合多个参数

可以在*路径操作函数*中混合使用 `Path`、`Query` 和请求体声明，FastAPI 会处理所有这些声明。

```python
from typing import Union

from fastapi import FastAPI, Path
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


@app.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int = Path(title="The ID of the item to get", ge=0, le=1000),
    q: Union[str, None] = None,
    item: Union[Item, None] = None,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    if item:
        results.update({"item": item})
    return results
```

## 嵌套模型

可以通过嵌套 Pydantic 模型在请求体中定义复杂的嵌套 JSON 对象。

例如，一个项目可以有一个标签列表。为此，可以将 `tags` 属性定义为一个列表。

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: list = []


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

为了更好的类型安全和编辑器支持，可以更具体地指定列表中的项，例如 `tags: list[str] = []`。也可以使用其他 Pydantic 模型的列表来创建更深层次的嵌套。

## 嵌入单个请求体参数

默认情况下，如果在函数中声明单个 Pydantic 模型，其内容将被视为请求的直接主体。但是，可以指示 FastAPI 期望一个带有特定键的 JSON 对象。可以通过使用 `Body` 来实现这一点。

```python
from typing import Union

from fastapi import Body, FastAPI
from pydantic import BaseModel, Field

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = Field(
        default=None, title="The description of the item", max_length=300
    )
    price: float = Field(gt=0, description="The price must be greater than zero")
    tax: Union[float, None] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item = Body(embed=True)):
    results = {"item_id": item_id, "item": item}
    return results
```

在这种情况下，FastAPI 会期望一个类似这样的请求体：

```json
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    }
}
```

而不是：

```json
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2
}
```

这也演示了如何使用 `Field` 为 Pydantic 模型属性添加额外的验证和元数据。

---

现在已经了解了如何处理从客户端发送的数据，接下来将探讨如何控制返回的内容。

接下来，学习如何配置[处理响应](./user-guide-handling-responses.md)。
