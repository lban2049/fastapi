# 路径和查询参数

构建 API 时，通常需要通过 URL 从客户端获取信息。FastAPI 可以轻松处理两种常见方式：**路径参数**和**查询参数**。通过使用标准的 Python 类型提示，你可以免费获得数据验证、转换和文档功能，从而使你的 API 更加健壮且易于使用。

本指南将引导你了解如何声明、指定类型和验证这两种参数。

## 路径参数

路径参数是 URL 路径的一部分，用大括号 `{}` 括起来。它们通常用于识别特定资源，例如物品的 ID。

### 声明路径参数

你可以在装饰器中和函数参数中声明路径参数。两者的名称必须匹配。

```python icon=logos:python title=main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

路径中 `item_id` 的值将作为参数 `item_id` 传递给你的函数。如果你运行此代码并访问 `http://127.0.0.1:8000/items/foo`，你将看到：

```json
{
  "item_id": "foo"
}
```

### 带类型的路径参数

你可以使用标准的 Python 类型提示来声明路径参数的类型。这正是 FastAPI 的强大之处。

```python icon=logos:python title=main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

现在，`item_id` 被声明为 `int`。FastAPI 将自动处理：

*   **验证：** 如果你访问 `/items/foo`，你会得到一个明确的 HTTP 422 Unprocessable Entity 错误，因为 'foo' 不是整数。
*   **转换：** 如果你访问 `/items/3`，FastAPI 会在将字符串 "3" 传递给你的函数之前，将其转换为整数 `3`。

这种自动验证和转换有助于防止错误，并为你的 API 用户改善开发者体验。

### 顺序很重要

当有多个路径操作可以匹配同一个 URL 时，FastAPI 会按照声明的顺序进行评估。固定路径应始终在可变路径之前声明。

```python icon=logos:python title=main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}


@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

由于 `/users/me` 在 `/users/{user_id}` 之前声明，因此对 `/users/me` 的请求将正确执行 `read_user_me` 函数。如果顺序颠倒，FastAPI 会将 "me" 视为 `user_id` 参数的值。

### 使用枚举预定义值

如果你的路径参数只应接受少数几个特定值，你可以使用标准的 Python `Enum`。

```python icon=logos:python title=main.py
from enum import Enum

from fastapi import FastAPI


class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"


app = FastAPI()


@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}

    return {"model_name": model_name, "message": "Have some residuals"}
```

通过使用 `ModelName` 作为类型提示，路径参数 `model_name` 将根据枚举的成员进行验证。对 `/models/alexnet` 的请求会成功，但对 `/models/unknown` 的请求将导致 422 错误，并附带一条有用的消息，指明允许的值。

### 包含路径的路径参数

有时你需要让路径参数包含文件路径，而文件路径中包含斜杠 (`/`)。你可以通过在装饰器中使用特殊语法 `{param_name:path}` 来告诉 FastAPI 捕获路径。

```python icon=logos:python title=main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

现在，对 `/files/home/johndoe/myfile.txt` 的请求将能正常工作，`file_path` 参数将包含完整的字符串 `home/johndoe/myfile.txt`。

## 查询参数

查询参数是 URL 中 `?` 之后出现的一组键值对。它们比路径参数更灵活，常用于过滤、排序或分页。

任何不属于路径的函数参数都会被自动解释为查询参数。

### 默认值

你可以像为任何 Python 函数参数提供默认值一样，为查询参数提供默认值。

```python icon=logos:python title=main.py
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

*   对 `/items/` 的请求将使用默认值：`skip=0` 和 `limit=10`。
*   对 `/items/?skip=20` 的请求将使用 `skip=20` 和 `limit=10`。
*   对 `/items/?skip=0&limit=5` 的请求将覆盖两个默认值。

### 可选参数

要使查询参数成为可选参数，你可以将其默认值声明为 `None`。

```python icon=logos:python title=main.py
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Union[str, None] = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

在这里，`q` 是一个可选的查询参数。如果客户端发送请求到 `/items/foo-item?q=somequery`，响应将包含 `q`。如果他们请求 `/items/foo-item`，`q` 参数将为 `None`，`if q:` 代码块将不会执行。

### 高级验证

FastAPI 允许使用 `Query` 函数对查询参数进行更强大的验证。你可以声明诸如最大长度、正则表达式等元数据。

```python icon=logos:python title=main.py
from typing import Union

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Union[str, None] = Query(default=None, max_length=50)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

在此示例中，`q` 参数仍然是可选的，但如果提供了该参数，其长度不能超过 50 个字符。这种声明式验证可以使你的应用程序逻辑保持整洁，并确保 API 的安全。

## 总结

你现在已经了解了如何处理最常见的 URL 参数类型。FastAPI 对现代 Python 特性的运用使你的 API 更加健壮、易于编写且能自生成文档。

*   **路径参数**在路径字符串中用 `{}` 声明。
*   **查询参数**被声明为不在路径中的函数参数。
*   **类型提示**为两者提供自动数据转换和验证。

接下来，我们将探讨如何处理在请求体中发送的数据，这对于在你的 API 中创建或更新项目至关重要。

[下一步：请求体](./tutorials-request-body.md)