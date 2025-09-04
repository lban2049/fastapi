# FastAPI Application

The `FastAPI` class is the main entry point for creating and managing your application. It provides the core functionality for defining routes, handling requests, and configuring your API.

This document serves as a comprehensive API reference for the `FastAPI` class, its configuration parameters, instance attributes, and methods. For a step-by-step introduction, see the [Getting Started](./getting-started.md) guide.

## Basic Usage

To begin, import `FastAPI` and create an application instance:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}
```

## Class Diagram

This diagram illustrates the core components and relationships within the `FastAPI` application structure. `FastAPI` inherits from Starlette and is primarily composed of an `APIRouter` that manages all the routes.

```d2
direction: down

Starlette: { shape: class }
APIRouter: { shape: class }

FastAPI: {
  shape: class
  
  "router: APIRouter"
}

FastAPI -> Starlette: "Inherits from"
FastAPI."router: APIRouter" -> APIRouter: "Composed of"

Path-Operation-Decorators: {
    label: "Path Operation Decorators\n(@app.get, @app.post, etc)"
    shape: rectangle
}

include_router: {
    label: "include_router()"
    shape: rectangle
}

Path-Operation-Decorators -> FastAPI."router: APIRouter": "Modify"
include_router -> FastAPI."router: APIRouter": "Modify"
```

## Parameters

The `FastAPI` class constructor accepts several parameters to configure your application's behavior, metadata, and documentation.

| Parameter | Type | Description |
|---|---|---|
| `title` | `str` | The title of your API. Default: `"FastAPI"`. |
| `description` | `str` | A description of the API, supporting Markdown. Default: `""`. |
| `summary` | `Optional[str]` | A short summary of the API. Default: `None`. |
| `version` | `str` | The version of your application. Default: `"0.1.0"`. |
| `openapi_url` | `Optional[str]` | The URL for the OpenAPI schema. Set to `None` to disable. Default: `"/openapi.json"`. |
| `docs_url` | `Optional[str]` | The URL for the Swagger UI documentation. Set to `None` to disable. Default: `"/docs"`. |
| `redoc_url` | `Optional[str]` | The URL for the ReDoc documentation. Set to `None` to disable. Default: `"/redoc"`. |
| `dependencies` | `Optional[Sequence[Depends]]` | A sequence of global dependencies applied to all path operations. |
| `default_response_class` | `Type[Response]` | The default response class to use. Default: `JSONResponse`. |
| `exception_handlers` | `Optional[Dict]` | A dictionary of exception handlers. |
| `lifespan` | `Optional[Lifespan]` | A lifespan context manager for handling startup and shutdown events. |
| `openapi_tags` | `Optional[List[Dict]]` | Metadata for tags used in path operations. |
| `servers` | `Optional[List[Dict]]` | A list of server definitions for the OpenAPI schema. |
| `contact` | `Optional[Dict]` | Contact information for the API. |
| `license_info` | `Optional[Dict]` | License information for the API. |
| `root_path` | `str` | A path prefix handled by a proxy. Default: `""`. |
| `debug` | `bool` | Enable debug mode. Default: `False`. |
| `routes` | `Optional[List[BaseRoute]]` | A list of routes, inherited from Starlette for compatibility. |
| `redirect_slashes` | `bool` | Whether to redirect trailing slashes. Default: `True`. |
| `swagger_ui_oauth2_redirect_url` | `Optional[str]` | OAuth2 redirect URL for Swagger UI. Default: `"/docs/oauth2-redirect"`. |
| `swagger_ui_init_oauth` | `Optional[Dict]` | OAuth2 configuration for Swagger UI. Default: `None`. |
| `middleware` | `Optional[Sequence[Middleware]]` | A list of middleware to add on instantiation. |
| `on_startup` / `on_shutdown` | `Optional[Sequence[Callable]]` | Deprecated event handlers. Use `lifespan` instead. |
| `terms_of_service` | `Optional[str]` | A URL to the Terms of Service. |
| `root_path_in_servers` | `bool` | Whether to include the `root_path` in the OpenAPI `servers` field. Default: `True`. |
| `responses` | `Optional[Dict]` | Additional global responses for OpenAPI. |
| `callbacks` | `Optional[List[BaseRoute]]` | OpenAPI callbacks for all path operations. |
| `webhooks` | `Optional[APIRouter]` | An `APIRouter` for OpenAPI webhooks. |
| `deprecated` | `Optional[bool]` | Mark all path operations as deprecated. Default: `None`. |
| `include_in_schema` | `bool` | Whether to include all path operations in the OpenAPI schema. Default: `True`. |
| `swagger_ui_parameters` | `Optional[Dict]` | Parameters to configure Swagger UI. |
| `generate_unique_id_function` | `Callable[[APIRoute], str]` | Function to generate unique IDs for path operations. |
| `separate_input_output_schemas` | `bool` | Generate separate schemas for request and response models. Default: `True`. |

### Metadata and Documentation Configuration

You can configure the metadata for your API, which is used in the OpenAPI schema and the automatic documentation interfaces.

```python
from fastapi import FastAPI

tags_metadata = [
    {
        "name": "users",
        "description": "Operations with users.",
    },
    {
        "name": "items",
        "description": "Manage items.",
    },
]

app = FastAPI(
    title="ChimichangApp",
    description="ChimichangApp API helps you do awesome stuff. 🚀",
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

### Global Dependencies

You can add dependencies that will be applied to all *path operations* in the application.

```python
from fastapi import Depends, FastAPI, Header, HTTPException

async def verify_token(x_token: str = Header()):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")

app = FastAPI(dependencies=[Depends(verify_token)])

@app.get("/items/")
async def read_items():
    return [{"item": "Portal Gun"}, {"item": "Plumbus"}]
```

## Instance Attributes

A `FastAPI` instance has several attributes you can access or modify.

- `router` (`APIRouter`): The main router for the application. All path operations are registered here.
- `dependency_overrides` (`Dict`): A dictionary to override dependencies, mainly used for testing. For more details, see [Testing Dependencies with Overrides](./advanced-testing.md).
- `state` (`State`): An object to store arbitrary application state, inherited from Starlette.
- `openapi_schema` (`Optional[Dict]`): Caches the generated OpenAPI schema. The first time it's accessed, the schema is generated and stored here.
- `openapi_version` (`str`): The OpenAPI version string. Defaults to `"3.1.0"` but can be modified if needed for compatibility with older tools.
- `webhooks` (`APIRouter`): An `APIRouter` for documenting OpenAPI webhooks.

## Methods

### Path Operation Decorators

FastAPI uses decorators to associate functions with specific URL paths and HTTP methods. These decorators share a common set of parameters for configuration.

- `@app.get(path, **kwargs)`
- `@app.post(path, **kwargs)`
- `@app.put(path, **kwargs)`
- `@app.delete(path, **kwargs)`
- `@app.patch(path, **kwargs)`
- `@app.options(path, **kwargs)`
- `@app.head(path, **kwargs)`
- `@app.trace(path, **kwargs)`

**Common Parameters**

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | The URL path for the endpoint. |
| `response_model` | `Any` | The Pydantic model used for the response. |
| `status_code` | `int` | The default HTTP status code for the response. |
| `tags` | `List[str]` | A list of tags for grouping in the API docs. |
| `summary` | `str` | A short summary of the endpoint. |
| `description` | `str` | A detailed description, supporting Markdown. |
| `dependencies` | `Sequence[Depends]` | A list of dependencies specific to this endpoint. |
| `deprecated` | `bool` | Marks the endpoint as deprecated in the docs. |

**Example: `@app.post()`**

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

Includes an `APIRouter` in the application, which is useful for structuring larger applications. See [Bigger Applications](./advanced-bigger-applications.md) for more details.

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
    responses={404: {"description": "Not found"}}
)
```

### `@app.websocket`

Decorates a function to handle WebSocket connections.

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f"Message text was: {data}")
    except WebSocketDisconnect:
        print("Client disconnected")
```

### `@app.middleware`

Adds middleware to the application. The only supported type is `"http"`.

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

Registers a function to handle a specific exception type.

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

## Next Steps

Now that you are familiar with the main `FastAPI` application class, you might want to explore how to structure your application with routers.

<x-cards>
  <x-card data-title="Routing" data-icon="lucide:milestone" data-href="/api-reference/routing">
    Learn about APIRouter to organize your path operations into separate modules.
  </x-card>
  <x-card data-title="Bigger Applications" data-icon="lucide:layout-grid" data-href="/advanced/bigger-applications">
    Discover strategies for structuring large, production-ready applications.
  </x-card>
</x-cards>