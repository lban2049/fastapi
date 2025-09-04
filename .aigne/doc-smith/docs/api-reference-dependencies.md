# Dependencies

FastAPI's dependency injection system is a powerful mechanism for managing shared logic, database connections, authentication, and more. This reference details the classes used to declare dependencies.

For a step-by-step guide and more conceptual examples, please refer to the [User Guide on Dependency Injection](./user-guide-dependency-injection.md).

## Dependency Injection Flow

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
  shape: package
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

The `Depends` class is the primary tool for declaring a dependency in a *path operation function*. It signals to FastAPI that a parameter should be populated by the result of the specified dependency function (or callable).

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

### Parameters

| Name         | Type                           | Description                                                                                                                                                             |
|--------------|--------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `dependency` | `Optional[Callable[..., Any]]` | The dependency callable. This can be a function, a class, or any other callable. If not provided, the parameter's type annotation is used as the dependency.         |
| `use_cache`  | `bool`                         | If `True` (the default), the result of the dependency is cached for a single request. Subsequent dependencies requiring the same callable will receive the cached value. |

### Example Usage

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

In this example, `read_items` depends on `common_parameters`. FastAPI will call `common_parameters` with the request's query parameters and inject the returned dictionary into the `commons` parameter.

---

## `Security`

The `Security` class is a subclass of `Depends` and is used specifically for dependencies related to security schemes. It provides an additional `scopes` parameter that integrates with the OpenAPI documentation to specify required security scopes.

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

### Parameters

| Name         | Type                           | Description                                                                                                                                                             |
|--------------|--------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `dependency` | `Optional[Callable[..., Any]]` | The security dependency callable, typically an instance of a security scheme like `OAuth2PasswordBearer`.                                                               |
| `scopes`     | `Optional[Sequence[str]]`      | A list of security scope strings required to access this endpoint. These are used in the OpenAPI UI to request permissions.                                           |
| `use_cache`  | `bool`                         | If `True` (the default), the result of the dependency is cached for a single request.                                                                                   |

### Example Usage

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
In this example, the `read_users_me` endpoint requires the `me` scope. The `Security` function ensures that the dependency `get_current_user` is called and also documents the scope requirement in the API schema.

---

## Internal Model: `Dependant`

The `Dependant` class is an internal data structure that FastAPI uses to model a dependency and all of its sub-dependencies. Developers typically do not interact with this class directly, but it is documented here for reference and for those building tools on top of FastAPI.

FastAPI analyzes each *path operation function* and its parameters to build a `Dependant` object, which forms a graph of all dependencies.

### Key Attributes

| Attribute                 | Type                                | Description                                                                                             |
|---------------------------|-------------------------------------|---------------------------------------------------------------------------------------------------------|
| `path_params`             | `List[ModelField]`                  | A list of Pydantic model fields representing path parameters.                                           |
| `query_params`            | `List[ModelField]`                  | A list of Pydantic model fields representing query parameters.                                          |
| `header_params`           | `List[ModelField]`                  | A list of Pydantic model fields representing header parameters.                                         |
| `cookie_params`           | `List[ModelField]`                  | A list of Pydantic model fields representing cookie parameters.                                         |
| `body_params`             | `List[ModelField]`                  | A list of Pydantic model fields representing body parameters.                                           |
| `dependencies`            | `List["Dependant"]`                | A list of `Dependant` objects for sub-dependencies.                                                     |
| `security_requirements`   | `List[SecurityRequirement]`         | A list of security requirements for this dependency.                                                    |
| `name`                    | `Optional[str]`                     | The name of the parameter that this dependency will be injected into.                                   |
| `call`                    | `Optional[Callable[..., Any]]`      | The callable that will be executed to resolve the dependency.                                           |
| `use_cache`               | `bool`                              | Whether to cache the result of this dependency for the scope of a single request.                       |
| `path`                    | `Optional[str]`                     | The path of the operation this dependency belongs to.                                                   |
| `cache_key`               | `Tuple`                             | A tuple used as the key for caching the dependency's result. It's composed of the `call` and `security_scopes`. |

Next, you may want to learn about the utilities used for security. See the [Security Utilities API Reference](./api-reference-security.md) for more details.