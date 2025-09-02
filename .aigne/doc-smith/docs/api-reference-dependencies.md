# Dependencies

FastAPI's dependency injection system is a powerful mechanism for managing dependencies, sharing logic, and handling tasks like authentication and database connections. This reference details the classes used to declare dependencies.

For a more tutorial-style guide on how to use dependency injection, please refer to the [User Guide - Dependency Injection](./user-guide-dependency-injection.md).

## `Depends` Class

The `Depends` class is the primary tool for declaring a dependency in a *path operation function*. When FastAPI sees a parameter with a `Depends` default value, it will resolve and inject the result of the dependency.

```python
class Depends:
    def __init__(
        self, dependency: Optional[Callable[..., Any]] = None, *, use_cache: bool = True
    ):
        self.dependency = dependency
        self.use_cache = use_cache
```

### Parameters

| Parameter | Type | Description |
|---|---|---|
| `dependency` | `Optional[Callable[..., Any]]` | The dependency callable. This can be a function, a class, or any other callable. If not provided, it is inferred from the type annotation. |
| `use_cache` | `bool` | If `True` (the default), the result of the dependency is cached for the scope of a single request. Subsequent calls to the same dependency (with the same parameters) within the same request will return the cached value instead of re-executing the callable. Set to `False` to re-run the dependency every time it's required in the same request. |

### Example

Here is an example of a dependency that provides common query parameters. This dependency is then used in a *path operation function*.

```python
from typing import Annotated

from fastapi import Depends, FastAPI

app = FastAPI()


def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: Annotated[dict, Depends(common_parameters)]):
    return commons


@app.get("/users/")
async def read_users(commons: Annotated[dict, Depends(common_parameters)]):
    return commons
```

## `Security` Class

The `Security` class is a subclass of `Depends` and is used specifically for handling security-related dependencies. It extends `Depends` by adding a `scopes` parameter, which is essential for implementing authorization logic, such as with OAuth2.

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

### Parameters

| Parameter | Type | Description |
|---|---|---|
| `dependency` | `Optional[Callable[..., Any]]` | The security dependency callable, which is typically an instance of a security scheme class (e.g., `OAuth2PasswordBearer`). |
| `scopes` | `Optional[Sequence[str]]` | A sequence of strings representing the security scopes (e.g., permissions) required to access this endpoint. |
| `use_cache` | `bool` | Caching behavior, identical to its function in `Depends`. |

### Example

This example demonstrates how to use `Security` to protect an endpoint, requiring a valid token and specific scopes.

```python
from typing import Annotated

from fastapi import Depends, FastAPI, Security
from fastapi.security import OAuth2PasswordBearer, SecurityScopes

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


async def get_current_user(security_scopes: SecurityScopes, token: Annotated[str, Depends(oauth2_scheme)]):
    # In a real application, you would decode the token, validate scopes,
    # and fetch the user from the database.
    print(f"Token: {token}, Required Scopes: {security_scopes.scopes}")
    return {"username": "admin", "scopes": ["users", "me"]}


@app.get("/users/me")
async def read_users_me(current_user: Annotated[dict, Security(get_current_user, scopes=["me"])]):
    return current_user
```

## Internal Dependency Model

Internally, FastAPI analyzes each *path operation function* and its dependencies to build a tree-like structure represented by the `Dependant` class. This object holds all the information about the parameters (path, query, body, etc.) and sub-dependencies for a given callable.

This `Dependant` tree is then used during request processing to resolve all dependencies in the correct order, handle caching, and pass the results to your functions.

```d2
direction: right

"Path Operation": {
    "Parameter 1 (Path)"
    "Parameter 2 (Query)"
    "Parameter 3 (Depends)"
}

"FastAPI Analyzer": {
    shape: cloud
    "get_dependant()": "Analyzes Path Operation Signature"
}

"Dependant Object": {
    shape: package
    "path_params: [Parameter 1]"
    "query_params: [Parameter 2]"
    "dependencies: [Sub-Dependant for Param 3]"
    "call: Path Operation Callable"
    "use_cache: true"
}

"Path Operation" -> "FastAPI Analyzer": "On startup"
"FastAPI Analyzer" -> "Dependant Object": "Builds Dependency Tree"

```

This diagram shows how the FastAPI analyzer processes a path operation to build a `Dependant` object, which is the core model for the dependency injection system.

---

For more information on the available security schemes and utilities, please see the [Security Utilities](./api-reference-security.md) reference.