# APIRouter

The `APIRouter` class is a powerful tool for structuring your FastAPI application. It allows you to group *path operations*, which is particularly useful for organizing your code into multiple files or modules. You can think of an `APIRouter` as a mini `FastAPI` application that can be included in a main app or even in another router.

For a step-by-step guide on how to structure larger applications, please refer to the [Bigger Applications - Multiple Files](./tutorials-advanced-bigger-applications.md) tutorial.

## Basic Usage

Here's a simple example of how to use `APIRouter`:

```python icon=logos:python title="main.py"
from fastapi import APIRouter, FastAPI

app = FastAPI()
router = APIRouter()


@router.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]


app.include_router(router)
```

In this example, a router is created, a path operation is added to it, and then the router is included in the main `FastAPI` application.

## Parameters

When you create an instance of `APIRouter`, you can configure it with several parameters that will apply to all the *path operations* it contains.

<x-field data-name="prefix" data-type="string" data-default="" data-desc="An optional path prefix for all routes in the router. It must start with a '/'. For example, a prefix of '/users' and a route path of '/me' will result in a final path of '/users/me'."></x-field>
<x-field data-name="tags" data-type="list[str | Enum]" data-desc="A list of tags to apply to all path operations in this router. These tags are used for grouping operations in the OpenAPI documentation."></x-field>
<x-field data-name="dependencies" data-type="list[Depends]" data-desc="A list of dependencies to be applied to all path operations in this router."></x-field>
<x-field data-name="default_response_class" data-type="Type[Response]" data-default="JSONResponse" data-desc="The default response class to use for path operations."></x-field>
<x-field data-name="responses" data-type="dict" data-desc="Additional responses to be shown in the OpenAPI documentation for all path operations."></x-field>
<x-field data-name="callbacks" data-type="list[BaseRoute]" data-desc="A list of OpenAPI callbacks that apply to all path operations in this router."></x-field>
<x-field data-name="deprecated" data-type="bool" data-desc="A boolean indicating whether to mark all path operations in this router as deprecated in the OpenAPI schema."></x-field>
<x-field data-name="include_in_schema" data-type="bool" data-default="true" data-desc="A boolean indicating whether to include all path operations from this router in the generated OpenAPI schema."></x-field>
<x-field data-name="lifespan" data-type="Lifespan[Any]" data-desc="A Lifespan context manager handler that replaces the deprecated startup and shutdown event handlers."></x-field>
<x-field data-name="on_startup" data-type="list[Callable]" data-deprecated="true" data-desc="A list of startup event handler functions. Use 'lifespan' instead."></x-field>
<x-field data-name="on_shutdown" data-type="list[Callable]" data-deprecated="true" data-desc="A list of shutdown event handler functions. Use 'lifespan' instead."></x-field>

## Path Operation Decorators

Like the `FastAPI` app object, `APIRouter` provides decorator methods for all standard HTTP operations to add routes.

### Common Parameters

All path operation decorators share a common set of parameters for configuration and documentation:

<x-field data-name="path" data-type="str" data-required="true" data-desc="The URL path for this path operation."></x-field>
<x-field data-name="response_model" data-type="Any" data-desc="The Pydantic model to use for the response. It's used for validation, serialization, and documentation."></x-field>
<x-field data-name="status_code" data-type="int" data-desc="The default HTTP status code for the response."></x-field>
<x-field data-name="tags" data-type="list[str | Enum]" data-desc="A list of tags for this path operation, used for grouping in the OpenAPI schema."></x-field>
<x-field data-name="dependencies" data-type="list[Depends]" data-desc="A list of dependencies for this path operation."></x-field>
<x-field data-name="summary" data-type="str" data-desc="A short summary for the path operation, shown in the OpenAPI UI."></x-field>
<x-field data-name="description" data-type="str" data-desc="A detailed description for the path operation. Supports Markdown. If not provided, it's extracted from the function's docstring."></x-field>
<x-field data-name="response_description" data-type="str" data-default="Successful Response" data-desc="The description for the default response."></x-field>
<x-field data-name="responses" data-type="dict" data-desc="A dictionary of additional possible responses, including different status codes and their models."></x-field>
<x-field data-name="deprecated" data-type="bool" data-desc="A boolean to mark this path operation as deprecated in the OpenAPI schema."></x-field>
<x-field data-name="operation_id" data-type="str" data-desc="A unique string used to identify the path operation. If not provided, it is generated automatically."></x-field>
<x-field data-name="response_model_include" data-type="set | dict" data-desc="Fields to include in the response model."></x-field>
<x-field data-name="response_model_exclude" data-type="set | dict" data-desc="Fields to exclude from the response model."></x-field>
<x-field data-name="response_model_by_alias" data-type="bool" data-default="true" data-desc="Whether to serialize the response model by alias."></x-field>
<x-field data-name="response_model_exclude_unset" data-type="bool" data-default="false" data-desc="Whether to exclude fields that were not explicitly set from the response."></x-field>
<x-field data-name="response_model_exclude_defaults" data-type="bool" data-default="false" data-desc="Whether to exclude fields that have their default values from the response."></x-field>
<x-field data-name="response_model_exclude_none" data-type="bool" data-default="false" data-desc="Whether to exclude fields with a value of None from the response."></x-field>
<x-field data-name="include_in_schema" data-type="bool" data-default="true" data-desc="Whether to include this path operation in the OpenAPI schema."></x-field>
<x-field data-name="response_class" data-type="Type[Response]" data-desc="The response class to be used for this path operation."></x-field>
<x-field data-name="callbacks" data-type="list[BaseRoute]" data-desc="A list of path operations that will be used as OpenAPI callbacks."></x-field>
<x-field data-name="openapi_extra" data-type="dict" data-desc="Extra metadata to be included in the OpenAPI schema for this path operation."></x-field>

### HTTP Methods

Below are examples for each HTTP method decorator available on an `APIRouter` instance.

#### `@router.get()`
Adds a path operation for HTTP GET requests.
```python icon=logos:python
@router.get("/items/")
def read_items():
    return [{"name": "Empanada"}, {"name": "Arepa"}]
```

#### `@router.post()`
Adds a path operation for HTTP POST requests.
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
Adds a path operation for HTTP PUT requests.
```python icon=logos:python
@router.put("/items/{item_id}")
def replace_item(item_id: str, item: Item):
    return {"message": "Item replaced", "id": item_id}
```

#### `@router.delete()`
Adds a path operation for HTTP DELETE requests.
```python icon=logos:python
@router.delete("/items/{item_id}")
def delete_item(item_id: str):
    return {"message": "Item deleted"}
```

#### `@router.patch()`
Adds a path operation for HTTP PATCH requests.
```python icon=logos:python
@router.patch("/items/")
def update_item(item: Item):
    return {"message": "Item updated in place"}
```

#### `@router.options()`
Adds a path operation for HTTP OPTIONS requests.
```python icon=logos:python
@router.options("/items/")
def get_item_options():
    return {"additions": ["Aji", "Guacamole"]}
```

#### `@router.head()`
Adds a path operation for HTTP HEAD requests.
```python icon=logos:python
from fastapi import Response

@router.head("/items/", status_code=204)
def get_items_headers(response: Response):
    response.headers["X-Cat-Dog"] = "Alone in the world"
```

#### `@router.trace()`
Adds a path operation for HTTP TRACE requests.
```python icon=logos:python
@router.trace("/items/{item_id}")
def trace_item(item_id: str):
    return None
```

## Including Other Routers

You can include one `APIRouter` into another `APIRouter` or into a `FastAPI` app using the `include_router` method. This is the primary mechanism for building larger, modular applications.

```python icon=logos:python title="main.py"
from fastapi import APIRouter, FastAPI

app = FastAPI()

# Router for internal/admin endpoints
internal_router = APIRouter()

# Router for user-facing endpoints
users_router = APIRouter(
    prefix="/users",
    tags=["users"],
)

@users_router.get("/")
def read_users():
    return [{"name": "Rick"}, {"name": "Morty"}]

# Include users_router into internal_router
internal_router.include_router(users_router)

# Include the combined router into the main app
app.include_router(internal_router, prefix="/api/v1")
```

### `include_router` Parameters

When including a router, you can specify parameters that will apply to all of its routes, which are combined with any parameters already set on the router itself.

<x-field data-name="router" data-type="APIRouter" data-required="true" data-desc="The APIRouter instance to include."></x-field>
<x-field data-name="prefix" data-type="string" data-default="" data-desc="A path prefix to be prepended to all routes in the included router."></x-field>
<x-field data-name="tags" data-type="list[str | Enum]" data-desc="Tags to be added to all path operations in the included router."></x-field>
<x-field data-name="dependencies" data-type="list[Depends]" data-desc="Dependencies to be added to all path operations in the included router."></x-field>
<x-field data-name="default_response_class" data-type="Type[Response]" data-desc="The default response class to use, overriding the one in the included router."></x-field>
<x-field data-name="responses" data-type="dict" data-desc="Additional responses to be added to all path operations."></x-field>
<x-field data-name="deprecated" data-type="bool" data-desc="Mark all path operations in the included router as deprecated."></x-field>
<x-field data-name="include_in_schema" data-type="bool" data-default="true" data-desc="Whether to include the routes from this router in the OpenAPI schema."></x-field>

## WebSockets

`APIRouter` also supports WebSocket routes using the `.websocket()` decorator.

### `@router.websocket()`

Decorates a function to handle WebSocket connections.

<x-field data-name="path" data-type="str" data-required="true" data-desc="The WebSocket path."></x-field>
<x-field data-name="name" data-type="str" data-desc="An optional internal name for the WebSocket route."></x-field>
<x-field data-name="dependencies" data-type="list[Depends]" data-desc="A list of dependencies for this WebSocket connection."></x-field>

**Example**

```python icon=logos:python
from fastapi import WebSocket

@router.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```