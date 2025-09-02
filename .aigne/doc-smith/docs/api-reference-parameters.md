# Parameters

FastAPI uses parameter-defining functions to declare and configure API parameters within your *path operation functions*. These functions provide data validation, conversion, documentation for OpenAPI, and editor support (e.g., type hints).

These functions—`Path`, `Query`, `Header`, `Cookie`, `Body`, `Form`, and `File`—are used with `typing.Annotated` to provide additional metadata for your parameters.

They all share a common set of parameters for validation and documentation, derived from Pydantic's `FieldInfo` and FastAPI's `Param` class. For more complex dependency-injection scenarios, see the [Dependencies](./api-reference-dependencies.md) reference.

---

## Path

Declares a parameter that is part of the URL path. Path parameters are always required.

### Usage Example

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
|---|---|---|
| `default` | `Any` | Must be `...` as path parameters are always required. Available for compatibility. |
| `alias` | `str` | An alternative name for the parameter, used for data extraction and OpenAPI schema. |
| `title` | `str` | A human-readable title for the parameter. |
| `description` | `str` | A human-readable description for the parameter. |
| `gt` | `float` | "Greater than". Value must be greater than this. |
| `ge` | `float` | "Greater than or equal". Value must be greater than or equal to this. |
| `lt` | `float` | "Less than". Value must be less than this. |
| `le` | `float` | "Less than or equal". Value must be less than or equal to this. |
| `min_length` | `int` | Minimum length for string values. |
| `max_length` | `int` | Maximum length for string values. |
| `pattern` | `str` | A regular expression pattern that the string value must match. |
| `deprecated` | `bool` | Marks the parameter as deprecated in the OpenAPI documentation. |
| `examples` | `List[Any]` | A list of example values. |
| `openapi_examples` | `Dict[str, Example]` | OpenAPI-specific examples with more details. |
| `include_in_schema` | `bool` | Whether to include this parameter in the generated OpenAPI schema. Defaults to `True`. |
| `json_schema_extra` | `Dict[str, Any]` | Any additional JSON schema data to include. |

---

## Query

Declares a query parameter, which is the part of the URL that follows the `?`.

### Usage Example

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
|---|---|---|
| `default` | `Any` | The default value if the parameter is not provided. Can be `None` to make it optional. |
| `alias` | `str` | An alternative name for the parameter, used for data extraction and OpenAPI schema. |
| `title` | `str` | A human-readable title for the parameter. |
| `description` | `str` | A human-readable description for the parameter. |
| `gt` | `float` | "Greater than". Value must be greater than this. |
| `ge` | `float` | "Greater than or equal". Value must be greater than or equal to this. |
| `lt` | `float` | "Less than". Value must be less than this. |
| `le` | `float` | "Less than or equal". Value must be less than or equal to this. |
| `min_length` | `int` | Minimum length for string values. |
| `max_length` | `int` | Maximum length for string values. |
| `pattern` | `str` | A regular expression pattern that the string value must match. |
| `deprecated` | `bool` | Marks the parameter as deprecated in the OpenAPI documentation. |
| `examples` | `List[Any]` | A list of example values. |
| `openapi_examples` | `Dict[str, Example]` | OpenAPI-specific examples with more details. |
| `include_in_schema` | `bool` | Whether to include this parameter in the generated OpenAPI schema. Defaults to `True`. |
| `json_schema_extra` | `Dict[str, Any]` | Any additional JSON schema data to include. |

---

## Header

Declares a request header parameter. Header names are case-insensitive.

### Usage Example

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
|---|---|---|
| `convert_underscores` | `bool` | If `True` (the default), converts underscores (`_`) in the parameter name to hyphens (`-`) to match standard HTTP header format. |
| `default` | `Any` | The default value if the header is not provided. |
| `alias` | `str` | An alternative name for the parameter. Useful if the header name is not a valid Python identifier. |
| `title` | `str` | A human-readable title. |
| `description` | `str` | A human-readable description. |
| `gt`, `ge`, `lt`, `le` | `float` | Numeric validations. |
| `min_length`, `max_length` | `int` | String length validations. |
| `pattern` | `str` | A regular expression pattern. |
| `deprecated` | `bool` | Marks the header as deprecated. |
| `examples` | `List[Any]` | A list of example values. |
| `openapi_examples` | `Dict[str, Example]` | OpenAPI-specific examples. |
| `include_in_schema` | `bool` | Whether to include this header in the schema. |
| `json_schema_extra` | `Dict[str, Any]` | Any additional JSON schema data. |

---

## Cookie

Declares a request cookie parameter.

### Usage Example

```python
from typing import Annotated
from fastapi import FastAPI, Cookie

app = FastAPI()


@app.get("/items/")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    return {"ads_id": ads_id}
```

### Parameters

`Cookie` shares the same validation and documentation parameters as `Query` (e.g., `default`, `alias`, `title`, `description`, numeric and string validations, etc.).

---

## Body

Declares a parameter that comes from the request body. It is often used with Pydantic models to define complex data structures.

### Usage Example

```python
from typing import Annotated
from fastapi import FastAPI, Body
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

app = FastAPI()

# With a Pydantic model (most common)
@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, "item": item}

# With a single body parameter
@app.put("/items/importance/{item_id}")
async def update_importance(
    item_id: int, 
    importance: Annotated[int, Body(embed=True)]
): 
    return {"item_id": item_id, "importance": importance}
```

### Parameters

| Parameter | Type | Description |
|---|---|---|
| `embed` | `bool` | If `True`, the parameter will be expected as a key in the JSON body, rather than being the entire body. This happens automatically if you declare more than one `Body` parameter. |
| `media_type` | `str` | The media type of the request body. Defaults to `application/json`. |
| `default` | `Any` | The default value if the parameter is not provided. |
| `alias` | `str` | An alternative name for the parameter key in the body. |
| `title` | `str` | A human-readable title. |
| `description` | `str` | A human-readable description. |
| `gt`, `ge`, `lt`, `le` | `float` | Numeric validations. |
| `min_length`, `max_length` | `int` | String length validations. |
| `pattern` | `str` | A regular expression pattern. |
| `deprecated` | `bool` | Marks the parameter as deprecated. |
| `examples` | `List[Any]` | A list of example values. |
| `openapi_examples` | `Dict[str, Example]` | OpenAPI-specific examples. |
| `include_in_schema` | `bool` | Whether to include this parameter in the schema. |
| `json_schema_extra` | `Dict[str, Any]` | Any additional JSON schema data. |

---

## Form

Declares a form data parameter. This is used when the client sends data as `application/x-www-form-urlencoded`.

### Usage Example

```python
from typing import Annotated
from fastapi import FastAPI, Form

app = FastAPI()


@app.post("/login/")
async def login(username: Annotated[str, Form()], password: Annotated[str, Form()]):
    return {"username": username}
```

### Parameters

`Form` inherits from `Body` and shares the same parameters, but its `media_type` defaults to `application/x-www-form-urlencoded`.

---

## File

Declares a file upload parameter. This requires the client to send data as `multipart/form-data`.

### Usage Example

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

### Parameters

`File` inherits from `Form` and shares the same parameters, but its `media_type` defaults to `multipart/form-data`.