# FastAPI Application

The `FastAPI` class is the main entry point of your application. It provides all the functionality for your API, inheriting from `starlette.applications.Starlette` but adding features like automatic documentation, data validation, and dependency injection.

## Basic Usage

Creating an instance of `FastAPI` is the first step in building your API. Here's a simple example:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}
```

This creates an application instance `app` and defines a single path operation for the root URL `/`.

## Application Configuration

The `FastAPI` application can be configured with several parameters during initialization to customize its behavior, metadata, and documentation.

```d2
direction: down

"FastAPI App": {
  shape: cloud
  "Configuration": {
    "Metadata": "title, description, version, etc."
    "API Docs": "docs_url, redoc_url, openapi_url"
    "Behavior": "dependencies, middleware, lifespan"
    "Routing": "routes, redirect_slashes"
  }

  "Core Components": {
    "Router": {
      "Path Operations": "@app.get(), @app.post(), etc."
      "Included Routers": "app.include_router()"
    }
    "Middleware Stack": "Processes requests/responses"
    "Dependency Injection": "Manages dependencies"
    "Exception Handlers": "Handles errors gracefully"
  }
  "Configuration" -> "Core Components": "Initializes"
}
```

### Initialization Parameters

Here is a comprehensive list of parameters available when creating a `FastAPI` instance:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `debug` | `bool` | `False` | Enable debug mode. If `True`, error tracebacks are returned in responses. |
| `routes` | `Optional[List[BaseRoute]]` | `None` | A list of Starlette routes. It's recommended to use path operation decorators like `@app.get()` instead. |
| `title` | `str` | `"FastAPI"` | The title of your API, which appears in the OpenAPI schema and documentation UIs. |
| `summary` | `Optional[str]` | `None` | A short summary of the API. |
| `description` | `str` | `""` | A detailed description of your API. Supports Markdown. |
| `version` | `str` | `"0.1.0"` | The version of your application's API (e.g., "1.2.0" or "v2-beta"). |
| `openapi_url` | `Optional[str]` | `"/openapi.json"` | The URL path for the OpenAPI schema. Set to `None` to disable it and the docs UIs. |
| `openapi_tags` | `Optional[List[Dict[str, Any]]]` | `None` | A list of dictionaries to define and order tags used in path operations for documentation. |
| `servers` | `Optional[List[Dict]]` | `None` | A list of server definitions for the OpenAPI schema, useful for specifying different environments (e.g., staging, production). |
| `dependencies` | `Optional[Sequence[Depends]]` | `None` | A list of global dependencies that will be applied to all path operations in the application. |
| `default_response_class` | `Type[Response]` | `JSONResponse` | The default response class to be used for path operations. |
| `redirect_slashes` | `bool` | `True` | Whether to automatically redirect requests if a path is accessed without a trailing slash. |
| `docs_url` | `Optional[str]` | `"/docs"` | The URL path for the interactive Swagger UI documentation. Set to `None` to disable. |
| `redoc_url` | `Optional[str]` | `"/redoc"` | The URL path for the ReDoc documentation. Set to `None` to disable. |
| `swagger_ui_oauth2_redirect_url` | `Optional[str]` | `"/docs/oauth2-redirect"` | The OAuth2 redirect URL for the Swagger UI. |
| `swagger_ui_init_oauth` | `Optional[Dict]` | `None` | A dictionary for configuring OAuth2 in Swagger UI. |
| `swagger_ui_parameters`| `Optional[Dict]` | `None` | A dictionary of parameters to customize the Swagger UI. |
| `middleware` | `Optional[Sequence[Middleware]]` | `None` | A sequence of Starlette middleware to add to the application. Using `app.add_middleware()` is more common. |
| `exception_handlers` | `Optional[Dict]` | `None` | A dictionary of exception handlers. Using the `@app.exception_handler()` decorator is preferred. |
| `on_startup` | `Optional[Sequence[Callable]]` | `None` | (Deprecated) A list of functions to run on application startup. Use `lifespan` instead. |
| `on_shutdown` | `Optional[Sequence[Callable]]` | `None` | (Deprecated) A list of functions to run on application shutdown. Use `lifespan` instead. |
| `lifespan` | `Optional[Lifespan]` | `None` | A context manager for handling startup and shutdown events. This is the recommended approach. |
| `terms_of_service` | `Optional[str]` | `None` | A URL to the terms of service for the API. |
| `contact` | `Optional[Dict]` | `None` | A dictionary with contact information for the API (e.g., `name`, `url`, `email`). |
| `license_info` | `Optional[Dict]` | `None` | A dictionary with license information for the API (e.g., `name`, `url`). |
| `root_path` | `str` | `""` | A path prefix for the application, useful when behind a reverse proxy. |
| `root_path_in_servers` | `bool` | `True` | If `True`, the `root_path` is automatically added to the `servers` list in the OpenAPI schema. |
| `responses` | `Optional[Dict]` | `None` | Additional global responses to include in all path operations in the OpenAPI schema. |
| `callbacks` | `Optional[List[BaseRoute]]` | `None` | A list of OpenAPI callbacks to apply to all path operations. |
| `webhooks` | `Optional[APIRouter]` | `None` | An `APIRouter` instance to declare OpenAPI webhooks. |
| `deprecated` | `Optional[bool]` | `None` | If `True`, marks all path operations in the application as deprecated. |
| `include_in_schema` | `bool` | `True` | Whether to include all path operations in the OpenAPI schema by default. |
| `generate_unique_id_function` | `Callable` | `generate_unique_id` | A function to generate unique IDs for each path operation in the OpenAPI schema. |
| `separate_input_output_schemas` | `bool` | `True` | Whether to generate separate schemas for input (request) and output (response) models. |

## Core Methods

### Path Operation Decorators

FastAPI uses decorators to define API endpoints. These decorators correspond to HTTP methods and are the primary way to add routes to your application.

- `@app.get()`
- `@app.post()`
- `@app.put()`
- `@app.delete()`
- `@app.patch()`
- `@app.options()`
- `@app.head()`
- `@app.trace()`

**Example:**

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

### Structuring Applications with Routers

The `include_router` method allows you to structure your application by splitting it into multiple `APIRouter` instances, which is essential for larger applications.

```python
from fastapi import FastAPI, APIRouter

app = FastAPI()

router = APIRouter()

@router.get("/users/")
def read_users():
    return [{"username": "user1"}, {"username": "user2"}]

app.include_router(router, prefix="/api/v1", tags=["users"])
```
For more details, see the documentation on [Bigger Applications](./advanced-bigger-applications.md).

### WebSockets

FastAPI provides first-class support for WebSockets via the `@app.websocket()` decorator.

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
For more advanced use cases, refer to the [WebSockets](./advanced-websockets.md) guide.

### Middleware

You can add middleware to your application to process every request before it reaches a path operation and every response before it's sent to the client.

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
Learn more about middleware in the [Advanced Middleware](./advanced-middleware.md) section.

### Exception Handlers

Custom exception handlers allow you to define how your application responds to specific exceptions.

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

### Lifespan Events

The `lifespan` context manager is the recommended way to handle logic that needs to run before the application starts up (e.g., initializing a database connection pool) and when it shuts down.

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

This reference provides a comprehensive overview of the `FastAPI` application class. For more detailed information on structuring your routes, please proceed to the [Routing](./api-reference-routing.md) API reference.