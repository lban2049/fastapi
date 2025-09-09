# 查询参数

当您声明的函数参数不属于路径时，它们会被自动解释为“查询”参数。

查询是 URL 中 `?` 之后、由 `&` 字符分隔的键值对集合。例如，在 URL `http://1227.0.0.1:8000/items/?skip=0&limit=10` 中，查询参数是 `skip` 和 `limit`。

由于它们是 URL 的一部分，因此它们“天然”是字符串。但是，当您使用 Python 类型（例如 `int`、`float`、`bool`）声明它们时，FastAPI 会自动进行转换和验证。

本指南介绍如何定义、验证和记录查询参数。有关路径参数的详细信息，请参阅[路径参数](./user-guide-path-parameters.md)指南。

## 默认值

查询参数可以有默认值。如果客户端没有为带默认值的参数提供值，FastAPI 将使用该默认值。

在这里，`skip` 和 `limit` 的默认值分别为 `0` 和 `10`：

```python title="tutorial001.py"
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]

```

如果您访问 URL：

`http://127.0.0.1:8000/items/`

`skip` 和 `limit` 的值将分别为 `0` 和 `10`。

如果您访问：

`http://127.0.0.1:8000/items/?skip=20`

那么 `skip` 的值将为 `20`，而 `limit` 的值将保持为 `10`。

## 可选参数

您可以通过使用 `Union`（或 Python 3.10+ 中的 `|`）并将默认值设置为 `None` 来声明可选的查询参数。

```python title="tutorial002.py"
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Union[str, None] = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}

```

在这种情况下，参数 `q` 是可选的。如果客户端提供该参数，则会使用其值。否则，其值将为 `None`。

## 查询参数类型转换

FastAPI 会自动将 URL 中的字符串值转换为指定的 Python 类型。例如，`bool` 类型的参数会将 `true`、`1`、`on`、`yes` 等值识别为 `True`，将 `false`、`0`、`off`、`no` 等值识别为 `False`。

```python title="tutorial003.py"
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

如果您访问 `http://127.0.0.1:8000/items/foo?short=1` 或 `http://127.0.0.1:8000/items/foo?short=true`，`short` 参数将为 `True`，响应中将省略长描述。

## 多个路径和查询参数

您可以按任意顺序声明多个路径参数和查询参数。FastAPI 非常智能，能够根据路径字符串和函数签名判断出哪个是哪个。

```python title="tutorial004.py"
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

FastAPI 知道 `user_id` 和 `item_id` 是路径的一部分，而 `q` 和 `short` 是查询参数。

## 必需的查询参数

要使查询参数成为必需项，只需在声明时不用默认值即可。

```python title="tutorial005.py"
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item

```

在这里，该函数需要一个必需的查询参数 `needy`。如果您尝试调用 `http://127.0.0.1:8000/items/foo-item` 而不添加 `needy` 参数，FastAPI 将返回一个明确的 HTTP 错误。

一个有效的请求是：`http://127.0.0.1:8000/items/foo-item?needy=sooooneedy`。

## 混合使用必需、可选和带默认值的参数

您可以自由地混合使用必需参数、带默认值的参数和可选参数。

```python title="tutorial006.py"
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

## 使用 `Query` 进行额外验证

对于查询参数的更高级验证，您可以使用 `Query` 函数。这允许您设置最小/最大长度、正则表达式等约束。

首先，从 `fastapi` 导入 `Query`：

```python
from fastapi import FastAPI, Query
```

### 字符串验证

您可以为字符串参数设置 `min_length` 和 `max_length` 等约束。

要使用 `Query`，您需要将参数的默认值替换为 `Query(...)`。

例如，要声明一个最小长度为 3、最大长度为 50 的可选 `q` 参数，您可以这样做：

```python title="tutorial003.py"
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

在这里，`q` 参数是可选的（因为 `default=None`），但如果提供了该参数，其长度必须在 3 到 50 个字符之间。

### 正则表达式验证

您还可以使用 `pattern` 参数来强制执行正则表达式模式。

```python title="tutorial004.py"
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

在此示例中，`q` 的值必须正好是 `fixedquery`。

### 列表 / 多个值

当您使用 `List` 类型提示定义查询参数时，它可以接受多个值。例如，`List[str]`。

```python
from typing import List, Union

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Union[List[str], None] = Query(default=None)):
    query_items = {"q": q}
    return query_items
```

这样，您就可以发送类似 `http://127.0.0.1:8000/items/?q=foo&q=bar` 的请求。FastAPI 会将 `q` 正确解析为一个 Python 列表：`["foo", "bar"]`。

### 其他验证

`Query` 支持许多其他验证参数。以下是一些最常见的参数：

| 参数 | 描述 |
|---|---|
| `alias` | URL 中参数的别名（例如，用 `item-query` 代替 `q`）。 |
| `title` | 文档中参数的可读标题。 |
| `description` | 参数的详细描述。 |
| `deprecated` | 在 OpenAPI 文档中将参数标记为已弃用。 |
| `gt`、`ge`、`lt`、`le` | 用于数字验证：大于、大于等于、小于、小于等于。 |
| `include_in_schema` | 设置为 `False` 可将参数从生成的 OpenAPI 模式中排除。 |


## 总结

FastAPI 提供了一种强大而直观的方式来处理查询参数。您可以在函数签名中直接使用类型和默认值来定义它们，FastAPI 将处理转换和基本验证。对于更复杂的场景，`Query` 提供了一套丰富的验证选项，可以对 API 的输入实施细粒度的约束。

处理完路径和查询参数后，下一步通常是处理请求体中发送的数据。在下一章[请求体](./user-guide-request-body.md)中了解更多相关信息。