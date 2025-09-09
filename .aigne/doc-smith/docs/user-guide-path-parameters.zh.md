# 路径参数

你可以使用与 Python 格式化字符串相同的语法来声明路径“参数”或“变量”。这使你能够捕获 URL 路径的一部分，并在你的*路径操作函数*中使用它们。

## 声明路径参数

路径参数在路径中使用花括号 `{}` 定义。

```python title="main.py" icon=logos:python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

路径参数 `item_id` 的值将作为参数 `item_id` 传递给你的函数。因此，如果你运行此示例并访问 `http://127.0.0.1:8000/items/foo`，你将看到以下响应：

```json
{
  "item_id": "foo"
}
```

## 带类型的路径参数

你可以使用标准的 Python 类型提示，在函数中声明路径参数的类型。

```python title="main.py" icon=logos:python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

在这种情况下，`item_id` 被声明为 `int`。通过此类型声明，FastAPI 为你提供了自动请求“解析”功能。如果你在浏览器中访问 `http://127.0.0.1:8000/items/3`，路径中的值 `"3"` 将被解析并转换为整数 `3`。

响应将是：

```json
{
  "item_id": 3
}
```

### 数据校验

如果使用 `int` 类型提示访问 URL `/items/foo`，你将看到一个清晰的 HTTP 错误消息，指示路径参数的类型无效。

```json
{
  "detail": [
    {
      "loc": [
        "path",
        "item_id"
      ],
      "msg": "value is not a valid integer",
      "type": "type_error.integer"
    }
  ]
}
```

这种自动校验由 Pydantic 提供，FastAPI 在底层使用它。

## 顺序很重要

在创建*路径操作*时，你可能会遇到这样的情况：你有一个固定路径（如 `/users/me`）和一个捕获参数的路径（如 `/users/{user_id}`）。

由于路径操作是按顺序评估的，你需要确保固定端点的路径在带参数的路径*之前*声明。

```python title="main.py" icon=logos:python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}


@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

如果 `/users/{user_id}` 先声明，它将匹配 `/users/me`，并认为 `user_id` 参数是字符串 `"me"`。

## 使用枚举的预定义值

如果你有一个只能接受少数预定义值的路径参数，你可以使用标准的 Python `Enum`。

创建一个继承自 `str` 和 `Enum` 的 `Enum` 类。

```python title="main.py" icon=logos:python
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

FastAPI 将使用该枚举来校验路径参数，并且还会在交互式 API 文档中包含可用值。

## 包含路径的路径参数

在某些情况下，你可能需要一个路径参数来包含文件路径，其中包含斜杠 (`/`)。你可以使用来自 Starlette（底层 ASGI 框架）的路径转换器来实现这一点。

要捕获路径，请使用语法 `{file_path:path}`。

```python title="main.py" icon=logos:python
from fastapi import FastAPI

app = FastAPI()


@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

如果你向 `/files/home/johndoe/myfile.txt` 发出请求，`file_path` 参数将包含完整路径 `home/johndoe/myfile.txt`。

## 数值校验

对于更高级的校验，特别是针对数字，你可以使用 `Path()` 函数。

首先，从 `fastapi` 导入 `Path`：

```python
from fastapi import FastAPI, Path
```

你可以使用 `Path()` 来添加额外的元数据和校验检查。

### 添加元数据

你可以为路径参数添加 `title` 和其他元数据。这些信息将用于生成的 OpenAPI 模式和交互式 API 文档。

```python title="main.py" icon=logos:python
from typing import Union

from fastapi import FastAPI, Path, Query

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    item_id: int = Path(title="The ID of the item to get"),
    q: Union[str, None] = Query(default=None, alias="item-query"),
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

### 按需排序参数

当你使用 `Path()` 时，你可能想要对参数重新排序。例如，将一个必需的查询参数 `q` 放在路径参数之前。Python 要求带默认值的参数必须在没有默认值的参数之后。你可以在函数参数中使用 `*` 来表示所有后续参数都只能通过关键字传递。

```python title="main.py" icon=logos:python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(*, item_id: int = Path(title="The ID of the item to get"), q: str):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

### 数值校验：大于或等于

使用 `Path()`，你可以声明数值约束。例如，要确保 `item_id` 是一个大于或等于 1 的整数，你可以使用 `ge=1`。

```python title="main.py" icon=logos:python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    *, item_id: int = Path(title="The ID of the item to get", ge=1), q: str
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

### 数值校验：大于和小于或等于

你也可以使用 `gt`（大于）和 `le`（小于或等于）。

```python title="main.py" icon=logos:python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    *,
    item_id: int = Path(title="The ID of the item to get", gt=0, le=1000),
    q: str,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

### 使用浮点数进行数值校验

数值校验也适用于 `float` 值。此示例展示了如何结合路径参数和查询参数的校验。

```python title="main.py" icon=logos:python
from fastapi import FastAPI, Path, Query

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    *,
    item_id: int = Path(title="The ID of the item to get", ge=0, le=1000),
    q: str,
    size: float = Query(gt=0, lt=10.5),
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    if size:
        results.update({"size": size})
    return results
```

## 总结

你可以使用类似 f-string 的语法来声明路径参数。FastAPI 为路径参数提供了强大的功能：

*   **类型提示**：自动解析和数据校验。
*   **顺序重要性**：固定路径应在带变量的路径之前声明。
*   **枚举**：用于预定义的、允许的值。
*   **路径转换器**：用于捕获包含斜杠的路径。
*   **`Path()`**：用于添加丰富的元数据和数值校验（`gt`、`ge`、`lt`、`le`）。

接下来，我们将探讨如何声明[查询参数](./user-guide-query-parameters.md)。
