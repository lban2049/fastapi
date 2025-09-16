# Dependencies

FastAPI includes a powerful and easy-to-use Dependency Injection system. It allows you to manage dependencies (like database sessions, security requirements, or shared logic) in a structured and reusable way. FastAPI handles calling your dependencies and injecting their results into your *path operation functions*.

This system helps you:
-   Share logic and code.
-   Share database connections.
-   Implement security schemes (authentication and authorization).
-   And much more...

This page serves as a technical reference for the `Depends` and `Security` functions. For a step-by-step guide, please see the [Tutorial on Dependencies](./tutorials-dependencies-and-security.md).

## Depends

The `Depends` function is the primary tool for declaring a dependency in your application. You provide it with a callable (a function, a class, etc.), and FastAPI will execute it, resolve any sub-dependencies, and inject the returned value into your function's parameter.

### Parameters

<x-field data-name="dependency" data-type="Optional[Callable[..., Any]]" data-default="None" data-required="false" data-desc="A 'dependable' callable, such as a function. You should pass the function object itself, not the result of calling it. FastAPI will execute it for you."></x-field>
<x-field data-name="use_cache" data-type="bool" data-default="true" data-required="false" data-desc="If `True` (the default), the dependency's result is cached for the duration of a single request. If the same dependency is required multiple times in one request, it will only be executed once, and the cached value will be reused."></x-field>

### Example

Here's how to create a dependency that provides common query parameters. This dependency can then be reused across multiple *path operations*.

```python title="dependency_example.py" icon=logos:python
from typing import Annotated, Union

from fastapi import Depends, FastAPI

app = FastAPI()


async def common_parameters(q: Union[str, None] = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: Annotated[dict, Depends(common_parameters)]):
    return commons


@app.get("/users/")
async def read_users(commons: Annotated[dict, Depends(common_parameters)]):
    return commons
```

In this example, `common_parameters` is a dependency. When a request comes to `/items/` or `/users/`, FastAPI will:
1.  Call the `common_parameters` function.
2.  Extract `q`, `skip`, and `limit` from the request's query string.
3.  Pass them as arguments to `common_parameters`.
4.  Take the dictionary returned by `common_parameters`.
5.  Inject that dictionary into the `commons` parameter of `read_items` or `read_users`.

## Security

The `Security` function is a special sub-class of `Depends`. It works in the exact same way but adds the ability to define security scopes, which are then integrated into your OpenAPI schema and the interactive API documentation (e.g., at `/docs`).

This is particularly useful for documenting authentication and authorization requirements, especially with schemes like OAuth2.

### Parameters

<x-field data-name="dependency" data-type="Optional[Callable[..., Any]]" data-default="None" data-required="false" data-desc="A 'dependable' callable that implements a security scheme, for example, by verifying a token or API key and returning the current user."></x-field>
<x-field data-name="scopes" data-type="Optional[Sequence[str]]" data-default="None" data-required="false" data-desc="A sequence of strings (a list or tuple) representing the OAuth2 scopes required for this specific dependency. This information is used for the OpenAPI schema."></x-field>
<x-field data-name="use_cache" data-type="bool" data-default="true" data-required="false" data-desc="Caches the result of the security dependency within a single request. Defaults to `True`."></x-field>

### Example

In this example, `Security` is used to protect an endpoint. It relies on a dependency `get_current_active_user` and specifies that the `items` scope is required.

```python title="security_dependency_example.py" icon=logos:python
from typing import Annotated, Union

from pydantic import BaseModel
from fastapi import Depends, FastAPI, Security
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

class User(BaseModel):
    username: str
    email: Union[str, None] = None

# This is a simplified example. In a real application,
# you would decode and validate the token properly.
def get_current_active_user(token: Annotated[str, Depends(oauth2_scheme)]):
    # In a real app, you would decode the token and get the user from the database
    user = User(username=token + "_faked", email="user@example.com")
    return user

@app.get("/users/me/items/")
async def read_own_items(
    current_user: Annotated[User, Security(get_current_active_user, scopes=["items"])],
):
    return [{"item_id": "Foo", "owner": current_user.username}]

```

The use of `Security(get_current_active_user, scopes=["items"])` tells FastAPI to:
1.  Treat `get_current_active_user` as a dependency.
2.  Inject the result into the `current_user` parameter.
3.  In the OpenAPI documentation, mark this endpoint as secured by the scheme defined in `get_current_active_user` (which itself depends on `oauth2_scheme`) and requiring the `items` scope.

For a more in-depth look at security utilities, please see the [Security API Reference](./api-reference-security.md).