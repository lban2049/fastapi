# Path Parameters

You can declare path "parameters" or "variables" with the same syntax used by Python format strings. This allows you to capture parts of the URL path and use them in your *path operation function*.

## Declare Path Parameters

A path parameter is defined in the path using curly braces `{}`.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

The value of the path parameter `item_id` will be passed to your function as the argument `item_id`. So, if you run this example and go to `http://127.0.0.1:8000/items/foo`, you will see the response:

```json
{
  "item_id": "foo"
}
```

## Path Parameters with Types

You can declare the type of a path parameter in the function, using standard Python type hints.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

In this case, `item_id` is declared to be an `int`. With this type declaration, FastAPI gives you automatic request "parsing". If you go to `http://127.0.0.1:8000/items/3` in your browser, the value `"3"` from the path is parsed and converted into the integer `3`.

The response will be:

```json
{
  "item_id": 3
}
```

### Data Validation

If you go to the URL `/items/foo` with the `int` type hint, you will see a clear HTTP error message, indicating that the path parameter has an invalid type.

```json
{
  "detail": [
    {
      "loc": [
        "path",
        "item_id"
      ],
      "msg": "value is not a valid integer",
      "type": "type_error.integer"
    }
  ]
}
```

This automatic validation is provided by Pydantic, which FastAPI uses under the hood.

## Order Matters

When creating *path operations*, you might have a situation where you have a fixed path (like `/users/me`) and a path that captures a parameter (like `/users/{user_id}`).

Because path operations are evaluated in order, you need to make sure that the path for the fixed endpoint is declared *before* the one with the parameter.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}


@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

If `/users/{user_id}` were declared first, it would match `/users/me`, thinking that the `user_id` parameter is the string `"me"`.

## Predefined Values with Enums

If you have a path parameter that can only accept a few predefined values, you can use a standard Python `Enum`.

Create an `Enum` class that inherits from `str` and `Enum`.

```python
from enum import Enum

from fastapi import FastAPI


class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"


app = FastAPI()


@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}

    return {"model_name": model_name, "message": "Have some residuals"}
```

FastAPI will use the enum to validate the path parameter and will also include the available values in the interactive API documentation.

## Path Parameters Containing Paths

There might be cases where you need a path parameter to contain a file path, which includes slashes (`/`). You can use a path converter from Starlette (the underlying ASGI framework) to achieve this.

To capture a path, use the syntax `{file_path:path}`.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

If you make a request to `/files/home/johndoe/myfile.txt`, the `file_path` parameter will contain the full path `home/johndoe/myfile.txt`.

## Numeric Validations

For more advanced validation, especially for numbers, you can use the `Path()` function.

First, import `Path` from `fastapi`:

```python
from fastapi import FastAPI, Path
```

You can use `Path()` to add extra metadata and validation checks.

### Add Metadata

You can add a `title` and other metadata to your path parameter. This information will be used in the generated OpenAPI schema and the interactive API docs.

```python
from typing import Union

from fastapi import FastAPI, Path, Query

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    item_id: int = Path(title="The ID of the item to get"),
    q: Union[str, None] = Query(default=None, alias="item-query"),
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

### Order the parameters as you need

When you use `Path()`, you might want to reorder the parameters. For example, having a required query parameter `q` before a path parameter. Python requires that parameters with default values come after those without. You can use a `*` in the function arguments to indicate that all subsequent arguments are keyword-only.

```python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(*, item_id: int = Path(title="The ID of the item to get"), q: str):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

### Number validations: greater than or equal

With `Path()`, you can declare numeric constraints. For instance, to ensure `item_id` is an integer greater than or equal to 1, you can use `ge=1`.

```python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    *, item_id: int = Path(title="The ID of the item to get", ge=1), q: str
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

### Number validations: greater than and less than or equal

You can also use `gt` (greater than) and `le` (less than or equal to).

```python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    *,
    item_id: int = Path(title="The ID of the item to get", gt=0, le=1000),
    q: str,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

### Number validations with floats

Number validations also work for `float` values. This example shows how you can combine path and query parameter validations.

```python
from fastapi import FastAPI, Path, Query

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    *,
    item_id: int = Path(title="The ID of the item to get", ge=0, le=1000),
    q: str,
    size: float = Query(gt=0, lt=10.5),
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    if size:
        results.update({"size": size})
    return results
```

## Recap

You can declare path parameters using f-string-like syntax. FastAPI provides powerful features for path parameters:

*   **Type Hinting**: Automatic parsing and data validation.
*   **Order Importance**: Fixed paths should be declared before paths with variables.
*   **Enums**: For predefined, allowed values.
*   **Path Converter**: To capture paths that include slashes.
*   **`Path()`**: For adding rich metadata and numeric validations (`gt`, `ge`, `lt`, `le`).

Next, we will explore how to declare [Query Parameters](./user-guide-query-parameters.md).