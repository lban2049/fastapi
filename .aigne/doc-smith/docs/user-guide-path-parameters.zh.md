# 路径参数

你可以使用与 Python f-strings 相同的语法来声明路径“参数”或“变量”：

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

路径参数 `item_id` 的值将作为参数 `item_id` 传递给你的函数。

因此，如果你运行此示例并访问 `http://127.0.0.1:8000/items/foo`，你将看到如下响应：

```json
{
  "item_id": "foo"
}
```

## 带类型的路径参数

你可以使用标准的 Python 类型注解在函数中声明路径参数的类型。

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

在本例中，`item_id` 被声明为 `int` 类型。

这将在你的函数内部为你提供编辑器支持，包括错误检查、代码补全等功能。

通过此类型声明，FastAPI 提供了自动的数据解析和验证。如果你访问 `http://127.0.0.1:8000/items/3`，`item_id` 将被转换为整数 `3`。但是，如果你访问 `http://127.0.0.1:8000/items/foo`，你将看到一个有用的 HTTP 错误：

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

这是因为路径参数 `item_id` 未能成功解析为 `int`。

## 顺序很重要

在创建*路径操作*时，你可能会遇到这样的情况：你有一个固定路径，如 `/users/me`，还有一个带参数的路径，如 `/users/{user_id}`。由于路径操作是按顺序评估的，你需要确保 `/users/me` 的路径在 `/users/{user_id}` 之前声明。

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}


@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

否则，`/users/{user_id}` 的路径也会匹配 `/users/me`，并认为它正在接收一个值为 `"me"` 的参数 `user_id`。

## 预定义值

如果你有一个*路径操作*只应接收一组预定义的值，你可以使用标准的 Python `Enum`。

创建一个继承自 `str` 和 `Enum` 的 `Enum`。

```python
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

通过继承 `str`，API 文档将能够知道这些值必须是字符串，并能够正确地呈现它们。

然后你可以在类型注解中使用它。路径参数将根据枚举中的值集进行验证。

如果你访问 `http://127.0.0.1:8000/models/resnet`，你将得到如下响应：

```json
{
  "model_name": "resnet",
  "message": "Have some residuals"
}
```

交互式文档将自动在下拉菜单中显示可用值。

## 包含路径的路径参数

你可以使用 URL 转换器声明一个本身包含路径的路径参数。

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

在此示例中，参数 `file_path` 可以包含斜杠 (`/`)，例如 `home/johndoe/myfile.txt`。

如果你访问 `http://127.0.0.1:8000/files/home/johndoe/myfile.txt`，响应将是：

```json
{
  "file_path": "home/johndoe/myfile.txt"
}
```

## 路径参数和数值校验

FastAPI 允许你为参数声明额外的校验和元数据。对于路径参数，你可以使用 `Path`。

要使用它，你首先需要从 `fastapi` 导入 `Path`：

```python
from fastapi import FastAPI, Path
```

由于路径参数始终是必需的，你必须使用 `...` 作为默认值来声明它们，以将其标记为必需。

```python
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

当你需要声明查询参数 `q` 和路径参数 `item_id` 时，Python 的语法规则规定，带默认值的参数必须位于没有默认值的参数之后。

但是，你可以在函数签名中使用 `*` 来重新排序它们。这会告诉 Python，所有后续参数都是仅限关键字的参数，它们的顺序无关紧要。

```python
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

使用 `Path` 你还可以声明数值校验。

参数 `ge=1` 将强制要求 `item_id` 必须是“大于或等于”1 的整数。

```python
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

你还可以使用 `gt` (大于) 和 `le` (小于或等于)。

```python
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

数值校验也适用于 `float` 值。这不仅对 `Path` 有用，对 `Query` 参数也同样有用。

这里我们添加了一个 `size` 查询参数，它必须大于 0 且小于 10.5。

```python
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

你已经了解了如何：
- 为路径参数使用 Python 类型提示。
- 控制路径操作的顺序。
- 使用 `Enum` 定义预定义的、允许的路径参数值。
- 定义本身可以包含路径的路径参数。
- 使用 `Path` 声明元数据和数值校验。

现在你可以继续学习[查询参数](./user-guide-query-parameters.md)。