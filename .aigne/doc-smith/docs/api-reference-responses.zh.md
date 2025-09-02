# 响应

FastAPI 提供了一系列响应类，用于返回特定类型的数据以及自定义标头、Cookie 和状态码。这些类继承自 Starlette，为各种用例提供了灵活性。虽然你可以直接返回数据并让 FastAPI 处理转换，但使用这些响应类可以让你拥有更多控制权。

若想了解如何在路径操作中使用这些响应类的实际示例，请参阅 [Handling Responses](./user-guide-handling-responses.md) 用户指南。

## 标准响应类

FastAPI 包含多个标准响应类，适用于常见的 Web 开发需求。以下类可直接从 `fastapi.responses` 中获取。

| Class | Description |
|---|---|
| `Response` | 基础响应类。它可以接受字节或字符串形式的 `content`，并允许手动设置 `media_type`、`status_code` 和 `headers`。 |
| `HTMLResponse` | 用于返回 HTML 内容的响应。它会自动将 `Content-Type` 标头设置为 `text/html`。 |
| `PlainTextResponse` | 用于返回纯文本。它会将 `Content-Type` 标头设置为 `text/plain`。 |
| `JSONResponse` | 大多数 FastAPI 操作的默认响应类型。它将给定的数据结构编码为 JSON 字符串，并将 `Content-Type` 标头设置为 `application/json`。 |
| `RedirectResponse` | 返回 HTTP 重定向。默认情况下，它使用 307 临时重定向状态码。 |
| `StreamingResponse` | 以流式传输响应正文。这对于不想一次性加载到内存中的大型响应（例如生成大型 CSV 文件）非常有用。 |
| `FileResponse` | 一种专门用于从指定路径发送文件的流式响应。它会根据文件名扩展名推断媒体类型，并添加 `Content-Disposition` 等适当的标头。 |

### 示例：使用 `HTMLResponse`

你可以直接从路径操作函数返回一个 `HTMLResponse`。

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

## 高性能 JSON 响应

对于需要最高性能的应用程序，FastAPI 提供了可利用更快 JSON 库的替代 JSON 响应类。

### UJSONResponse

`UJSONResponse` 使用 `ujson` 库进行高性能 JSON 序列化。它比标准库的 `json` 模块快得多。

要使用它，你必须先安装 `ujson`：

```bash
pip install ujson
```

然后，你可以在应用程序中通过设置路径操作装饰器中的 `response_class` 参数来使用它。

```python
from fastapi import FastAPI
from fastapi.responses import UJSONResponse

app = FastAPI()

@app.get("/items/", response_class=UJSONResponse)
async def read_items():
    return [{"item_id": "Foo"}]
```

该类重写了标准的 `JSONResponse`，使用 `ujson.dumps` 进行序列化，这可以为重度依赖 JSON 的 API 提供显著的速度提升。

### ORJSONResponse

`ORJSONResponse` 使用 `orjson` 库提供了另一种高性能替代方案。`orjson` 以其卓越的速度和无需额外配置即可正确、快速地序列化 datetime 和 dataclass 等常见数据类型的能力而闻名。

首先，安装 `orjson`：

```bash
pip install orjson
```

然后，将其用作 `response_class`。

```python
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse
import datetime

app = FastAPI()

@app.get("/data/", response_class=ORJSONResponse)
async def read_data():
    return {"timestamp": datetime.datetime.now(), "status": "ok"}

```

`ORJSONResponse` 经过配置，可以处理非字符串键并序列化 NumPy 数组，使其成为数据密集型应用的稳健选择。

---

本参考涵盖了 FastAPI 中可用的响应类。要继续浏览 API 参考，请前往 [Security Utilities](./api-reference-security.md) 部分。