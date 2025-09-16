# Responses

FastAPI provides a flexible way to handle HTTP responses. While you can return dictionaries, lists, or Pydantic models directly and let FastAPI handle the conversion to a JSON response, you can also have full control by returning a `Response` object directly. This is particularly useful for setting custom headers, cookies, or returning non-JSON content types.

FastAPI re-exports several common response classes from Starlette for your convenience. For more narrative-driven examples, you might want to check the tutorial guide on [Custom Responses](https://fastapi.tiangolo.com/advanced/custom-response/).

## Standard Response Classes

These are the most commonly used response classes, directly available from `fastapi.responses`.

| Class | Description |
| --- | --- |
| `Response` | The base class for all responses. You can use it for custom responses with specific media types, headers, etc. |
| `HTMLResponse` | A response that automatically sets the `Content-Type` header to `text/html`. |
| `PlainTextResponse` | A response that sets the `Content-Type` header to `text/plain`. |
| `JSONResponse` | The default response class. It takes a Python object and returns a JSON-encoded response. |
| `RedirectResponse` | Returns an HTTP redirect (307 Temporary Redirect by default). |
| `StreamingResponse` | Takes an async generator or a regular generator/iterator and streams the response body. |
| `FileResponse` | Asynchronously streams a file as the response. |

### Example: Using HTMLResponse

You can return an HTML response directly from your path operation.

```python Using HTMLResponse icon=logos:python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()

@app.get("/", response_class=HTMLResponse)
async def read_root():
    return """
    <html>
        <head>
            <title>Some HTML in here</title>
        </head>
        <body>
            <h1>Look ma! HTML!</h1>
        </body>
    </html>
    """
```

## High-Performance JSON Responses

For applications requiring maximum performance, FastAPI provides alternative JSON response classes that leverage faster JSON libraries like `ujson` and `orjson`.

### UJSONResponse

This response class uses the `ujson` library for JSON serialization, which can be significantly faster than the standard `json` module.

To use it, first install `ujson`:

```bash Terminal icon=mdi:bash
pip install ujson
```

Then, you can set it as the default response class for your application or for a specific path operation.

```python Setting UJSONResponse as Default icon=logos:python
from fastapi import FastAPI
from fastapi.responses import UJSONResponse

app = FastAPI(default_response_class=UJSONResponse)

@app.get("/items/")
async def read_items():
    return [{"item_id": "Foo"}, {"item_id": "Bar"}]
```

### ORJSONResponse

This response class uses the `orjson` library, another high-performance alternative that is particularly good at serializing dataclasses, datetimes, and UUIDs.

To use it, first install `orjson`:

```bash Terminal icon=mdi:bash
pip install orjson
```

`ORJSONResponse` is configured to handle non-string keys and NumPy objects, making it very versatile.

```python Using ORJSONResponse for a specific endpoint icon=logos:python
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse
import numpy as np

app = FastAPI()

@app.get("/data", response_class=ORJSONResponse)
async def read_data():
    return {"numbers": np.array([1, 2, 3]).tolist(), 1: "integer_key"}
```

## JSON Compatible Encoder

FastAPI uses a special function, `jsonable_encoder`, to convert Python objects (like Pydantic models or `datetime` objects) into JSON-compatible data structures (like `dict` and `list`). This function is called internally before sending a `JSONResponse`, but you can also use it manually, for example, to prepare data before storing it in a database.

### Parameters

<x-field data-name="obj" data-type="Any" data-required="true" data-desc="The input object to convert to JSON."></x-field>
<x-field data-name="include" data-type="set | dict" data-required="false" data-desc="Pydantic's include parameter, to set the fields to include."></x-field>
<x-field data-name="exclude" data-type="set | dict" data-required="false" data-desc="Pydantic's exclude parameter, to set the fields to exclude."></x-field>
<x-field data-name="by_alias" data-type="bool" data-default="true" data-required="false" data-desc="If the output should use Pydantic model alias names."></x-field>
<x-field data-name="exclude_unset" data-type="bool" data-default="false" data-required="false" data-desc="Exclude fields that were not explicitly set (and only have default values)."></x-field>
<x-field data-name="exclude_defaults" data-type="bool" data-default="false" data-required="false" data-desc="Exclude fields that have the same value as the default, even if set explicitly."></x-field>
<x-field data-name="exclude_none" data-type="bool" data-default="false" data-required="false" data-desc="Exclude any fields that have a None value."></x-field>
<x-field data-name="custom_encoder" data-type="dict" data-required="false" data-desc="A dictionary of custom encoders for specific types."></x-field>
<x-field data-name="sqlalchemy_safe" data-type="bool" data-default="true" data-required="false" data-desc="Exclude any fields that start with _sa, for compatibility with SQLAlchemy objects."></x-field>

### Usage Example

Here's how you can use `jsonable_encoder` to convert a Pydantic model containing a `datetime` object into a dictionary with an ISO-formatted string.

```python Encoding a Pydantic Model icon=logos:python
from datetime import datetime
from pydantic import BaseModel
from fastapi.encoders import jsonable_encoder

class Item(BaseModel):
    title: str
    timestamp: datetime
    description: str | None = None

item_obj = Item(title="Foo", timestamp=datetime.now())

# Convert the Pydantic model to a dict
json_compatible_item_data = jsonable_encoder(item_obj)

# json_compatible_item_data will be something like:
# {
#   "title": "Foo",
#   "timestamp": "2023-10-27T10:00:00.123456",
#   "description": null
# }
print(json_compatible_item_data)
```

### Automatically Encoded Types

`jsonable_encoder` has built-in support for many common types that are not directly JSON-serializable.

| Original Type | Encoded To |
| --- | --- |
| `datetime.datetime` | `str` (ISO 8601) |
| `datetime.date` | `str` (ISO 8601) |
| `datetime.time` | `str` (ISO 8601) |
| `datetime.timedelta`| `float` (total seconds) |
| `UUID` | `str` |
| `Decimal` | `int` or `float` |
| `Enum` | The enum's value |
| `set`, `frozenset`, `deque` | `list` |
| `bytes` | `str` (decoded) |
| `Path` objects | `str` |
| Pydantic `SecretStr`, `SecretBytes` | `str` |
| Pydantic networking types | `str` |

For more advanced scenarios, such as handling exceptions and returning appropriate error responses, please refer to the [Exceptions](./api-reference-exceptions.md) documentation.