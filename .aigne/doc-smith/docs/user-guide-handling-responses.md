# Handling Responses

When building an API, you have fine-grained control over what is sent back to the client. FastAPI allows you to define the shape of the response, change the default HTTP status code, and set custom headers or cookies. This ensures your API is predictable, well-documented, and behaves exactly as clients expect.

This section covers the primary ways to manage your API's output.

## Use the `response_model` parameter

The most common way to control the response is by declaring a `response_model` in your *path operation decorator*. This model, typically a Pydantic model, serves several purposes:

- **Data Filtering**: It ensures the returned data conforms to the model's schema. Any data in your return object that is not defined in the `response_model` will be excluded.
- **Data Validation**: It validates the output data. If your return object has incorrect types (e.g., a `float` where an `int` is expected), FastAPI will raise an error.
- **Documentation**: It adds the response schema to your API's OpenAPI documentation, making it clear to users what data they should expect.

### Response Model for a Single Item

Here's how you can declare a `response_model` for an endpoint that creates an item. Even though the function receives and returns the same `item` object, the `response_model` guarantees the output matches the `Item` model's structure.

```python title="main.py" icon=logos:python
from typing import Any, List, Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: List[str] = []


@app.post("/items/", response_model=Item)
async def create_item(item: Item) -> Any:
    return item
```

FastAPI will use this `response_model` to filter, validate, and document the output.

### Response Model for a List of Items

You can also use type hints from Python's `typing` module, like `List`, in the `response_model`.

```python title="main.py" icon=logos:python
@app.get("/items/", response_model=List[Item])
async def read_items() -> Any:
    return [
        {"name": "Portal Gun", "price": 42.0},
        {"name": "Plumbus", "price": 32.0},
    ]
```

In this case, FastAPI will ensure the response is a JSON array where each object conforms to the `Item` model.

## Change the Status Code

By default, successful responses use the `200 OK` status code. You can easily override this by adding a `status_code` argument to the *path operation decorator*. This is particularly useful for creation endpoints, where a `201 Created` status code is more appropriate.

```python title="main.py" icon=logos:python
from fastapi import FastAPI

app = FastAPI()


@app.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}
```

Now, a successful POST request to `/items/` will return a `201 Created` status code.

## Set Custom Headers and Cookies

For more advanced control, such as setting custom headers or cookies, you can return a `Response` object directly. FastAPI provides several `Response` subclasses, with `JSONResponse` being the most common for APIs.

### Custom Headers

To add custom headers to your response, create a `JSONResponse` instance and pass the headers as a dictionary.

```python title="main.py" icon=logos:python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()


@app.get("/headers/")
def get_headers():
    content = {"message": "Hello World"}
    headers = {"X-Cat-Dog": "alone in the world", "Content-Language": "en-US"}
    return JSONResponse(content=content, headers=headers)
```

The client will now receive the custom `X-Cat-Dog` and `Content-Language` headers.

### Setting Cookies

Similarly, you can set cookies by creating a `JSONResponse` object and using its `set_cookie` method.

```python title="main.py" icon=logos:python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()


@app.post("/cookie/")
def create_cookie():
    content = {"message": "Come to the dark side, we have cookies"}
    response = JSONResponse(content=content)
    response.set_cookie(key="fakesession", value="fake-cookie-session-value")
    return response
```

## Return a Response Directly

Returning a `Response` object gives you full control. This is also useful when you need to serialize data types that are not native to JSON, such as `datetime` objects. FastAPI provides a `jsonable_encoder` utility for this purpose.

```python title="main.py" icon=logos:python
from datetime import datetime
from typing import Union

from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from fastapi.responses import JSONResponse
from pydantic import BaseModel


class Item(BaseModel):
    title: str
    timestamp: datetime
    description: Union[str, None] = None


app = FastAPI()


@app.put("/items/{id}")
def update_item(id: str, item: Item):
    json_compatible_item_data = jsonable_encoder(item)
    return JSONResponse(content=json_compatible_item_data)
```

Here, `jsonable_encoder` converts the `datetime` object in the `Item` model into a string format suitable for JSON before it's passed to `JSONResponse`.

## More Response Types

FastAPI, building on Starlette, provides a range of response classes for different use cases. You can import them directly from `fastapi.responses`.

<x-cards data-columns="3">
  <x-card data-title="JSONResponse" data-icon="lucide:code-json">The default response type for JSON data. Supports high-performance encoders.</x-card>
  <x-card data-title="HTMLResponse" data-icon="lucide:code">Used for returning HTML content directly to the browser.</x-card>
  <x-card data-title="PlainTextResponse" data-icon="lucide:file-text">For sending plain text responses.</x-card>
  <x-card data-title="RedirectResponse" data-icon="lucide:corner-up-right">Issues an HTTP redirect to a different URL.</x-card>
  <x-card data-title="StreamingResponse" data-icon="lucide:workflow">Streams the response body, useful for large files or real-time data.</x-card>
  <x-card data-title="FileResponse" data-icon="lucide:file">Streams a file from disk as the response.</x-card>
</x-cards>

For more detailed information, see the [API Reference for Responses](./api-reference-responses.md).

With these tools, you can precisely control every aspect of your API's responses. Next, you'll learn about a powerful system for managing dependencies and sharing logic.

---

Next, let's explore how to structure your code and handle dependencies with [Dependency Injection](./user-guide-dependency-injection.md).
