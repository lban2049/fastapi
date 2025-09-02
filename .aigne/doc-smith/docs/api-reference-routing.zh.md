# 路由

FastAPI 使用强大的路由系统将 URL 路径与处理它们的特定函数相关联。这主要由两个关键类管理：`APIRouter` 和 `APIRoute`。理解这些组件对于构建应用程序至关重要，尤其是在应用程序变得复杂时。

- **`APIRouter`**：一个允许你对*路径操作*进行分组的类。它是将你的 API 拆分为多个模块化文件的标准工具。你可以在一个路由上定义路径操作，然后将该路由包含在你的主 `FastAPI` 应用程序中，甚至可以包含在另一个路由中。
- **`APIRoute`**：实际处理单个*路径操作*的类。当你使用像 `@app.get("/items")` 这样的装饰器时，你实际上是在后台创建了一个 `APIRoute` 的实例。

有关构建大型项目的分步指南，请参阅[更大型的应用](./advanced-bigger-applications.md)用户指南。

```d2
direction: down

"FastAPI 应用" {
  shape: cloud
  "app = FastAPI()"
}

"用户路由 (APIRouter)" {
  shape: package
  "router = APIRouter(prefix='/users')"

  "GET /: APIRoute"
  "POST /: APIRoute"
}

"物品路由 (APIRouter)" {
  shape: package
  "router = APIRouter(prefix='/items')"

  "GET /{item_id}: APIRoute"
}

"FastAPI 应用" -> "用户路由 (APIRouter)": "app.include_router(users_router)"
"FastAPI 应用" -> "物品路由 (APIRouter)": "app.include_router(items_router)"
```

## APIRouter

`APIRouter` 类用于创建一组模块化的路径操作，这些操作可以在之后被包含到主 `FastAPI` 应用程序中。这是为了代码可维护性而推荐的结构化方式。

### 创建 `APIRouter`

你可以使用多个参数来实例化 `APIRouter`，以配置其包含的所有路径操作。

```python
from fastapi import APIRouter, FastAPI

app = FastAPI()
router = APIRouter(
    prefix="/items",
    tags=["items"],
    dependencies=[Depends(get_token_header)],
    responses={404: {"description": "Not found"}},
)


@router.get("/")
async def read_items():
    return [{"name": "Item Foo"}, {"name": "Item Bar"}]


app.include_router(router)
```

### `APIRouter` 参数

创建 `APIRouter` 实例时，可以提供以下参数：

| Parameter | Type | Description |
|---|---|---|
| `prefix` | `str` | 此路由中所有路由的可选路径前缀。必须以 `/` 开头。 |
| `tags` | `Optional[List[Union[str, Enum]]]` | 应用于此路由中所有路径操作的标签列表，用于 OpenAPI 文档。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 应用于此路由中所有路径操作的依赖项序列。 |
| `default_response_class` | `Type[Response]` | 用于路径操作的默认响应类。默认为 `JSONResponse`。 |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | 要包含在所有路径操作的 OpenAPI schema 中的额外响应。 |
| `callbacks` | `Optional[List[BaseRoute]]` | 应用于此路由中所有路径操作的 OpenAPI 回调列表。 |
| `deprecated` | `Optional[bool]` | 如果为 `True`，则在 OpenAPI schema 中将此路由中的所有路径操作标记为已弃用。 |
| `include_in_schema` | `bool` | 如果为 `False`，则从生成的 OpenAPI schema 中排除此路由中的所有路径操作。默认为 `True`。 |
| `generate_unique_id_function` | `Callable[[APIRoute], str]` | 用于自定义路径操作唯一 ID 生成的函数。 |
| `route_class` | `Type[APIRoute]` | 用于此路由中路径操作的自定义 `APIRoute` 类。 |

### 包含路由

在主 `FastAPI` 应用中，用于处理路由的最重要方法是 `include_router`。它会将路由中的所有路径操作挂载到应用程序中。

```python
from fastapi import APIRouter, FastAPI

app = FastAPI()
users_router = APIRouter()

@users_router.get("/users/")
def read_users():
    return [{"name": "Rick"}, {"name": "Morty"}]

# You include the router in your main app
app.include_router(users_router)
```

#### `include_router` 参数

你还可以向 `include_router` 提供参数，以便为包含的路由添加进一步的配置。

| Parameter | Type | Description |
|---|---|---|
| `router` | `APIRouter` | 要包含的 `APIRouter` 实例。 |
| `prefix` | `str` | 一个可选的路径前缀，将加在所有来自被包含路由的路由之前。 |
| `tags` | `Optional[List[Union[str, Enum]]]` | 要添加到来自被包含路由的所有路径操作的标签列表。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 要添加到来自被包含路由的所有路径操作的依赖项序列。 |
| `default_response_class` | `Type[Response]` | 被包含路由的默认响应类。 |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | 被包含路由的额外响应。 |
| `deprecated` | `Optional[bool]` | 将来自被包含路由的所有路径操作标记为已弃用。 |
| `include_in_schema` | `bool` | 一个布尔值，指示被包含的路由是否应包含在 OpenAPI schema 中。 |

### 路径操作装饰器

`APIRouter` 为所有标准 HTTP 动词（`.get()`、`.post()`、`.put()`、`.patch()`、`.delete()` 等）提供了装饰器方法，用于添加路径操作。这些装饰器接受众多参数来配置路由、其验证、文档和响应。

以下是所有路径操作装饰器可用的通用参数。

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | 此操作的 URL 路径。 |
| `response_model` | `Any` | 用于响应的 Pydantic 模型，可用于验证和序列化。 |
| `status_code` | `Optional[int]` | 响应的默认 HTTP 状态码。 |
| `tags` | `Optional[List[Union[str, Enum]]]` | 用于在 OpenAPI 文档中对此操作进行分组的标签列表。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 此路径操作的依赖项列表。 |
| `summary` | `Optional[str]` | 在 OpenAPI 中对此操作的简短摘要。 |
| `description` | `Optional[str]` | 详细描述，支持 Markdown。如果未提供，则从函数的文档字符串中提取。 |
| `response_description` | `str` | 默认响应的描述。默认为“Successful Response”。 |
| `responses` | `Optional[Dict[...]]` | 包含其他可能响应及其模型、描述等的字典。 |
| `deprecated` | `Optional[bool]` | 如果为 `True`，则在 OpenAPI 中将此操作标记为已弃用。 |
| `operation_id` | `Optional[str]` | 用于标识操作的唯一字符串。对客户端生成很有用。 |
| `response_model_include` | `Optional[IncEx]` | 响应模型中要包含的字段。 |
| `response_model_exclude` | `Optional[IncEx]` | 响应模型中要排除的字段。 |
| `response_model_by_alias` | `bool` | 是否按别名序列化响应模型。默认为 `True`。 |
| `response_model_exclude_unset` | `bool` | 是否从响应中排除未明确设置的字段。默认为 `False`。 |
| `response_model_exclude_defaults`| `bool` | 是否从响应中排除具有默认值的字段。默认为 `False`。 |
| `response_model_exclude_none` | `bool` | 是否从响应中排除值为 `None` 的字段。默认为 `False`。 |
| `include_in_schema` | `bool` | 是否在 OpenAPI schema 中包含此路径操作。默认为 `True`。 |
| `response_class` | `Type[Response]` | 用于此操作响应的自定义 `Response` 类。 |
| `name` | `Optional[str]` | 路由的名称，用于 URL 路径生成。仅在内部使用。 |
| `callbacks` | `Optional[List[BaseRoute]]` | 此操作的 OpenAPI 回调列表。 |
| `openapi_extra` | `Optional[Dict[str, Any]]` | 要包含在此操作的 OpenAPI schema 中的额外元数据字典。 |

### WebSocket 装饰器

对于实时通信，`APIRouter` 提供了一个 `websocket` 装饰器。

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

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | WebSocket 路径。 |
| `name` | `Optional[str]` | WebSocket 路由的内部名称。 |
| `dependencies` | `Optional[Sequence[params.Depends]]`| 此 WebSocket 的依赖项列表。 |

---

## APIRoute

`APIRoute` 表示单个 API 端点（一个*路径操作*）。虽然通常使用 `FastAPI` 或 `APIRouter` 对象上的装饰器来创建路由，但你也可以直接实例化此类或对其进行子类化以进行高级自定义。

### `APIRoute` 参数

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | 此路由的 URL 路径。 |
| `endpoint` | `Callable[..., Any]` | 处理对此路径请求的函数。 |
| `response_model` | `Any` | 用于响应的 Pydantic 模型。 |
| `status_code` | `Optional[int]` | 默认 HTTP 状态码。 |
| `tags` | `Optional[List[...]]` | 用于 OpenAPI 文档的标签列表。 |
| `dependencies` | `Optional[Sequence[...]]` | 此路由的依赖项序列。 |
| `summary` | `Optional[str]` | 操作的简短摘要。 |
| `description` | `Optional[str]` | 详细描述。 |
| `response_description` | `str` | 成功响应的描述。 |
| `responses` | `Optional[Dict[...]]` | 其他响应的字典。 |
| `deprecated` | `Optional[bool]` | 将路由标记为已弃用。 |
| `name` | `Optional[str]` | 路由的内部名称。 |
| `methods` | `Optional[Union[Set[str], List[str]]]` | 此路由的 HTTP 方法集或列表（例如 `["GET", "POST"]`）。 |
| `operation_id` | `Optional[str]` | 操作的唯一 ID。 |
| `response_model_include` | `Optional[IncEx]` | 响应中要包含的字段。 |
| `response_model_exclude` | `Optional[IncEx]` | 响应中要排除的字段。 |
| `response_model_by_alias` | `bool` | 是否按别名序列化响应。 |
| `response_model_exclude_unset`| `bool` | 是否从响应中排除未设置的字段。 |
| `response_model_exclude_defaults`| `bool` | 是否从响应中排除具有默认值的字段。 |
| `response_model_exclude_none` | `bool` | 是否从响应中排除设置为 `None` 的字段。 |
| `include_in_schema` | `bool` | 是否在 OpenAPI schema 中包含此路由。 |
| `response_class` | `Union[Type[Response], ...]` | 要使用的响应类。 |
| `dependency_overrides_provider`| `Optional[Any]` | 用于内部处理依赖项覆盖。 |
| `callbacks` | `Optional[List[BaseRoute]]` | OpenAPI 回调列表。 |
| `openapi_extra` | `Optional[Dict[str, Any]]` | 额外的 OpenAPI 元数据。 |
| `generate_unique_id_function` | `Callable[[APIRoute], str]` | 用于生成唯一 ID 的函数。 |

---

## APIWebSocketRoute

此类等效于用于 WebSocket 连接的 `APIRoute`。当你使用 `@router.websocket()` 装饰器时会创建它。

### `APIWebSocketRoute` 参数

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | WebSocket 的 URL 路径。 |
| `endpoint` | `Callable[..., Any]` | 处理 WebSocket 连接的函数。 |
| `name` | `Optional[str]` | 路由的内部名称。 |
| `dependencies` | `Optional[Sequence[params.Depends]]` | 此 WebSocket 路由的依赖项序列。 |
| `dependency_overrides_provider`| `Optional[Any]` | 用于内部处理依赖项覆盖。 |
