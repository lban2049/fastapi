# 更大的应用 - 多文件

当应用规模变大时，将其拆分为多个文件有助于提高可维护性和组织性。FastAPI 的设计旨在使用 `APIRouter` 来支持这种做法。

## 示例文件结构

构建大型应用的一种常见方法是将路径操作分离到不同的模块中。您可以将相关的端点分组到一个 `routers` 目录中。

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

这种结构有助于分离关注点，例如，将所有与用户相关的端点放在 `users.py` 中，将与项目相关的端点放在 `items.py` 中。

## 创建路由器

您可以在每个模块中创建独立的 `APIRouter` 实例。`APIRouter` 的工作方式与 `FastAPI` 应用非常相似，允许您声明路径操作。

以下是 `app/routers/users.py` 的示例：

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

同样，`app/routers/items.py` 也定义了自己的路由器。您可以为路由器配置参数，这些参数将应用于其所有路径操作，例如 `prefix`、`tags`、`dependencies` 和默认 `responses`。

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

在这种情况下，`items.py` 中的所有路径操作的路径都将以 `/items` 开头，被标记为 `items`，并需要 `get_token_header` 依赖。

## 主应用

主应用文件 `app/main.py` 是您将所有内容整合在一起的地方。您可以从路由器模块中导入路由器对象，并将它们包含在主 `FastAPI` 应用中。

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

### 包含路由器

您可以使用 `app.include_router()` 来挂载路由器。此方法允许您添加路由器，并可以选择性地覆盖或增加其配置。

对于 `admin.router`，我们添加了 `/admin` 前缀、一个 `admin` 标签、一个额外的依赖项和一个默认响应。这些参数会应用于 `admin.router` 中定义的所有路径操作，并叠加在路由器本身已配置的任何参数之上。

这种结构使您的代码保持组织性和可扩展性。

### 应用结构图

下图说明了主应用如何包含不同的路由器，以及如何组合前缀以形成最终的 API 路径。

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

通过使用 `APIRouter`，您可以有效地将应用组织成逻辑模块，从而使您的代码库在不断增长的过程中更清晰、更易于维护和导航。这种方法促进了代码重用和关注点分离。

要了解有关路由的更多具体信息，请参阅[路由 API 参考](./api-reference-routing.md)。要了解如何测试您的结构化应用，请访问[测试](./advanced-testing.md)指南。