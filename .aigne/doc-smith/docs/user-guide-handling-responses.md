# Handling Responses

In FastAPI, you have fine-grained control over the response sent back to the client. This includes defining the data structure, setting the HTTP status code, and adding custom headers or cookies. This section covers the primary ways to manage your API's output.

## Define a Response Model

You can declare the model used for the response with the `response_model` parameter in any of the *path operation decorators*. FastAPI uses this `response_model` to:

- Convert the output data to the model's type definition.
- Validate the data.
- Add a JSON Schema for the response to the OpenAPI path operation.
- Filter the output data, so only the fields defined in the model will be included in the response.

### Example: Filtering Response Data

Here, the `response_model` is set to the `Item` model. Even if the function returns more data than defined in `Item`, FastAPI will filter it to match the model.

```python
from typing import Any, List, Union

from fastapi import FastAPI
from pydantic import BaseModel

ap = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: List[str] = []


@app.post("/items/", response_model=Item)
async def create_item(item: Item) -> Any:
    return item


@app.get("/items/", response_model=List[Item])
async def read_items() -> Any:
    return [
        {"name": "Portal Gun", "price": 42.0},
        {"name": "Plumbus", "price": 32.0},
    ]
```

In this example, `create_item` will return the item, but only the fields defined in the `Item` Pydantic model will be sent. For `read_items`, the response will be a list of objects, each conforming to the `Item` model.

## Change the Status Code

By default, path operations return a `200 OK` status code. You can override this for successful responses using the `status_code` parameter in the decorator.

```python
from fastapi import FastAPI

app = FastAPI()


@app.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}
```

Here, creating an item will now return a `201 Created` status code, which is the standard HTTP status for successfully creating a new resource.

## Use a Direct Response Object

For advanced scenarios like setting custom headers or cookies, you can return a `Response` object directly. FastAPI provides several helpers, like `JSONResponse`, to make this easier.

### Custom Headers

To add custom headers, you can create a `JSONResponse` and pass a dictionary of headers to it.

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()


@app.get("/headers/")
def get_headers():
    content = {"message": "Hello World"}
    headers = {"X-Cat-Dog": "alone in the world", "Content-Language": "en-US"}
    return JSONResponse(content=content, headers=headers)
```

The response from this endpoint will include the custom `X-Cat-Dog` and `Content-Language` headers.

### Set Cookies

To set a cookie, create a `JSONResponse` instance and then use its `set_cookie()` method.

```python
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

This gives you full control over cookie attributes like `key`, `value`, `domain`, `path`, etc.

## Handling Complex Data Types with `jsonable_encoder`

Sometimes you need to return data that contains non-JSON-compatible types, like `datetime` objects or Pydantic models. FastAPI provides the `jsonable_encoder` utility to convert such data into a JSON-compatible structure.

```python
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

Here, `jsonable_encoder` converts the `item` object, including its `timestamp` field, into a dictionary with a string representation of the datetime. This dictionary can then be safely passed as content to `JSONResponse`.

## Other Response Types

FastAPI offers a variety of response classes for different needs, inheriting directly from Starlette. You can return an instance of any of these directly from your path operation.

| Class               | Description                                                 |
|---------------------|-------------------------------------------------------------|
| `Response`          | The base class, can be used for custom responses.           |
| `JSONResponse`      | The default, for JSON-encoded data.                         |
| `HTMLResponse`      | For returning HTML content.                                 |
| `PlainTextResponse` | For returning plain text.                                   |
| `RedirectResponse`  | For sending an HTTP redirect (307).                         |
| `StreamingResponse` | For streaming a response body.                              |
| `FileResponse`      | For streaming a file as the response.                       |

For high-performance applications, you can also use `UJSONResponse` or `ORJSONResponse` after installing the respective libraries (`ujson` or `orjson`).

## Next Steps

You now have the tools to control every aspect of your API's response. To learn how to manage shared logic and dependencies, proceed to the next section on [Dependency Injection](./user-guide-dependency-injection.md).
