# Path Parameters

You can declare path "parameters" or "variables" with the same syntax used by Python f-strings:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

The value of the path parameter `item_id` will be passed to your function as the argument `item_id`.

So, if you run this example and go to `http://127.0.0.1:8000/items/foo`, you will see a response of:

```json
{
  "item_id": "foo"
}
```

## Path parameters with types

You can declare the type of a path parameter in the function, using standard Python type annotations.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

In this case, `item_id` is declared to be an `int`.

This will give you editor support inside of your function, with error checks, completion, etc.

With this type declaration, FastAPI provides automatic data parsing and validation. If you go to `http://127.0.0.1:8000/items/3`, the `item_id` will be converted to the integer `3`. However, if you go to `http://127.0.0.1:8000/items/foo`, you will see a helpful HTTP error:

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

This is because the path parameter `item_id` failed to parse as an `int`.

## Order matters

When creating *path operations*, you may find a situation where you have a fixed path, like `/users/me`, and a path with a parameter, like `/users/{user_id}`. Because path operations are evaluated in order, you need to ensure that the path for `/users/me` is declared before the one for `/users/{user_id}`.

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

Otherwise, the path for `/users/{user_id}` would also match `/users/me`, thinking that it's receiving a parameter `user_id` with a value of `"me"`.

## Predefined values

If you have a *path operation* that should only receive a predefined set of values, you can use a standard Python `Enum`.

Create an `Enum` that inherits from `str` and `Enum`.

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

By inheriting from `str`, the API docs will be able to know that the values must be strings and will be able to render correctly.

Then you can use it in a type annotation. The path parameter will be validated against the set of values in the enum.

If you visit `http://127.0.0.1:8000/models/resnet`, you'll get a response like:

```json
{
  "model_name": "resnet",
  "message": "Have some residuals"
}
```

The interactive docs will automatically show the available values in a dropdown menu.

## Path parameters containing paths

You can declare a path parameter that contains a path itself, using a URL converter.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

In this example, the parameter `file_path` can contain slashes (`/`), such as `home/johndoe/myfile.txt`.

If you go to `http://127.0.0.1:8000/files/home/johndoe/myfile.txt`, the response will be:

```json
{
  "file_path": "home/johndoe/myfile.txt"
}
```

## Path parameters and numeric validations

FastAPI allows you to declare additional validations and metadata for your parameters. For path parameters, you can use `Path`.

To use it, you first need to import `Path` from `fastapi`:

```python
from fastapi import FastAPI, Path
```

Since path parameters are always required, you must declare them with `...` as the default value to mark them as required.

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

When you need to declare a query parameter `q` and a path parameter `item_id`, Python's syntax rules state that parameters with a default value must come after those without one. 

However, you can reorder them by using `*` in the function signature. This tells Python that all subsequent arguments are keyword-only arguments and their order doesn't matter.

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

### Number validations: greater or equal

With `Path` you can also declare numeric validations.

The `ge=1` parameter will enforce that `item_id` must be an integer "greater than or equal to" 1.

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

Number validations also work for `float` values. This is useful not only for `Path` but also for `Query` parameters.

Here we add a `size` query parameter that must be greater than 0 and less than 10.5.

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

You have seen how to:
- Use Python type hints for path parameters.
- Control the order of path operations.
- Use `Enum` for predefined, allowed path parameter values.
- Define path parameters that can contain paths themselves.
- Use `Path` to declare metadata and numeric validations.

Now you can proceed to learn about [Query Parameters](./user-guide-query-parameters.md).