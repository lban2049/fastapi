# 请求体

当需要从客户端（例如浏览器）向 API 发送数据时，会以**请求体**的形式发送。这通常用于创建或更新数据的操作，例如 `POST`、`PUT` 和 `PATCH`。

FastAPI 利用 Pydantic 模型来定义、验证和记录这些请求体，从而可以用极少的代码轻松处理复杂的数据结构。

## 创建第一个请求体

首先，将数据结构定义为 Pydantic 模型。该模型声明了所期望数据的结构、字段及其类型。

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

在此示例中：
- 我们定义了一个 `Item` 模型，包含 `name`、`description`、`price` 和 `tax` 字段。
- `create_item` 函数接受一个类型提示为 `Item` 的 `item` 参数。

仅需这一个类型声明，FastAPI 就会：
1.  读取 JSON 格式的请求体。
2.  将类型转换为相应的 Python 类型。
3.  验证数据。如果数据无效，它会返回一个清晰的错误，指明问题所在。
4.  在 `item` 参数中提供接收到的数据。
5.  为模型生成 JSON Schema，该 Schema 将用于 OpenAPI 文档。

## 使用模型

在函数内部，可以直接访问模型对象的所有属性。如果需要，也可以将模型转换为字典。

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

在这里，如果提供了 `tax`，我们会将传入的 `item` 转换为字典，并添加一个计算得出的 `price_with_tax` 字段。

## 组合路径、查询和请求体参数

可以在同一个函数中声明路径参数、查询参数和请求体参数。FastAPI 会正确识别每一个参数，并从相应的来源获取数据。

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

在这个 `update_item` 函数中：
- `item_id` 是一个**路径参数**。
- `item` 是一个**请求体参数**。
- `q` 是一个**查询参数**。

FastAPI 会同时处理所有这些参数。

## 多个请求体参数和字段

有时可能需要接收多个请求体参数，或者将单个模型嵌入一个 JSON 键中。对于这些情况，可以使用 `Body` 工具。

### 嵌入单个请求体参数

如果希望请求体是一个 JSON 对象，且该对象包含一个特定键（例如 `"item"`），该键的值为模型数据，那么可以使用 `Body(embed=True)`。

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

请求体将不再是这样：
```json
{
    "name": "Foo",
    "description": "A very nice Item",
    "price": 35.4,
    "tax": 3.2
}
```

FastAPI 现在将期望的请求体是这样：
```json
{
    "item": {
        "name": "Foo",
        "description": "A very nice Item",
        "price": 35.4,
        "tax": 3.2
    }
}
```

### 使用 `Field` 添加丰富验证

注意，在上面的示例中，我们还使用了 Pydantic 的 `Field`。这允许为模型的属性添加额外的验证和元数据，例如 `title`、`description`、`max_length` 以及像 `gt`（大于）这样的数值约束。

这些额外信息也会用于为 API 文档生成更详细、更准确的 OpenAPI schema。

## 嵌套模型

通过在 Pydantic 模型中嵌套使用其他 Pydantic 模型，可以定义复杂的嵌套 JSON 对象。例如，可以包含子模型列表，或将其他模型作为属性。

下面是一个 `Item` 包含标签列表的示例。

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

该端点的一个有效请求体可能如下所示：

```json
{
    "name": "Foo",
    "description": "A very nice Item",
    "price": 35.4,
    "tax": 3.2,
    "tags": ["electronics", "hardware", "computer"]
}
```

FastAPI 会自动处理这些嵌套结构的验证。`tags` 字段甚至可以是 `list[OtherModel]`，以支持更深层嵌套的数据。

既然已经了解了如何处理客户端发送的数据，接下来让我们在[处理响应](./user-guide-handling-responses.md)部分探讨如何控制返回的数据。