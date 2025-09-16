# 请求体

当你需要从客户端（如浏览器）向 API 发送数据时，你会将其作为**请求体**发送。请求体是客户端发送给 API 的数据。**响应体**是 API 发送回客户端的数据。

你的 API 几乎总是需要发送响应体。但客户端不一定总是需要发送请求体。要声明请求体，你可以使用 Pydantic 模型，它为你提供了数据验证、转换和文档化的所有功能。

## 创建你的 Pydantic 模型

首先，你需要将数据结构定义为 Pydantic `BaseModel`。

```python title="main.py" icon=logos:python
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

通过使用类型提示 `Item` 声明 `item` 参数，FastAPI 将：

*   以 JSON 格式读取请求体。
*   转换相应的数据类型（如果需要）。
*   验证数据。如果数据无效，它将返回一个明确的错误，指明错误数据的确切位置和内容。
*   在参数 `item` 中为你提供接收到的数据。
*   为你的模型生成 JSON Schema 定义，这些定义将用于 OpenAPI 文档。

### 参数详情

<x-field data-name="item" data-type="Item" data-required="true" data-desc="在请求体中接收到的 item 对象。">
  <x-field data-name="name" data-type="string" data-required="true" data-desc="item 的名称。">
  </x-field>
  <x-field data-name="description" data-type="string | None" data-required="false" data-desc="item 的可选描述。">
  </x-field>
  <x-field data-name="price" data-type="float" data-required="true" data-desc="item 的价格。">
  </x-field>
  <x-field data-name="tax" data-type="float | None" data-required="false" data-desc="可选的税额。">
  </x-field>
</x-field>

### 响应示例

当你发送一个包含有效 JSON 请求体的请求时，API 将原样返回。

```json
{
  "name": "Sample Item",
  "description": "A sample description",
  "price": 19.99,
  "tax": 1.60
}
```

## 使用模型

在你的*路径操作函数*内部，你可以直接访问模型对象的所有属性：

```python title="main.py" icon=logos:python
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

在此示例中，我们使用 `item.dict()` 将 Pydantic 模型转换为字典，如果提供了 `tax`，则计算一个新的 `price_with_tax` 值，并返回更新后的字典。

### 响应示例

```json
{
  "name": "Sample Item",
  "description": "A sample description",
  "price": 19.99,
  "tax": 1.60,
  "price_with_tax": 21.59
}
```

## 混合路径、查询和请求体

你可以同时声明路径参数、查询参数和请求体参数。FastAPI 会识别它们中的每一个，并从正确的位置获取数据。

```python title="main.py" icon=logos:python
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

参数将被识别如下：

*   **`item_id`**：路径参数，因为它在路径中声明。
*   **`q`**：查询参数，因为它是一个单一类型。
*   **`item`**：请求体参数，因为它被声明为 Pydantic 模型。

## 使用 `Field` 添加额外验证

你可以使用 Pydantic 的 `Field` 为模型属性声明更多的验证和元数据。

```python title="main.py" icon=logos:python
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

在这里，我们添加了：
*   为 `description` 添加了 `title` 和 `max_length`。
*   为 `price` 添加了 `description` 和一个验证规则（`gt=0`，即大于 0）。

### 嵌入单个请求体参数

注意函数签名中的 `item: Item = Body(embed=True)`。默认情况下，FastAPI 期望直接接收 JSON 请求体。但是，如果使用 `Body(embed=True)`，它会期望请求体嵌入在一个键中。客户端必须发送 `{"item": {"name": "Foo", ...}}`，而不是 `{"name": "Foo", ...}`。

## 嵌套模型

Pydantic 模型可以嵌套。你可以定义一个模型，其属性包含其他模型、列表等。

```python title="main.py" icon=logos:python
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

在此示例中，`Item` 模型包含一个 `tags` 属性，它是一个列表。客户端可以为此字段发送一个 JSON 数组。

**请求体示例**
```json
{
  "name": "T-Shirt",
  "description": "A nice cotton t-shirt",
  "price": 15.50,
  "tax": 1.24,
  "tags": ["clothing", "apparel", "summer"]
}
```

## 表单数据

当你需要接收表单字段而不是 JSON 时，可以使用 `Form`。这通常用于通过 `application/x-www-form-urlencoded` 发送的数据。

要使用表单，你首先需要安装 `python-multipart`：

```bash
pip install python-multipart
```

然后，在你的路径操作中使用 `Form`：

```python title="main.py" icon=logos:python
from fastapi import FastAPI, Form

app = FastAPI()


@app.post("/login/")
async def login(username: str = Form(), password: str = Form()):
    return {"username": username}
```

## 文件上传

FastAPI 也支持使用 `File` 和 `UploadFile` 进行文件上传。这也需要安装 `python-multipart`。

处理上传主要有两种方式：

1.  **作为字节**: 使用 `bytes = File()`。这适用于小文件，因为它将全部内容存储在内存中。
2.  **作为 `UploadFile`**: 使用 `file: UploadFile`。这对于大文件更高效，因为它会将文件流式传输到磁盘。

```python title="main.py" icon=logos:python
from fastapi import FastAPI, File, UploadFile

app = FastAPI()


@app.post("/files/")
async def create_file(file: bytes = File()):
    return {"file_size": len(file)}


@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile):
    return {"filename": file.filename}
```

`UploadFile` 对象有几个有用的属性和方法，包括：
*   `filename`：上传文件的名称。
*   `content_type`：文件的内容类型（MIME 类型）。
*   `file`：一个 `SpooledTemporaryFile`（一个类文件对象）。
*   `async` 方法，如 `read()`、`write()` 和 `seek()`。

现在你对如何处理各种类型的请求体有了扎实的理解。你可以接收简单的 JSON、嵌套结构、表单数据，甚至文件上传。

接下来，让我们探讨如何管理依赖项并为你的应用程序添加安全性。你可以继续阅读[依赖项和安全性](./tutorials-dependencies-and-security.md)指南。