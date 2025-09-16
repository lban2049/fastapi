# APIRouter

`APIRouter` 类是构建 FastAPI 应用的强大工具。它允许你对*路径操作*进行分组，这对于将代码组织到多个文件或模块中特别有用。你可以将 `APIRouter` 视为一个微型的 `FastAPI` 应用，它可以被包含在主应用中，甚至可以包含在另一个路由器中。

有关如何构建更大型应用的分步指南，请参阅 [更大规模的应用 - 多个文件](./tutorials-advanced-bigger-applications.md) 教程。

## 基本用法

以下是一个如何使用 `APIRouter` 的简单示例：

```python icon=logos:python title="main.py"
from fastapi import APIRouter, FastAPI

app = FastAPI()
router = APIRouter()


@router.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]


app.include_router(router)
```

在此示例中，创建了一个路由器，向其添加了一个路径操作，然后将该路由器包含在主 `FastAPI` 应用中。

## 参数

创建 `APIRouter` 实例时，你可以使用多个参数对其进行配置，这些参数将应用于它包含的所有*路径操作*。

<x-field data-name="prefix" data-type="string" data-default="" data-desc="路由器中所有路由的可选路径前缀。它必须以“/”开头。例如，前缀“/users”和路由路径“/me”将产生最终路径“/users/me”。"></x-field>
<x-field data-name="tags" data-type="list[str | Enum]" data-desc="应用于此路由器中所有路径操作的标签列表。这些标签用于在 OpenAPI 文档中对操作进行分组。"></x-field>
<x-field data-name="dependencies" data-type="list[Depends]" data-desc="应用于此路由器中所有路径操作的依赖项列表。"></x-field>
<x-field data-name="default_response_class" data-type="Type[Response]" data-default="JSONResponse" data-desc="用于路径操作的默认响应类。"></x-field>
<x-field data-name="responses" data-type="dict" data-desc="在 OpenAPI 文档中为所有路径操作显示的其他响应。"></x-field>
<x-field data-name="callbacks" data-type="list[BaseRoute]" data-desc="适用于此路由器中所有路径操作的 OpenAPI 回调列表。"></x-field>
<x-field data-name="deprecated" data-type="bool" data-desc="一个布尔值，指示是否在此路由器的 OpenAPI 结构中将所有路径操作标记为已弃用。"></x-field>
<x-field data-name="include_in_schema" data-type="bool" data-default="true" data-desc="一个布尔值，指示是否将此路由器中的所有路径操作包含在生成的 OpenAPI 结构中。"></x-field>
<x-field data-name="lifespan" data-type="Lifespan[Any]" data-desc="一个 Lifespan 上下文管理器处理器，用于替换已弃用的启动和关闭事件处理器。"></x-field>
<x-field data-name="on_startup" data-type="list[Callable]" data-deprecated="true" data-desc="启动事件处理器函数列表。请改用 'lifespan'。"></x-field>
<x-field data-name="on_shutdown" data-type="list[Callable]" data-deprecated="true" data-desc="关闭事件处理器函数列表。请改用 'lifespan'。"></x-field>

## 路径操作装饰器

与 `FastAPI` 应用对象一样，`APIRouter` 为所有标准 HTTP 操作提供装饰器方法以添加路由。

### 通用参数

所有路径操作装饰器共享一组用于配置和文档的通用参数：

<x-field data-name="path" data-type="str" data-required="true" data-desc="此路径操作的 URL 路径。"></x-field>
<x-field data-name="response_model" data-type="Any" data-desc="用于响应的 Pydantic 模型。它用于验证、序列化和文档记录。"></x-field>
<x-field data-name="status_code" data-type="int" data-desc="响应的默认 HTTP 状态码。"></x-field>
<x-field data-name="tags" data-type="list[str | Enum]" data-desc="此路径操作的标签列表，用于在 OpenAPI 结构中进行分组。"></x-field>
<x-field data-name="dependencies" data-type="list[Depends]" data-desc="此路径操作的依赖项列表。"></x-field>
<x-field data-name="summary" data-type="str" data-desc="路径操作的简短摘要，显示在 OpenAPI UI 中。"></x-field>
<x-field data-name="description" data-type="str" data-desc="路径操作的详细描述。支持 Markdown。如果未提供，则从函数的文档字符串中提取。"></x-field>
<x-field data-name="response_description" data-type="str" data-default="Successful Response" data-desc="默认响应的描述。"></x-field>
<x-field data-name="responses" data-type="dict" data-desc="包含其他可能响应的字典，包括不同的状态码及其模型。"></x-field>
<x-field data-name="deprecated" data-type="bool" data-desc="一个布尔值，用于在 OpenAPI 结构中将此路径操作标记为已弃用。"></x-field>
<x-field data-name="operation_id" data-type="str" data-desc="用于标识路径操作的唯一字符串。如果未提供，则会自动生成。"></x-field>
<x-field data-name="response_model_include" data-type="set | dict" data-desc="要包含在响应模型中的字段。"></x-field>
<x-field data-name="response_model_exclude" data-type="set | dict" data-desc="要从响应模型中排除的字段。"></x-field>
<x-field data-name="response_model_by_alias" data-type="bool" data-default="true" data-desc="是否按别名序列化响应模型。"></x-field>
<x-field data-name="response_model_exclude_unset" data-type="bool" data-default="false" data-desc="是否从响应中排除未显式设置的字段。"></x-field>
<x-field data-name="response_model_exclude_defaults" data-type="bool" data-default="false" data-desc="是否从响应中排除具有默认值的字段。"></x-field>
<x-field data-name="response_model_exclude_none" data-type="bool" data-default="false" data-desc="是否从响应中排除值为 None 的字段。"></x-field>
<x-field data-name="include_in_schema" data-type="bool" data-default="true" data-desc="是否在 OpenAPI 结构中包含此路径操作。"></x-field>
<x-field data-name="response_class" data-type="Type[Response]" data-desc="用于此路径操作的响应类。"></x-field>
<x-field data-name="callbacks" data-type="list[BaseRoute]" data-desc="将用作 OpenAPI 回调的路径操作列表。"></x-field>
<x-field data-name="openapi_extra" data-type="dict" data-desc="要包含在此路径操作的 OpenAPI 结构中的额外元数据。"></x-field>

### HTTP 方法

以下是 `APIRouter` 实例上可用的每个 HTTP 方法装饰器的示例。

#### `@router.get()`
为 HTTP GET 请求添加路径操作。
```python icon=logos:python
@router.get("/items/")
def read_items():
    return [{"name": "Empanada"}, {"name": "Arepa"}]
```

#### `@router.post()`
为 HTTP POST 请求添加路径操作。
```python icon=logos:python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None

@router.post("/items/")
def create_item(item: Item):
    return {"message": "Item created"}
```

#### `@router.put()`
为 HTTP PUT 请求添加路径操作。
```python icon=logos:python
@router.put("/items/{item_id}")
def replace_item(item_id: str, item: Item):
    return {"message": "Item replaced", "id": item_id}
```

#### `@router.delete()`
为 HTTP DELETE 请求添加路径操作。
```python icon=logos:python
@router.delete("/items/{item_id}")
def delete_item(item_id: str):
    return {"message": "Item deleted"}
```

#### `@router.patch()`
为 HTTP PATCH 请求添加路径操作。
```python icon=logos:python
@router.patch("/items/")
def update_item(item: Item):
    return {"message": "Item updated in place"}
```

#### `@router.options()`
为 HTTP OPTIONS 请求添加路径操作。
```python icon=logos:python
@router.options("/items/")
def get_item_options():
    return {"additions": ["Aji", "Guacamole"]}
```

#### `@router.head()`
为 HTTP HEAD 请求添加路径操作。
```python icon=logos:python
from fastapi import Response

@router.head("/items/", status_code=204)
def get_items_headers(response: Response):
    response.headers["X-Cat-Dog"] = "Alone in the world"
```

#### `@router.trace()`
为 HTTP TRACE 请求添加路径操作。
```python icon=logos:python
@router.trace("/items/{item_id}")
def trace_item(item_id: str):
    return None
```

## 包含其他路由器

你可以使用 `include_router` 方法将一个 `APIRouter` 包含到另一个 `APIRouter` 或 `FastAPI` 应用中。这是构建更大型、模块化应用的主要机制。

```python icon=logos:python title="main.py"
from fastapi import APIRouter, FastAPI

app = FastAPI()

# 用于内部/管理端点的路由器
internal_router = APIRouter()

# 用于面向用户的端点的路由器
users_router = APIRouter(
    prefix="/users",
    tags=["users"],
)

@users_router.get("/")
def read_users():
    return [{"name": "Rick"}, {"name": "Morty"}]

# 将 users_router 包含到 internal_router 中
internal_router.include_router(users_router)

# 将组合后的路由器包含到主应用中
app.include_router(internal_router, prefix="/api/v1")
```

### `include_router` 参数

包含路由器时，你可以指定将应用于其所有路由的参数，这些参数会与路由器本身已设置的任何参数相结合。

<x-field data-name="router" data-type="APIRouter" data-required="true" data-desc="要包含的 APIRouter 实例。"></x-field>
<x-field data-name="prefix" data-type="string" data-default="" data-desc="要添加到所含路由器中所有路由的路径前缀。"></x-field>
<x-field data-name="tags" data-type="list[str | Enum]" data-desc="要添加到所含路由器中所有路径操作的标签。"></x-field>
<x-field data-name="dependencies" data-type="list[Depends]" data-desc="要添加到所含路由器中所有路径操作的依赖项。"></x-field>
<x-field data-name="default_response_class" data-type="Type[Response]" data-desc="要使用的默认响应类，它会覆盖所含路由器中的默认响应类。"></x-field>
<x-field data-name="responses" data-type="dict" data-desc="要添加到所有路径操作中的其他响应。"></x-field>
<x-field data-name="deprecated" data-type="bool" data-desc="将所含路由器中的所有路径操作标记为已弃用。"></x-field>
<x-field data-name="include_in_schema" data-type="bool" data-default="true" data-desc="是否在 OpenAPI 结构中包含此路由器的路由。"></x-field>

## WebSocket

`APIRouter` 还支持使用 `.websocket()` 装饰器的 WebSocket 路由。

### `@router.websocket()`

装饰一个函数以处理 WebSocket 连接。

<x-field data-name="path" data-type="str" data-required="true" data-desc="WebSocket 路径。"></x-field>
<x-field data-name="name" data-type="str" data-desc="WebSocket 路由的可选内部名称。"></x-field>
<x-field data-name="dependencies" data-type="list[Depends]" data-desc="此 WebSocket 连接的依赖项列表。"></x-field>

**示例**

```python icon=logos:python
from fastapi import WebSocket

@router.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```