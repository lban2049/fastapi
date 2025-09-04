# Dependency Injection

FastAPI includes an intuitive Dependency Injection (DI) system. It's a way for your code to declare things it requires to work, like database sessions, authentication credentials, or shared parameters. FastAPI then takes care of providing these dependencies to your code.

This is very useful for:
- Sharing logic and code.
- Sharing database connections.
- Enforcing security, authentication, and role requirements.
- And many other cases.

Let's start with a simple example.

## Create a Dependency, or "Dependable"

Imagine you have multiple endpoints that share the same query parameters, for example, for pagination (`skip`, `limit`) and an optional query string (`q`).

Instead of repeating these parameters in every path operation function, you can define them once in a shared function. This function is our dependency.

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

Here's what's happening:
1.  We created a function `common_parameters` that takes the same parameters as a path operation function (`q`, `skip`, `limit`).
2.  This function returns a dictionary containing these values.
3.  In our path operation functions `read_items` and `read_users`, we declare a parameter `commons`.
4.  We provide a default value for this parameter: `Depends(common_parameters)`. `Depends` is a special marker that tells FastAPI that this parameter depends on another function.

FastAPI will then:
- Call the dependency function (`common_parameters`) with the required parameters from the request.
- Take the return value of that function.
- Assign that return value to the parameter in the path operation function (`commons`).

Now, both the `/items/` and `/users/` endpoints share the same set of query parameters, defined in one place.

## Classes as Dependencies

While functions are great for simple dependencies, you can also use classes. This can be beneficial for organizing your code, especially as dependencies become more complex.

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

When you declare a dependency with a class like `Depends(CommonQueryParams)`, FastAPI understands that it needs to create an instance of that class. It will inspect the `__init__` method and provide the necessary parameters from the request, just as it would for a function.

The benefit here is that your editor will provide better autocompletion and type-checking because it knows `commons` is an instance of `CommonQueryParams`.

## Shortcut: `Depends()`

You might have noticed we are repeating `CommonQueryParams` in the type hint and inside `Depends`. FastAPI provides a convenient shortcut for this common pattern.

If you pass nothing to `Depends()`, it will use the type annotation of the parameter to determine the dependency.

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

This code is equivalent to the previous example but is more concise. The `commons: CommonQueryParams = Depends()` syntax is the most common and recommended way to declare a class-based dependency.

### Without Type Hint

You could also write `commons = Depends(CommonQueryParams)` without a type hint, but this is not recommended as you lose the benefits of type checking and editor autocompletion.

## Sub-dependencies

Dependencies can declare their own dependencies. FastAPI's dependency injection system will resolve this chain of dependencies automatically.

Here's an example where one dependency relies on another:

```python
from typing import Union, Annotated

from fastapi import Cookie, Depends, FastAPI

app = FastAPI()


def query_extractor(q: Union[str, None] = None):
    # This dependency gets the raw query parameter 'q'
    return q


def query_or_cookie_extractor(
    q: Annotated[str, Depends(query_extractor)],
    last_query: Union[str, None] = Cookie(default=None),
):
    # This dependency depends on query_extractor.
    # It returns the query 'q' if it exists, otherwise it falls back to a cookie value.
    if not q:
        return last_query
    return q


@app.get("/items/")
async def read_query(
    query_or_default: Annotated[str, Depends(query_or_cookie_extractor)],
):
    # The path operation only needs to depend on the final dependency.
    return {"q_or_cookie": query_or_default}

```

In this flow:
1. The path operation `read_query` depends on `query_or_cookie_extractor`.
2. `query_or_cookie_extractor` in turn depends on `query_extractor`.
3. FastAPI first resolves `query_extractor`, gets the value of `q`, and passes it to `query_or_cookie_extractor`.
4. The result of `query_or_cookie_extractor` is then passed to the path operation.

This creates a dependency graph that FastAPI traverses and resolves for each request.

## Dependencies with `yield`

For dependencies that need to perform cleanup actions after a response is sent (like closing a database connection), you can use a generator with `yield`.

The code before the `yield` statement is executed before the response is generated. The yielded value is injected into the path operation. The code after the `yield` is executed after the response has been sent.

This pattern is ideal for managing resources.

```python
from fastapi import Depends, FastAPI

app = FastAPI()

# A dummy "database" connection class for demonstration
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
    # The 'db' object is the yielded value from get_db_session
    return {"message": "Items read", "db_connected": db.connected}

```

When a request is made to `/items/`, FastAPI will:
1. Call `get_db_session()`.
2. Execute the code up to `yield db`, creating a `DummyDB` instance.
3. Inject the `db` instance into `read_items`.
4. Execute `read_items` and generate a response.
5. After the response is sent, execute the code in the `finally` block, calling `db.close()`.

This ensures that resources are always released, even if an error occurs during the request.

## Dependency Caching

By default, FastAPI caches the return value of a dependency within the scope of a single request. If multiple parts of your application (e.g., a path operation and a sub-dependency) depend on the same dependency, it will only be executed once, and the result will be reused.

For example, if `dependency_b` depends on `dependency_a`, and the path operation depends on both `dependency_a` and `dependency_b`, `dependency_a` will only be run once.

To disable this behavior and force the re-execution of a dependency every time it's called, you can set `use_cache=False`.

```python
from fastapi import Depends, FastAPI

app = FastAPI()


async def get_value(use_cache: bool = True):
    # A dummy function to demonstrate caching
    return {"value": "cached" if use_cache else "new"}

# Cached dependency (default behavior)
@app.get("/cached/")
async def read_cached(val: dict = Depends(get_value)):
    # 'get_value' is called once per request
    return val

# Non-cached dependency
@app.get("/no-cache/")
async def read_no_cache(val: dict = Depends(get_value, use_cache=False)):
    # 'get_value' is called every time it appears as a dependency
    return val
```

Caching is generally desirable as it prevents redundant computations, like repeatedly fetching a user's data from a database within the same request.

## Summary

FastAPI's dependency injection provides a simple yet powerful way to manage dependencies and reuse code. You can define dependencies as either functions or classes and inject them into your path operations using `Depends`. This system supports sub-dependencies, resource management with `yield`, and caching, forming the foundation for many features like security and database connection management.

To see how this is applied for security, proceed to the [Security](./advanced-security.md) section.