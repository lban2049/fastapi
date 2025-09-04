# Routing

FastAPI uses `APIRouter` to organize and structure your application's endpoints. This is particularly useful for splitting your API into multiple files and modules, which is a common practice for building larger, maintainable applications. Each individual endpoint is represented internally by an `APIRoute`.

This page provides a detailed reference for these classes. For a step-by-step guide on how to structure your project with routers, see the tutorial on [Bigger Applications](./advanced-bigger-applications.md).

## `APIRouter`

The `APIRouter` class allows you to group *path operations*. You can think of it as a mini `FastAPI` application. You can define routes, dependencies, and tags on a router, and then include it in the main application or even in another router.

### Basic Usage

Here's a simple example of how to use `APIRouter`:

```python
from fastapi import APIRouter, FastAPI

app = FastAPI()
router = APIRouter()


@router.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]


app.include_router(router)
```

### `APIRouter` Parameters

When you create an instance of `APIRouter`, you can pass several parameters to configure all the routes it contains.

| Parameter | Type | Description |
|---|---|---|
| `prefix` | `str` | An optional path prefix for all routes in this router. It must start with a `/`. |
| `tags` | `Optional[List[Union[str, Enum]]]` | A list of tags to apply to all *path operations* in this router. These are used for grouping in the OpenAPI documentation. |
| `dependencies` | `Optional[Sequence[params.Depends]]` | A list of dependencies (`Depends()`) to be applied to all *path operations* in this router. |
| `default_response_class` | `Type[Response]` | The default response class to be used. Defaults to `JSONResponse`. |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | Additional responses to be applied to all routes, which will be shown in the OpenAPI documentation. |
| `callbacks` | `Optional[List[BaseRoute]]` | OpenAPI callbacks that apply to all *path operations* in this router. |
| `deprecated` | `Optional[bool]` | If `True`, marks all *path operations* in this router as deprecated in the OpenAPI documentation. |
| `include_in_schema` | `bool` | If `False`, excludes all *path operations* in this router from the generated OpenAPI schema. Defaults to `True`. |
| `generate_unique_id_function` | `Callable[[APIRoute], str]` | A function to customize the generation of unique IDs for *path operations*. |
| `route_class` | `Type[APIRoute]` | A custom `APIRoute` class to be used for the routes in this router. |
| `lifespan` | `Optional[Lifespan[Any]]` | A `Lifespan` context manager to handle startup and shutdown events for this router. |

### Path Operation Decorators

`APIRouter` provides decorators for all standard HTTP methods to add new *path operations*.

- `@router.get()`
- `@router.put()`
- `@router.post()`
- `@router.delete()`
- `@router.options()`
- `@router.head()`
- `@router.patch()`
- `@router.trace()`

All these decorators accept the same parameters as the `APIRoute` class, which are detailed below.

```python
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter()

class Item(BaseModel):
    name: str
    description: str | None = None

@router.post("/items/", status_code=201, tags=["items"])
async def create_item(item: Item):
    return {"message": "Item created successfully", "item": item}
```

### `websocket()` Decorator

To add a WebSocket endpoint, use the `@router.websocket()` decorator.

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | The WebSocket path. |
| `name` | `Optional[str]` | An internal name for the WebSocket route. |
| `dependencies` | `Optional[Sequence[params.Depends]]` | A list of dependencies for this WebSocket. |

**Example**
```python
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

This method is used to mount another `APIRouter` onto the current one or onto a `FastAPI` app. This is the key to building modular applications.

```python
# In items.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/items/")
def get_items():
    return ["Portal Gun", "Plumbus"]

# In main.py
from fastapi import FastAPI
# from . import items # Assuming items.py is in the same directory

app = FastAPI()
app.include_router(router, prefix="/api/v1", tags=["items"])
```

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `router` | `APIRouter` | The `APIRouter` instance to include. |
| `prefix` | `str` | An optional path prefix to be prepended to all routes from the included router. |
| `tags` | `Optional[List[Union[str, Enum]]]` | A list of tags to add to all routes from the included router. |
| `dependencies` | `Optional[Sequence[params.Depends]]` | A list of dependencies to add to all routes from the included router. |
| `default_response_class` | `Type[Response]` | A default response class for the included routes. |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | Additional responses for the included routes. |
| `callbacks` | `Optional[List[BaseRoute]]` | OpenAPI callbacks for the included routes. |
| `deprecated` | `Optional[bool]` | Mark all routes from the included router as deprecated. |
| `include_in_schema` | `bool` | Whether to include the routes in the OpenAPI schema. Defaults to `True`. |
| `generate_unique_id_function` | `Callable[[APIRoute], str]` | A custom function to generate unique IDs for the included routes. |

---

## `APIRoute`

The `APIRoute` class represents a single *path operation*. You typically don't create instances of `APIRoute` directly. Instead, they are created by `APIRouter`'s path operation decorators (e.g., `@router.get(...)`). The parameters listed here are available in all those decorators.

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | The URL path for this operation. |
| `endpoint` | `Callable[..., Any]` | The function that will handle requests to this path. |
| `response_model` | `Any` | The Pydantic model used for the response. It helps with data validation, serialization, and documentation. |
| `status_code` | `Optional[int]` | The default HTTP status code for the response. |
| `tags` | `Optional[List[Union[str, Enum]]]` | A list of tags for OpenAPI documentation grouping. |
| `dependencies` | `Optional[Sequence[params.Depends]]` | A sequence of dependencies for this path operation. |
| `summary` | `Optional[str]` | A short summary for the path operation, used in OpenAPI. |
| `description` | `Optional[str]` | A detailed description, supporting Markdown. Extracted from the function's docstring if not provided. |
| `response_description` | `str` | The description for the default response. Defaults to "Successful Response". |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | A dictionary of additional possible responses with their models, descriptions, etc. |
| `deprecated` | `Optional[bool]` | If `True`, marks the path operation as deprecated in OpenAPI. |
| `methods` | `Optional[Union[Set[str], List[str]]]` | A set or list of HTTP methods for this route (e.g., `["GET", "POST"]`). |
| `operation_id` | `Optional[str]` | A unique string used to identify the operation in OpenAPI. Useful for client generation. |
| `response_model_include` | `Optional[IncEx]` | Fields to include in the response model. |
| `response_model_exclude` | `Optional[IncEx]` | Fields to exclude from the response model. |
| `response_model_by_alias` | `bool` | Whether to use Pydantic model aliases in the response. Defaults to `True`. |
| `response_model_exclude_unset` | `bool` | Whether to exclude fields that were not explicitly set from the response. Defaults to `False`. |
| `response_model_exclude_defaults` | `bool` | Whether to exclude fields that have their default values from the response. Defaults to `False`. |
| `response_model_exclude_none` | `bool` | Whether to exclude fields with a value of `None` from the response. Defaults to `False`. |
| `include_in_schema` | `bool` | Whether to include this path operation in the OpenAPI schema. Defaults to `True`. |
| `response_class` | `Union[Type[Response], DefaultPlaceholder]` | The response class to use, e.g., `JSONResponse`, `HTMLResponse`. |
| `name` | `Optional[str]` | An internal name for the route. |
| `callbacks` | `Optional[List[BaseRoute]]` | A list of OpenAPI callbacks. |
| `openapi_extra` | `Optional[Dict[str, Any]]]` | A dictionary of extra OpenAPI information to be merged into the operation schema. |

---

## `APIWebSocketRoute`

This class is the equivalent of `APIRoute` for WebSocket connections. It is created by the `@router.websocket()` decorator.

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | The URL path for the WebSocket connection. |
| `endpoint` | `Callable[..., Any]` | The function that will handle the WebSocket session. |
| `name` | `Optional[str]` | An internal name for the WebSocket route. |
| `dependencies` | `Optional[Sequence[params.Depends]]` | A sequence of dependencies for this WebSocket route. |

After understanding how to structure your routes, you may want to dive deeper into how parameters are defined. For more information, please see the [Parameters](./api-reference-parameters.md) reference.