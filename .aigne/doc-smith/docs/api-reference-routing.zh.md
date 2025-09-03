# 路由

FastAPI 使用 `APIRouter` 来组织和结构化应用的端点。这对于将 API 拆分到多个文件和模块中特别有用，也是构建更大型、可维护应用的常见实践。每个单独的端点在内部由一个 `APIRoute` 表示。

本页为这些类提供了详细的参考资料。关于如何使用路由器来组织项目结构的分步指南，请参阅 [更大型的应用](./advanced-bigger-applications.md) 教程。

## `APIRouter`

`APIRouter` 类允许你对*路径操作*进行分组。你可以将其视为一个迷你的 `FastAPI` 应用。你可以在路由器上定义路由、依赖项和标签，然后将其包含在主应用甚至另一个路由器中。

### 基本用法

以下是一个如何使用 `APIRouter` 的简单示例：

```python
from fastapi import APIRouter, FastAPI

app = FastAPI()
router = APIRouter()


@router.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]


app.include_router(router)
```

### `APIRouter` 参数

创建 `APIRouter` 实例时，可以传递几个参数来配置其包含的所有路由。

| Parameter | Type | Description |
|---|---|---|
| `prefix` | `str` | 此路由器中所有路由的可选路径前缀。它必须以 `/` 开头。 |
| `tags` | `Optional[List[Union[str, Enum]]]` | 应用于此路由器中所有*路径操作*的标签列表。这些标签用于在 OpenAPI 文档中进行分组。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 应用于此路由器中所有*路径操作*的依赖项（`Depends()`）列表。 |
| `default_response_class` | `Type[Response]` | 要使用的默认响应类。默认为 `JSONResponse`。 |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | 应用于所有路由的额外响应，将显示在 OpenAPI 文档中。 |
| `callbacks` | `Optional[List[BaseRoute]]` | 应用于此路由器中所有*路径操作*的 OpenAPI 回调。 |
| `deprecated` | `Optional[bool]` | 如果为 `True`，则在 OpenAPI 文档中将此路由器中的所有*路径操作*标记为已弃用。 |
| `include_in_schema` | `bool` | 如果为 `False`，则从生成的 OpenAPI 架构中排除此路由器中的所有*路径操作*。默认为 `True`。 |
| `generate_unique_id_function` | `Callable[[APIRoute], str]` | 用于自定义生成*路径操作*唯一 ID 的函数。 |
| `route_class` | `Type[APIRoute]` | 用于此路由器中路由的自定义 `APIRoute` 类。 |
| `lifespan` | `Optional[Lifespan[Any]]` | 用于处理此路由器启动和关闭事件的 `Lifespan` 上下文管理器。 |

### 路径操作装饰器

`APIRouter` 为所有标准 HTTP 方法提供了添加新*路径操作*的装饰器。

- `@router.get()`
- `@router.put()`
- `@router.post()`
- `@router.delete()`
- `@router.options()`
- `@router.head()`
- `@router.patch()`
- `@router.trace()`

所有这些装饰器都接受与 `APIRoute` 类相同的参数，具体将在下文详述。

```python
from fastapi import APIRouter

router = APIRouter()

@router.post("/items/", status_code=201, tags=["items"])
async def create_item(item: dict):
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
```python
from fastapi import APIRouter, WebSocket

router = APIRouter()

@router.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```

### `include_router()`

此方法用于将另一个 `APIRouter` 挂载到当前路由器或 `FastAPI` 应用上。这是构建模块化应用的关键。

```python
from fastapi import FastAPI

# In items.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/items/")
def get_items():
    return ["Portal Gun", "Plumbus"]

# In main.py
app = FastAPI()
app.include_router(router, prefix="/api/v1", tags=["items"])
```

**参数**

| Parameter | Type | Description |
|---|---|---|
| `router` | `APIRouter` | 要包含的 `APIRouter` 实例。 |
| `prefix` | `str` | 一个可选的路径前缀，将添加到所包含路由器的所有路由之前。 |
| `tags` | `Optional[List[Union[str, Enum]]]` | 要添加到所包含路由器的所有路由的标签列表。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 要添加到所包含路由器的所有路由的依赖项列表。 |
| `default_response_class` | `Type[Response]` | 所包含路由的默认响应类。 |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | 所包含路由的额外响应。 |
| `callbacks` | `Optional[List[BaseRoute]]` | 所包含路由的 OpenAPI 回调。 |
| `deprecated` | `Optional[bool]` | 将所包含路由器的所有路由标记为已弃用。 |
| `include_in_schema` | `bool` | 是否在 OpenAPI 架构中包含这些路由。默认为 `True`。 |
| `generate_unique_id_function` | `Callable[[APIRoute], str]` | 用于为所包含路由生成唯一 ID 的自定义函数。 |

---

## `APIRoute`

`APIRoute` 类代表单个*路径操作*。通常情况下，你不会直接创建 `APIRoute` 的实例。它们由 `APIRouter` 的路径操作装饰器（例如 `@router.get(...)`）创建。此处列出的参数在所有这些装饰器中都可用。

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | 此操作的 URL 路径。 |
| `endpoint` | `Callable[..., Any]` | 将处理对此路径请求的函数。 |
| `response_model` | `Any` | 用于响应的 Pydantic 模型。它有助于数据验证、序列化和文档生成。 |
| `status_code` | `Optional[int]` | 响应的默认 HTTP 状态码。 |
| `tags` | `Optional[List[Union[str, Enum]]]` | 用于 OpenAPI 文档分组的标签列表。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 此路径操作的依赖项序列。 |
| `summary` | `Optional[str]` | 路径操作的简短摘要，用于 OpenAPI。 |
| `description` | `Optional[str]` | 详细描述，支持 Markdown。如果未提供，则从函数的文档字符串中提取。 |
| `response_description` | `str` | 默认响应的描述。默认为“Successful Response”。 |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | 包含其他可能响应及其模型、描述等的字典。 |
| `deprecated` | `Optional[bool]` | 如果为 `True`，则在 OpenAPI 中将路径操作标记为已弃用。 |
| `methods` | `Optional[Union[Set[str], List[str]]]` | 此路由的 HTTP 方法集或列表（例如 `["GET", "POST"]`）。 |
| `operation_id` | `Optional[str]` | 用于在 OpenAPI 中标识操作的唯一字符串。对客户端生成很有用。 |
| `response_model_include` | `Optional[IncEx]` | 要包含在响应模型中的字段。 |
| `response_model_exclude` | `Optional[IncEx]` | 要从响应模型中排除的字段。 |
| `response_model_by_alias` | `bool` | 是否在响应中使用 Pydantic 模型别名。默认为 `True`。 |
| `response_model_exclude_unset` | `bool` | 是否从响应中排除未明确设置的字段。默认为 `False`。 |
| `response_model_exclude_defaults` | `bool` | 是否从响应中排除具有默认值的字段。默认为 `False`。 |
| `response_model_exclude_none` | `bool` | 是否从响应中排除值为 `None` 的字段。默认为 `False`。 |
| `include_in_schema` | `bool` | 是否在 OpenAPI 架构中包含此路径操作。默认为 `True`。 |
| `response_class` | `Union[Type[Response], DefaultPlaceholder]` | 要使用的响应类，例如 `JSONResponse`、`HTMLResponse`。 |
| `name` | `Optional[str]` | 路由的内部名称。 |
| `callbacks` | `Optional[List[BaseRoute]]` | OpenAPI 回调列表。 |
| `openapi_extra` | `Optional[Dict[str, Any]]]` | 要合并到操作架构中的额外 OpenAPI 信息字典。 |

---

## `APIWebSocketRoute`

此类等同于用于 WebSocket 连接的 `APIRoute`。它由 `@router.websocket()` 装饰器创建。

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | WebSocket 连接的 URL 路径。 |
| `endpoint` | `Callable[..., Any]` | 将处理 WebSocket 会话的函数。 |
| `name` | `Optional[str]` | WebSocket 路由的内部名称。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 此 WebSocket 路由的依赖项序列。 |

了解如何组织路由后，你可能希望深入研究如何定义参数。更多信息，请参阅 [参数](./api-reference-parameters.md) 参考。