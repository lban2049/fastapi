# Responses

FastAPI provides a variety of response classes to send specific types of data, status codes, and headers. Most of these are inherited directly from Starlette, providing a robust and flexible system for crafting API responses. Additionally, FastAPI offers specialized classes that leverage high-performance JSON libraries for improved serialization speed.

For a more task-oriented guide on using responses, see the [Handling Responses](./user-guide-handling-responses.md) user guide.

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

### Response

The base `Response` class can be used to return any `bytes` or `str` content with a specific media type.

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/legacy-data")
def get_legacy_data():
    data = "<legacyformat>some_data</legacyformat>"
    return Response(content=data, media_type="application/xml")
```

### HTMLResponse

Use `HTMLResponse` to return an HTML string that the browser will render.

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

### PlainTextResponse

For returning simple text or any content that should be interpreted as plain text.

```python
from fastapi import FastAPI
from fastapi.responses import PlainTextResponse

app = FastAPI()

@app.get("/readme", response_class=PlainTextResponse)
async def get_readme():
    return "This is a plain text response."
```

### JSONResponse

This is the default response used by FastAPI. You can use it directly to return a JSON response, for example, when returning a dictionary from a path operation.

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.get("/items/")
async def read_items():
    return JSONResponse(content={"message": "Here are your items"})
```

### RedirectResponse

Performs an HTTP redirect. By default, it returns a `307 Temporary Redirect` status code.

```python
from fastapi import FastAPI
from fastapi.responses import RedirectResponse

app = FastAPI()

@app.get("/portal")
async def redirect_to_docs():
    return RedirectResponse(url="/docs")
```

### StreamingResponse

Streams the response body from an async generator or a standard generator/iterator. This is useful for large responses that you don't want to load into memory all at once.

```python
import asyncio
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

async def fake_video_streamer():
    for i in range(10):
        yield b"some chunk of data"
        await asyncio.sleep(0.1)

@app.get("/stream")
async def stream_data():
    return StreamingResponse(fake_video_streamer(), media_type="video/mp4")
```

### FileResponse

Asynchronously streams a file as the response. It is highly efficient for sending large files.

```python
from fastapi import FastAPI
from fastapi.responses import FileResponse

app = FastAPI()

# Assume you have a file named 'my_image.png' in the same directory
image_path = "my_image.png"

@app.get("/file")
async def get_file():
    return FileResponse(image_path, media_type="image/png")
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

`ORJSONResponse` uses the `orjson` library, another high-performance JSON library known for its speed and correctness. It supports serializing many types that standard libraries do not, such as dataclasses, `datetime`, `UUID`, and NumPy arrays, without extra configuration.

To use it, you first need to install `orjson`:

```bash
pip install orjson
```

Then, set it as the `response_class` in your path operation. It's particularly useful for data-intensive applications.

```python
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse
import numpy as np

app = FastAPI()

@app.get("/data", response_class=ORJSONResponse)
async def read_numpy_data():
    # orjson can serialize numpy arrays directly
    return {"matrix": np.arange(9).reshape(3, 3)}
```

This response class is configured with options (`OPT_NON_STR_KEYS | OPT_SERIALIZE_NUMPY`) to enable its advanced serialization features.