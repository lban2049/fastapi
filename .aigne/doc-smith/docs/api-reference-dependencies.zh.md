# 依赖项

FastAPI 的依赖注入系统是一种强大的机制，用于管理共享逻辑、数据库连接、身份验证等。本文档为用于声明和管理依赖项的类提供了技术参考。

有关分步指南和更多概念性示例，请参阅 [依赖注入用户指南](./user-guide-dependency-injection.md)。

## 依赖项解析流程

下图说明了 FastAPI 为传入请求解析依赖项的概要流程。

```d2
direction: down

Incoming-Request: {
  label: "传入请求"
  shape: circle
}

FastAPI-Router: {
  label: "FastAPI 路由器"
  shape: rectangle
}

Dependency-Resolution-Engine: {
  label: "依赖项解析引擎"
  shape: rectangle
  grid-columns: 1

  Dependant-Graph: {
    label: "依赖关系图"
    shape: rectangle

    Sub-Dependency-A: {
      label: "子依赖项 A"
      shape: class
    }
    Sub-Dependency-B: {
      label: "子依赖项 B"
      shape: class
    }
    Path-Operation-Function: {
      label: "路径操作函数"
      shape: class
    }

    Sub-Dependency-A -> Path-Operation-Function: "注入结果"
    Sub-Dependency-B -> Path-Operation-Function: "注入结果"
  }
}

Generated-Response: {
  label: "生成的响应"
  shape: circle
}

Incoming-Request -> FastAPI-Router: "匹配路径操作"
FastAPI-Router -> Dependency-Resolution-Engine: "触发依赖项解析"
Dependency-Resolution-Engine -> Generated-Response: "执行函数并返回"
```

---

## `Depends`

`Depends` 类是在*路径操作函数*中声明依赖项的主要工具。它向 FastAPI 表明，应使用指定依赖项可调用对象的结果来填充参数。

```python class Depends icon=logos:python
class Depends:
    def __init__(
        self,
        dependency: Optional[Callable[..., Any]] = None,
        *,
        use_cache: bool = True,
    ):
        self.dependency = dependency
        self.use_cache = use_cache
```

### 参数

| Name         | Type                           | Description                                                                                                                                                                                  |
|--------------|--------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `dependency` | `Optional[Callable[..., Any]]` | 依赖项可调用对象（例如，函数或类）。如果为 `None`，则使用参数的类型注解作为依赖项。                                                                 |
| `use_cache`  | `bool`                         | 如果为 `True`（默认值），依赖项的结果将在单个请求的作用域内被缓存。后续需要相同可调用对象（具有相同作用域）的依赖项将接收缓存的值。 |

### 示例用法

```python Example icon=logos:python
from typing import Annotated

from fastapi import Depends, FastAPI

app = FastAPI()


async def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: Annotated[dict, Depends(common_parameters)]):
    return commons
```

在此示例中，`read_items` 依赖于 `common_parameters`。FastAPI 将使用请求的查询参数调用 `common_parameters`，并将返回的字典注入到 `commons` 参数中。

---

## `Security`

`Security` 类是 `Depends` 的一个子类，专门用于处理与安全方案相关的依赖项。它增加了一个 `scopes` 参数，用于与 OpenAPI 文档集成，从而为端点指定所需的安全作用域。

```python class Security icon=logos:python
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

| Name         | Type                           | Description                                                                                                       |
|--------------|--------------------------------|-------------------------------------------------------------------------------------------------------------------|
| `dependency` | `Optional[Callable[..., Any]]` | 安全依赖项可调用对象，通常是安全方案的实例，例如 `OAuth2PasswordBearer`。         |
| `scopes`     | `Optional[Sequence[str]]`      | 此端点所需的安全作用域字符串列表。这些作用域将用于 OpenAPI 架构中。              |
| `use_cache`  | `bool`                         | 如果为 `True`（默认值），结果将在请求期间被缓存。                                    |

### 示例用法

```python Example icon=logos:python
from typing import Annotated

from fastapi import Depends, FastAPI, Security
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    # In a real app, you would decode the token and get the user
    return {"token": token, "scopes": ["me", "items"]}


@app.get("/users/me")
async def read_users_me(
    current_user: Annotated[dict, Security(get_current_user, scopes=["me"])],
):
    return current_user
```

---

## 内部模型

以下数据类由 FastAPI 内部用于构建和管理依赖关系图。虽然通常您不会直接与它们交互，但了解这些类对于在 FastAPI 基础上构建工具或进行高级定制会很有帮助。

### `Dependant`

`Dependant` 类为单个依赖项及其所有子依赖项、参数和安全要求进行建模。FastAPI 会分析每个*路径操作函数*及其参数来构建一个 `Dependant` 对象，该对象构成依赖关系图中的一个节点。

#### 关键属性

| Attribute                 | Type                                | Description                                                                                             |
|---------------------------|-------------------------------------|---------------------------------------------------------------------------------------------------------|
| `path_params`             | `List[ModelField]`                  | 路径参数的 Pydantic 模型字段列表。                                                      |
| `query_params`            | `List[ModelField]`                  | 查询参数的 Pydantic 模型字段列表。                                                     |
| `header_params`           | `List[ModelField]`                  | 标头参数的 Pydantic 模型字段列表。                                                    |
| `cookie_params`           | `List[ModelField]`                  | Cookie 参数的 Pydantic 模型字段列表。                                                    |
| `body_params`             | `List[ModelField]`                  | 请求体参数的 Pydantic 模型字段列表。                                                      |
| `dependencies`            | `List["Dependant"]`                | 子依赖项的 `Dependant` 对象列表。                                                       |
| `security_requirements`   | `List[SecurityRequirement]`         | 此依赖项的安全要求列表。                                                      |
| `name`                    | `Optional[str]`                     | 此依赖项注入的参数名称。                                             |
| `call`                    | `Optional[Callable[..., Any]]`      | 为解析依赖项而执行的可调用对象。                                                |
| `use_cache`               | `bool`                              | 是否缓存此依赖项的结果。                                                         |
| `path`                    | `Optional[str]`                     | 此依赖项所属操作的路径。                                                   |
| `cache_key`               | `Tuple`                             | 用于缓存的唯一键，由 `call` 和排序后的 `security_scopes` 组成。                          |

### `SecurityRequirement`

该数据类表示一个特定的安全方案及其所需的作用域。

```python class SecurityRequirement icon=logos:python
from dataclasses import dataclass
from typing import Optional, Sequence
from fastapi.security.base import SecurityBase

@dataclass
class SecurityRequirement:
    security_scheme: SecurityBase
    scopes: Optional[Sequence[str]] = None
```

#### 属性

| Attribute         | Type                         | Description                                                              |
|-------------------|------------------------------|--------------------------------------------------------------------------|
| `security_scheme` | `SecurityBase`               | 安全方案实例（例如 `OAuth2` 实例）。               |
| `scopes`          | `Optional[Sequence[str]]`    | 此方案所需的作用域列表。                             |

---

本参考涵盖了 FastAPI 依赖注入系统的核心组件。有关常用作依赖项的安全方案的详细信息，请参阅 [安全工具 API 参考](./api-reference-security.md)。