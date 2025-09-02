# 依赖项

FastAPI 的依赖注入系统是一个强大的机制，用于管理依赖项、共享逻辑以及处理身份验证和数据库连接等任务。本参考文档详细介绍了用于声明依赖项的类。

有关如何使用依赖注入的更详尽的教程式指南，请参阅[用户指南 - 依赖注入](./user-guide-dependency-injection.md)。

## `Depends` 类

`Depends` 类是在*路径操作函数*中声明依赖项的主要工具。当 FastAPI 看到一个带有 `Depends` 默认值的参数时，它会解析并注入该依赖项的结果。

```python
class Depends:
    def __init__(
        self, dependency: Optional[Callable[..., Any]] = None, *, use_cache: bool = True
    ):
        self.dependency = dependency
        self.use_cache = use_cache
```

### 参数

| Parameter | Type | Description |
|---|---|---|
| `dependency` | `Optional[Callable[..., Any]]` | 依赖项的可调用对象。这可以是一个函数、一个类或任何其他可调用对象。如果未提供，则从类型注解中推断。 |
| `use_cache` | `bool` | 如果为 `True`（默认值），依赖项的结果将在单个请求的作用域内被缓存。在同一次请求中，后续对同一依赖项（使用相同参数）的调用将返回缓存的值，而不会重新执行可调用对象。设置为 `False` 可在同一次请求中每次需要时都重新运行依赖项。 |

### 示例

以下是一个提供通用查询参数的依赖项示例。该依赖项随后被用于*路径操作函数*中。

```python
from typing import Annotated

from fastapi import Depends, FastAPI

app = FastAPI()


def def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: Annotated[dict, Depends(common_parameters)]):
    return commons


@app.get("/users/")
async def read_users(commons: Annotated[dict, Depends(common_parameters)]):
    return commons
```

## `Security` 类

`Security` 类是 `Depends` 的子类，专门用于处理与安全相关的依赖项。它通过添加一个 `scopes` 参数来扩展 `Depends`，这对于实现授权逻辑（例如使用 OAuth2）至关重要。

```python
class Security(Depends):
    def __init__(
        self,
        dependency: Optional[Callable[..., Any]] = None,
        *,
        scopes: Optional[Sequence[str]] = None,
        use_cache: bool = True,
    ):
        super().__init__(dependency=dependency, use_cache=use_cache)
        self.scopes = scopes or []
```

### 参数

| Parameter | Type | Description |
|---|---|---|
| `dependency` | `Optional[Callable[..., Any]]` | 安全依赖项的可调用对象，通常是安全方案类的实例（例如 `OAuth2PasswordBearer`）。 |
| `scopes` | `Optional[Sequence[str]]` | 一个表示访问此端点所需的安全作用域（例如权限）的字符串序列。 |
| `use_cache` | `bool` | 缓存行为，其功能与在 `Depends` 中相同。 |

### 示例

此示例演示了如何使用 `Security` 来保护一个端点，要求提供有效的令牌和特定的作用域。

```python
from typing import Annotated

from fastapi import Depends, FastAPI, Security
from fastapi.security import OAuth2PasswordBearer, SecurityScopes

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


async def get_current_user(security_scopes: SecurityScopes, token: Annotated[str, Depends(oauth2_scheme)]):
    # 在实际应用中，您需要解码令牌、验证作用域，
    # 并从数据库中获取用户。
    print(f"Token: {token}, Required Scopes: {security_scopes.scopes}")
    return {"username": "admin", "scopes": ["users", "me"]}


@app.get("/users/me")
async def read_users_me(current_user: Annotated[dict, Security(get_current_user, scopes=["me"])]):
    return current_user
```

## 内部依赖模型

在内部，FastAPI 会分析每个*路径操作函数*及其依赖项，以构建一个由 `Dependant` 类表示的树状结构。该对象包含了关于给定可调用对象的所有参数（路径、查询、请求体等）和子依赖项的信息。

这个 `Dependant` 树随后在请求处理过程中被用来按正确顺序解析所有依赖项、处理缓存并将结果传递给您的函数。

```d2
direction: right

"路径操作": {
    "参数 1 (路径)"
    "参数 2 (查询)"
    "参数 3 (依赖)"
}

"FastAPI 分析器": {
    shape: cloud
    "get_dependant()": "分析路径操作签名"
}

"Dependant 对象": {
    shape: package
    "path_params: [参数 1]"
    "query_params: [参数 2]"
    "dependencies: [参数 3 的子 Dependant]"
    "call: 路径操作可调用对象"
    "use_cache: true"
}

"路径操作" -> "FastAPI 分析器": "启动时"
"FastAPI 分析器" -> "Dependant 对象": "构建依赖树"

```

此图显示了 FastAPI 分析器如何处理路径操作以构建一个 `Dependant` 对象，该对象是依赖注入系统的核心模型。

---

有关可用的安全方案和实用工具的更多信息，请参阅[安全实用工具](./api-reference-security.md)参考。