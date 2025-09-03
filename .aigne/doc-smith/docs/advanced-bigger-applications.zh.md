# 更大型的应用

随着应用的增长，将其拆分成多个文件会很有益处。这有助于组织代码、提高可维护性，并允许不同团队成员同时处理 API 的不同部分。**FastAPI** 提供了一个名为 `APIRouter` 的强大工具，可以有效地组织应用结构。

## `APIRouter`

`APIRouter` 就像一个迷你的 `FastAPI` 应用。你可以在其上声明*路径操作*，然后将其包含在主 `FastAPI` 应用中。这使你可以将相关的端点分组到不同的 Python 模块中。

### 一个简单的路由示例

让我们从为用户相关的端点创建一个简单的路由开始。你可以将此代码放在类似 `app/routers/users.py` 的文件中。

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

在这里，我们创建了一个 `APIRouter` 实例，并使用与 `FastAPI` 实例相同的装饰器（`@router.get`、`@router.post` 等）向其添加*路径操作*。

### 包含路由

现在，你可以将此路由包含在主应用文件中，例如 `app/main.py`。

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

通过使用 `app.include_router(users.router)`，来自 `users.router` 的所有路由现在都成为主应用的一部分。最终的 API 将拥有像 `/users/` 和 `/users/me` 这样的路径。

下图直观地展示了主应用如何包含不同的路由：

```d2
direction: down

"app/main.py" {
  shape: document
  "FastAPI App" {
    shape: rectangle
  }
  "app.include_router(users.router)" {}
  "app.include_router(items.router)" {}
  "app.include_router(admin.router)" {}
}

"app/routers/users.py" {
  shape: document
  "users_router = APIRouter()"
}

"app/routers/items.py" {
  shape: document
  "items_router = APIRouter()"
}

"app/internal/admin.py" {
  shape: document
  "admin_router = APIRouter()"
}

"app/main.py" -> "app/routers/users.py": "includes"
"app/main.py" -> "app/routers/items.py": "includes"
"app/main.py" -> "app/internal/admin.py": "includes"
```

## 路由参数：`prefix`、`tags`、`dependencies` 和 `responses`

你可以为 `APIRouter` 配置参数，这些参数将应用于其所有的*路径操作*。这对于避免代码重复很有用。

不妨看看 `app/routers/items.py` 中的另一个路由：

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

让我们来分解 `APIRouter` 中使用的参数：

- **`prefix`**：此路由中所有路径的路径前缀。在这里，`"/items"` 意味着 `read_items` 的路径将是 `/items/`，而 `read_item` 的路径将是 `/items/{item_id}`。
- **`tags`**：应用于所有路径操作的标签列表，用于在 API 文档中进行分组。
- **`dependencies`**：将为此路由中所有路径操作执行的依赖项列表。
- **`responses`**：应用于所有路径操作的附加响应字典。

请注意，你仍然可以基于单个操作覆盖这些设置。例如，`update_item` 操作添加了一个 `"custom"` 标签和一个特定的 `403` 响应。

## 使用自定义参数包含路由

你也可以在调用 `app.include_router()` 时提供这些参数。这对于为一组路由添加前缀，或为 API 中需要身份验证的部分应用依赖项非常有用。

在 `app/main.py` 中，你可以使用特定设置包含另一个路由：

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

在此示例中，来自 `admin.router` 的所有路由都将以 `/admin` 为前缀，标记为 `"admin"`，需要 `get_token_header` 依赖项，并在文档中定义一个额外的 `418` 响应。

通过使用 `APIRouter`，你可以有效地构建大型应用的结构，使代码保持井然有序、可重用且易于管理。

现在你的应用结构良好，下一步是确保其可靠性。在 [测试](./advanced-testing.md) 部分学习如何为你的路径操作和依赖项编写测试。