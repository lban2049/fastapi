# Bigger Applications

As your application grows, it becomes beneficial to split it into multiple files. This helps organize your code, improve maintainability, and allow different team members to work on different parts of the API simultaneously. **FastAPI** provides a powerful tool called `APIRouter` to structure your application effectively.

## The `APIRouter`

An `APIRouter` is like a mini-`FastAPI` application. You can declare *path operations* on it, and then include it in the main `FastAPI` app. This allows you to group related endpoints in separate Python modules.

### A Simple Router Example

Let's start by creating a simple router for user-related endpoints. You can place this code in a file like `app/routers/users.py`.

```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]


@router.get("/users/me", tags=["users"])
async def read_user_me():
    return {"username": "fakecurrentuser"}


@router.get("/users/{username}", tags=["users"])
async def read_user(username: str):
    return {"username": username}
```

Here, we create an instance of `APIRouter` and add *path operations* to it using the same decorators (`@router.get`, `@router.post`, etc.) as you would with a `FastAPI` instance.

### Including the Router

Now, you can include this router in your main application file, for example, `app/main.py`.

```python
from fastapi import FastAPI
from .routers import items, users

app = FastAPI()

app.include_router(users.router)
app.include_router(items.router)

@app.get("/")
async def root():
    return {"message": "Hello Bigger Applications!"}
```

By using `app.include_router(users.router)`, all the routes from `users.router` are now part of the main application. The final API will have paths like `/users/` and `/users/me`.

Here's a visual representation of how the main application includes different routers:

```d2
direction: down

"app/main.py": {
  shape: document
  label: "app/main.py"
  "FastAPI App": {
    shape: rectangle
    label: "FastAPI App"
  }
  "app.include_router(users.router)": {label: "app.include_router(users.router)"}
  "app.include_router(items.router)": {label: "app.include_router(items.router)"}
  "app.include_router(admin.router)": {label: "app.include_router(admin.router)"}
}

"app/routers/users.py": {
  shape: document
  label: "app/routers/users.py"
  "users_router": {
    label: "users_router = APIRouter()"
  }
}

"app/routers/items.py": {
  shape: document
  label: "app/routers/items.py"
  "items_router": {
    label: "items_router = APIRouter()"
  }
}

"app/internal/admin.py": {
  shape: document
  label: "app/internal/admin.py"
  "admin_router": {
    label: "admin_router = APIRouter()"
  }
}

"app/main.py" -> "app/routers/users.py": includes
"app/main.py" -> "app/routers/items.py": includes
"app/main.py" -> "app/internal/admin.py": includes
```

## Router Parameters: `prefix`, `tags`, `dependencies`, and `responses`

You can configure the `APIRouter` with parameters that will apply to all of its *path operations*. This is useful for avoiding code duplication.

Consider another router in `app/routers/items.py`:

```python
from fastapi import APIRouter, Depends, HTTPException

from ..dependencies import get_token_header

router = APIRouter(
    prefix="/items",
    tags=["items"],
    dependencies=[Depends(get_token_header)],
    responses={404: {"description": "Not found"}},
)


fake_items_db = {"plumbus": {"name": "Plumbus"}, "gun": {"name": "Portal Gun"}}


@router.get("/")
async def read_items():
    return fake_items_db


@router.get("/{item_id}")
async def read_item(item_id: str):
    if item_id not in fake_items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"name": fake_items_db[item_id]["name"], "item_id": item_id}


@router.put(
    "/{item_id}",
    tags=["custom"],
    responses={403: {"description": "Operation forbidden"}},
)
async def update_item(item_id: str):
    if item_id != "plumbus":
        raise HTTPException(
            status_code=403, detail="You can only update the item: plumbus"
        )
    return {"item_id": item_id, "name": "The great Plumbus"}
```

Let's break down the parameters used in `APIRouter`:

- **`prefix`**: A path prefix for all routes in this router. Here, `"/items"` means that the path for `read_items` will be `/items/` and for `read_item` will be `/items/{item_id}`.
- **`tags`**: A list of tags to apply to all path operations, which is used for grouping in the API documentation.
- **`dependencies`**: A list of dependencies that will be executed for all path operations in this router.
- **`responses`**: A dictionary of additional responses that apply to all path operations.

Notice that you can still override these settings on a per-operation basis. For example, the `update_item` operation adds a `"custom"` tag and a specific `403` response.

## Including a Router with Custom Parameters

You can also provide these parameters when you call `app.include_router()`. This is useful for adding a prefix to a group of routers or applying a dependency to a section of your API that requires authentication.

In `app/main.py`, you can include another router with specific settings:

```python
from fastapi import Depends, FastAPI

from .dependencies import get_query_token, get_token_header
from .internal import admin
from .routers import items, users

app = FastAPI(dependencies=[Depends(get_query_token)])


app.include_router(users.router)
app.include_router(items.router)
app.include_router(
    admin.router,
    prefix="/admin",
    tags=["admin"],
    dependencies=[Depends(get_token_header)],
    responses={418: {"description": "I'm a teapot"}},
)


@app.get("/")
async def root():
    return {"message": "Hello Bigger Applications!"}
```

In this example, all routes from `admin.router` will be prefixed with `/admin`, will be tagged with `"admin"`, will require the `get_token_header` dependency, and will have an additional `418` response defined in the documentation.

By using `APIRouter`, you can effectively structure large applications, keeping your code organized, reusable, and easy to manage.

Now that your application is well-structured, the next step is to ensure its reliability. Learn how to write tests for your path operations and dependencies in the [Testing](./advanced-testing.md) section.