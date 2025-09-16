# Parameters

FastAPI allows you to declare information about the parameters your *path operation* receives. This includes validation, metadata for documentation, and more.

These parameter-declaring functions (`Path`, `Query`, `Header`, etc.) are used as arguments within `typing.Annotated`. They provide FastAPI with the necessary details to validate incoming data and generate accurate OpenAPI documentation.

## Path

Use `Path` to declare a parameter that is part of the URL path. Path parameters are always required.

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

### Parameters

<x-field data-name="default" data-type="any" data-default="..." data-required="true" data-desc="Path parameters are always required. This is set to `...` to indicate that a value must be provided."></x-field>
<x-field data-name="alias" data-type="str" data-required="false" data-desc="An alternative name for the parameter. Used for data extraction and OpenAPI documentation."></x-field>
<x-field data-name="title" data-type="str" data-required="false" data-desc="A human-readable title for the parameter."></x-field>
<x-field data-name="description" data-type="str" data-required="false" data-desc="A human-readable description for the parameter."></x-field>
<x-field data-name="gt" data-type="float" data-required="false" data-desc="Greater than. The value must be greater than this number."></x-field>
<x-field data-name="ge" data-type="float" data-required="false" data-desc="Greater than or equal. The value must be greater than or equal to this number."></x-field>
<x-field data-name="lt" data-type="float" data-required="false" data-desc="Less than. The value must be less than this number."></x-field>
<x-field data-name="le" data-type="float" data-required="false" data-desc="Less than or equal. The value must be less than or equal to this number."></x-field>
<x-field data-name="min_length" data-type="int" data-required="false" data-desc="Minimum length for string values."></x-field>
<x-field data-name="max_length" data-type="int" data-required="false" data-desc="Maximum length for string values."></x-field>
<x-field data-name="pattern" data-type="str" data-required="false" data-desc="A regular expression pattern that the string value must match."></x-field>
<x-field data-name="deprecated" data-type="bool" data-required="false" data-desc="Marks this parameter as deprecated in the OpenAPI documentation."></x-field>
<x-field data-name="examples" data-type="List[Any]" data-required="false" data-desc="A list of example values for the parameter."></x-field>
<x-field data-name="openapi_examples" data-type="Dict[str, Example]" data-required="false" data-desc="OpenAPI-specific examples for extended documentation in tools like Swagger UI."></x-field>
<x-field data-name="include_in_schema" data-type="bool" data-default="true" data-required="false" data-desc="Whether to include this parameter in the generated OpenAPI schema."></x-field>

## Query

Use `Query` to declare a query parameter, which is the part of the URL after the `?`.

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

### Parameters

<x-field data-name="default" data-type="any" data-required="false" data-desc="The default value of the parameter if it is not provided by the client. If `None`, the parameter is optional."></x-field>
<x-field data-name="alias" data-type="str" data-required="false" data-desc="An alternative name for the parameter. Used for data extraction and OpenAPI documentation."></x-field>
<x-field data-name="title" data-type="str" data-required="false" data-desc="A human-readable title for the parameter."></x-field>
<x-field data-name="description" data-type="str" data-required="false" data-desc="A human-readable description for the parameter."></x-field>
<x-field data-name="gt" data-type="float" data-required="false" data-desc="Greater than. The value must be greater than this number."></x-field>
<x-field data-name="ge" data-type="float" data-required="false" data-desc="Greater than or equal. The value must be greater than or equal to this number."></x-field>
<x-field data-name="lt" data-type="float" data-required="false" data-desc="Less than. The value must be less than this number."></x-field>
<x-field data-name="le" data-type="float" data-required="false" data-desc="Less than or equal. The value must be less than or equal to this number."></x-field>
<x-field data-name="min_length" data-type="int" data-required="false" data-desc="Minimum length for string values."></x-field>
<x-field data-name="max_length" data-type="int" data-required="false" data-desc="Maximum length for string values."></x-field>
<x-field data-name="pattern" data-type="str" data-required="false" data-desc="A regular expression pattern that the string value must match."></x-field>
<x-field data-name="deprecated" data-type="bool" data-required="false" data-desc="Marks this parameter as deprecated in the OpenAPI documentation."></x-field>
<x-field data-name="examples" data-type="List[Any]" data-required="false" data-desc="A list of example values for the parameter."></x-field>
<x-field data-name="openapi_examples" data-type="Dict[str, Example]" data-required="false" data-desc="OpenAPI-specific examples for extended documentation in tools like Swagger UI."></x-field>
<x-field data-name="include_in_schema" data-type="bool" data-default="true" data-required="false" data-desc="Whether to include this parameter in the generated OpenAPI schema."></x-field>

## Header

Use `Header` to declare a request header parameter. 

```python Header Parameter Example icon=logos:python
from typing import Annotated

from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    return {"User-Agent": user_agent}
```

### Parameters

`Header` accepts the same parameters as `Query`, with one addition:

<x-field data-name="convert_underscores" data-type="bool" data-default="true" data-required="false" data-desc="If `True`, underscores (`_`) in the parameter name will be automatically converted to hyphens (`-`) to match standard HTTP header format."></x-field>

## Cookie

Use `Cookie` to declare a parameter that comes from a request cookie.

```python Cookie Parameter Example icon=logos:python
from typing import Annotated

from fastapi import FastAPI, Cookie

app = FastAPI()


@app.get("/items/")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    return {"ads_id": ads_id}
```

### Parameters

`Cookie` accepts the same parameters as `Query`.

## Body

Use `Body` to declare a parameter that comes from the request body. It is often used with Pydantic models to define complex JSON objects.

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

### Parameters

`Body` accepts most of the same validation and metadata parameters as `Query`, plus the following:

<x-field data-name="embed" data-type="bool" data-default="false" data-required="false" data-desc="If `True`, the parameter will be expected inside a JSON body with the parameter name as the key. This is useful when you have a single primitive type in the body."></x-field>
<x-field data-name="media_type" data-type="str" data-default="application/json" data-required="false" data-desc="The media type of the request body."></x-field>

## Form

Use `Form` to declare parameters from form fields (`application/x-www-form-urlencoded`).

```python Form Parameter Example icon=logos:python
from typing import Annotated

from fastapi import FastAPI, Form

app = FastAPI()


@app.post("/login/")
async def login(username: Annotated[str, Form()], password: Annotated[str, Form()]):
    return {"username": username}
```

### Parameters

`Form` inherits from `Body` and accepts the same parameters. It sets the `media_type` to `application/x-www-form-urlencoded` by default.

## File

Use `File` to declare a parameter for file uploads (`multipart/form-data`). The parameter should be annotated with `bytes` or `UploadFile`.

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

### Parameters

`File` inherits from `Form` and accepts the same parameters. It sets the `media_type` to `multipart/form-data` by default.