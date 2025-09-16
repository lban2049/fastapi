# 响应

FastAPI 提供了一种处理 HTTP 响应的灵活方式。虽然你可以直接返回字典、列表或 Pydantic 模型，并让 FastAPI 处理到 JSON 响应的转换，但你也可以通过直接返回 `Response` 对象来获得完全的控制权。这对于设置自定义标头、Cookie 或返回非 JSON 内容类型特别有用。

为方便起见，FastAPI 从 Starlette 中重新导出了几个常见的响应类。如需更多叙述性的示例，你可能需要查看关于[自定义响应](https://fastapi.tiangolo.com/advanced/custom-response/)的教程指南。

## 标准响应类

这些是最常用的响应类，可直接从 `fastapi.responses` 中获取。

| 类 | 描述 |
| --- | --- |
| `Response` | 所有响应的基类。你可以用它来创建具有特定媒体类型、标头等的自定义响应。 |
| `HTMLResponse` | 一种自动将 `Content-Type` 标头设置为 `text/html` 的响应。 |
| `PlainTextResponse` | 一种将 `Content-Type` 标头设置为 `text/plain` 的响应。 |
| `JSONResponse` | 默认的响应类。它接收一个 Python 对象并返回一个 JSON 编码的响应。 |
| `RedirectResponse` | 返回一个 HTTP 重定向（默认为 307 临时重定向）。 |
| `StreamingResponse` | 接收一个异步生成器或一个常规的生成器/迭代器，并流式传输响应体。 |
| `FileResponse` | 异步地将文件作为响应进行流式传输。 |

### 示例：使用 HTMLResponse

你可以直接从你的路径操作中返回 HTML 响应。

```python 使用 HTMLResponse icon=logos:python
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

对于要求最高性能的应用程序，FastAPI 提供了利用 `ujson` 和 `orjson` 等更快 JSON 库的替代 JSON 响应类。

### UJSONResponse

该响应类使用 `ujson` 库进行 JSON 序列化，其速度可能比标准 `json` 模块快得多。

要使用它，首先安装 `ujson`：

```bash Terminal icon=mdi:bash
pip install ujson
```

然后，你可以将其设置为应用程序或特定路径操作的默认响应类。

```python 设置 UJSONResponse 为默认响应 icon=logos:python
from fastapi import FastAPI
from fastapi.responses import UJSONResponse

app = FastAPI(default_response_class=UJSONResponse)

@app.get("/items/")
async def read_items():
    return [{"item_id": "Foo"}, {"item_id": "Bar"}]
```

### ORJSONResponse

该响应类使用 `orjson` 库，这是另一个高性能的替代方案，特别擅长序列化 dataclass、datetime 和 UUID。

要使用它，首先安装 `orjson`：

```bash Terminal icon=mdi:bash
pip install orjson
```

`ORJSONResponse` 被配置为可以处理非字符串键和 NumPy 对象，这使其功能非常多样。

```python 为特定端点使用 ORJSONResponse icon=logos:python
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse
import numpy as np

app = FastAPI()

@app.get("/data", response_class=ORJSONResponse)
async def read_data():
    return {"numbers": np.array([1, 2, 3]).tolist(), 1: "integer_key"}
```

## JSON 兼容编码器

FastAPI 使用一个特殊函数 `jsonable_encoder`，将 Python 对象（如 Pydantic 模型或 `datetime` 对象）转换为与 JSON 兼容的数据结构（如 `dict` 和 `list`）。该函数在发送 `JSONResponse` 之前会在内部被调用，但你也可以手动使用它，例如，在将数据存入数据库之前准备数据。

### 参数

<x-field data-name="obj" data-type="Any" data-required="true" data-desc="要转换为 JSON 的输入对象。"></x-field>
<x-field data-name="include" data-type="set | dict" data-required="false" data-desc="Pydantic 的 include 参数，用于设置要包含的字段。"></x-field>
<x-field data-name="exclude" data-type="set | dict" data-required="false" data-desc="Pydantic 的 exclude 参数，用于设置要排除的字段。"></x-field>
<x-field data-name="by_alias" data-type="bool" data-default="true" data-required="false" data-desc="输出是否应使用 Pydantic 模型的别名。"></x-field>
<x-field data-name="exclude_unset" data-type="bool" data-default="false" data-required="false" data-desc="排除未显式设置（且仅具有默认值）的字段。"></x-field>
<x-field data-name="exclude_defaults" data-type="bool" data-default="false" data-required="false" data-desc="排除与默认值相同值的字段，即使是显式设置的。"></x-field>
<x-field data-name="exclude_none" data-type="bool" data-default="false" data-required="false" data-desc="排除任何值为 None 的字段。"></x-field>
<x-field data-name="custom_encoder" data-type="dict" data-required="false" data-desc="用于特定类型的自定义编码器字典。"></x-field>
<x-field data-name="sqlalchemy_safe" data-type="bool" data-default="true" data-required="false" data-desc="排除任何以 _sa 开头的字段，以兼容 SQLAlchemy 对象。"></x-field>

### 用法示例

以下是如何使用 `jsonable_encoder` 将包含 `datetime` 对象的 Pydantic 模型转换为带有 ISO 格式字符串的字典。

```python 编码 Pydantic 模型 icon=logos:python
from datetime import datetime
from pydantic import BaseModel
from fastapi.encoders import jsonable_encoder

class Item(BaseModel):
    title: str
    timestamp: datetime
    description: str | None = None

item_obj = Item(title="Foo", timestamp=datetime.now())

# 将 Pydantic 模型转换为字典
json_compatible_item_data = jsonable_encoder(item_obj)

# json_compatible_item_data 将类似于：
# {
#   "title": "Foo",
#   "timestamp": "2023-10-27T10:00:00.123456",
#   "description": null
# }
print(json_compatible_item_data)
```

### 自动编码的类型

`jsonable_encoder` 内置支持许多不能直接进行 JSON 序列化的常见类型。

| 原始类型 | 编码为 |
| --- | --- |
| `datetime.datetime` | `str` (ISO 8601) |
| `datetime.date` | `str` (ISO 8601) |
| `datetime.time` | `str` (ISO 8601) |
| `datetime.timedelta`| `float` (总秒数) |
| `UUID` | `str` |
| `Decimal` | `int` 或 `float` |
| `Enum` | 枚举的值 |
| `set`、`frozenset`、`deque` | `list` |
| `bytes` | `str` (已解码) |
| `Path` 对象 | `str` |
| Pydantic `SecretStr`、`SecretBytes` | `str` |
| Pydantic networking types | `str` |

对于更高级的场景，例如处理异常并返回适当的错误响应，请参阅[异常](./api-reference-exceptions.md)文档。