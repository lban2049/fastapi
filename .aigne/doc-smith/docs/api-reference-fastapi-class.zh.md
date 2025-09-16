# FastAPI 类

`FastAPI` 类是创建和管理 API 的主要入口点。它继承自 `starlette.applications.Starlette`，为你的 API 提供了所有功能。

创建 `FastAPI` 类的实例时，你可以指定各种配置选项来自定义应用程序的行为、元数据、文档等。

### 第一步示例

创建一个 `FastAPI` 应用程序非常简单，只需导入该类并创建一个实例即可：

```python First Steps icon=logos:python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}
```

有关入门的完整指南，请查看[教程 - 第一步](./tutorials-first-steps.md)。

## 参数

`FastAPI` 类的构造函数接受多个参数来配置你的应用程序。以下是最重要的一些参数：

<x-field data-name="title" data-type="str" data-default="FastAPI" data-required="false" data-desc="API 的标题。它会显示在自动生成的 API 文档中（例如，位于 /docs 的 Swagger UI）。"></x-field>

```python Title Example icon=logos:python
from fastapi import FastAPI

app = FastAPI(title="ChimichangApp")
```

<x-field data-name="version" data-type="str" data-default="0.1.0" data-required="false" data-desc="API 的版本。这是你应用程序的版本，而不是 OpenAPI 规范的版本。它也会显示在 API 文档中。"></x-field>

```python Version Example icon=logos:python
from fastapi import FastAPI

app = FastAPI(version="0.0.1")
```

<x-field data-name="description" data-type="str" data-default="" data-required="false" data-desc="API 的描述。它支持 Markdown，并会显示在 API 文档中。"></x-field>

```python Description Example icon=logos:python
from fastapi import FastAPI

app = FastAPI(
    description="""
    ChimichangApp API helps you do awesome stuff. 🚀

    ## Items

    You can **read items**.

    ## Users

    You will be able to:

    * **Create users** (_not implemented_).
    * **Read users** (_not implemented_).

    """
)
```

<x-field data-name="dependencies" data-type="Optional[Sequence[Depends]]" data-default="None" data-required="false" data-desc="一个全局依赖项列表，将应用于应用程序中的每个路径操作，包括子路由器中的路径操作。"></x-field>

```python Global Dependencies Example icon=logos:python
from fastapi import Depends, FastAPI
from .dependencies import func_dep_1, func_dep_2

app = FastAPI(dependencies=[Depends(func_dep_1), Depends(func_dep_2)])
```

<x-field data-name="debug" data-type="bool" data-default="False" data-required="false" data-desc="一个布尔值，指示在服务器错误时是否应返回调试跟踪信息。"></x-field>

<x-field data-name="openapi_url" data-type="Optional[str]" data-default="/openapi.json" data-required="false" data-desc="提供 OpenAPI 模式的 URL 路径。设置为 `None` 可禁用 OpenAPI 模式和自动文档 UI。"></x-field>

<x-field data-name="docs_url" data-type="Optional[str]" data-default="/docs" data-required="false" data-desc="交互式 Swagger UI 文档的 URL 路径。设置为 `None` 可禁用它。"></x-field>

<x-field data-name="redoc_url" data-type="Optional[str]" data-default="/redoc" data-required="false" data-desc="备选 ReDoc 文档的 URL 路径。设置为 `None` 可禁用它。"></x-field>

<x-field data-name="default_response_class" data-type="Type[Response]" data-default="JSONResponse" data-required="false" data-desc="用于路径操作的默认响应类。例如，你可以使用它来设置不同的默认 JSON 渲染器。"></x-field>

```python Default Response Class Example icon=logos:python
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse

app = FastAPI(default_response_class=ORJSONResponse)
```

<x-field data-name="lifespan" data-type="Optional[Lifespan[AppType]]" data-default="None" data-required="false" data-desc="一个生命周期上下文管理器，用于处理启动和关闭事件。这是管理应用程序启动和关闭逻辑的推荐方法。"></x-field>

<x-field data-name="root_path" data-type="str" data-default="" data-required="false" data-desc="由代理处理的路径前缀，应用程序看不到它，但在文档中生成正确的 URL 时是必需的。"></x-field>

<x-field data-name="openapi_tags" data-type="Optional[List[Dict[str, Any]]]" data-default="None" data-required="false" data-desc="一个字典列表，为路径操作中使用的标签提供元数据。这可用于控制 API 文档中标签的顺序和描述。"></x-field>

<x-field data-name="servers" data-type="Optional[List[Dict[str, Union[str, Any]]]]" data-default="None" data-required="false" data-desc="OpenAPI 模式的服务器定义列表，当你的 API 在多个 URL 上可用时非常有用。"></x-field>

<x-field data-name="exception_handlers" data-type="Optional[Dict[...]]" data-default="None" data-required="false" data-desc="异常处理程序的字典。更常见的做法是使用 `@app.exception_handler()` 装饰器。"></x-field>

<x-field data-name="middleware" data-type="Optional[Sequence[Middleware]]" data-default="None" data-required="false" data-desc="添加到应用程序的 Starlette 中间件列表。更常见的做法是使用 `app.add_middleware()` 或 `@app.middleware('http')` 装饰器。"></x-field>

## 属性

`FastAPI` 类的实例有几个有用的属性：

<x-field data-name="router" data-type="APIRouter" data-desc="应用程序的主路由器。所有路径操作都在此路由器上注册。"></x-field>
<x-field data-name="dependency_overrides" data-type="Dict[Callable, Callable]" data-desc="一个用于覆盖依赖项的字典，主要用于测试目的。"></x-field>
<x-field data-name="state" data-type="State" data-desc="一个 Starlette 状态对象，用于存储任意应用程序级别的状态。虽然可用，但在 FastAPI 中使用依赖项通常是更好的方法。"></x-field>
<x-field data-name="openapi_schema" data-type="Optional[Dict[str, Any]]" data-desc="缓存的 OpenAPI 模式。它在首次需要时生成，然后存储在此处。你可以修改此字典以自定义 OpenAPI 模式。"></x-field>
<x-field data-name="webhooks" data-type="APIRouter" data-desc="一个专用于路径操作的 APIRouter，这些路径操作将被记录为 OpenAPI Webhook。"></x-field>

## 方法

### 路径操作装饰器

向应用程序添加路由的主要方法是使用路径操作装饰器。这些装饰器对应于 HTTP 方法（`GET`、`POST`、`PUT`、`DELETE` 等）。

#### @app.get()

此装饰器注册一个函数来处理给定路径的 HTTP GET 请求。

```python GET Example icon=logos:python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/")
def read_items():
    return [{"name": "Empanada"}, {"name": "Arepa"}]
```

`.get()` 装饰器以及所有其他路径操作装饰器（`.post()`、`.put()` 等）都接受许多参数来配置路由。以下是一些最常见的参数：

<x-field data-name="path" data-type="str" data-required="true" data-desc="此路径操作的 URL 路径。"></x-field>
<x-field data-name="response_model" data-type="Any" data-required="false" data-desc="用于响应的 Pydantic 模型或类型。它确保响应数据符合此模型，并过滤掉任何额外数据。"></x-field>
<x-field data-name="status_code" data-type="Optional[int]" data-required="false" data-desc="响应的默认 HTTP 状态码。"></x-field>
<x-field data-name="tags" data-type="Optional[List[Union[str, Enum]]]" data-required="false" data-desc="用于在 API 文档中对此路径操作进行分组的标签列表。"></x-field>
<x-field data-name="summary" data-type="Optional[str]" data-required="false" data-desc="路径操作的简短摘要，显示在 API 文档中。"></x-field>
<x-field data-name="description" data-type="Optional[str]" data-required="false" data-desc="路径操作的较长描述，支持 Markdown。如果未提供，则从函数的文档字符串中获取。"></x-field>
<x-field data-name="deprecated" data-type="Optional[bool]" data-required="false" data-desc="在 API 文档中将此路径操作标记为已弃用。"></x-field>
<x-field data-name="dependencies" data-type="Optional[Sequence[Depends]]" data-required="false" data-desc="特定于此路径操作的依赖项列表。"></x-field>

其他路径操作装饰器，如 `@app.post()`、`@app.put()`、`@app.delete()`、`@app.patch()`、`@app.options()` 和 `@app.head()`，共享相同的配置参数。

### include_router()

在应用程序中包含一个 `APIRouter`，这对于在多个文件中构建大型应用程序至关重要。

```python Include Router Example icon=logos:python
from fastapi import FastAPI
from .users import users_router

app = FastAPI()

app.include_router(users_router)
```

**参数**

<x-field data-name="router" data-type="APIRouter" data-required="true" data-desc="要包含的 APIRouter 实例。"></x-field>
<x-field data-name="prefix" data-type="str" data-default="" data-required="false" data-desc="应用于路由器中所有路由的 URL 前缀。"></x-field>
<x-field data-name="tags" data-type="Optional[List[Union[str, Enum]]]" data-required="false" data-desc="应用于路由器中所有路由的标签。"></x-field>
<x-field data-name="dependencies" data-type="Optional[Sequence[Depends]]" data-required="false" data-desc="应用于路由器中所有路由的依赖项。"></x-field>
<x-field data-name="responses" data-type="Optional[Dict[...]]" data-required="false" data-desc="应用于路由器中所有路由以供文档使用的附加响应。"></x-field>

有关更多详细信息，请参阅[大型应用](./tutorials-advanced-bigger-applications.md)的文档。

### @app.websocket()

用于创建 WebSocket 端点的装饰器。

```python WebSocket Example icon=logos:python
from fastapi import FastAPI, WebSocket

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```

**参数**

<x-field data-name="path" data-type="str" data-required="true" data-desc="WebSocket 的 URL 路径。"></x-field>
<x-field data-name="name" data-type="Optional[str]" data-required="false" data-desc="WebSocket 路由的内部名称。"></x-field>
<x-field data-name="dependencies" data-type="Optional[Sequence[Depends]]" data-required="false" data-desc="此 WebSocket 的依赖项列表。"></x-field>

### @app.middleware()

用于向应用程序添加中间件的装饰器。目前，仅支持 `"http"` 中间件以这种方式添加。

```python Middleware Example icon=logos:python
import time
from typing import Awaitable, Callable

from fastapi import FastAPI, Request, Response

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(
    request: Request, call_next: Callable[[Request], Awaitable[Response]]
) -> Response:
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

### @app.exception_handler()

用于为应用程序注册自定义异常处理程序的装饰器。

```python Exception Handler Example icon=logos:python
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
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )
```

### openapi()

生成并返回应用程序的 OpenAPI 模式。它会缓存结果，因此后续调用速度很快。你可以覆盖此方法以自定义生成的模式。

```python Custom OpenAPI Example icon=logos:python
from fastapi import FastAPI
from fastapi.openapi.utils import get_openapi

app = FastAPI()

def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema
    openapi_schema = get_openapi(
        title="Custom title",
        version="2.5.0",
        summary="This is a very custom OpenAPI schema",
        description="Here's a longer description of the custom **OpenAPI** schema",
        routes=app.routes,
    )
    app.openapi_schema = openapi_schema
    return app.openapi_schema

app.openapi = custom_openapi
```

---

现在你已经了解了主要的 `FastAPI` 应用程序类，接下来一个很好的步骤是学习如何使用 [APIRouter](./api-reference-apirouter.md) 来构建你的应用程序。