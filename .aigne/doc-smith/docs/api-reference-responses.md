# Responses

FastAPI provides a collection of response classes to return specific types of data and to customize headers, cookies, and status codes. These classes are inherited from Starlette and offer flexibility for various use cases. While you can return data directly and let FastAPI handle the conversion, using these response classes gives you more control.

For practical examples on how to use these in your path operations, refer to the [Handling Responses](./user-guide-handling-responses.md) user guide.

## Standard Response Classes

FastAPI includes several standard response classes suitable for common web development needs. The following classes are available directly from `fastapi.responses`.

| Class | Description |
|---|---|
| `Response` | The base response class. It can take `content` as bytes or a string and allows for manual setting of `media_type`, `status_code`, and `headers`. |
| `HTMLResponse` | A response for returning HTML content. It automatically sets the `Content-Type` header to `text/html`. |
| `PlainTextResponse` | Used for returning plain text. It sets the `Content-Type` header to `text/plain`. |
| `JSONResponse` | The default response type for most FastAPI operations. It encodes a given data structure into a JSON string and sets the `Content-Type` header to `application/json`. |
| `RedirectResponse` | Returns an HTTP redirect. By default, it uses a 307 Temporary Redirect status code. |
| `StreamingResponse` | Streams the response body. This is useful for large responses that you don't want to load into memory all at once, such as generating a large CSV file. |
| `FileResponse` | A specialized streaming response for sending a file from a specified path. It infers the media type from the filename extension and adds appropriate headers like `Content-Disposition`. |

### Example: Using `HTMLResponse`

You can return an `HTMLResponse` directly from your path operation function.

```python
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

For applications that require maximum performance, FastAPI provides alternative JSON response classes that leverage faster JSON libraries.

### UJSONResponse

`UJSONResponse` uses the `ujson` library for high-performance JSON serialization. It can be significantly faster than the standard library's `json` module.

To use it, you must first install `ujson`:

```bash
pip install ujson
```

Then, you can use it in your application by setting the `response_class` parameter in your path operation decorator.

```python
from fastapi import FastAPI
from fastapi.responses import UJSONResponse

app = FastAPI()

@app.get("/items/", response_class=UJSONResponse)
async def read_items():
    return [{"item_id": "Foo"}]
```

This class overrides the standard `JSONResponse` to use `ujson.dumps` for serialization, which can provide a noticeable speed boost for JSON-heavy APIs.

### ORJSONResponse

`ORJSONResponse` provides another high-performance alternative using the `orjson` library. `orjson` is known for its exceptional speed and its ability to correctly and quickly serialize common data types like datetimes and dataclasses without extra configuration.

First, install `orjson`:

```bash
pip install orjson
```

Then, use it as the `response_class`.

```python
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse
import datetime

app = FastAPI()

@app.get("/data/", response_class=ORJSONResponse)
async def read_data():
    return {"timestamp": datetime.datetime.now(), "status": "ok"}

```

`ORJSONResponse` is configured to handle non-string keys and serialize NumPy arrays, making it a robust choice for data-intensive applications.

---

This reference covers the response classes available in FastAPI. To continue exploring the API reference, proceed to the [Security Utilities](./api-reference-security.md) section.