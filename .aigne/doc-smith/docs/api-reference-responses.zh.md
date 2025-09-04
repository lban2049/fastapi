# 响应

FastAPI 提供了多种响应类，用于发送特定类型的数据、状态码和标头。这些类大多直接继承自 Starlette，为构建 API 响应提供了一个健壮而灵活的系统。此外，FastAPI 还提供了利用高性能 JSON 库来提高序列化速度的专用类。

如需更侧重于任务的响应使用指南，请参阅[用户指南 - 处理响应](./user-guide-handling-responses.md)。

## 标准响应类

这些是适用于常见用例的核心响应类。为方便起见，它们都从 `starlette.responses` 导入，并由 `fastapi.responses` 重新导出。

| Class | Description |
|---|---|
| `Response` | 所有响应对象的基类。可用于包含原始字节的自定义响应。 |
| `HTMLResponse` | 用于返回媒体类型为 `text/html` 的内容。 |
| `PlainTextResponse` | 用于返回媒体类型为 `text/plain` 的内容。 |
| `JSONResponse` | 路径操作的默认响应。将 Python `dict` 或 Pydantic 模型序列化为 JSON。 |
| `RedirectResponse` | 用于通过返回 `307` 状态码和 `Location` 标头来执行 HTTP 重定向。 |
| `StreamingResponse` | 从异步生成器或普通生成器/迭代器中流式传输响应正文内容。 |
| `FileResponse` | 异步地将文件作为响应流式传输。 |

### Response

`Response` 基类可用于返回任何带有特定媒体类型的 `bytes` 或 `str` 内容。

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/legacy-data")
def get_legacy_data():
    data = "<legacyformat>some_data</legacyformat>"
    return Response(content=data, media_type="application/xml")
```

### HTMLResponse

使用 `HTMLResponse` 返回一个 HTML 字符串，浏览器将对其进行渲染。

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

用于返回纯文本或任何应被解释为纯文本的内容。

```python
from fastapi import FastAPI
from fastapi.responses import PlainTextResponse

app = FastAPI()

@app.get("/readme", response_class=PlainTextResponse)
async def get_readme():
    return "This is a plain text response."
```

### RedirectResponse

执行 HTTP 重定向。默认情况下，它会返回 `307 Temporary Redirect` 状态码。

```python
from fastapi import FastAPI
from fastapi.responses import RedirectResponse

app = FastAPI()

@app.get("/docs")
async def redirect_to_swagger():
    return RedirectResponse(url="/docs/index.html")
```

### StreamingResponse

从异步生成器或标准生成器/迭代器中流式传输响应正文。这对于不想一次性将全部内容加载到内存中的大型响应非常有用。

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

异步地将文件作为响应流式传输。它对于发送大文件非常高效。

```python
from fastapi import FastAPI
from fastapi.responses import FileResponse

app = FastAPI()

# 假设在同一目录下有一个名为 'my_image.png' 的文件
image_path = "my_image.png"

@app.get("/file")
async def get_file():
    return FileResponse(image_path, media_type="image/png")
```

## 高性能 JSON 响应

对于需要尽可能快的 JSON 序列化的应用程序，FastAPI 提供了与 `ujson` 和 `orjson` 集成的响应类。

### UJSONResponse

`UJSONResponse` 使用 `ujson` 库序列化数据，其速度可能远快于标准的 `json` 库。

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

`ORJSONResponse` 使用 `orjson` 库，这是另一个以其速度和正确性而闻名的高性能 JSON 库。它支持序列化许多标准库不支持的类型，例如 dataclasses、`datetime`、`UUID` 和 NumPy 数组，且无需额外配置。

要使用它，首先需要安装 `orjson`：

```bash
pip install orjson
```

然后，在路径操作中将其设置为 `response_class`。它对于数据密集型应用程序特别有用。

```python
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse
import numpy as np

app = FastAPI()

@app.get("/data", response_class=ORJSONResponse)
async def read_numpy_data():
    # orjson 可以直接序列化 numpy 数组
    return {"matrix": np.arange(9).reshape(3, 3)}
```

该响应类配置了选项 (`OPT_NON_STR_KEYS | OPT_SERIALIZE_NUMPY`) 以启用其高级序列化功能。