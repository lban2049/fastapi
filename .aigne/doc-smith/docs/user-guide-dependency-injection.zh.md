# 依赖注入

FastAPI 包含一个功能强大且直观的依赖注入系统。它旨在简化使用，可帮助你管理依赖、共享逻辑、处理身份验证、管理数据库连接等，同时确保代码保持整洁且结构良好。

该系统允许你声明*路径操作函数*所需的依赖项，FastAPI 会负责提供这些依赖项。

## 一个简单的依赖

我们从一个基本示例开始。假设你有多个端点共享通用查询参数，例如 `q`、`skip` 和 `limit`。

你无需在每个函数签名中重复这些参数，而可以在一个专用函数中将它们统一定义。

### 创建一个依赖

依赖只是一个函数（或可调用对象），它可以接受与*路径操作函数*相同的参数。

在这里，我们定义一个 `common_parameters` 函数来处理共享的查询参数：

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

### 工作原理

1.  **定义依赖**：`common_parameters` 函数就是我们的依赖。它接受标准查询参数 `q`、`skip` 和 `limit`。
2.  **“依赖”它**：在我们的*路径操作函数*（`read_items` 和 `read_users`）中，我们添加一个参数 `commons`。
3.  **使用 `Depends`**：我们将默认值 `Depends(common_parameters)` 赋给 `commons` 参数。这会告诉 FastAPI，`commons` 不是一个常规参数，而是一个需要被解析的依赖。

当请求访问 `/items/` 或 `/users/` 时，FastAPI 将会：
*   使用请求中的查询参数调用 `common_parameters` 函数。
*   获取返回值（一个字典）。
*   将该字典作为 `commons` 参数传递给 `read_items` 或 `read_users`。

这使你可以在多个端点之间重用相同的参数逻辑，而无需重复代码。

## 作为依赖的类

虽然函数非常适合简单的依赖，但对于更复杂的逻辑，类能提供更好的组织性。你可以将类用作依赖，FastAPI 会处理其实例化。

让我们重构前面的示例，改用类来实现。

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

在这里，FastAPI 看到 `Depends(CommonQueryParams)` 后，会明白它需要：
1.  检查 `CommonQueryParams` 类的 `__init__` 方法。
2.  从请求中解析 `__init__` 的参数（即查询参数 `q`、`skip` 和 `limit`）。
3.  使用这些参数创建一个 `CommonQueryParams` 的实例。
4.  将该实例作为 `commons` 参数传递给 `read_items`。

这种方法结构更清晰，并且与面向对象的原则高度契合。

### 更简洁的语法

FastAPI 提供了一个方便的快捷方式。如果你为依赖使用了类型提示，则无需再次将可调用对象传递给 `Depends`。

你可以直接使用 `Depends()`：

```python
@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends()):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

FastAPI 非常智能，它能看到 `CommonQueryParams` 这个类型提示，并理解这就是你想要注入的依赖。这是使用基于类的依赖时最常用和推荐的方式。

## 依赖注入流程

依赖注入系统遵循一个清晰且可预测的流程来解析依赖并将其提供给你的路径操作。

```d2
direction: down

request: "传入的请求"
path_op: "路径操作函数"

subgraph "FastAPI 引擎" {
  direction: right
  
  dep_marker: "检测 `Depends()`"
  inspector: "检查依赖签名（例如 `__init__`）"
  param_solver: "从请求中解析参数（查询、路径等）"
  dep_callable: "调用依赖（例如创建类实例）"
  
  dep_marker -> inspector -> param_solver -> dep_callable
}

request -> path_op
path_op -> dep_marker
param_solver -> request: "获取参数"

dep_result: "依赖结果"
dep_callable -> dep_result
dep_result -> path_op: "注入结果"

```

## 缓存

依赖注入系统包含一个缓存。对于单个请求，如果代码的多个部分依赖于同一个依赖（具有相同的参数），那么该依赖只会被调用一次。其结果会被缓存，并在同一请求内的所有后续需要中重用。

这由 `Depends` 中的 `use_cache` 参数控制，其默认值为 `True`。

```python
class Depends:
    def __init__(
        self, dependency: Optional[Callable[..., Any]] = None, *, use_cache: bool = True
    ):
        self.dependency = dependency
        self.use_cache = use_cache
```

这对于建立数据库连接或执行昂贵计算的依赖项尤其有用，可以确保它们不会在每次请求中不必要地多次运行。

通过掌握依赖注入，你可以构建复杂、可维护且稳健的 API。这里学到的原则为更高级的主题（如身份验证和授权）奠定了基础。

下一步，请参阅 [安全](./advanced-security.md) 指南，了解如何应用这些概念来保护你的应用程序。