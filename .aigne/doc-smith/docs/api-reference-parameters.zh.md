# 参数

FastAPI 允许你声明有关*路径操作*所接收参数的信息。这包括验证、用于文档的元数据等。

这些参数声明函数（`Path`、`Query`、`Header` 等）在 `typing.Annotated` 中作为参数使用。它们为 FastAPI 提供了验证传入数据和生成准确 OpenAPI 文档所需的详细信息。

## Path

使用 `Path` 声明作为 URL 路径一部分的参数。路径参数始终是必需的。

```python Path Parameter Example icon=logos:python
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

<x-field data-name="default" data-type="any" data-default="..." data-required="true" data-desc="路径参数始终是必需的。此项设置为 `...` 以表示必须提供值。"></x-field>
<x-field data-name="alias" data-type="str" data-required="false" data-desc="参数的备用名称。用于数据提取和 OpenAPI 文档。"></x-field>
<x-field data-name="title" data-type="str" data-required="false" data-desc="供人阅读的参数标题。"></x-field>
<x-field data-name="description" data-type="str" data-required="false" data-desc="供人阅读的参数描述。"></x-field>
<x-field data-name="gt" data-type="float" data-required="false" data-desc="大于。该值必须大于此数字。"></x-field>
<x-field data-name="ge" data-type="float" data-required="false" data-desc="大于或等于。该值必须大于或等于此数字。"></x-field>
<x-field data-name="lt" data-type="float" data-required="false" data-desc="小于。该值必须小于此数字。"></x-field>
<x-field data-name="le" data-type="float" data-required="false" data-desc="小于或等于。该值必须小于或等于此数字。"></x-field>
<x-field data-name="min_length" data-type="int" data-required="false" data-desc="字符串值的最小长度。"></x-field>
<x-field data-name="max_length" data-type="int" data-required="false" data-desc="字符串值的最大长度。"></x-field>
<x-field data-name="pattern" data-type="str" data-required="false" data-desc="字符串值必须匹配的正则表达式模式。"></x-field>
<x-field data-name="deprecated" data-type="bool" data-required="false" data-desc="在 OpenAPI 文档中将此参数标记为已弃用。"></x-field>
<x-field data-name="examples" data-type="List[Any]" data-required="false" data-desc="参数的示例值列表。"></x-field>
<x-field data-name="openapi_examples" data-type="Dict[str, Example]" data-required="false" data-desc="用于在 Swagger UI 等工具中进行扩展文档的 OpenAPI 特定示例。"></x-field>
<x-field data-name="include_in_schema" data-type="bool" data-default="true" data-required="false" data-desc="是否将此参数包含在生成的 OpenAPI 模式中。"></x-field>

## Query

使用 `Query` 声明查询参数，即 URL 中 `?` 之后的部分。

```python Query Parameter Example icon=logos:python
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

<x-field data-name="default" data-type="any" data-required="false" data-desc="如果客户端未提供参数，则该参数的默认值。如果为 `None`，则该参数是可选的。"></x-field>
<x-field data-name="alias" data-type="str" data-required="false" data-desc="参数的备用名称。用于数据提取和 OpenAPI 文档。"></x-field>
<x-field data-name="title" data-type="str" data-required="false" data-desc="供人阅读的参数标题。"></x-field>
<x-field data-name="description" data-type="str" data-required="false" data-desc="供人阅读的参数描述。"></x-field>
<x-field data-name="gt" data-type="float" data-required="false" data-desc="大于。该值必须大于此数字。"></x-field>
<x-field data-name="ge" data-type="float" data-required="false" data-desc="大于或等于。该值必须大于或等于此数字。"></x-field>
<x-field data-name="lt" data-type="float" data-required="false" data-desc="小于。该值必须小于此数字。"></x-field>
<x-field data-name="le" data-type="float" data-required="false" data-desc="小于或等于。该值必须小于或等于此数字。"></x-field>
<x-field data-name="min_length" data-type="int" data-required="false" data-desc="字符串值的最小长度。"></x-field>
<x-field data-name="max_length" data-type="int" data-required="false" data-desc="字符串值的最大长度。"></x-field>
<x-field data-name="pattern" data-type="str" data-required="false" data-desc="字符串值必须匹配的正则表达式模式。"></x-field>
<x-field data-name="deprecated" data-type="bool" data-required="false" data-desc="在 OpenAPI 文档中将此参数标记为已弃用。"></x-field>
<x-field data-name="examples" data-type="List[Any]" data-required="false" data-desc="参数的示例值列表。"></x-field>
<x-field data-name="openapi_examples" data-type="Dict[str, Example]" data-required="false" data-desc="用于在 Swagger UI 等工具中进行扩展文档的 OpenAPI 特定示例。"></x-field>
<x-field data-name="include_in_schema" data-type="bool" data-default="true" data-required="false" data-desc="是否将此参数包含在生成的 OpenAPI 模式中。"></x-field>

## Header

使用 `Header` 声明请求头参数。

```python Header Parameter Example icon=logos:python
from typing import Annotated

from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    return {"User-Agent": user_agent}
```

### 参数

`Header` 接受与 `Query` 相同的参数，并额外增加一个参数：

<x-field data-name="convert_underscores" data-type="bool" data-default="true" data-required="false" data-desc="如果为 `True`，参数名称中的下划线 (`_`) 将自动转换成连字符 (`-`)，以匹配标准的 HTTP 请求头格式。"></x-field>

## Cookie

使用 `Cookie` 声明来自请求 cookie 的参数。

```python Cookie Parameter Example icon=logos:python
from typing import Annotated

from fastapi import FastAPI, Cookie

app = FastAPI()


@app.get("/items/")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    return {"ads_id": ads_id}
```

### 参数

`Cookie` 接受与 `Query` 相同的参数。

## Body

使用 `Body` 声明来自请求体的参数。它通常与 Pydantic 模型一起使用以定义复杂的 JSON 对象。

```python Body Parameter Example icon=logos:python
from typing import Annotated
from fastapi import FastAPI, Body
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.put("/items/{item_id}")
async def update_item(
    item_id: int, 
    item: Item, 
    importance: Annotated[int, Body(gt=0)]
):
    results = {"item_id": item_id, "item": item, "importance": importance}
    return results
```

### 参数

`Body` 接受与 `Query` 大部分相同的验证和元数据参数，此外还包括以下参数：

<x-field data-name="embed" data-type="bool" data-default="false" data-required="false" data-desc="如果为 `True`，则参数应位于 JSON 请求体中，并以参数名作为键。当请求体中只有一个原始类型时，此功能非常有用。"></x-field>
<x-field data-name="media_type" data-type="str" data-default="application/json" data-required="false" data-desc="请求体的媒体类型。"></x-field>

## Form

使用 `Form` 声明来自表单字段（`application/x-www-form-urlencoded`）的参数。

```python Form Parameter Example icon=logos:python
from typing import Annotated

from fastapi import FastAPI, Form

app = FastAPI()


@app.post("/login/")
async def login(username: Annotated[str, Form()], password: Annotated[str, Form()]):
    return {"username": username}
```

### 参数

`Form` 继承自 `Body`，并接受相同的参数。默认情况下，它会将 `media_type` 设置为 `application/x-www-form-urlencoded`。

## File

使用 `File` 声明用于文件上传（`multipart/form-data`）的参数。该参数应使用 `bytes` 或 `UploadFile` 进行注解。

```python File Upload Example icon=logos:python
from typing import Annotated

from fastapi import FastAPI, File, UploadFile

app = FastAPI()


@app.post("/files/")
async def create_file(file: Annotated[bytes, File()])-> dict:
    return {"file_size": len(file)}


@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile) -> dict:
    return {"filename": file.filename, "content_type": file.content_type}
```

### 参数

`File` 继承自 `Form`，并接受相同的参数。默认情况下，它会将 `media_type` 设置为 `multipart/form-data`。