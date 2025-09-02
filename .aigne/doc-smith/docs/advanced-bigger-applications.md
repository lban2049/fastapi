# Bigger Applications - Multiple Files

When applications grow, it's beneficial to split them into multiple files to improve maintainability and organization. FastAPI is designed to support this practice using `APIRouter`.

## Example File Structure

A common way to structure a larger application is to separate path operations into different modules. You can group related endpoints into a `routers` directory.

```text
. / app /
│
├── __init__.py
├── main.py
├── dependencies.py
└── routers /
    ├── __init__.py
    ├── items.py
    └── users.py
```

This structure helps in separating concerns, for example, keeping all user-related endpoints in `users.py` and item-related ones in `items.py`.

## Creating the Routers

You can create separate `APIRouter` instances in each of your modules. An `APIRouter` works very similarly to a `FastAPI` app, allowing you to declare path operations.

Here's an example of `app/routers/users.py`:

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

Similarly, `app/routers/items.py` defines its own router. You can configure the router with parameters that will apply to all of its path operations, such as a `prefix`, `tags`, `dependencies`, and default `responses`.

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

In this case, all path operations in `items.py` will have their paths start with `/items`, be tagged with `items`, and require the `get_token_header` dependency.

## The Main Application

The main application file, `app/main.py`, is where you bring everything together. You import the router objects from your router modules and include them in the main `FastAPI` app.

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

### Including Routers

You use `app.include_router()` to mount the routers. This method allows you to add a router and optionally override or add to its configuration.

For the `admin.router`, we add a `/admin` prefix, an `admin` tag, an additional dependency, and a default response. These parameters apply to all path operations defined in `admin.router`, on top of any parameters already configured on the router itself.

This structure keeps your code organized and scalable.

### Application Structure Diagram

The following diagram illustrates how the main application includes the different routers and how prefixes are combined to form the final API paths.

```d2
direction: down

"FastAPI App (main.py)": {
  shape: cloud

  "app.include_router(users.router)": {}
  "app.include_router(items.router)": {}
  "app.include_router(admin.router, prefix='/admin')": {}
}

"users.router (APIRouter)": {
  shape: package
  "/users/": {}
  "/users/me": {}
}

"items.router (APIRouter(prefix='/items'))": {
  shape: package
  "/": "-> /items/"
  "/{item_id}": "-> /items/{item_id}"
}

"admin.router (APIRouter)": {
  shape: package
  "/": "-> /admin/"
  "/dashboard": "-> /admin/dashboard"
}

"FastAPI App (main.py)"."app.include_router(users.router)" -> "users.router (APIRouter)": {
  label: "Includes"
  style: {
    animated: true
  }
}
"FastAPI App (main.py)"."app.include_router(items.router)" -> "items.router (APIRouter(prefix='/items'))": {
  label: "Includes"
  style: {
    animated: true
  }
}
"FastAPI App (main.py)"."app.include_router(admin.router, prefix='/admin')" -> "admin.router (APIRouter)": {
  label: "Includes with prefix"
  style: {
    animated: true
  }
}
```

By using `APIRouter`, you can effectively organize your application into logical modules, making your codebase cleaner, more maintainable, and easier to navigate as it grows. This approach promotes code reuse and separation of concerns.

To learn more about the specifics of routing, see the [Routing API Reference](./api-reference-routing.md). To understand how to test your structured application, visit the [Testing](./advanced-testing.md) guide.