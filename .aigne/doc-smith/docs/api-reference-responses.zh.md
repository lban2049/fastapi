# 响应

FastAPI 提供了多种响应类，用于发送特定类型的数据、状态码和标头。其中大部分直接继承自 Starlette，为构建 API 响应提供了一个强大而灵活的系统。此外，FastAPI 还提供了利用高性能 JSON 库来提高序列化速度的专用类。

有关使用响应的更面向任务的指南，请参阅[用户指南 - 处理响应](./user-guide-handling-responses.md)。

## 标准响应类

这些是可用于常见用例的核心响应类。它们都从 `starlette.responses` 导入，并由 `fastapi.responses` 重新导出，以方便使用。

| Class | Description |
|---|---|
| `Response` | 所有响应对象的基类。可用于包含原始字节的自定义响应。 |
| `HTMLResponse` | 用于返回媒体类型为 `text/html` 的内容。 |
| `PlainTextResponse` | 用于返回媒体类型为 `text/plain` 的内容。 |
| `JSONResponse` | 路径操作的默认响应。将 Python `dict` 或 Pydantic 模型序列化为 JSON。 |
| `RedirectResponse` | 用于通过返回 `307` 状态码和 `Location` 标头来执行 HTTP 重定向。 |
| `StreamingResponse` | 从异步生成器或普通生成器/迭代器中流式传输响应正文内容。 |
| `FileResponse` | 异步地将文件作为响应进行流式传输。 |

### 示例：使用 HTMLResponse

你可以直接在路径操作装饰器中指定响应类。

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

## 高性能 JSON 响应

对于需要尽可能快的 JSON 序列化的应用，FastAPI 提供了与 `ujson` 和 `orjson` 集成的响应类。

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

`ORJSONResponse` 使用 `orjson` 库，这是另一个以速度和正确性著称的高性能 JSON 库。它支持序列化许多标准库不支持的类型，如 dataclasses、`datetime`、`UUID` 和 NumPy 数组，无需额外配置。

要使用它，首先需要安装 `orjson`：

```bash
pip install orjson
```

然后，在路径操作中将其设置为 `response_class`。它对于数据密集型应用（例如，使用 NumPy 的应用）尤其有用。

```python
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse
import numpy as np

app = FastAPI()

@app.get("/data", response_class=ORJSONResponse)
async def read_numpy_data():
    return {"matrix": np.arange(9).reshape(3, 3)}
```

该响应类配置了选项 (`OPT_NON_STR_KEYS | OPT_SERIALIZE_NUMPY`) 以启用其高级序列化功能。