# 参数

FastAPI 使用参数定义函数在*路径操作函数*中声明和配置 API 参数。这些函数提供数据验证、转换、OpenAPI 文档以及编辑器支持（例如类型提示）。

`Path`、`Query`、`Header`、`Cookie`、`Body`、`Form` 和 `File` 这些函数与 `typing.Annotated` 结合使用，为参数提供额外的元数据。

它们都共享一套用于验证和文档的通用参数，这些参数派生自 Pydantic 的 `FieldInfo` 和 FastAPI 的 `Param` 类。如需了解更复杂的依赖注入场景，请参阅 [依赖项](./api-reference-dependencies.md) 参考。

---

## Path

声明一个属于 URL 路径一部分的参数。路径参数始终是必需的。

### 用法示例

```python
from typing import Annotated
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get", ge=1)],
): 
    return {"item_id": item_id}
```

### 参数

| Parameter | Type | Description |
|---|---|---|
| `default` | `Any` | 必须为 `...`，因为路径参数始终是必需的。为兼容性而提供。 |
| `alias` | `str` | 参数的别名，用于数据提取和 OpenAPI 结构。 |
| `title` | `str` | 供人阅读的参数标题。 |
| `description` | `str` | 供人阅读的参数描述。 |
| `gt` | `float` | “大于”。值必须大于此值。 |
| `ge` | `float` | “大于或等于”。值必须大于或等于此值。 |
| `lt` | `float` | “小于”。值必须小于此值。 |
| `le` | `float` | “小于或等于”。值必须小于或等于此值。 |
| `min_length` | `int` | 字符串值的最小长度。 |
| `max_length` | `int` | 字符串值的最大长度。 |
| `pattern` | `str` | 字符串值必须匹配的正则表达式模式。 |
| `deprecated` | `bool` | 在 OpenAPI 文档中将参数标记为已弃用。 |
| `examples` | `List[Any]` | 示例值列表。 |
| `openapi_examples` | `Dict[str, Example]` | 包含更多详细信息的 OpenAPI 专用示例。 |
| `include_in_schema` | `bool` | 是否在生成的 OpenAPI 结构中包含此参数。默认为 `True`。 |
| `json_schema_extra` | `Dict[str, Any]` | 要包含的任何额外 JSON 结构数据。 |

---

## Query

声明一个查询参数，即 URL 中 `?` 之后的部分。

### 用法示例

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(max_length=50)] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

### 参数

| Parameter | Type | Description |
|---|---|---|
| `default` | `Any` | 如果未提供参数，则使用默认值。可以为 `None` 使其成为可选参数。 |
| `alias` | `str` | 参数的别名，用于数据提取和 OpenAPI 结构。 |
| `title` | `str` | 供人阅读的参数标题。 |
| `description` | `str` | 供人阅读的参数描述。 |
| `gt` | `float` | “大于”。值必须大于此值。 |
| `ge` | `float` | “大于或等于”。值必须大于或等于此值。 |
| `lt` | `float` | “小于”。值必须小于此值。 |
| `le` | `float` | “小于或等于”。值必须小于或等于此值。 |
| `min_length` | `int` | 字符串值的最小长度。 |
| `max_length` | `int` | 字符串值的最大长度。 |
| `pattern` | `str` | 字符串值必须匹配的正则表达式模式。 |
| `deprecated` | `bool` | 在 OpenAPI 文档中将参数标记为已弃用。 |
| `examples` | `List[Any]` | 示例值列表。 |
| `openapi_examples` | `Dict[str, Example]` | 包含更多详细信息的 OpenAPI 专用示例。 |
| `include_in_schema` | `bool` | 是否在生成的 OpenAPI 结构中包含此参数。默认为 `True`。 |
| `json_schema_extra` | `Dict[str, Any]` | 要包含的任何额外 JSON 结构数据。 |

---

## Header

声明一个请求头参数。标头名称不区分大小写。

### 用法示例

```python
from typing import Annotated
from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    return {"User-Agent": user_agent}
```

### 参数

| Parameter | Type | Description |
|---|---|---|
| `convert_underscores` | `bool` | 如果为 `True`（默认值），则将参数名称中的下划线（`_`）转换成连字符（`-`），以匹配标准的 HTTP 标头格式。 |
| `default` | `Any` | 如果未提供标头，则使用默认值。 |
| `alias` | `str` | 参数的别名。如果标头名称不是有效的 Python 标识符，则此项很有用。 |
| `title` | `str` | 供人阅读的标题。 |
| `description` | `str` | 供人阅读的描述。 |
| `gt`, `ge`, `lt`, `le` | `float` | 数值验证。 |
| `min_length`, `max_length` | `int` | 字符串长度验证。 |
| `pattern` | `str` | 正则表达式模式。 |
| `deprecated` | `bool` | 将标头标记为已弃用。 |
| `examples` | `List[Any]` | 示例值列表。 |
| `openapi_examples` | `Dict[str, Example]` | OpenAPI 专用示例。 |
| `include_in_schema` | `bool` | 是否在结构中包含此标头。 |
| `json_schema_extra` | `Dict[str, Any]` | 任何额外的 JSON 结构数据。 |

---

## Cookie

声明一个请求 cookie 参数。

### 用法示例

```python
from typing import Annotated
from fastapi import FastAPI, Cookie

app = FastAPI()


@app.get("/items/")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    return {"ads_id": ads_id}
```

### 参数

`Cookie` 与 `Query` 共享相同的验证和文档参数（例如 `default`、`alias`、`title`、`description`、数值和字符串验证等）。

---

## Body

声明一个来自请求体的参数。它通常与 Pydantic 模型一起使用来定义复杂的数据结构。

### 用法示例

```python
from typing import Annotated
from fastapi import FastAPI, Body
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

app = FastAPI()

# 使用 Pydantic 模型（最常见）
@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, "item": item}

# 使用单个请求体参数
@app.put("/items/importance/{item_id}")
async def update_importance(
    item_id: int, 
    importance: Annotated[int, Body(embed=True)]
): 
    return {"item_id": item_id, "importance": importance}
```

### 参数

| Parameter | Type | Description |
|---|---|---|
| `embed` | `bool` | 如果为 `True`，则参数将被视为 JSON 请求体中的一个键，而不是整个请求体。如果声明了多个 `Body` 参数，则会自动发生这种情况。 |
| `media_type` | `str` | 请求体的媒体类型。默认为 `application/json`。 |
| `default` | `Any` | 如果未提供参数，则使用默认值。 |
| `alias` | `str` | 请求体中参数键的别名。 |
| `title` | `str` | 供人阅读的标题。 |
| `description` | `str` | 供人阅读的描述。 |
| `gt`, `ge`, `lt`, `le` | `float` | 数值验证。 |
| `min_length`, `max_length` | `int` | 字符串长度验证。 |
| `pattern` | `str` | 正则表达式模式。 |
| `deprecated` | `bool` | 将参数标记为已弃用。 |
| `examples` | `List[Any]` | 示例值列表。 |
| `openapi_examples` | `Dict[str, Example]` | OpenAPI 专用示例。 |
| `include_in_schema` | `bool` | 是否在结构中包含此参数。 |
| `json_schema_extra` | `Dict[str, Any]` | 任何额外的 JSON 结构数据。 |

---

## Form

声明一个表单数据参数。当客户端以 `application/x-www-form-urlencoded` 格式发送数据时使用。

### 用法示例

```python
from typing import Annotated
from fastapi import FastAPI, Form

app = FastAPI()


@app.post("/login/")
async def login(username: Annotated[str, Form()], password: Annotated[str, Form()]):
    return {"username": username}
```

### 参数

`Form` 继承自 `Body` 并共享相同的参数，但其 `media_type` 默认为 `application/x-www-form-urlencoded`。

---

## File

声明一个文件上传参数。这要求客户端以 `multipart/form-data` 格式发送数据。

### 用法示例

```python
from typing import Annotated
from fastapi import FastAPI, File, UploadFile

app = FastAPI()


@app.post("/files/")
async def create_file(file: Annotated[bytes, File()]):
    return {"file_size": len(file)}


@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile):
    return {"filename": file.filename, "content_type": file.content_type}
```

### 参数

`File` 继承自 `Form` 并共享相同的参数，但其 `media_type` 默认为 `multipart/form-data`。