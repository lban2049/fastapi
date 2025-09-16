# FastAPI Class

The `FastAPI` class is the main entry point to create and manage your API. It provides all the functionality for your API, inheriting from `starlette.applications.Starlette`.

When you create an instance of the `FastAPI` class, you can specify various configuration options to customize your application's behavior, metadata, documentation, and more.

### First Steps Example

Creating a `FastAPI` application is as simple as importing the class and creating an instance:

```python First Steps icon=logos:python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}
```

For a complete guide on getting started, check out the [Tutorial - First Steps](./tutorials-first-steps.md).

## Parameters

The `FastAPI` class constructor accepts several parameters to configure your application. Here are the most important ones:

<x-field data-name="title" data-type="str" data-default="FastAPI" data-required="false" data-desc="The title of your API. This is displayed in the automatic API documentation (e.g., Swagger UI at /docs)."></x-field>

```python Title Example icon=logos:python
from fastapi import FastAPI

app = FastAPI(title="ChimichangApp")
```

<x-field data-name="version" data-type="str" data-default="0.1.0" data-required="false" data-desc="The version of your API. This is the version of your application, not the OpenAPI specification version. It's also displayed in the API docs."></x-field>

```python Version Example icon=logos:python
from fastapi import FastAPI

app = FastAPI(version="0.0.1")
```

<x-field data-name="description" data-type="str" data-default="" data-required="false" data-desc="A description of your API. It supports Markdown and is shown in the API docs."></x-field>

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

<x-field data-name="dependencies" data-type="Optional[Sequence[Depends]]" data-default="None" data-required="false" data-desc="A list of global dependencies that will be applied to every path operation in the application, including those in sub-routers."></x-field>

```python Global Dependencies Example icon=logos:python
from fastapi import Depends, FastAPI
from .dependencies import func_dep_1, func_dep_2

app = FastAPI(dependencies=[Depends(func_dep_1), Depends(func_dep_2)])
```

<x-field data-name="debug" data-type="bool" data-default="False" data-required="false" data-desc="A boolean indicating if debug tracebacks should be returned on server errors."></x-field>

<x-field data-name="openapi_url" data-type="Optional[str]" data-default="/openapi.json" data-required="false" data-desc="The URL path from which the OpenAPI schema will be served. Set to `None` to disable the OpenAPI schema and the automatic documentation UIs."></x-field>

<x-field data-name="docs_url" data-type="Optional[str]" data-default="/docs" data-required="false" data-desc="The URL path for the interactive Swagger UI documentation. Set to `None` to disable it."></x-field>

<x-field data-name="redoc_url" data-type="Optional[str]" data-default="/redoc" data-required="false" data-desc="The URL path for the alternative ReDoc documentation. Set to `None` to disable it."></x-field>

<x-field data-name="default_response_class" data-type="Type[Response]" data-default="JSONResponse" data-required="false" data-desc="The default response class to be used for path operations. You can use this to set a different default JSON renderer, for example."></x-field>

```python Default Response Class Example icon=logos:python
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse

app = FastAPI(default_response_class=ORJSONResponse)
```

<x-field data-name="lifespan" data-type="Optional[Lifespan[AppType]]" data-default="None" data-required="false" data-desc="A lifespan context manager to handle startup and shutdown events. This is the recommended way to manage application startup and shutdown logic."></x-field>

<x-field data-name="root_path" data-type="str" data-default="" data-required="false" data-desc="A path prefix handled by a proxy that is not seen by the application but is necessary for generating correct URLs in the documentation."></x-field>

<x-field data-name="openapi_tags" data-type="Optional[List[Dict[str, Any]]]" data-default="None" data-required="false" data-desc="A list of dictionaries that provide metadata for the tags used in path operations. This can be used to control the order and description of tags in the API docs."></x-field>

<x-field data-name="servers" data-type="Optional[List[Dict[str, Union[str, Any]]]]" data-default="None" data-required="false" data-desc="A list of server definitions for the OpenAPI schema, useful when your API is available at multiple URLs."></x-field>

<x-field data-name="exception_handlers" data-type="Optional[Dict[...]]" data-default="None" data-required="false" data-desc="A dictionary of exception handlers. It's more common to use the `@app.exception_handler()` decorator."></x-field>

<x-field data-name="middleware" data-type="Optional[Sequence[Middleware]]" data-default="None" data-required="false" data-desc="A list of Starlette middleware to add to the application. It's more common to use `app.add_middleware()` or the `@app.middleware('http')` decorator."></x-field>

## Attributes

An instance of the `FastAPI` class has several useful attributes:

<x-field data-name="router" data-type="APIRouter" data-desc="The main router for the application. All path operations are registered on this router."></x-field>
<x-field data-name="dependency_overrides" data-type="Dict[Callable, Callable]" data-desc="A dictionary to override dependencies, mainly used for testing purposes."></x-field>
<x-field data-name="state" data-type="State" data-desc="A Starlette state object for storing arbitrary application-level state. While available, using dependencies is often a better approach in FastAPI."></x-field>
<x-field data-name="openapi_schema" data-type="Optional[Dict[str, Any]]" data-desc="The cached OpenAPI schema. It's generated the first time it's needed and then stored here. You can modify this dictionary to customize the OpenAPI schema."></x-field>
<x-field data-name="webhooks" data-type="APIRouter" data-desc="An APIRouter dedicated to path operations that will be documented as OpenAPI webhooks."></x-field>

## Methods

### Path Operation Decorators

The primary way to add routes to your application is by using path operation decorators. These decorators correspond to HTTP methods (`GET`, `POST`, `PUT`, `DELETE`, etc.).

#### @app.get()

This decorator registers a function to handle HTTP GET requests for a given path.

```python GET Example icon=logos:python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/")
def read_items():
    return [{"name": "Empanada"}, {"name": "Arepa"}]
```

The `.get()` decorator, and all other path operation decorators (`.post()`, `.put()`, etc.), accept numerous parameters to configure the route. Here are some of the most common:

<x-field data-name="path" data-type="str" data-required="true" data-desc="The URL path for this path operation."></x-field>
<x-field data-name="response_model" data-type="Any" data-required="false" data-desc="A Pydantic model or type to be used for the response. It ensures the response data conforms to this model, filtering out any extra data."></x-field>
<x-field data-name="status_code" data-type="Optional[int]" data-required="false" data-desc="The default HTTP status code for the response."></x-field>
<x-field data-name="tags" data-type="Optional[List[Union[str, Enum]]]" data-required="false" data-desc="A list of tags to group this path operation in the API documentation."></x-field>
<x-field data-name="summary" data-type="Optional[str]" data-required="false" data-desc="A short summary for the path operation, shown in the API docs."></x-field>
<x-field data-name="description" data-type="Optional[str]" data-required="false" data-desc="A longer description for the path operation, supporting Markdown. If not provided, it's taken from the function's docstring."></x-field>
<x-field data-name="deprecated" data-type="Optional[bool]" data-required="false" data-desc="Marks this path operation as deprecated in the API documentation."></x-field>
<x-field data-name="dependencies" data-type="Optional[Sequence[Depends]]" data-required="false" data-desc="A list of dependencies specific to this path operation."></x-field>

Other path operation decorators like `@app.post()`, `@app.put()`, `@app.delete()`, `@app.patch()`, `@app.options()`, and `@app.head()` share the same configuration parameters.

### include_router()

Includes an `APIRouter` in the application, which is essential for structuring larger applications across multiple files.

```python Include Router Example icon=logos:python
from fastapi import FastAPI
from .users import users_router

app = FastAPI()

app.include_router(users_router)
```

**Parameters**

<x-field data-name="router" data-type="APIRouter" data-required="true" data-desc="The APIRouter instance to include."></x-field>
<x-field data-name="prefix" data-type="str" data-default="" data-required="false" data-desc="A URL prefix to apply to all routes in the router."></x-field>
<x-field data-name="tags" data-type="Optional[List[Union[str, Enum]]]" data-required="false" data-desc="Tags to apply to all routes in the router."></x-field>
<x-field data-name="dependencies" data-type="Optional[Sequence[Depends]]" data-required="false" data-desc="Dependencies to apply to all routes in the router."></x-field>
<x-field data-name="responses" data-type="Optional[Dict[...]]" data-required="false" data-desc="Additional responses to apply to all routes in the router for documentation."></x-field>

For more details, see the documentation on [Bigger Applications](./tutorials-advanced-bigger-applications.md).

### @app.websocket()

A decorator to create a WebSocket endpoint.

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

**Parameters**

<x-field data-name="path" data-type="str" data-required="true" data-desc="The URL path for the WebSocket."></x-field>
<x-field data-name="name" data-type="Optional[str]" data-required="false" data-desc="An internal name for the WebSocket route."></x-field>
<x-field data-name="dependencies" data-type="Optional[Sequence[Depends]]" data-required="false" data-desc="A list of dependencies for this WebSocket."></x-field>

### @app.middleware()

A decorator to add middleware to the application. Currently, only `"http"` middleware is supported this way.

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

A decorator to register a custom exception handler for the application.

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

Generates and returns the OpenAPI schema for the application. It caches the result, so subsequent calls are fast. You can override this method to customize the generated schema.

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

Now that you understand the main `FastAPI` application class, a great next step is to learn how to structure your application using the [APIRouter](./api-reference-apirouter.md).