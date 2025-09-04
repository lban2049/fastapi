# 参数

FastAPI 使用参数定义函数来声明 API 端点接收的输入。这些函数不仅用于声明，还处理数据验证、序列化以及 OpenAPI 的自动文档生成。

当需要为参数声明元数据或验证时，可以在 `typing.Annotated` 中使用这些函数。

本页为每个核心参数定义函数提供了详细参考。

## 参数来源概览

下图说明了在传入的 HTTP 请求中，每种类型的参数是从哪里提取的。

```d2
direction: down

"HTTP 请求": {
  shape: package
  grid-columns: 1

  "请求行": {
    shape: rectangle
    "GET /items/{item_id}?q=search HTTP/1.1"

    "路径": {
      label: "/items/{item_id}"
      shape: rectangle
    }

    "查询": {
      label: "?q=search"
      shape: rectangle
    }

    "请求行" -> "路径"
    "请求行" -> "查询"
  }

  "标头": {
    shape: rectangle
    "Host: example.com\nUser-Agent: curl/7.64.1\nCookie: session_id=abc123"

    "标头": {
      label: "User-Agent"
      shape: rectangle
    }

    "Cookie": {
      label: "Cookie"
      shape: rectangle
    }

    "标头" -> "标头"
    "标头" -> "Cookie"
  }

  "正文": {
    shape: rectangle
    "{\"name\": \"Foo\", \"price\": 42.0}"
  }

  "HTTP 请求" -> "请求行"
  "HTTP 请求" -> "标头"
  "HTTP 请求" -> "正文"
}

"FastAPI 参数函数": {
  shape: package
  grid-columns: 2

  "Path()": {shape: oval}
  "Query()": {shape: oval}
  "Header()": {shape: oval}
  "Cookie()": {shape: oval}
  "Body()": {shape: oval}
  "Form()": {shape: oval}
  "File()": {shape: oval}
}

"HTTP 请求"."请求行"."路径" -> "FastAPI 参数函数"."Path()": "提取 {item_id}" { style.stroke-dash: 2 }
"HTTP 请求"."请求行"."查询" -> "FastAPI 参数函数"."Query()": "提取 q" { style.stroke-dash: 2 }
"HTTP 请求"."标头"."标头" -> "FastAPI 参数函数"."Header()": "提取 User-Agent" { style.stroke-dash: 2 }
"HTTP 请求"."标头"."Cookie" -> "FastAPI 参数函数"."Cookie()": "提取 session_id" { style.stroke-dash: 2 }
"HTTP 请求"."正文" -> "FastAPI 参数函数"."Body()": "解析 JSON" { style.stroke-dash: 2 }

```

---

## `Path()`

声明路径参数。路径参数是 URL 路径的一部分，因此始终为必需项。

### 示例

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
| --- | --- | --- |
| `default` | `Any` | 必须为 `...`，因为路径参数始终为必需项。为兼容性而提供。 |
| `alias` | `str` | 参数的别名，用于 OpenAPI 模式。 |
| `title` | `str` | 参数的可读标题。 |
| `description` | `str` | 可读的描述。 |
| `gt` | `float` | 值必须大于此值。 |
| `ge` | `float` | 值必须大于或等于此值。 |
| `lt` | `float` | 值必须小于此值。 |
| `le` | `float` | 值必须小于或等于此值。 |
| `min_length` | `int` | 字符串值的最小长度。 |
| `max_length` | `int` | 字符串值的最大长度。 |
| `pattern` | `str` | 字符串值必须匹配的正则表达式模式。 |
| `deprecated` | `bool` | 在 OpenAPI 文档中将参数标记为已弃用。 |
| `examples` | `List[Any]` | 示例值列表。 |
| `openapi_examples` | `Dict[str, Example]` | 包含更多详细信息的 OpenAPI 特定示例。 |
| `include_in_schema`| `bool` | 是否在 OpenAPI 模式中包含此参数。默认为 `True`。 |
| `json_schema_extra`| `Dict[str, Any]` | 要包含的任何附加 JSON 模式数据。 |

---

## `Query()`

声明查询参数。这些是 URL 中 `?` 之后的键值对。

### 示例

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
| --- | --- | --- |
| `default` | `Any` | 未提供参数时的默认值。如果为 `...`，则该参数为必需项。 |
| `alias` | `str` | 参数的别名，用于提取数据和在 OpenAPI 中使用。 |
| `title` | `str` | 参数的可读标题。 |
| `description` | `str` | 可读的描述。 |
| `gt` | `float` | 值必须大于此值。 |
| `ge` | `float` | 值必须大于或等于此值。 |
| `lt` | `float` | 值必须小于此值。 |
| `le` | `float` | 值必须小于或等于此值。 |
| `min_length` | `int` | 字符串值的最小长度。 |
| `max_length` | `int` | 字符串值的最大长度。 |
| `pattern` | `str` | 字符串值必须匹配的正则表达式模式。 |
| `deprecated` | `bool` | 在 OpenAPI 文档中将参数标记为已弃用。 |
| `examples` | `List[Any]` | 示例值列表。 |
| `openapi_examples` | `Dict[str, Example]` | 包含更多详细信息的 OpenAPI 特定示例。 |
| `include_in_schema`| `bool` | 是否在 OpenAPI 模式中包含此参数。默认为 `True`。 |
| `json_schema_extra`| `Dict[str, Any]` | 要包含的任何附加 JSON 模式数据。 |

---

## `Header()`

声明标头参数。它从请求标头中读取。

### 示例

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
| --- | --- | --- |
| `default` | `Any` | 未提供标头时的默认值。 |
| `convert_underscores` | `bool` | 如果为 `True`（默认值），则将参数名称中的下划线 `_` 转换为连字符 `-` 来查找标头。 |
| `alias` | `str` | 参数的别名。 |
| `title` | `str` | 参数的可读标题。 |
| `description` | `str` | 可读的描述。 |
| `gt` | `float` | 值必须大于此值。 |
| `ge` | `float` | 值必须大于或等于此值。 |
| `lt` | `float` | 值必须小于此值。 |
| `le` | `float` | 值必须小于或等于此值。 |
| `min_length` | `int` | 字符串值的最小长度。 |
| `max_length` | `int` | 字符串值的最大长度。 |
| `pattern` | `str` | 字符串值必须匹配的正则表达式模式。 |
| `deprecated` | `bool` | 在 OpenAPI 文档中将参数标记为已弃用。 |
| `examples` | `List[Any]` | 示例值列表。 |
| `openapi_examples` | `Dict[str, Example]` | 包含更多详细信息的 OpenAPI 特定示例。 |
| `include_in_schema`| `bool` | 是否在 OpenAPI 模式中包含此参数。默认为 `True`。 |

---

## `Cookie()`

声明 Cookie 参数。它从请求 Cookie 中读取。

### 示例

```python
from typing import Annotated
from fastapi import Cookie, FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    return {"ads_id": ads_id}
```

### 参数

`Cookie` 与 `Query` 和 `Header` 共享大部分相同的验证和元数据参数，例如 `default`、`alias`、`title`、`description`、数值验证（`gt`、`ge` 等）和字符串验证（`min_length`、`max_length` 等）。

---

## `Body()`

声明来自请求正文的参数。它通常与 Pydantic 模型一起使用。

### 示例

```python
from typing import Annotated
from fastapi import FastAPI, Body
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item, importance: Annotated[int, Body(gt=0)]):
    return {"item": item, "importance": importance}
```

### 参数

| Parameter | Type | Description |
| --- | --- | --- |
| `default` | `Any` | 如果字段不在正文中，则使用默认值。 |
| `embed` | `bool` | 如果为 `True`，则参数应位于 JSON 正文内，并以其参数名称作为键。如果声明了多个 `Body` 参数，则会自动发生这种情况。 |
| `media_type` | `str` | 请求正文的媒体类型。默认为 `application/json`。 |
| `alias` | `str` | 参数字段的别名。 |
| `title` | `str` | 可读的标题。 |
| `description` | `str` | 可读的描述。 |
| `examples` | `List[Any]` | 示例值列表。 |
| `openapi_examples` | `Dict[str, Example]` | 包含更多详细信息的 OpenAPI 特定示例。 |
| `json_schema_extra`| `Dict[str, Any]` | 要包含的任何附加 JSON 模式数据。 |

它还支持与 `Path` 和 `Query` 相同的数值和字符串验证参数（`gt`、`ge`、`min_length` 等）。

---

## `Form()`

声明表单字段。当请求的媒体类型为 `application/x-www-form-urlencoded` 时使用。

### 示例

```python
from typing import Annotated
from fastapi import FastAPI, Form

app = FastAPI()

@app.post("/login/")
async def login(username: Annotated[str, Form()], password: Annotated[str, Form()]):
    return {"username": username}
```

### 参数

`Form` 继承自 `Body` 并共享所有相同的参数。`media_type` 默认为 `application/x-www-form-urlencoded`。

---

## `File()`

声明文件上传。当请求的媒体类型为 `multipart/form-data` 时使用。

### 示例

```python
from typing import Annotated
from fastapi import FastAPI, File, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(file: Annotated[bytes, File()])-> dict:
    return {"file_size": len(file)}

@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile)-> dict:
    return {"filename": file.filename, "content_type": file.content_type}
```

### 参数

`File` 继承自 `Form` 并共享所有相同的参数。`media_type` 默认为 `multipart/form-data`。