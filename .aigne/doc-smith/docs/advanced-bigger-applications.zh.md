# 更大型的应用

在构建应用程序时，通常会从单个文件（如 `main.py`）开始。然而，随着应用程序的增长，将所有内容都放在一个文件中会变得难以管理。

FastAPI 提供了一种简单而优雅的解决方案，用于将应用程序构建在多个文件和模块中：`APIRouter`。通过使用路由器，你可以对相关的*路径操作*进行分组，使你的代码库更有条理、模块化且更易于维护，尤其是在团队协作时。

## `APIRouter`

可以将 `APIRouter` 视为一个迷你的 `FastAPI` 应用程序。它允许你在一个单独的模块中声明路由、依赖项和其他配置，然后可以将其包含在主应用程序中。

我们来创建一个路由器来处理与用户相关的端点。你可以将此代码放在名为 `app/routers/users.py` 的文件中。

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

如你所见，我们导入了 `APIRouter`，创建了一个实例，然后使用 `@router.get()` 装饰器来定义我们的*路径操作*，就像在 `FastAPI` 实例上使用 `@app.get()` 一样。

### 在主应用中包含路由器

定义好路由器后，需要将其包含在主 `FastAPI` 应用程序中。在你的 `app/main.py` 文件中，你可以使用 `app.include_router()` 来添加来自 `users` 模块的所有路由。

```python app/main.py icon=logos:python
from fastapi import FastAPI

from .routers import users

app = FastAPI()

app.include_router(users.router)


@app.get("/")
async def root():
    return {"message": "Hello Bigger Applications!"}

```

通过此设置，你的主应用程序现在包含了来自 `users.router` 的所有路由。对 `/users/` 的请求将被正确路由到 `read_users` *路径操作*。

下图说明了主应用如何可以包含来自不同文件的多个路由器：

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

## 配置 `APIRouter`

为了避免代码重复，你可以为 `APIRouter` 配置参数，这些参数将应用于其所有的*路径操作*。这对于设置通用的路径前缀、添加标签、定义依赖项或指定通用响应特别有用。

我们来看另一个位于 `app/routers/items.py` 中使用这些参数的路由器：

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

以下是每个参数的作用：

- **`prefix`**：此路由器中所有*路径操作*的 URL 路径前缀。由于我们设置了 `prefix="/items"`，`read_items` 的路径变为 `/items/`，`read_item` 的路径变为 `/items/{item_id}`。
- **`tags`**：应用于所有操作的标签列表，有助于在 OpenAPI 文档中对它们进行分组。
- **`dependencies`**：将为此路由器的每个端点请求执行的依赖项列表。
- **`responses`**：将包含在所有操作文档中的附加响应字典。

即使有这些全局设置，你仍然可以自定义每个*路径操作*。例如，`update_item` 操作添加了一个额外的 `"custom"` 标签，并定义了一个特定的 `403` 响应，它将与路由器的全局设置合并。

## 包含带参数的路由器

你也可以直接在 `app.include_router()` 中提供这些配置参数。这些参数将应用于所包含路由器内的所有路由，并将覆盖在 `APIRouter` 实例本身上设置的任何参数。

这对于从主应用程序文件对整组路由应用路径前缀或身份验证依赖项等设置特别有用。

我们来更新 `app/main.py` 以包含多个路由器，其中一些带有自定义配置：

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

在此示例中：
- `users.router` 和 `items.router` 使用它们自己的预定义设置被包含进来。
- `admin.router` 使用一组新的配置被包含进来：其所有路由都将以 `/admin` 为前缀，标记为 `"admin"`，受 `get_token_header` 依赖项保护，并在文档中有一个额外的 `418` 响应。

通过利用 `APIRouter`，你可以有效地构建大型应用程序，使你的代码保持有条理、可重用且易于管理。

现在你的应用程序结构良好，下一步是确保其可靠性。在[测试](./advanced-testing.md)部分了解如何为你的路径操作和依赖项编写测试。