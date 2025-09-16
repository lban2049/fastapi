# Bigger Applications - Multiple Files

As your application grows, it becomes beneficial to split it into smaller, more manageable files. A common approach is to group related API endpoints into their own modules. FastAPI provides a powerful and intuitive tool for this purpose: `APIRouter`.

`APIRouter` works like a mini-`FastAPI` application. You can define *path operations*, dependencies, and other configurations on it, and then include it in your main application. This guide will walk you through a practical example of structuring a larger application.

## The Main Application File

Let's start with the main file, which will serve as the entry point for our application. This file is responsible for creating the main `FastAPI` instance and including all the routers from other modules.

```python main.py icon=logos:python
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

The key function here is `app.include_router()`. It takes a router object and incorporates all of its paths into the main application. You can also see that some dependencies are applied to the entire application in the `FastAPI` constructor.

## A Simple Router

Now, let's create a separate module for user-related endpoints. This helps keep our code organized by feature.

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

In this file, we create an instance of `APIRouter` and use its decorators (`@router.get`) to define *path operations*, just as you would with a `FastAPI` app instance. This router is then imported and included in our `main.py` file.

## A Router with Configuration

`APIRouter` can be initialized with parameters that will apply to all of its *path operations*. This is a convenient way to set a common path prefix, add tags, specify dependencies, and define common responses for a group of endpoints.

Here's a router for items with its own configuration:

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

Because we set `prefix="/items"` on the router, the path for the `read_item` function will be `/items/{item_id}`. All endpoints in this router will be tagged under "items", will depend on `get_token_header`, and will have a default 404 response documented.

Notice that you can still add extra tags or responses to individual path operations, like in `update_item`. These will be combined with the router's configuration.

## Including a Router with Additional Configuration

You can also provide configuration parameters directly within `app.include_router()`. These settings will be applied to every *path operation* in the included router, in addition to any configuration already defined on the router itself.

This is particularly useful for mounting a set of endpoints under a specific path, like `/admin`, or for applying a higher level of authentication to an entire module.

First, let's look at a simple admin router:

```python app/internal/admin.py icon=logos:python
from fastapi import APIRouter

router = APIRouter()


@router.post("/")
async def update_admin():
    return {"message": "Admin getting schwifty"}
```

Now, see how it's included in `main.py` with extra parameters:

```python main.py icon=logos:python
# ...
app.include_router(
    admin.router,
    prefix="/admin",
    tags=["admin"],
    dependencies=[Depends(get_token_header)],
    responses={418: {"description": "I'm a teapot"}},
)
# ...
```

Even though the path in `admin.py` is `/`, the `prefix` in `include_router` ensures that the final path for `update_admin` is `/admin/`. Similarly, the tags, dependencies, and responses defined here will be applied to all endpoints from `admin.router`.

## Summary

By using `APIRouter`, you can effectively structure your FastAPI application into multiple files and modules. This approach enhances code organization, reusability, and maintainability as your project scales. You can apply common configurations at either the router definition or inclusion point, giving you flexible control over your API's structure.

Now that you've learned how to structure your application, let's explore how to add custom processing for requests and responses. Continue to the [Middleware](./tutorials-advanced-middleware.md) guide.