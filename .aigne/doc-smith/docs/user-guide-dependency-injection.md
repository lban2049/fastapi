# Dependency Injection

FastAPI includes a powerful but intuitive Dependency Injection (DI) system. It's a way for your code to declare things it requires to work, like database sessions, authentication credentials, or shared parameters. FastAPI then takes care of providing these dependencies to your code.

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

You could also write `commons = Depends(CommonQueryParams)` without a type hint, but this is not recommended. You lose the benefits of type checking and editor autocompletion.

## How it Works

The dependency injection system follows a clear flow when a request comes in.

```d2
direction: down

"Client": {
  shape: person
}

"FastAPI App": {
  shape: package
  grid-columns: 1

  "/items/ endpoint": {
    shape: rectangle
    "read_items(commons: CommonQueryParams = Depends())"
  }

  "Dependency Injector": {
    shape: diamond
  }

  "CommonQueryParams": {
    label: "CommonQueryParams class"
    shape: class
    "__init__(self, q, skip, limit)"
  }
}

"HTTP Response": {
  shape: document
}

"Client" -> "FastAPI App"."/items/ endpoint": "1. GET /items/?q=foo"

"FastAPI App"."/items/ endpoint" -> "FastAPI App"."Dependency Injector": "2. Sees Depends() on 'commons' parameter"

"FastAPI App"."Dependency Injector" -> "FastAPI App"."CommonQueryParams": "3. Resolves dependency from type hint\n- Extracts q, skip, limit from request\n- Creates instance: CommonQueryParams(q='foo', skip=0, limit=100)"

"FastAPI App"."CommonQueryParams" -> "FastAPI App"."/items/ endpoint": "4. Injects instance into 'commons' argument"

"FastAPI App"."/items/ endpoint" -> "HTTP Response": "5. Path operation runs with the dependency result"

"HTTP Response" -> "Client": "6. Sends response back"
```

## Summary

FastAPI's dependency injection provides a simple yet powerful way to manage dependencies and reuse code. You can define dependencies as either functions or classes and inject them into your path operations using `Depends`. This system is the foundation for many advanced features, including security and database connection management.

To dive deeper, explore the [Advanced Topics](./advanced.md) for more complex use cases and patterns.