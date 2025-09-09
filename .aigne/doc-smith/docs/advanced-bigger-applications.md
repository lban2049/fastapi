# Bigger Applications

When building applications, it's common to start with a single file, like `main.py`. However, as your application grows, keeping everything in one file can become difficult to manage. 

FastAPI provides a simple and elegant solution for structuring your application across multiple files and modules: `APIRouter`. By using routers, you can group related *path operations*, making your codebase organized, modular, and easier to maintain, especially when working in a team.

## The `APIRouter`

Think of an `APIRouter` as a mini-`FastAPI` application. It allows you to declare routes, dependencies, and other configurations in a separate module, which can then be included in your main application.

Let's create a router to handle user-related endpoints. You can place this code in a file named `app/routers/users.py`.

```python app/routers/users.py icon=logos:python
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

As you can see, we import `APIRouter`, create an instance, and then use the `@router.get()` decorator to define our *path operations*, just as we would with `@app.get()` on a `FastAPI` instance.

### Including the Router in the Main App

Once you have your router defined, you need to include it in your main `FastAPI` application. In your `app/main.py` file, you can use `app.include_router()` to add all the routes from your `users` module.

```python app/main.py icon=logos:python
from fastapi import FastAPI

from .routers import users

app = FastAPI()

app.include_router(users.router)


@app.get("/")
async def root():
    return {"message": "Hello Bigger Applications!"}

```

With this setup, your main application now includes all the routes from `users.router`. A request to `/users/` will be correctly routed to the `read_users` *path operation*.

Here's a diagram illustrating how the main app can include multiple routers from different files:

```d2
direction: down

app-main: {
  label: "app/main.py\n(FastAPI App)"
  shape: rectangle
}

app-routers-users: {
  label: "app/routers/users.py\n(users.router)"
  shape: rectangle
}

app-routers-items: {
  label: "app/routers/items.py\n(items.router)"
  shape: rectangle
}

app-internal-admin: {
  label: "app/internal/admin.py\n(admin.router)"
  shape: rectangle
}

app-main -> app-routers-users: "app.include_router(users.router)"
app-main -> app-routers-items: "app.include_router(items.router)"
app-main -> app-internal-admin: "app.include_router(admin.router)"
```

## Configuring an `APIRouter`

To avoid duplicating code, you can configure your `APIRouter` with parameters that will be applied to all of its *path operations*. This is particularly useful for setting a common path prefix, adding tags, defining dependencies, or specifying common responses.

Let's look at another router in `app/routers/items.py` that uses these parameters:

```python app/routers/items.py icon=logos:python
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

Here's what each parameter does:

- **`prefix`**: A URL path prefix for all *path operations* in this router. Since we set `prefix="/items"`, the path for `read_items` becomes `/items/` and for `read_item` becomes `/items/{item_id}`.
- **`tags`**: A list of tags to apply to all operations, which helps group them in the OpenAPI documentation.
- **`dependencies`**: A list of dependencies that will be executed for every request to this router's endpoints.
- **`responses`**: A dictionary of additional responses that will be included in the documentation for all operations.

Even with these global settings, you can still customize each *path operation*. For example, the `update_item` operation adds an extra `"custom"` tag and defines a specific `403` response, which will be merged with the router's global settings.

## Including a Router with Parameters

You can also provide these configuration parameters directly in `app.include_router()`. These parameters will apply to all routes within the included router, and will override any parameters set on the `APIRouter` instance itself.

This is especially useful for applying settings like a path prefix or authentication dependency to a whole group of routes from your main application file.

Let's update `app/main.py` to include multiple routers, some with custom configurations:

```python app/main.py icon=logos:python
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

In this example:
- `users.router` and `items.router` are included with their own predefined settings.
- `admin.router` is included with a new set of configurations: all its routes will be prefixed with `/admin`, tagged with `"admin"`, protected by the `get_token_header` dependency, and will have an additional `418` response in the documentation.

By leveraging `APIRouter`, you can effectively structure large applications, keeping your code organized, reusable, and easy to manage.

Now that your application is well-structured, the next step is to ensure its reliability. Learn how to write tests for your path operations and dependencies in the [Testing](./advanced-testing.md) section.