# 查询参数

当你声明的函数参数不属于路径参数时，它们会被自动解析为“查询”参数。

查询参数是 URL 中位于 `?` 之后、由 `&` 符号分隔的键值对。例如，在 URL `http://127.0.0.1:8000/items/?skip=0&limit=10` 中，查询参数是 `skip` 和 `limit`。

由于查询参数是 URL 的一部分，所以它们“天然”是字符串。但当你用 Python 类型（例如 `int`、`float`、`bool`）声明它们时，FastAPI 会自动进行类型转换和数据校验。

本指南将介绍如何定义、校验和记录查询参数。有关路径参数的详细信息，请参阅 [路径参数](./user-guide-path-parameters.md) 指南。

## 默认值

查询参数可以设置默认值。如果客户端没有为带默认值的参数提供值，FastAPI 将使用该默认值。

在这里，`skip` 和 `limit` 的默认值分别为 `0` 和 `10`：

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

如果你访问以下 URL：

`http://127.0.0.1:8000/items/`

`skip` 和 `limit` 的值将分别为 `0` 和 `10`。

如果你访问：

`http://127.0.0.1:8000/items/?skip=20`

那么 `skip` 的值将是 `20`，而 `limit` 仍为 `10`。

## 可选参数

你可以通过使用 `Union`（或在 Python 3.10+ 中使用 `|`）并将默认值设置为 `None` 来声明可选的查询参数。

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

在这种情况下，参数 `q` 是可选的。如果客户端提供了该参数，其值将被使用。如果没有提供，其值将为 `None`。

## 查询参数类型转换

FastAPI 会自动将 URL 中的字符串值转换为指定的 Python 类型。例如，一个 `bool` 类型的参数会将 `true`、`1`、`on`、`yes` 等值识别为 `True`，并将 `false`、`0`、`off`、`no` 等值识别为 `False`。

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

如果你访问 `http://127.0.0.1:8000/items/foo?short=1` 或 `http://127.0.0.1:8000/items/foo?short=true`，`short` 参数的值将为 `True`，响应中将省略详细描述。

## 多个路径和查询参数

你可以按任意顺序声明多个路径参数和查询参数。FastAPI 非常智能，能够根据路径字符串和函数签名区分它们。

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

FastAPI 知道 `user_id` 和 `item_id` 是路径参数，而 `q` 和 `short` 是查询参数。

## 必需查询参数

要将查询参数设为必需，只需在声明时不为其提供默认值即可。

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item
```

这里，函数需要一个必需的查询参数 `needy`。如果你尝试调用 `http://127.0.0.1:8000/items/foo-item` 但未添加 `needy` 参数，FastAPI 将返回一个明确的 HTTP 错误。

一个有效的请求应为：`http://127.0.0.1:8000/items/foo-item?needy=sooooneedy`。

## 混合使用必需、可选和带默认值的参数

你可以自由混合使用必需参数、带默认值的参数和可选参数。

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

- `item_id`：一个必需的路径参数。
- `needy`：一个必需的查询参数。
- `skip`：一个默认值为 `0` 的查询参数。
- `limit`：一个可选的查询参数。

## 使用 Query 进行额外校验

要对查询参数进行更高级的校验，你可以使用 `Query` 函数。它允许你设置最小/最大长度、正则表达式等约束。

首先，从 `fastapi` 导入 `Query`：

```python
from fastapi import FastAPI, Query
```

### 字符串校验

你可以为字符串参数设置 `min_length` 和 `max_length` 等约束。

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

这里，`q` 参数是可选的，但如果提供了该参数，其长度必须在 3 到 50 个字符之间。

### 正则表达式校验

你还可以强制要求参数符合某个正则表达式模式。

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

在此示例中，`q` 的值必须严格为 `fixedquery`。

### 其他校验

`Query` 支持许多其他校验参数，包括：

- `alias`：为 URL 中的参数使用不同的名称（例如，用 `item-query` 代替 `q`）。
- `title`：在文档中为参数提供一个人类可读的标题。
- `description`：参数的描述。
- `deprecated`：在 OpenAPI 文档中将参数标记为已弃用。
- `gt`、`ge`、`lt`、`le`：用于数值校验（大于、大于等于等）。

你还可以声明一个参数来接收一个值列表：

```python
from typing import List, Union

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Union[List[str], None] = Query(default=None)):
    query_items = {"q": q}
    return query_items
```

这样，你就可以发送类似 `http://127.0.0.1:8000/items/?q=foo&q=bar` 的请求，FastAPI 会正确地将 `q` 解析为 `["foo", "bar"]`。

## 总结

FastAPI 提供了一种强大而直观的方式来处理查询参数。你可以直接在函数签名中通过类型和默认值来定义它们。对于更复杂的场景，`Query` 提供了一套丰富的校验选项。

处理完路径参数和查询参数后，下一步通常是处理请求体中发送的数据。请在下一章关于 [请求体](./user-guide-request-body.md) 的内容中了解更多信息。