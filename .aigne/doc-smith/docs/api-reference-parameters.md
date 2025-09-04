# Parameters

FastAPI uses parameter-defining functions to declare the inputs your API endpoints receive. These functions are not just for declaration; they also handle data validation, serialization, and automatic documentation generation for OpenAPI.

When you need to declare metadata or validation for a parameter, you use these functions inside of `typing.Annotated`.

This page provides a detailed reference for each of the core parameter-defining functions.

## Overview of Parameter Sources

The following diagram illustrates where each type of parameter is extracted from in an incoming HTTP request.

```d2
direction: down

"HTTP Request": {
  shape: package
  grid-columns: 1

  "Request Line": {
    shape: rectangle
    "GET /items/{item_id}?q=search HTTP/1.1"

    "Path": {
      label: "/items/{item_id}"
      shape: rectangle
    }

    "Query": {
      label: "?q=search"
      shape: rectangle
    }

    "Request Line" -> "Path"
    "Request Line" -> "Query"
  }

  "Headers": {
    shape: rectangle
    "Host: example.com\nUser-Agent: curl/7.64.1\nCookie: session_id=abc123"

    "Header": {
      label: "User-Agent"
      shape: rectangle
    }

    "Cookie": {
      label: "Cookie"
      shape: rectangle
    }

    "Headers" -> "Header"
    "Headers" -> "Cookie"
  }

  "Body": {
    shape: rectangle
    "{\"name\": \"Foo\", \"price\": 42.0}"
  }

  "HTTP Request" -> "Request Line"
  "HTTP Request" -> "Headers"
  "HTTP Request" -> "Body"
}

"FastAPI Parameter Functions": {
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

"HTTP Request"."Request Line"."Path" -> "FastAPI Parameter Functions"."Path()": "Extracts {item_id}" { style.stroke-dash: 2 }
"HTTP Request"."Request Line"."Query" -> "FastAPI Parameter Functions"."Query()": "Extracts q" { style.stroke-dash: 2 }
"HTTP Request"."Headers"."Header" -> "FastAPI Parameter Functions"."Header()": "Extracts User-Agent" { style.stroke-dash: 2 }
"HTTP Request"."Headers"."Cookie" -> "FastAPI Parameter Functions"."Cookie()": "Extracts session_id" { style.stroke-dash: 2 }
"HTTP Request"."Body" -> "FastAPI Parameter Functions"."Body()": "Parses JSON" { style.stroke-dash: 2 }

```

---

## `Path()`

Declares a path parameter. Path parameters are always required as they are part of the URL path.

### Example

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

### Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| `default` | `Any` | Must be `...` as path parameters are always required. Available for compatibility. |
| `alias` | `str` | An alternative name for the parameter, used in the OpenAPI schema. |
| `title` | `str` | A human-readable title for the parameter. |
| `description` | `str` | A human-readable description. |
| `gt` | `float` | Value must be greater than this. |
| `ge` | `float` | Value must be greater than or equal to this. |
| `lt` | `float` | Value must be less than this. |
| `le` | `float` | Value must be less than or equal to this. |
| `min_length` | `int` | Minimum length for string values. |
| `max_length` | `int` | Maximum length for string values. |
| `pattern` | `str` | A regular expression pattern that the string value must match. |
| `deprecated` | `bool` | Marks the parameter as deprecated in the OpenAPI documentation. |
| `examples` | `List[Any]` | A list of example values. |
| `openapi_examples` | `Dict[str, Example]` | OpenAPI-specific examples with more details. |
| `include_in_schema`| `bool` | Whether to include this parameter in the OpenAPI schema. Defaults to `True`. |
| `json_schema_extra`| `Dict[str, Any]` | Any additional JSON schema data to be included. |

---

## `Query()`

Declares a query parameter. These are the key-value pairs in the URL that come after the `?`.

### Example

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

### Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| `default` | `Any` | Default value if the parameter is not provided. If `...`, the parameter is required. |
| `alias` | `str` | An alternative name for the parameter, used for extracting data and in OpenAPI. |
| `title` | `str` | A human-readable title for the parameter. |
| `description` | `str` | A human-readable description. |
| `gt` | `float` | Value must be greater than this. |
| `ge` | `float` | Value must be greater than or equal to this. |
| `lt` | `float` | Value must be less than this. |
| `le` | `float` | Value must be less than or equal to this. |
| `min_length` | `int` | Minimum length for string values. |
| `max_length` | `int` | Maximum length for string values. |
| `pattern` | `str` | A regular expression pattern that the string value must match. |
| `deprecated` | `bool` | Marks the parameter as deprecated in the OpenAPI documentation. |
| `examples` | `List[Any]` | A list of example values. |
| `openapi_examples` | `Dict[str, Example]` | OpenAPI-specific examples with more details. |
| `include_in_schema`| `bool` | Whether to include this parameter in the OpenAPI schema. Defaults to `True`. |
| `json_schema_extra`| `Dict[str, Any]` | Any additional JSON schema data to be included. |

---

## `Header()`

Declares a header parameter. It reads from the request headers.

### Example

```python
from typing import Annotated
from fastapi import FastAPI, Header

app = FastAPI()

@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    return {"User-Agent": user_agent}
```

### Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| `default` | `Any` | Default value if the header is not provided. |
| `convert_underscores` | `bool` | If `True` (the default), converts underscores `_` in the parameter name to hyphens `-` to look for the header. |
| `alias` | `str` | An alternative name for the parameter. |
| `title` | `str` | A human-readable title for the parameter. |
| `description` | `str` | A human-readable description. |
| `gt` | `float` | Value must be greater than this. |
| `ge` | `float` | Value must be greater than or equal to this. |
| `lt` | `float` | Value must be less than this. |
| `le` | `float` | Value must be less than or equal to this. |
| `min_length` | `int` | Minimum length for string values. |
| `max_length` | `int` | Maximum length for string values. |
| `pattern` | `str` | A regular expression pattern that the string value must match. |
| `deprecated` | `bool` | Marks the parameter as deprecated in the OpenAPI documentation. |
| `examples` | `List[Any]` | A list of example values. |
| `openapi_examples` | `Dict[str, Example]` | OpenAPI-specific examples with more details. |
| `include_in_schema`| `bool` | Whether to include this parameter in the OpenAPI schema. Defaults to `True`. |

---

## `Cookie()`

Declares a cookie parameter. It reads from the request cookies.

### Example

```python
from typing import Annotated
from fastapi import Cookie, FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    return {"ads_id": ads_id}
```

### Parameters

`Cookie` shares most of the same validation and metadata parameters as `Query` and `Header`, such as `default`, `alias`, `title`, `description`, numeric validations (`gt`, `ge`, etc.), and string validations (`min_length`, `max_length`, etc.).

---

## `Body()`

Declares a parameter that comes from the request body. It is often used with Pydantic models.

### Example

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

### Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| `default` | `Any` | Default value if the field is not in the body. |
| `embed` | `bool` | If `True`, the parameter will be expected inside a JSON body with its parameter name as the key. This happens automatically if you declare more than one `Body` parameter. |
| `media_type` | `str` | The media type of the request body. Defaults to `application/json`. |
| `alias` | `str` | An alternative name for the parameter field. |
| `title` | `str` | A human-readable title. |
| `description` | `str` | A human-readable description. |
| `examples` | `List[Any]` | A list of example values. |
| `openapi_examples` | `Dict[str, Example]` | OpenAPI-specific examples with more details. |
| `json_schema_extra`| `Dict[str, Any]` | Any additional JSON schema data to be included. |

It also supports the same numeric and string validation parameters as `Path` and `Query` (`gt`, `ge`, `min_length`, etc.).

---

## `Form()`

Declares a form field. This is used when the request has a media type of `application/x-www-form-urlencoded`.

### Example

```python
from typing import Annotated
from fastapi import FastAPI, Form

app = FastAPI()

@app.post("/login/")
async def login(username: Annotated[str, Form()], password: Annotated[str, Form()]):
    return {"username": username}
```

### Parameters

`Form` inherits from `Body` and shares all the same parameters. The `media_type` defaults to `application/x-www-form-urlencoded`.

---

## `File()`

Declares a file upload. This is used when the request has a media type of `multipart/form-data`.

### Example

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

### Parameters

`File` inherits from `Form` and shares all the same parameters. The `media_type` defaults to `multipart/form-data`.
