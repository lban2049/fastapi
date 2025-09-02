# 查询参数

当你声明的函数参数不属于路径参数时，它们将被自动解析为“查询”参数。

查询是指 URL 中 `?` 符号之后出现的键值对集合。例如，在 URL `http://127.0.0.1:8000/items/?skip=0&limit=10` 中，查询参数就是 `skip` 和 `limit`。

它们是 URL 的标准组成部分，FastAPI 知道如何提取它们。

要了解如何声明 URL 路径中的参数，请参阅[路径参数](./user-guide-path-parameters.md)文档。

## 默认值

查询参数可以像 Python 函数的参数一样定义默认值。如果客户端没有为带有默认值的参数提供值，则会使用该默认值。

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

在此示例中，`skip` 和 `limit` 是查询参数。由于它们有默认值，你可以像这样访问端点：

*   `http://127.0.0.1:8000/items/`

这将使用默认值 `skip=0` 和 `limit=10`。你也可以提供特定值：

*   `http://127.0.0.1:8000/items/?skip=20`（默认使用 `limit=10`）
*   `http://127.0.0.1:8000/items/?skip=0&limit=5`

## 可选参数

你可以通过将参数的默认值设置为 `None` 并使用 Python 的 `typing` 库中的 `Union` 或 `Optional`，来声明可选的查询参数。

```python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Union[str, None] = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}

```

这里，查询参数 `q` 是可选的。如果你调用 `/items/foo`，`q` 将为 `None`。如果你调用 `/items/foo?q=somequery`，`q` 将为 `"somequery"`。

## 查询参数的类型转换

FastAPI 会根据你的类型提示自动转换查询参数的类型。例如，布尔类型的参数会正确解析 `true`、`on`、`1` 或 `yes` 等值。

```python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Union[str, None] = None, short: bool = False):
    item = {"item_id": item_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```

如果你访问 URL `/items/foo?short=true`，`short` 参数将被转换为布尔值 `True`。

## 多个路径参数和查询参数

你可以按任意顺序声明多个路径参数和查询参数。FastAPI 非常智能，能够根据它们的名称和声明位置来区分它们。

```python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: Union[str, None] = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```

在此示例中，`user_id` 和 `item_id` 是路径参数，而 `q` 和 `short` 是查询参数。

## 必需的查询参数

如果你声明一个没有默认值的查询参数，它就成为必需参数。客户端必须在 URL 中为其提供一个值。

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item
```

如果你尝试在没有 `needy` 查询参数的情况下访问 `/items/foo-item`，FastAPI 将返回一个清晰的 HTTP 错误，指示该参数缺失。

你也可以混合使用必需参数、可选参数和带默认值的参数：

```python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_user_item(
    item_id: str, needy: str, skip: int = 0, limit: Union[int, None] = None
):
    item = {"item_id": item_id, "needy": needy, "skip": skip, "limit": limit}
    return item
```

在这里：
- `needy` 是一个必需的 `str`。
- `skip` 是一个 `int`，默认值为 `0`。
- `limit` 是一个可选的 `int`。

## 使用 `Query` 进行高级校验和元数据定义

如需进行更高级的校验或为查询参数添加元数据，你可以使用 `Query` 函数。它允许你设置最小长度、最大长度和正则表达式等约束条件。

首先，从 `fastapi` 导入 `Query`：

```python
from fastapi import Query
```

### 字符串校验

你可以对字符串参数应用多种校验。例如，你可以强制规定最小和最大长度。

```python
from typing import Union

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Union[str, None] = Query(default=None, min_length=3, max_length=50),
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

现在，如果你提供的 `q` 参数短于 3 个字符或长于 50 个字符，你将收到一个错误。

你还可以强制使用正则表达式模式：

```python
from typing import Union

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Union[str, None] = Query(
        default=None, min_length=3, max_length=50, pattern="^fixedquery$"
    ),
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```
在这种情况下，`q` 参数必须是 `fixedquery`。

### 参数别名

有时你需要一个不是有效 Python 标识符的查询参数名称（例如 `item-query`）。你可以使用 `Query` 中的 `alias` 参数为该参数定义一个别名。

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: str = Query(alias="item-query")):
    return {"q": q}
```

现在，你可以使用 `http://127.0.0.1:8000/items/?item-query=somevalue` 来调用该端点。

### 弃用参数

你可以通过设置 `deprecated=True` 来将参数标记为已弃用。这将在交互式 API 文档中反映出来。

```python
from typing import Union

from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Union[str, None] = Query(
        default=None,
        alias="item-query",
        title="Query string",
        description="Query string for the items to search in the database",
        min_length=3,
        deprecated=True,
    )
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

---

## 总结

你已经学会了如何：
- 定义带有默认值、可选值和必需值的查询参数。
- 让 FastAPI 处理自动类型转换。
- 使用 `Query` 添加 `min_length`、`max_length` 和 `pattern` 等高级校验。
- 为参数创建别名并将其标记为已弃用。

接下来，你将学习如何处理请求体中发送的数据。

<x-card data-title="下一步：请求体" data-icon="lucide:file-json-2" data-href="/user-guide/request-body">
  学习如何使用 Pydantic 模型接收和校验来自请求体的数据。
</x-card>
