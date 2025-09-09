# 路由

FastAPI 使用 `APIRouter` 来组织和结构化应用的端点。这对于将 API 拆分为多个文件和模块特别有用，这是构建更大、可维护应用的常见做法。每个单独的端点在内部由一个 `APIRoute` 表示。

本页面为这些类提供了详细的参考。有关如何使用路由器构建项目结构的分步指南，请参阅关于[更大型应用](./advanced-bigger-applications.md)的教程。

## `APIRouter`

`APIRouter` 类允许您对*路径操作*进行分组。您可以将其视为一个迷你的 `FastAPI` 应用。您可以在路由器上定义路由、依赖项和标签，然后将其包含在主应用中，甚至包含在另一个路由器中。

### 基本用法

以下是一个如何使用 `APIRouter` 的简单示例：

```python title="main.py" icon=logos:python
from fastapi import APIRouter, FastAPI

app = FastAPI()
router = APIRouter()


@router.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]


app.include_router(router)
```

### `APIRouter` 参数

当您创建 `APIRouter` 的实例时，可以传递几个参数来配置它包含的所有路由。

| Parameter | Type | Description |
|---|---|---|
| `prefix` | `str` | 此路由器中所有路由的可选路径前缀。必须以 `/` 开头。 |
| `tags` | `Optional[List[Union[str, Enum]]]` | 应用于此路由器中所有*路径操作*的标签列表。这些标签用于在 OpenAPI 文档中进行分组。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 应用于此路由器中所有*路径操作*的依赖项 (`Depends()`) 列表。 |
| `default_response_class` | `Type[Response]` | 要使用的默认响应类。默认为 `JSONResponse`。 |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | 应用于所有路由的额外响应，将显示在 OpenAPI 文档中。 |
| `callbacks` | `Optional[List[BaseRoute]]` | 应用于此路由器中所有*路径操作*的 OpenAPI 回调。 |
| `deprecated` | `Optional[bool]` | 如果为 `True`，则在 OpenAPI 文档中将此路由器中的所有*路径操作*标记为已弃用。 |
| `include_in_schema` | `bool` | 如果为 `False`，则从生成的 OpenAPI 模式中排除此路由器中的所有*路径操作*。默认为 `True`。 |
| `generate_unique_id_function` | `Callable[[APIRoute], str]` | 用于自定义*路径操作*唯一 ID 生成的函数。 |
| `route_class` | `Type[APIRoute]` | 用于此路由器中路由的自定义 `APIRoute` 类。 |
| `lifespan` | `Optional[Lifespan[Any]]` | 用于处理此路由器启动和关闭事件的 `Lifespan` 上下文管理器。这是推荐的事件处理方法。 |
| `on_startup` / `on_shutdown` | `Optional[Sequence[Callable[[], Any]]]` | **已弃用。** 启动或关闭事件处理程序的列表。请改用 `lifespan`。 |

### 路径操作装饰器

`APIRouter` 为所有标准 HTTP 方法提供装饰器，以添加新的*路径操作*。这些装饰器创建 `APIRoute` 实例，并接受下面 `APIRoute` 部分中记录的所有参数。

- `@router.get()`
- `@router.put()`
- `@router.post()`
- `@router.delete()`
- `@router.options()`
- `@router.head()`
- `@router.patch()`
- `@router.trace()`

```python icon=logos:python
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter()

class Item(BaseModel):
    name: str
    description: str | None = None

@router.post("/items/", status_code=201, tags=["items"], summary="Create an item")
async def create_item(item: Item):
    return {"message": "Item created successfully", "item": item}
```

### `websocket()` 装饰器

要添加 WebSocket 端点，请使用 `@router.websocket()` 装饰器。

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | WebSocket 路径。 |
| `name` | `Optional[str]` | WebSocket 路由的内部名称。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 此 WebSocket 的依赖项列表。 |

**示例**
```python icon=logos:python
from fastapi import APIRouter, WebSocket, WebSocketDisconnect

router = APIRouter()

@router.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f"Message text was: {data}")
    except WebSocketDisconnect:
        print("Client disconnected")
```

### `include_router()`

此方法用于将另一个 `APIRouter` 挂载到当前的路由器或 `FastAPI` 应用上。这是构建模块化应用的关键。

```python title="routers/items.py" icon=logos:python
# In routers/items.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/items/")
def get_items():
    return ["Portal Gun", "Plumbus"]
```

```python title="main.py" icon=logos:python
# In main.py
from fastapi import FastAPI
from .routers import items

app = FastAPI()

# Include the items router with a prefix and tags
app.include_router(items.router, prefix="/api/v1", tags=["items"])
```

**参数**

| Parameter | Type | Description |
|---|---|---|
| `router` | `APIRouter` | 要包含的 `APIRouter` 实例。 |
| `prefix` | `str` | 可选的路径前缀，将添加到所包含路由器的所有路由之前。 |
| `tags` | `Optional[List[Union[str, Enum]]]` | 添加到所包含路由器的所有路由的标签列表。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 添加到所包含路由器的所有路由的依赖项列表。 |
| `default_response_class` | `Type[Response]` | 所包含路由的默认响应类。 |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | 所包含路由的额外响应。 |
| `callbacks` | `Optional[List[BaseRoute]]` | 所包含路由的 OpenAPI 回调。 |
| `deprecated` | `Optional[bool]` | 将所包含路由器的所有路由标记为已弃用。 |
| `include_in_schema` | `bool` | 是否在 OpenAPI 模式中包含这些路由。默认为 `True`。 |
| `generate_unique_id_function` | `Callable[[APIRoute], str]` | 用于为所包含路由生成唯一 ID 的自定义函数。 |

---

## `APIRoute`

`APIRoute` 类表示单个*路径操作*。通常您不会直接创建 `APIRoute` 的实例。相反，它们由 `APIRouter` 的路径操作装饰器（例如 `@router.get(...)`）创建。此处列出的参数在所有这些装饰器中都可用。

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | 此操作的 URL 路径。 |
| `endpoint` | `Callable[..., Any]` | 将处理对此路径的请求的函数。 |
| `response_model` | `Any` | 用于响应的 Pydantic 模型。它有助于数据验证、序列化和文档生成。 |
| `status_code` | `Optional[int]` | 响应的默认 HTTP 状态码。 |
| `tags` | `Optional[List[Union[str, Enum]]]` | 用于 OpenAPI 文档分组的标签列表。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 此路径操作的依赖项序列。 |
| `summary` | `Optional[str]` | 路径操作的简短摘要，用于 OpenAPI。 |
| `description` | `Optional[str]` | 详细描述，支持 Markdown。如果未提供，则从函数的文档字符串中提取。 |
| `response_description` | `str` | 默认响应的描述。默认为“Successful Response”。 |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | 包含其他可能响应及其模型、描述等的字典。 |
| `deprecated` | `Optional[bool]` | 如果为 `True`，则在 OpenAPI 中将此路径操作标记为已弃用。 |
| `methods` | `Optional[Union[Set[str], List[str]]]` | 此路由的 HTTP 方法集合或列表（例如 `["GET", "POST"]`）。 |
| `operation_id` | `Optional[str]` | 用于在 OpenAPI 中标识操作的唯一字符串。对客户端生成很有用。 |
| `response_model_include` | `Optional[IncEx]` | 要包含在响应模型中的字段。 |
| `response_model_exclude` | `Optional[IncEx]` | 要从响应模型中排除的字段。 |
| `response_model_by_alias` | `bool` | 是否在响应中使用 Pydantic 模型别名。默认为 `True`。 |
| `response_model_exclude_unset` | `bool` | 是否从响应中排除未显式设置的字段。默认为 `False`。 |
| `response_model_exclude_defaults` | `bool` | 是否从响应中排除具有默认值的字段。默认为 `False`。 |
| `response_model_exclude_none` | `bool` | 是否从响应中排除值为 `None` 的字段。默认为 `False`。 |
| `include_in_schema` | `bool` | 是否在 OpenAPI 模式中包含此路径操作。默认为 `True`。 |
| `response_class` | `Union[Type[Response], DefaultPlaceholder]` | 要使用的响应类，例如 `JSONResponse`、`HTMLResponse`。 |
| `name` | `Optional[str]` | 路由的内部名称。 |
| `callbacks` | `Optional[List[BaseRoute]]` | OpenAPI 回调列表。 |
| `openapi_extra` | `Optional[Dict[str, Any]]]` | 要合并到操作模式中的额外 OpenAPI 信息的字典。 |

---

## `APIWebSocketRoute`

此类等同于用于 WebSocket 连接的 `APIRoute`。它由 `@router.websocket()` 装饰器创建。

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | WebSocket 连接的 URL 路径。 |
| `endpoint` | `Callable[..., Any]` | 将处理 WebSocket 会话的函数。 |
| `name` | `Optional[str]` | WebSocket 路由的内部名称。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 此 WebSocket 路由的依赖项序列。 |

在了解了如何构建路由之后，您可能希望更深入地了解如何定义参数。更多信息，请参阅[参数](./api-reference-parameters.md)参考。