# FastAPI 应用

`FastAPI` 类是创建和管理应用的主要入口点。它提供了定义路由、处理请求和配置 API 的核心功能。

本文档是 `FastAPI` 类及其配置参数、实例属性和方法的综合 API 参考。有关分步介绍，请参阅 [入门](./getting-started.md) 教程。

## 基本用法

首先，导入 `FastAPI` 并创建一个应用实例：

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}
```

## 类图

该图说明了 `FastAPI` 应用结构中的核心组件和关系。`FastAPI` 继承自 Starlette，主要由一个管理所有路由的 `APIRouter` 组成。

```d2
direction: down

"Starlette": { shape: class }
"APIRouter": { shape: class }

"FastAPI": {
  shape: class
  
  "router: APIRouter"
}

"FastAPI" -> "Starlette": "继承自"
"FastAPI"."router: APIRouter" -> "APIRouter": "由...组成"

"路径操作装饰器\n(@app.get, @app.post 等)": {
    shape: rectangle
}

"include_router()": {
    shape: rectangle
}

"路径操作装饰器\n(@app.get, @app.post 等)" -> "FastAPI"."router: APIRouter": "修改"
"include_router()" -> "FastAPI"."router: APIRouter": "修改"
```

## 参数

`FastAPI` 类的构造函数接受多个参数，用于配置应用的行为、元数据和文档。

| Parameter | Type | Description |
|---|---|---|
| `title` | `str` | API 的标题。默认值：`"FastAPI"`。 |
| `description` | `str` | API 的描述，支持 Markdown。默认值：`""`。 |
| `summary` | `Optional[str]` | API 的简短摘要。默认值：`None`。 |
| `version` | `str` | 应用的版本。默认值：`"0.1.0"`。 |
| `openapi_url` | `Optional[str]` | OpenAPI 模式的 URL。设置为 `None` 可禁用。默认值：`"/openapi.json"`。 |
| `docs_url` | `Optional[str]` | Swagger UI 文档的 URL。设置为 `None` 可禁用。默认值：`"/docs"`。 |
| `redoc_url` | `Optional[str]` | ReDoc 文档的 URL。设置为 `None` 可禁用。默认值：`"/redoc"`。 |
| `dependencies` | `Optional[Sequence[Depends]]` | 应用于所有路径操作的全局依赖项序列。 |
| `default_response_class` | `Type[Response]` | 要使用的默认响应类。默认值：`JSONResponse`。 |
| `exception_handlers` | `Optional[Dict]` | 异常处理程序的字典。 |
| `lifespan` | `Optional[Lifespan]` | 用于处理启动和关闭事件的生命周期上下文管理器。 |
| `openapi_tags` | `Optional[List[Dict]]` | 用于路径操作中标签的元数据。 |
| `servers` | `Optional[List[Dict]]` | OpenAPI 模式的服务器定义列表。 |
| `contact` | `Optional[Dict]` | API 的联系信息。 |
| `license_info` | `Optional[Dict]` | API 的许可证信息。 |
| `root_path` | `str` | 由代理处理的路径前缀。 |
| `...and others` | | 完整列表请参阅源代码。 |

### 元数据和文档配置

您可以为 API 配置元数据，这些元数据将用于 OpenAPI 模式和自动文档界面。

```python
from fastapi import FastAPI

tags_metadata = [
    {
        "name": "users",
        "description": "与用户相关的操作。",
    },
    {
        "name": "items",
        "description": "管理项目。",
    },
]

app = FastAPI(
    title="ChimichangApp",
    description="ChimichangApp API 帮助您完成出色的工作。🚀",
    version="2.5.0",
    terms_of_service="http://example.com/terms/",
    contact={
        "name": "Deadpoolio the Amazing",
        "url": "http://x-force.example.com/contact/",
        "email": "dp@x-force.example.com",
    },
    license_info={
        "name": "Apache 2.0",
        "url": "https://www.apache.org/licenses/LICENSE-2.0.html",
    },
    openapi_tags=tags_metadata
)

@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]
```

### 全局依赖项

您可以添加将应用于应用中所有*路径操作*的依赖项。

```python
from fastapi import Depends, FastAPI, Header, HTTPException

async def verify_token(x_token: str = Header()):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token 标头无效")

app = FastAPI(dependencies=[Depends(verify_token)])

@app.get("/items/")
async def read_items():
    return [{"item": "Portal Gun"}, {"item": "Plumbus"}]
```

## 实例属性

`FastAPI` 实例有几个您可以访问或修改的属性。

- `router` (`APIRouter`)：应用的主路由器。所有路径操作都在此注册。
- `dependency_overrides` (`Dict`)：用于覆盖依赖项的字典，主要用于测试。更多详情请参阅 [使用覆盖测试依赖项](./advanced-testing.md)。
- `state` (`State`)：一个用于存储任意应用状态的对象，继承自 Starlette。
- `openapi_schema` (`Optional[Dict]`)：缓存生成的 OpenAPI 模式。首次访问时，会生成并存储模式。
- `openapi_version` (`str`)：OpenAPI 版本字符串。默认为 `"3.1.0"`，但如果需要与旧工具兼容，可以修改。
- `webhooks` (`APIRouter`)：用于记录 OpenAPI Webhook 的 `APIRouter`。

## 方法

### 路径操作装饰器

FastAPI 使用装饰器将函数与特定的 URL 路径和 HTTP 方法关联起来。这些装饰器共享一组通用的配置参数。

- `@app.get(path, **kwargs)`
- `@app.post(path, **kwargs)`
- `@app.put(path, **kwargs)`
- `@app.delete(path, **kwargs)`
- `@app.patch(path, **kwargs)`
- `@app.options(path, **kwargs)`
- `@app.head(path, **kwargs)`
- `@app.trace(path, **kwargs)`

**通用参数**

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | 端点的 URL 路径。 |
| `response_model` | `Any` | 用于响应的 Pydantic 模型。 |
| `status_code` | `int` | 响应的默认 HTTP 状态码。 |
| `tags` | `List[str]` | 用于在 API 文档中分组的标签列表。 |
| `summary` | `str` | 端点的简短摘要。 |
| `description` | `str` | 详细描述，支持 Markdown。 |
| `dependencies` | `Sequence[Depends]` | 此端点特定的依赖项列表。 |
| `deprecated` | `bool` | 在文档中将端点标记为已弃用。 |

**示例：`@app.post()`**

```python
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.post("/items/", response_model=Item, status_code=201, tags=["items"])
async def create_item(item: Item):
    return item
```

### `include_router`

在应用中包含一个 `APIRouter`，这对于构建更大型的应用很有用。更多详情请参阅 [更大型的应用](./advanced-bigger-applications.md)。

```python
from fastapi import APIRouter, FastAPI

app = FastAPI()
router = APIRouter()

@router.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]

app.include_router(
    router,
    prefix="/api/v1",
    responses={404: {"description": "未找到"}}
)
```

### `@app.websocket`

装饰一个函数以处理 WebSocket 连接。

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f"消息文本为：{data}")
    except WebSocketDisconnect:
        print("客户端已断开连接")
```

### `@app.middleware`

向应用添加中间件。唯一支持的类型是 `"http"`。

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

### `@app.exception_handler`

注册一个函数来处理特定的异常类型。

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
        content={"message": f"糟糕！{exc.name} 出错了。"},
    )
```

## 后续步骤

现在您已经熟悉了主要的 `FastAPI` 应用类，您可能想了解如何使用路由器来构建应用。

<x-cards>
  <x-card data-title="路由" data-icon="lucide:milestone" data-href="/api-reference/routing">
    了解 APIRouter，将路径操作组织到不同的模块中。
  </x-card>
  <x-card data-title="更大型的应用" data-icon="lucide:layout-grid" data-href="/advanced/bigger-applications">
    探索构建大型、生产就绪型应用的策略。
  </x-card>
</x-cards>