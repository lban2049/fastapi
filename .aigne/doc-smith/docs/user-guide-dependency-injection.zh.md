# 依赖注入

FastAPI 包含一个非常直观的依赖注入 (DI) 系统。通过该系统，你的代码可以声明其运行所需的依赖项，例如数据库会话、身份验证凭据或共享参数。FastAPI 会负责为你的代码提供这些依赖项。

这对于以下方面非常有用：
- 共享逻辑和代码。
- 共享数据库连接。
- 强制执行安全性、身份验证和角色要求。
- 以及许多其他情况。

让我们从一个简单的示例开始。

## 创建一个依赖项，或称为“可依赖项”

假设你有多个端点共享相同的查询参数，例如，用于分页的（`skip`、`limit`）和一个可选的查询字符串（`q`）。

你可以在一个共享函数中一次性定义这些参数，而无需在每个路径操作函数中重复定义。这个函数就是我们的依赖项。

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

具体流程如下：
1.  我们创建了一个函数 `common_parameters`，它接受与路径操作函数（`q`、`skip`、`limit`）相同的参数。
2.  该函数返回一个包含这些值的字典。
3.  在我们的路径操作函数 `read_items` 和 `read_users` 中，我们声明了一个参数 `commons`。
4.  我们为该参数提供了一个默认值：`Depends(common_parameters)`。`Depends` 是一个特殊的标记，它告诉 FastAPI 该参数依赖于另一个函数。

然后，FastAPI 将会：
- 调用依赖函数（`common_parameters`），并从请求中传入所需参数。
- 获取该函数的返回值。
- 将该返回值赋给路径操作函数中的参数（`commons`）。

现在，`/items/` 和 `/users/` 这两个端点共享同一组查询参数，且这些参数在同一个地方定义。

## 将类作为依赖项

虽然函数非常适合简单的依赖项，但你也可以使用类。这有助于组织代码，尤其是在依赖项变得更加复杂时。

让我们使用类来重构前面的示例。

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

当你使用像 `Depends(CommonQueryParams)` 这样的类来声明依赖项时，FastAPI 会明白它需要创建该类的一个实例。它会检查 `__init__` 方法，并从请求中提供必要的参数，就像处理函数一样。

这样做的好处是，你的编辑器会提供更好的自动补全和类型检查功能，因为它知道 `commons` 是 `CommonQueryParams` 的一个实例。

## 快捷方式：`Depends()`

你可能已经注意到，我们在类型提示和 `Depends` 内部重复了 `CommonQueryParams`。FastAPI 为这种常见模式提供了一个方便的快捷方式。

如果你没有向 `Depends()` 传递任何参数，它将使用参数的类型注解来确定依赖项。

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

你也可以不使用类型提示，直接写成 `commons = Depends(CommonQueryParams)`，但不推荐这样做，因为你将失去类型检查和编辑器自动补全带来的好处。

## 子依赖项

依赖项可以声明自己的依赖项。FastAPI 的依赖注入系统会自动解析这个依赖链。

下面是一个依赖项依赖于另一个依赖项的示例：

```python
from typing import Union, Annotated

from fastapi import Cookie, Depends, FastAPI

app = FastAPI()


def query_extractor(q: Union[str, None] = None):
    # 此依赖项获取原始查询参数 'q'
    return q


def query_or_cookie_extractor(
    q: Annotated[str, Depends(query_extractor)],
    last_query: Union[str, None] = Cookie(default=None),
):
    # 此依赖项依赖于 query_extractor。
    # 如果查询参数 'q' 存在，则返回 'q'，否则返回 Cookie 值。
    if not q:
        return last_query
    return q


@app.get("/items/")
async def read_query(
    query_or_default: Annotated[str, Depends(query_or_cookie_extractor)],
):
    # 路径操作只需要依赖于最终的依赖项。
    return {"q_or_cookie": query_or_default}

```

在这个流程中：
1. 路径操作 `read_query` 依赖于 `query_or_cookie_extractor`。
2. `query_or_cookie_extractor` 又依赖于 `query_extractor`。
3. FastAPI 首先解析 `query_extractor`，获取 `q` 的值，并将其传递给 `query_or_cookie_extractor`。
4. 然后 `query_or_cookie_extractor` 的结果被传递给路径操作。

这会创建一个依赖关系图，FastAPI 会为每个请求遍历并解析该图。

## 带有 `yield` 的依赖项

对于需要在发送响应后执行清理操作（如关闭数据库连接）的依赖项，你可以使用带有 `yield` 的生成器。

`yield` 语句之前的代码在生成响应之前执行。`yield` 产生的值被注入到路径操作中。`yield` 之后的代码在响应发送后执行。

这种模式非常适合管理资源。

```python
from fastapi import Depends, FastAPI

app = FastAPI()

# 用于演示的虚拟“数据库”连接类
class DummyDB:
    def __init__(self):
        self.connected = True
        print("Connecting to DB")

    def close(self):
        self.connected = False
        print("Closing DB connection")

def get_db_session():
    db = DummyDB()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
async def read_items(db: DummyDB = Depends(get_db_session)):
    # 'db' 对象是 get_db_session yield 产生的值
    return {"message": "Items read", "db_connected": db.connected}

```

当向 `/items/` 发出请求时，FastAPI 将：
1. 调用 `get_db_session()`。
2. 执行到 `yield db` 为止的代码，创建一个 `DummyDB` 实例。
3. 将 `db` 实例注入到 `read_items` 中。
4. 执行 `read_items` 并生成响应。
5. 响应发送后，执行 `finally` 块中的代码，调用 `db.close()`。

这可以确保即使在请求期间发生错误，资源也总能被释放。

## 依赖项缓存

默认情况下，FastAPI 会在单个请求的作用域内缓存依赖项的返回值。如果你的应用程序的多个部分（例如，一个路径操作和一个子依赖项）都依赖于同一个依赖项，那么该依赖项将只会被执行一次，其结果将被重用。

例如，如果 `dependency_b` 依赖于 `dependency_a`，并且路径操作同时依赖于 `dependency_a` 和 `dependency_b`，那么 `dependency_a` 将只运行一次。

要禁用此行为并强制依赖项在每次被调用时都重新执行，你可以设置 `use_cache=False`。

```python
from fastapi import Depends, FastAPI

app = FastAPI()


async def get_value(use_cache: bool = True):
    # 一个用于演示缓存的虚拟函数
    return {"value": "cached" if use_cache else "new"}

# 缓存的依赖项（默认行为）
@app.get("/cached/")
async def read_cached(val: dict = Depends(get_value)):
    # 'get_value' 每个请求只调用一次
    return val

# 非缓存的依赖项
@app.get("/no-cache/")
async def read_no_cache(val: dict = Depends(get_value, use_cache=False)):
    # 'get_value' 在每次作为依赖项出现时都会被调用
    return val
```

缓存通常是可取的，因为它可以防止冗余计算，例如在同一次请求中重复从数据库中获取用户数据。

## 总结

FastAPI 的依赖注入提供了一种简单而强大的方式来管理依赖项和重用代码。你可以将依赖项定义为函数或类，并使用 `Depends` 将它们注入到你的路径操作中。该系统支持子依赖、使用 `yield` 进行资源管理以及缓存，为安全性、数据库连接管理等许多功能奠定了基础。

要了解如何将其应用于安全性，请继续阅读[安全性](./advanced-security.md)部分。