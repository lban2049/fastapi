# 响应

FastAPI 提供了多种响应类，用于发送特定类型的数据、状态码和标头。这些类大部分直接继承自 Starlette，为创建 API 响应提供了一个强大而灵活的系统。此外，FastAPI 还提供了利用高性能 JSON 库的专用类，以提高序列化速度。

如需了解更多关于使用响应的面向任务的指南，请参阅 [处理响应](./user-guide-handling-responses.md) 用户指南。

## 标准响应类

这些是可用于常见用例的核心响应类。它们都从 `starlette.responses` 中导入，并为方便起见由 `fastapi.responses` 重新导出。

| 类 | 描述 |
|---|---|
| `Response` | 所有响应对象的基类。可用于包含原始字节的自定义响应。 |
| `HTMLResponse` | 用于返回媒体类型为 `text/html` 的内容。 |
| `PlainTextResponse` | 用于返回媒体类型为 `text/plain` 的内容。 |
| `JSONResponse` | 路径操作的默认响应。将 Python `dict` 或 Pydantic 模型序列化为 JSON。 |
| `RedirectResponse` | 用于执行 HTTP 重定向，返回 `307` 状态码和 `Location` 标头。 |
| `StreamingResponse` | 从异步生成器或普通生成器/迭代器流式传输响应正文内容。 |
| `FileResponse` | 以异步方式将文件作为响应流式传输。 |

### Response

基类 `Response` 可用于返回带有特定媒体类型的任何 `bytes` 或 `str` 内容。

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/legacy-data")
def get_legacy_data():
    data = "<legacyformat>some_data</legacyformat>"
    return Response(content=data, media_type="application/xml")
```

### HTMLResponse

使用 `HTMLResponse` 返回浏览器将渲染的 HTML 字符串。

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

用于返回应被解释为纯文本的简单文本或任何内容。

```python
from fastapi import FastAPI
from fastapi.responses import PlainTextResponse

app = FastAPI()

@app.get("/readme", response_class=PlainTextResponse)
async def get_readme():
    return "This is a plain text response."
```

### JSONResponse

这是 FastAPI 使用的默认响应。你可以直接使用它来返回 JSON 响应，例如，从路径操作返回字典时。

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.get("/items/")
async def read_items():
    return JSONResponse(content={"message": "Here are your items"})
```

### RedirectResponse

执行 HTTP 重定向。默认情况下，它返回 `307 Temporary Redirect` 状态码。

```python
from fastapi import FastAPI
from fastapi.responses import RedirectResponse

app = FastAPI()

@app.get("/portal")
async def redirect_to_docs():
    return RedirectResponse(url="/docs")
```

### StreamingResponse

从异步生成器或标准生成器/迭代器流式传输响应正文。这对于不希望一次性加载到内存中的大型响应非常有用。

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

以异步方式将文件作为响应流式传输。对于发送大文件而言，它非常高效。

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

## 高性能 JSON 响应

对于需要尽可能快的 JSON 序列化的应用程序，FastAPI 提供了与 `ujson` 和 `orjson` 集成的响应类。

### UJSONResponse

`UJSONResponse` 使用 `ujson` 库来序列化数据，其速度可能远快于标准 `json` 库。

要使用它，首先需要安装 `ujson`：

```bash
pip install ujson
```

然后，将其用作 `response_class`：

```python
from fastapi import FastAPI
from fastapi.responses import UJSONResponse

app = FastAPI()

@app.get("/items", response_class=UJSONResponse)
async def read_items():
    return [{"item_id": "item1"}, {"item_id": "item2"}]
```

### ORJSONResponse

`ORJSONResponse` 使用 `orjson` 库，这是另一个以速度和正确性著称的高性能 JSON 库。它支持序列化许多标准库不支持的类型，例如 dataclasses、`datetime`、`UUID` 和 NumPy 数组，无需额外配置。

要使用它，首先需要安装 `orjson`：

```bash
pip install orjson
```

然后，在路径操作中将其设置为 `response_class`。它对数据密集型应用程序尤其有用。

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

该响应类配置了选项 (`OPT_NON_STR_KEYS | OPT_SERIALIZE_NUMPY`) 以启用其高级序列化功能。