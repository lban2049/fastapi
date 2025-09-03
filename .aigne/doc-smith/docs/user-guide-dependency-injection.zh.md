# 依赖注入

FastAPI 包含一个强大而直观的依赖注入（DI）系统。通过该系统，你的代码可以声明其运行所需的内容，如数据库会话、身份验证凭据或共享参数。然后，FastAPI 会负责为你的代码提供这些依赖项。

这对于以下方面非常有用：
- 共享逻辑和代码。
- 共享数据库连接。
- 强制执行安全、身份验证和角色要求。
- 以及许多其他情况。

让我们从一个简单的示例开始。

## 创建依赖项或“可依赖项”

假设你有多个端点共享相同的查询参数，例如用于分页的（`skip`、`limit`）和一个可选的查询字符串（`q`）。

你可以在一个共享函数中一次性定义这些参数，而不是在每个路径操作函数中重复它们。这个函数就是我们的依赖项。

```python
from typing import Union

from fastapi import Depends, FastAPI

app = FastAPI()


async def common_parameters(
    q: Union[str, None] = None, skip: int = 0, limit: int = 100
):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons


@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return commons
```

具体过程如下：
1.  我们创建了一个函数 `common_parameters`，它接受与路径操作函数相同的参数（`q`、`skip`、`limit`）。
2.  这个函数返回一个包含这些值的字典。
3.  在我们的路径操作函数 `read_items` 和 `read_users` 中，我们声明了一个参数 `commons`。
4.  我们为这个参数提供了一个默认值：`Depends(common_parameters)`。`Depends` 是一个特殊的标记，它告诉 FastAPI 这个参数依赖于另一个函数。

然后 FastAPI 将会：
- 使用请求中所需的参数调用依赖函数（`common_parameters`）。
- 获取该函数的返回值。
- 将该返回值赋给路径操作函数中的参数（`commons`）。

现在，`/items/` 和 `/users/` 端点共享同一组查询参数，且这些参数在同一个地方定义。

## 将类作为依赖项

虽然函数非常适合简单的依赖项，但你也可以使用类。这有助于组织代码，尤其是在依赖项变得更加复杂时。

让我们重构前面的示例来使用类。

```python
from typing import Union

from fastapi import Depends, FastAPI

app = FastAPI()


fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


class CommonQueryParams:
    def __init__(self, q: Union[str, None] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit


@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends(CommonQueryParams)):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

当你使用像 `Depends(CommonQueryParams)` 这样的类声明依赖项时，FastAPI 会理解它需要创建该类的一个实例。它会检查 `__init__` 方法并从请求中提供必要的参数，就像处理函数一样。

这样做的好处是，你的编辑器会提供更好的自动补全和类型检查，因为它知道 `commons` 是 `CommonQueryParams` 的一个实例。

## 快捷方式：`Depends()`

你可能已经注意到，我们在类型提示和 `Depends` 内部重复了 `CommonQueryParams`。对于这种常见模式，FastAPI 提供了一个方便的快捷方式。

如果你不向 `Depends()` 传递任何内容，它将使用参数的类型注解来确定依赖项。

```python
from typing import Union

from fastapi import Depends, FastAPI

app = FastAPI()


fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


class CommonQueryParams:
    def __init__(self, q: Union[str, None] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit


@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends()):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

这段代码与前面的示例等效，但更加简洁。`commons: CommonQueryParams = Depends()` 语法是声明基于类的依赖项最常用和推荐的方式。

### 不使用类型提示

你也可以写成 `commons = Depends(CommonQueryParams)` 而不加类型提示，但不推荐这样做。你会失去类型检查和编辑器自动补全带来的好处。

## 工作原理

当请求到达时，依赖注入系统遵循一个清晰的流程。

```d2
direction: down

"客户端": {
  shape: person
}

"FastAPI 应用": {
  shape: package
  grid-columns: 1

  "/items/ 端点": {
    shape: rectangle
    "read_items(commons: CommonQueryParams = Depends())"
  }

  "依赖注入器": {
    shape: diamond
  }

  "CommonQueryParams": {
    label: "CommonQueryParams 类"
    shape: class
    "__init__(self, q, skip, limit)"
  }
}

"HTTP 响应": {
  shape: document
}

"客户端" -> "FastAPI 应用"."/items/ 端点": "1. GET /items/?q=foo"

"FastAPI 应用"."/items/ 端点" -> "FastAPI 应用"."依赖注入器": "2. 在 'commons' 参数上看到 Depends()"

"FastAPI 应用"."依赖注入器" -> "FastAPI 应用"."CommonQueryParams": "3. 从类型提示解析依赖项\n- 从请求中提取 q、skip、limit\n- 创建实例：CommonQueryParams(q='foo', skip=0, limit=100)"

"FastAPI 应用"."CommonQueryParams" -> "FastAPI 应用"."/items/ 端点": "4. 将实例注入 'commons' 参数"

"FastAPI 应用"."/items/ 端点" -> "HTTP 响应": "5. 路径操作使用依赖项的结果运行"

"HTTP 响应" -> "客户端": "6. 发送响应"
```

## 总结

FastAPI 的依赖注入提供了一种简单而强大的方式来管理依赖项和重用代码。你可以将依赖项定义为函数或类，并使用 `Depends` 将它们注入到你的路径操作中。该系统是许多高级功能（包括安全和数据库连接管理）的基础。

要深入了解，请浏览[高级主题](./advanced.md)以获取更复杂的用例和模式。
