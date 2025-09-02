# FastAPI 应用

`FastAPI` 类是应用程序的主要入口点。它为 API 提供了所有功能，继承自 `starlette.applications.Starlette`，但增加了自动文档、数据验证和依赖注入等功能。

## 基本用法

创建 `FastAPI` 实例是构建 API 的第一步。以下是一个简单的示例：

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}
```

这将创建一个应用程序实例 `app`，并为根 URL `/` 定义一个单一的路径操作。

## 应用配置

`FastAPI` 应用程序可以在初始化时通过多个参数进行配置，以自定义其行为、元数据和文档。

```d2
direction: down

"FastAPI 应用": {
  shape: cloud
  "配置": {
    "元数据": "标题、描述、版本等"
    "API 文档": "docs_url、redoc_url、openapi_url"
    "行为": "依赖项、中间件、生命周期"
    "路由": "路由、重定向斜杠"
  }

  "核心组件": {
    "路由器": {
      "路径操作": "@app.get()、@app.post() 等"
      "包含的路由器": "app.include_router()"
    }
    "中间件栈": "处理请求/响应"
    "依赖注入": "管理依赖项"
    "异常处理器": "优雅地处理错误"
  }
  "配置" -> "核心组件": "初始化"
}
```

### 初始化参数

以下是创建 `FastAPI` 实例时可用的参数的完整列表：

| Parameter | Type | Default | Description |
|---|---|---|---|
| `debug` | `bool` | `False` | 启用调试模式。如果为 `True`，响应中会返回错误追溯信息。 |
| `routes` | `Optional[List[BaseRoute]]` | `None` | 一个 Starlette 路由列表。建议使用路径操作装饰器（如 `@app.get()`）代替。 |
| `title` | `str` | `"FastAPI"` | API 的标题，会显示在 OpenAPI 模式和文档 UI 中。 |
| `summary` | `Optional[str]` | `None` | API 的简短摘要。 |
| `description` | `str` | `""` | API 的详细描述。支持 Markdown。 |
| `version` | `str` | `"0.1.0"` | 应用程序 API 的版本（例如，“1.2.0” 或 “v2-beta”）。 |
| `openapi_url` | `Optional[str]` | `"/openapi.json"` | OpenAPI 模式的 URL 路径。设置为 `None` 可禁用它和文档 UI。 |
| `openapi_tags` | `Optional[List[Dict[str, Any]]]` | `None` | 一个用于定义和排序文档中路径操作所用标签的字典列表。 |
| `servers` | `Optional[List[Dict]]` | `None` | 一个 OpenAPI 模式的服务器定义列表，用于指定不同环境（如预发布环境、生产环境）。 |
| `dependencies` | `Optional[Sequence[Depends]]` | `None` | 一个全局依赖项列表，将应用于应用程序中的所有路径操作。 |
| `default_response_class` | `Type[Response]` | `JSONResponse` | 用于路径操作的默认响应类。 |
| `redirect_slashes` | `bool` | `True` | 如果路径访问时没有尾部斜杠，是否自动重定向请求。 |
| `docs_url` | `Optional[str]` | `"/docs"` | 交互式 Swagger UI 文档的 URL 路径。设置为 `None` 可禁用。 |
| `redoc_url` | `Optional[str]` | `"/redoc"` | ReDoc 文档的 URL 路径。设置为 `None` 可禁用。 |
| `swagger_ui_oauth2_redirect_url` | `Optional[str]` | `"/docs/oauth2-redirect"` | Swagger UI 的 OAuth2 重定向 URL。 |
| `swagger_ui_init_oauth` | `Optional[Dict]` | `None` | 一个用于在 Swagger UI 中配置 OAuth2 的字典。 |
| `swagger_ui_parameters`| `Optional[Dict]` | `None` | 一个用于自定义 Swagger UI 的参数字典。 |
| `middleware` | `Optional[Sequence[Middleware]]` | `None` | 一个要添加到应用程序的 Starlette 中间件序列。更常见的做法是使用 `app.add_middleware()`。 |
| `exception_handlers` | `Optional[Dict]` | `None` | 一个异常处理器字典。首选使用 `@app.exception_handler()` 装饰器。 |
| `on_startup` | `Optional[Sequence[Callable]]` | `None` | （已弃用）一个在应用程序启动时运行的函数列表。请改用 `lifespan`。 |
| `on_shutdown` | `Optional[Sequence[Callable]]` | `None` | （已弃用）一个在应用程序关闭时运行的函数列表。请改用 `lifespan`。 |
| `lifespan` | `Optional[Lifespan]` | `None` | 一个用于处理启动和关闭事件的上下文管理器。这是推荐的方法。 |
| `terms_of_service` | `Optional[str]` | `None` | API 服务条款的 URL。 |
| `contact` | `Optional[Dict]` | `None` | 一个包含 API 联系信息的字典（例如，`name`、`url`、`email`）。 |
| `license_info` | `Optional[Dict]` | `None` | 一个包含 API 许可证信息的字典（例如，`name`、`url`）。 |
| `root_path` | `str` | `""` | 应用程序的路径前缀，在反向代理后使用时很有用。 |
| `root_path_in_servers` | `bool` | `True` | 如果为 `True`，`root_path` 会自动添加到 OpenAPI 模式的 `servers` 列表中。 |
| `responses` | `Optional[Dict]` | `None` | 要包含在 OpenAPI 模式中所有路径操作里的额外全局响应。 |
| `callbacks` | `Optional[List[BaseRoute]]` | `None` | 一个应用于所有路径操作的 OpenAPI 回调列表。 |
| `webhooks` | `Optional[APIRouter]` | `None` | 一个用于声明 OpenAPI Webhook 的 `APIRouter` 实例。 |
| `deprecated` | `Optional[bool]` | `None` | 如果为 `True`，则将应用程序中的所有路径操作标记为已弃用。 |
| `include_in_schema` | `bool` | `True` | 默认情况下是否在 OpenAPI 模式中包含所有路径操作。 |
| `generate_unique_id_function` | `Callable` | `generate_unique_id` | 一个为 OpenAPI 模式中每个路径操作生成唯一 ID 的函数。 |
| `separate_input_output_schemas` | `bool` | `True` | 是否为输入（请求）和输出（响应）模型生成单独的模式。 |

## 核心方法

### 路径操作装饰器

FastAPI 使用装饰器来定义 API 端点。这些装饰器对应 HTTP 方法，是向应用程序添加路由的主要方式。

- `@app.get()`
- `@app.post()`
- `@app.put()`
- `@app.delete()`
- `@app.patch()`
- `@app.options()`
- `@app.head()`
- `@app.trace()`

**示例：**

```python
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

app = FastAPI()

@app.post("/items/")
def create_item(item: Item):
    return {"message": f"Item '{item.name}' created successfully."}
```

### 使用路由器构建应用程序

`include_router` 方法允许你通过将应用程序拆分为多个 `APIRouter` 实例来构建应用程序，这对于大型应用程序至关重要。

```python
from fastapi import FastAPI, APIRouter

app = FastAPI()

router = APIRouter()

@router.get("/users/")
def read_users():
    return [{"username": "user1"}, {"username": "user2"}]

app.include_router(router, prefix="/api/v1", tags=["users"])
```
更多详情，请参阅关于[更大型应用](./advanced-bigger-applications.md)的文档。

### WebSocket

FastAPI 通过 `@app.websocket()` 装饰器为 WebSocket 提供了一流的支持。

```python
from fastapi import FastAPI, WebSocket

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```
有关更高级的用例，请参阅 [WebSocket](./advanced-websockets.md) 指南。

### 中间件

你可以向应用程序添加中间件，以在每个请求到达路径操作之前以及每个响应发送给客户端之前对其进行处理。

```python
import time
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```
在[高级中间件](./advanced-middleware.md)部分了解有关中间件的更多信息。

### 异常处理器

自定义异常处理器允许你定义应用程序如何响应特定异常。

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name

app = FastAPI()

@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something wrong."},
    )
```

### 生命周期事件

`lifespan` 上下文管理器是处理需要在应用程序启动前（例如，初始化数据库连接池）和关闭时运行的逻辑的推荐方法。

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

db_connections = {}

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Code to run on startup
    print("Connecting to database...")
    db_connections["main"] = {"status": "connected"}
    yield
    # Code to run on shutdown
    print("Closing database connection...")
    db_connections.clear()

app = FastAPI(lifespan=lifespan)
```

---

本参考提供了 `FastAPI` 应用程序类的全面概述。有关构建路由的更多详细信息，请继续阅读 [路由](./api-reference-routing.md) API 参考。