# 依赖项

FastAPI 的依赖注入系统是一个强大的机制，可用于管理共享逻辑、数据库连接、身份验证等。本参考详细介绍了用于声明依赖项的类。

如需分步指南和更多概念性示例，请参阅[依赖注入用户指南](./user-guide-dependency-injection.md)。

## 依赖注入流程

下图展示了 FastAPI 如何为传入请求解析依赖项的宏观流程。

```d2
direction: down

"传入请求": {
  shape: circle
}

"FastAPI 路由器": {
  shape: rectangle
}

"依赖项解析引擎": {
  shape: package
  grid-columns: 1

  "依赖关系图": {
    shape: rectangle

    "子依赖项 A": {
      shape: class
    }
    "子依赖项 B": {
      shape: class
    }
    "路径操作函数": {
      shape: class
    }

    "子依赖项 A" -> "路径操作函数": "注入结果"
    "子依赖项 B" -> "路径操作函数": "注入结果"
  }
}

"生成的响应": {
  shape: circle
}

"传入请求" -> "FastAPI 路由器": "匹配路径操作"
"FastAPI 路由器" -> "依赖项解析引擎": "触发依赖项解析"
"依赖项解析引擎" -> "生成的响应": "执行函数并返回"

```

--- 

## `Depends`

`Depends` 类是在*路径操作函数*中声明依赖项的主要工具。它向 FastAPI 表明，一个参数应由指定依赖函数（或可调用对象）的结果填充。

```python
class Depends:
    def __init__(
        self, 
        dependency: Optional[Callable[..., Any]] = None, 
        *, 
        use_cache: bool = True
    ):
        self.dependency = dependency
        self.use_cache = use_cache
```

### 参数

| 名称 | 类型 | 描述 |
|---|---|---|
| `dependency` | `Optional[Callable[..., Any]]` | 依赖项可调用对象。它可以是函数、类或任何其他可调用对象。如果未提供，则使用参数的类型注解作为依赖项。 |
| `use_cache` | `bool` | 如果为 `True`（默认值），依赖项的结果将在单个请求中被缓存。后续需要相同可调用对象的依赖项将接收缓存的值。 |

### 使用示例

```python
from typing import Annotated

from fastapi import Depends, FastAPI

app = FastAPI()


async def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: Annotated[dict, Depends(common_parameters)]):
    return commons
```

在本示例中，`read_items` 依赖于 `common_parameters`。FastAPI 将使用请求的查询参数调用 `common_parameters`，并将返回的字典注入 `commons` 参数。

---

## `Security`

`Security` 类是 `Depends` 的子类，专门用于与安全方案相关的依赖项。它提供了一个额外的 `scopes` 参数，可与 OpenAPI 文档集成，用于指定所需的安全范围。

```python
class Security(Depends):
    def __init__(
        self, 
        dependency: Optional[Callable[..., Any]] = None, 
        *, 
        scopes: Optional[Sequence[str]] = None, 
        use_cache: bool = True
    ):
        super().__init__(dependency=dependency, use_cache=use_cache)
        self.scopes = scopes or []
```

### 参数

| 名称 | 类型 | 描述 |
|---|---|---|
| `dependency` | `Optional[Callable[..., Any]]` | 安全依赖项可调用对象，通常是安全方案（如 `OAuth2PasswordBearer`）的实例。 |
| `scopes` | `Optional[Sequence[str]]` | 访问此端点所需的安全范围字符串列表。这些范围用于在 OpenAPI UI 中请求权限。 |
| `use_cache` | `bool` | 如果为 `True`（默认值），依赖项的结果将在单个请求中被缓存。 |

### 使用示例

```python
from typing import Annotated

from fastapi import Depends, FastAPI, Security
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    # In a real app, you would decode the token and get the user
    return {"token": token}


@app.get("/users/me")
async def read_users_me(current_user: Annotated[dict, Security(get_current_user, scopes=["me"])])
    return current_user

```
在本示例中，`read_users_me` 端点需要 `me` 范围。`Security` 函数确保调用依赖项 `get_current_user`，同时在 API 架构中记录此范围要求。

---

## 内部模型：`Dependant`

`Dependant` 类是 FastAPI 用于为依赖项及其所有子依赖项建模的内部数据结构。开发者通常不直接与此类交互，但本文档在此提供相关信息，以供参考，也便于开发者在 FastAPI 基础上构建工具。

FastAPI 会分析每个*路径操作函数*及其参数，以构建一个 `Dependant` 对象，从而形成一个包含所有依赖项的图。

### 关键属性

| 属性 | 类型 | 描述 |
|---|---|---|
| `path_params` | `List[ModelField]` | 表示路径参数的 Pydantic 模型字段列表。 |
| `query_params` | `List[ModelField]` | 表示查询参数的 Pydantic 模型字段列表。 |
| `header_params` | `List[ModelField]` | 表示标头参数的 Pydantic 模型字段列表。 |
| `cookie_params` | `List[ModelField]` | 表示 Cookie 参数的 Pydantic 模型字段列表。 |
| `body_params` | `List[ModelField]` | 表示请求体参数的 Pydantic 模型字段列表。 |
| `dependencies` | `List["Dependant"]` | 用于子依赖项的 `Dependant` 对象列表。 |
| `security_requirements` | `List[SecurityRequirement]` | 此依赖项的安全要求列表。 |
| `name` | `Optional[str]` | 此依赖项将被注入的参数名称。 |
| `call` | `Optional[Callable[..., Any]]` | 为解析依赖项而将执行的可调用对象。 |
| `use_cache` | `bool` | 是否在单个请求范围内缓存此依赖项的结果。 |
| `path` | `Optional[str]` | 此依赖项所属操作的路径。 |
| `cache_key` | `Tuple` | 用于缓存依赖项结果的键的元组。它由 `call` 和 `security_scopes` 组成。 |

接下来，您可能希望了解用于安全的实用工具。更多详情请参阅[安全实用工具 API 参考](./api-reference-security.md)。