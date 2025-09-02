# Dependency Injection

FastAPI includes a powerful but intuitive Dependency Injection system. It's designed to be easy to use and helps you manage dependencies, share logic, handle authentication, manage database connections, and more, all while ensuring your code remains clean and well-structured.

This system allows you to declare dependencies that your *path operation functions* need, and FastAPI takes care of providing them.

## A Simple Dependency

Let's start with a basic example. Imagine you have multiple endpoints that share common query parameters, like `q`, `skip`, and `limit`.

Instead of repeating these parameters in every function signature, you can define them once in a dedicated function.

### Create a Dependency

A dependency is simply a function (or a callable) that can take the same parameters as a *path operation function*.

Here, we define a `common_parameters` function that will handle our shared query parameters:

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

### How it Works

1.  **Define the Dependency**: The `common_parameters` function is our dependency. It takes the standard query parameters `q`, `skip`, and `limit`.
2.  **"Depend" on it**: In our *path operation functions* (`read_items` and `read_users`), we add a parameter `commons`.
3.  **Use `Depends`**: We assign the default value `Depends(common_parameters)` to the `commons` parameter. This tells FastAPI that `commons` is not a regular parameter but a dependency that needs to be resolved.

When a request comes to `/items/` or `/users/`, FastAPI will:
*   Call the `common_parameters` function with the query parameters from the request.
*   Take the returned value (a dictionary).
*   Pass that dictionary as the `commons` argument to `read_items` or `read_users`.

This allows you to reuse the same parameter logic across multiple endpoints without duplicating code.

## Classes as Dependencies

While functions are great for simple dependencies, classes offer better organization for more complex logic. You can use a class as a dependency, and FastAPI will handle its instantiation.

Let's refactor the previous example to use a class.

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

Here, FastAPI sees `Depends(CommonQueryParams)` and understands it needs to:
1.  Inspect the `__init__` method of the `CommonQueryParams` class.
2.  Resolve the parameters for `__init__` (the query parameters `q`, `skip`, and `limit`) from the request.
3.  Create an instance of `CommonQueryParams` using those parameters.
4.  Pass that instance as the `commons` argument to `read_items`.

This approach is more structured and aligns well with object-oriented principles.

### A Simpler Syntax

FastAPI provides a convenient shortcut. If you use a type hint for the dependency, you don't need to pass the callable to `Depends` again.

You can simply use `Depends()`:

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

FastAPI is smart enough to see the type hint `CommonQueryParams` and understand that it is the dependency you want to inject. This is the most common and recommended way to use class-based dependencies.

## Dependency Injection Flow

The dependency injection system follows a clear and predictable flow to resolve and provide dependencies to your path operations.

```d2
direction: down

request: "Incoming Request"
path_op: "Path Operation Function"

subgraph "FastAPI Engine" {
  direction: right
  
  dep_marker: "Detects `Depends()`"
  inspector: "Inspects Dependency Signature (e.g., `__init__`)"
  param_solver: "Resolves Parameters (query, path, etc.) from Request"
  dep_callable: "Calls Dependency (e.g., creates class instance)"
  
  dep_marker -> inspector -> param_solver -> dep_callable
}

request -> path_op
path_op -> dep_marker
param_solver -> request: "gets params"

dep_result: "Dependency Result"
dep_callable -> dep_result
dep_result -> path_op: "Injects result"

```

## Caching

The dependency injection system includes a cache. For a single request, if multiple parts of your code depend on the same dependency (with the same parameters), it will only be called once. The result is cached and reused for all subsequent needs within that same request.

This is controlled by the `use_cache` parameter in `Depends`, which defaults to `True`.

```python
class Depends:
    def __init__(
        self, dependency: Optional[Callable[..., Any]] = None, *, use_cache: bool = True
    ):
        self.dependency = dependency
        self.use_cache = use_cache
```

This is particularly useful for dependencies that establish database connections or perform expensive computations, ensuring they don't run unnecessarily multiple times per request.

By mastering dependency injection, you can build complex, maintainable, and robust APIs. The principles you've learned here form the foundation for more advanced topics, such as authentication and authorization.

For the next step, see how to apply these concepts to secure your application in the [Security](./advanced-security.md) guide.