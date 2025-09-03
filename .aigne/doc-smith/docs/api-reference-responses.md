# Responses

FastAPI provides a variety of response classes to send specific types of data, status codes, and headers. Most of these are inherited directly from Starlette, providing a robust and flexible system for crafting API responses. Additionally, FastAPI offers specialized classes that leverage high-performance JSON libraries for improved serialization speed.

For a more task-oriented guide on using responses, see the [User Guide - Handling Responses](./user-guide-handling-responses.md).

## Standard Response Classes

These are the core response classes available for common use cases. They are all imported from `starlette.responses` and re-exported by `fastapi.responses` for convenience.

| Class | Description |
|---|---|
| `Response` | The base class for all response objects. Can be used for custom responses with raw bytes. |
| `HTMLResponse` | Used for returning content with a `text/html` media type. |
| `PlainTextResponse` | Used for returning content with a `text/plain` media type. |
| `JSONResponse` | The default response for path operations. Serializes Python `dict` or Pydantic models to JSON. |
| `RedirectResponse` | Used to perform an HTTP redirect by returning a `307` status code and a `Location` header. |
| `StreamingResponse` | Streams response body content from an async generator or a normal generator/iterator. |
| `FileResponse` | Asynchronously streams a file as the response. |

### Example: Using `HTMLResponse`

You can specify the response class directly in your path operation decorator.

```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()

@app.get("/", response_class=HTMLResponse)
async def get_html():
    return """
    <html>
        <head>
            <title>My Cool App</title>
        </head>
        <body>
            <h1>Welcome!</h1>
        </body>
    </html>
    """
```

## High-Performance JSON Responses

For applications that require the fastest possible JSON serialization, FastAPI provides response classes that integrate with `ujson` and `orjson`.

### UJSONResponse

`UJSONResponse` uses the `ujson` library to serialize data, which can be significantly faster than the standard `json` library.

To use it, you first need to install `ujson`:

```bash
pip install ujson
```

Then, use it as your `response_class`:

```python
from fastapi import FastAPI
from fastapi.responses import UJSONResponse

app = FastAPI()

@app.get("/items", response_class=UJSONResponse)
async def read_items():
    return [{"item_id": "item1"}, {"item_id": "item2"}]
```

### ORJSONResponse

`ORJSONResponse` uses the `orjson` library, another high-performance JSON library that is known for its speed and correctness. It supports serializing many types that standard libraries do not, such as dataclasses, `datetime`, `UUID`, and NumPy arrays, without extra configuration.

To use it, you first need to install `orjson`:

```bash
pip install orjson
```

Then, set it as the `response_class` in your path operation. It's particularly useful for data-intensive applications, for example, with NumPy.

```python
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse
import numpy as np

app = FastAPI()

@app.get("/data", response_class=ORJSONResponse)
async def read_numpy_data():
    return {"matrix": np.arange(9).reshape(3, 3)}
```

This response class is configured with options (`OPT_NON_STR_KEYS | OPT_SERIALIZE_NUMPY`) to enable its advanced serialization features.