# Dependencies

FastAPI's dependency injection system is a powerful mechanism for managing shared logic, database connections, authentication, and more. This document provides a technical reference for the classes used to declare and manage dependencies.

For a step-by-step guide and more conceptual examples, please see the [Dependency Injection User Guide](./user-guide-dependency-injection.md).

## Dependency Resolution Flow

The following diagram illustrates the high-level process of how FastAPI resolves dependencies for an incoming request.

```d2
direction: down

Incoming-Request: {
  label: "Incoming Request"
  shape: circle
}

FastAPI-Router: {
  label: "FastAPI Router"
  shape: rectangle
}

Dependency-Resolution-Engine: {
  label: "Dependency Resolution Engine"
  shape: rectangle
  grid-columns: 1

  Dependant-Graph: {
    label: "Dependant Graph"
    shape: rectangle

    Sub-Dependency-A: {
      label: "Sub-Dependency A"
      shape: class
    }
    Sub-Dependency-B: {
      label: "Sub-Dependency B"
      shape: class
    }
    Path-Operation-Function: {
      label: "Path Operation Function"
      shape: class
    }

    Sub-Dependency-A -> Path-Operation-Function: "Result Injected"
    Sub-Dependency-B -> Path-Operation-Function: "Result Injected"
  }
}

Generated-Response: {
  label: "Generated Response"
  shape: circle
}

Incoming-Request -> FastAPI-Router: "Matches path operation"
FastAPI-Router -> Dependency-Resolution-Engine: "Triggers dependency resolution"
Dependency-Resolution-Engine -> Generated-Response: "Executes function & returns"
```

---

## `Depends`

The `Depends` class is the primary tool for declaring a dependency in a *path operation function*. It signals to FastAPI that a parameter should be populated by the result of the specified dependency callable.

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

### Parameters

| Name         | Type                           | Description                                                                                                                                                                                  |
|--------------|--------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `dependency` | `Optional[Callable[..., Any]]` | The dependency callable (e.g., a function or class). If `None`, the parameter's type annotation is used as the dependency.                                                                 |
| `use_cache`  | `bool`                         | If `True` (default), the dependency's result is cached for the scope of a single request. Subsequent dependencies requiring the same callable (with the same scopes) will receive the cached value. |

### Example Usage

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

In this example, `read_items` depends on `common_parameters`. FastAPI will call `common_parameters` with the request's query parameters and inject the returned dictionary into the `commons` parameter.

---

## `Security`

The `Security` class is a subclass of `Depends` used specifically for dependencies related to security schemes. It adds a `scopes` parameter to integrate with OpenAPI documentation, specifying required security scopes for an endpoint.

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

### Parameters

| Name         | Type                           | Description                                                                                                       |
|--------------|--------------------------------|-------------------------------------------------------------------------------------------------------------------| 
| `dependency` | `Optional[Callable[..., Any]]` | The security dependency callable, typically an instance of a security scheme like `OAuth2PasswordBearer`.         |
| `scopes`     | `Optional[Sequence[str]]`      | A list of security scope strings required for this endpoint. These are used in the OpenAPI schema.              |
| `use_cache`  | `bool`                         | If `True` (the default), the result is cached for the duration of the request.                                    |

### Example Usage

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

## Internal Models

The following data classes are used internally by FastAPI to build and manage the dependency graph. While you typically don't interact with them directly, understanding them can be useful for building tools or advanced customizations on top of FastAPI.

### `Dependant`

The `Dependant` class models a single dependency and all of its sub-dependencies, parameters, and security requirements. FastAPI analyzes each *path operation function* and its parameters to build a `Dependant` object, which forms a node in the dependency graph.

#### Key Attributes

| Attribute                 | Type                                | Description                                                                                             |
|---------------------------|-------------------------------------|---------------------------------------------------------------------------------------------------------|
| `path_params`             | `List[ModelField]`                  | List of Pydantic model fields for path parameters.                                                      |
| `query_params`            | `List[ModelField]`                  | List of Pydantic model fields for query parameters.                                                     |
| `header_params`           | `List[ModelField]`                  | List of Pydantic model fields for header parameters.                                                    |
| `cookie_params`           | `List[ModelField]`                  | List of Pydantic model fields for cookie parameters.                                                    |
| `body_params`             | `List[ModelField]`                  | List of Pydantic model fields for body parameters.                                                      |
| `dependencies`            | `List["Dependant"]`                | List of `Dependant` objects for sub-dependencies.                                                       |
| `security_requirements`   | `List[SecurityRequirement]`         | List of security requirements for this dependency.                                                      |
| `name`                    | `Optional[str]`                     | The name of the parameter this dependency is injected into.                                             |
| `call`                    | `Optional[Callable[..., Any]]`      | The callable that is executed to resolve the dependency.                                                |
| `use_cache`               | `bool`                              | Whether to cache the result of this dependency.                                                         |
| `path`                    | `Optional[str]`                     | The path of the operation this dependency belongs to.                                                   |
| `cache_key`               | `Tuple`                             | A unique key for caching, composed of the `call` and sorted `security_scopes`.                          |

### `SecurityRequirement`

This data class represents a specific security scheme and the scopes required for it.

```python class SecurityRequirement icon=logos:python
from dataclasses import dataclass
from typing import Optional, Sequence
from fastapi.security.base import SecurityBase

@dataclass
class SecurityRequirement:
    security_scheme: SecurityBase
    scopes: Optional[Sequence[str]] = None
```

#### Attributes

| Attribute         | Type                         | Description                                                              |
|-------------------|------------------------------|--------------------------------------------------------------------------|
| `security_scheme` | `SecurityBase`               | The security scheme instance (e.g., an `OAuth2` instance).               |
| `scopes`          | `Optional[Sequence[str]]`    | The list of required scopes for this scheme.                             |

---

This reference covers the core components of FastAPI's dependency injection system. For details on the security schemes that are often used as dependencies, see the [Security Utilities API Reference](./api-reference-security.md).