# 更大型的应用 - 多文件

随着应用的增长，将其拆分成更小、更易于管理的文件会很有帮助。一种常见的做法是将相关的 API 端点分组到各自的模块中。FastAPI 为此提供了一个强大而直观的工具：`APIRouter`。

`APIRouter` 的工作方式类似于一个迷你的 `FastAPI` 应用。你可以在其上定义*路径操作*、依赖项和其他配置，然后将其包含到主应用中。本指南将通过一个实际示例，引导你了解如何构建一个更大型的应用。

## 主应用文件

我们从主文件开始，它将作为我们应用的入口点。该文件负责创建 `FastAPI` 主实例，并包含来自其他模块的所有路由。

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

这里的关键函数是 `app.include_router()`。它接收一个路由器对象，并将其所有路径整合到主应用中。你还可以看到，在 `FastAPI` 的构造函数中，一些依赖项被应用于整个应用。

## 一个简单的路由器

现在，我们为用户相关的端点创建一个单独的模块。这有助于我们按功能组织代码。

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

在这个文件中，我们创建了一个 `APIRouter` 实例，并使用其装饰器 (`@router.get`) 来定义*路径操作*，就像使用 `FastAPI` 应用实例一样。然后，这个路由器被导入并包含在我们的 `main.py` 文件中。

## 带配置的路由器

`APIRouter` 可以使用参数进行初始化，这些参数将应用于其所有的*路径操作*。这是一种为一组端点设置通用路径前缀、添加标签、指定依赖项和定义通用响应的便捷方法。

这是一个带有自定义配置的物品路由器：

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

因为我们在路由器上设置了 `prefix="/items"`，所以 `read_item` 函数的路径将是 `/items/{item_id}`。此路由器中的所有端点都将带有 "items" 标签，依赖于 `get_token_header`，并会记录一个默认的 404 响应。

请注意，你仍然可以为单个路径操作添加额外的标签或响应，就像在 `update_item` 中一样。这些将与路由器的配置相结合。

## 包含带附加配置的路由器

你也可以直接在 `app.include_router()` 中提供配置参数。这些设置将应用于所包含的路由器中的每个*路径操作*，并附加在路由器本身已定义的任何配置之上。

这对于将一组端点挂载到特定路径（如 `/admin`）下，或对整个模块应用更高级别的身份验证特别有用。

首先，我们来看一个简单的管理路由器：

```python app/internal/admin.py icon=logos:python
from fastapi import APIRouter

router = APIRouter()


@router.post("/")
async def update_admin():
    return {"message": "Admin getting schwifty"}
```

现在，看看它如何在 `main.py` 中通过额外的参数被包含进来：

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

尽管 `admin.py` 中的路径是 `/`，但 `include_router` 中的 `prefix` 确保了 `update_admin` 的最终路径是 `/admin/`。同样，这里定义的标签、依赖项和响应将应用于 `admin.router` 中的所有端点。

## 总结

通过使用 `APIRouter`，你可以有效地将 FastAPI 应用构建成多个文件和模块。随着项目的扩展，这种方法可以增强代码的组织性、可重用性和可维护性。你可以在路由器定义或包含点应用通用配置，从而灵活地控制 API 结构。

现在你已经学会了如何构建应用，接下来我们来探讨如何为请求和响应添加自定义处理。请继续阅读[中间件](./tutorials-advanced-middleware.md)指南。