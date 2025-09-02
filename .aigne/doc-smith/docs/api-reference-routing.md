# Routing

FastAPI uses a powerful routing system to associate URL paths with the specific functions that handle them. This is primarily managed through two key classes: `APIRouter` and `APIRoute`. Understanding these components is essential for structuring your application, especially as it grows in complexity.

- **`APIRouter`**: A class that allows you to group *path operations*. It's the standard tool for splitting your API into multiple, modular files. You can define path operations on a router and then include that router in your main `FastAPI` application or even in another router.
- **`APIRoute`**: The class that actually handles a single *path operation*. When you use a decorator like `@app.get("/items")`, you are creating an instance of `APIRoute` behind the scenes.

For a step-by-step guide on structuring larger projects, see the user guide on [Bigger Applications](./advanced-bigger-applications.md).

```d2
direction: down

"FastAPI App" {
  shape: cloud
  "app = FastAPI()"
}

"Users Router (APIRouter)" {
  shape: package
  "router = APIRouter(prefix='/users')"

  "GET /: APIRoute"
  "POST /: APIRoute"
}

"Items Router (APIRouter)" {
  shape: package
  "router = APIRouter(prefix='/items')"

  "GET /{item_id}: APIRoute"
}

"FastAPI App" -> "Users Router (APIRouter)": "app.include_router(users_router)"
"FastAPI App" -> "Items Router (APIRouter)": "app.include_router(items_router)"
```

## APIRouter

The `APIRouter` class is used to create a modular set of path operations that can be included later in a main `FastAPI` application. This is the preferred way to structure your code for maintainability.

### Creating an `APIRouter`

You can instantiate `APIRouter` with several parameters to configure all the path operations it contains.

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

### `APIRouter` Parameters

When creating an `APIRouter` instance, you can provide the following parameters:

| Parameter | Type | Description |
|---|---|---|
| `prefix` | `str` | An optional path prefix for all routes in this router. It must start with a `/`. |
| `tags` | `Optional[List[Union[str, Enum]]]` | A list of tags to apply to all path operations in this router, used for OpenAPI documentation. |
| `dependencies` | `Optional[Sequence[params.Depends]]` | A sequence of dependencies to apply to all path operations in this router. |
| `default_response_class` | `Type[Response]` | The default response class to use for path operations. Defaults to `JSONResponse`. |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | Additional responses to be included in the OpenAPI schema for all path operations. |
| `callbacks` | `Optional[List[BaseRoute]]` | A list of OpenAPI callbacks to apply to all path operations in this router. |
| `deprecated` | `Optional[bool]` | If `True`, marks all path operations in this router as deprecated in the OpenAPI schema. |
| `include_in_schema` | `bool` | If `False`, excludes all path operations in this router from the generated OpenAPI schema. Defaults to `True`. |
| `generate_unique_id_function` | `Callable[[APIRoute], str]` | A function to customize the generation of unique IDs for path operations. |
| `route_class` | `Type[APIRoute]` | A custom `APIRoute` class to be used for the path operations in this router. |

### Including a Router

The most important method on the main `FastAPI` app for working with routers is `include_router`. It mounts all the path operations from the router into the application.

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

#### `include_router` Parameters

You can also provide parameters to `include_router` to add further configuration to the included routes.

| Parameter | Type | Description |
|---|---|---|
| `router` | `APIRouter` | The `APIRouter` instance to include. |
| `prefix` | `str` | An optional path prefix to prepend to all routes from the included router. |
| `tags` | `Optional[List[Union[str, Enum]]]` | A list of tags to add to all path operations from the included router. |
| `dependencies` | `Optional[Sequence[params.Depends]]` | A sequence of dependencies to add to all path operations from the included router. |
| `default_response_class` | `Type[Response]` | A default response class for the included routes. |
| `responses` | `Optional[Dict[Union[int, str], Dict[str, Any]]]` | Additional responses for the included routes. |
| `deprecated` | `Optional[bool]` | Mark all path operations from the included router as deprecated. |
| `include_in_schema` | `bool` | A boolean to indicate if the included routes should be in the OpenAPI schema. |

### Path Operation Decorators

`APIRouter` provides decorator methods for all the standard HTTP verbs (`.get()`, `.post()`, `.put()`, `.patch()`, `.delete()`, etc.) to add path operations. These decorators accept numerous parameters to configure the route, its validation, its documentation, and its response.

Below are the common parameters available for all path operation decorators.

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | The URL path for this operation. |
| `response_model` | `Any` | The Pydantic model to be used for the response, enabling validation and serialization. |
| `status_code` | `Optional[int]` | The default HTTP status code for the response. |
| `tags` | `Optional[List[Union[str, Enum]]]` | A list of tags for grouping this operation in the OpenAPI documentation. |
| `dependencies` | `Optional[Sequence[params.Depends]]` | A list of dependencies for this path operation. |
| `summary` | `Optional[str]` | A short summary for the operation in OpenAPI. |
| `description` | `Optional[str]` | A detailed description, supporting Markdown. Extracted from the function's docstring if not provided. |
| `response_description` | `str` | A description for the default response. Defaults to "Successful Response". |
| `responses` | `Optional[Dict[...]]` | A dictionary of additional possible responses with their models, descriptions, etc. |
| `deprecated` | `Optional[bool]` | If `True`, marks this operation as deprecated in OpenAPI. |
| `operation_id` | `Optional[str]` | A unique string used to identify the operation. Useful for client generation. |
| `response_model_include` | `Optional[IncEx]` | Fields to include in the response model. |
| `response_model_exclude` | `Optional[IncEx]` | Fields to exclude from the response model. |
| `response_model_by_alias` | `bool` | Whether to serialize the response model by alias. Defaults to `True`. |
| `response_model_exclude_unset` | `bool` | Whether to exclude fields that were not explicitly set from the response. Defaults to `False`. |
| `response_model_exclude_defaults`| `bool` | Whether to exclude fields that have their default value from the response. Defaults to `False`. |
| `response_model_exclude_none` | `bool` | Whether to exclude fields with a value of `None` from the response. Defaults to `False`. |
| `include_in_schema` | `bool` | Whether to include this path operation in the OpenAPI schema. Defaults to `True`. |
| `response_class` | `Type[Response]` | A custom `Response` class to use for this operation's response. |
| `name` | `Optional[str]` | A name for the route, used for URL path generation. Only used internally. |
| `callbacks` | `Optional[List[BaseRoute]]` | A list of OpenAPI callbacks for this operation. |
| `openapi_extra` | `Optional[Dict[str, Any]]` | A dictionary of extra metadata to include in the OpenAPI schema for this operation. |

### WebSocket Decorator

For real-time communication, `APIRouter` provides a `websocket` decorator.

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
| `path` | `str` | The WebSocket path. |
| `name` | `Optional[str]` | An internal name for the WebSocket route. |
| `dependencies` | `Optional[Sequence[params.Depends]]`| A list of dependencies for this WebSocket. |

---

## APIRoute

An `APIRoute` represents a single API endpoint (a *path operation*). While you typically create routes using decorators on `FastAPI` or `APIRouter` objects, you can instantiate this class directly or subclass it for advanced customization.

### `APIRoute` Parameters

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | The URL path for this route. |
| `endpoint` | `Callable[..., Any]` | The function that handles requests to this path. |
| `response_model` | `Any` | Pydantic model for the response. |
| `status_code` | `Optional[int]` | The default HTTP status code. |
| `tags` | `Optional[List[...]]` | List of tags for OpenAPI documentation. |
| `dependencies` | `Optional[Sequence[...]]` | Sequence of dependencies for this route. |
| `summary` | `Optional[str]` | A short summary for the operation. |
| `description` | `Optional[str]` | A detailed description. |
| `response_description` | `str` | Description of the successful response. |
| `responses` | `Optional[Dict[...]]` | Dictionary of additional responses. |
| `deprecated` | `Optional[bool]` | Marks the route as deprecated. |
| `name` | `Optional[str]` | An internal name for the route. |
| `methods` | `Optional[Union[Set[str], List[str]]]` | A set or list of HTTP methods for this route (e.g., `["GET", "POST"]`). |
| `operation_id` | `Optional[str]` | A unique ID for the operation. |
| `response_model_include` | `Optional[IncEx]` | Fields to include in the response. |
| `response_model_exclude` | `Optional[IncEx]` | Fields to exclude from the response. |
| `response_model_by_alias` | `bool` | Whether to serialize the response by alias. |
| `response_model_exclude_unset`| `bool` | Whether to exclude unset fields from the response. |
| `response_model_exclude_defaults`| `bool` | Whether to exclude fields with default values from the response. |
| `response_model_exclude_none` | `bool` | Whether to exclude fields set to `None` from the response. |
| `include_in_schema` | `bool` | Whether to include this route in the OpenAPI schema. |
| `response_class` | `Union[Type[Response], ...]` | The response class to use. |
| `dependency_overrides_provider`| `Optional[Any]` | Used internally to handle dependency overrides. |
| `callbacks` | `Optional[List[BaseRoute]]` | List of OpenAPI callbacks. |
| `openapi_extra` | `Optional[Dict[str, Any]]` | Extra OpenAPI metadata. |
| `generate_unique_id_function` | `Callable[[APIRoute], str]` | Function to generate unique IDs. |

---

## APIWebSocketRoute

This class is the equivalent of `APIRoute` for WebSocket connections. It is created when you use the `@router.websocket()` decorator.

### `APIWebSocketRoute` Parameters

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | The URL path for the WebSocket. |
| `endpoint` | `Callable[..., Any]` | The function that handles the WebSocket connection. |
| `name` | `Optional[str]` | An internal name for the route. |
| `dependencies` | `Optional[Sequence[params.Depends]]` | A sequence of dependencies for this WebSocket route. |
| `dependency_overrides_provider`| `Optional[Any]` | Used internally to handle dependency overrides. |
