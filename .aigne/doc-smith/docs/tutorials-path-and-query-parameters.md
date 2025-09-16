# Path and Query Parameters

When you build an API, you often need to get information from the client through the URL. FastAPI makes it easy to handle two common ways of doing this: **Path Parameters** and **Query Parameters**. By using standard Python type hints, you get data validation, conversion, and documentation for free, making your API more robust and easier to use.

This guide will walk you through how to declare, type, and validate both types of parameters.

## Path Parameters

Path parameters are parts of the URL path itself, enclosed in curly braces `{}`. They are typically used to identify a specific resource, like an item's ID.

### Declare a Path Parameter

You can declare a path parameter in the decorator and as a function argument. The name must match.

```python icon=logos:python title=main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

The value of the `item_id` from the path will be passed to your function as the argument `item_id`. If you run this and go to `http://127.0.0.1:8000/items/foo`, you will see:

```json
{
  "item_id": "foo"
}
```

### Path Parameters with Types

You can declare the type of a path parameter using standard Python type hints. This is where FastAPI's power really shines.

```python icon=logos:python title=main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

Now, `item_id` is declared as an `int`. FastAPI will automatically handle:

*   **Validation:** If you go to `/items/foo`, you'll get a clear HTTP 422 Unprocessable Entity error because 'foo' is not an integer.
*   **Conversion:** If you go to `/items/3`, FastAPI will convert the string "3" to the integer `3` before passing it to your function.

This automatic validation and conversion helps prevent bugs and improves the developer experience for your API users.

### Order Matters

When you have multiple path operations that could match a URL, FastAPI evaluates them in the order they are declared. A fixed path should always be declared before a variable path.

```python icon=logos:python title=main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}


@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

Because `/users/me` is declared before `/users/{user_id}`, a request to `/users/me` will correctly execute the `read_user_me` function. If the order were reversed, FastAPI would think "me" is a value for the `user_id` parameter.

### Predefined Values with Enums

If you have a path parameter that should only accept a few specific values, you can use a standard Python `Enum`.

```python icon=logos:python title=main.py
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

By using `ModelName` as a type hint, the path parameter `model_name` will be validated against the members of the enum. A request to `/models/alexnet` will work, but a request to `/models/unknown` will result in a 422 error with a helpful message indicating the allowed values.

### Path Parameters Containing Paths

Sometimes you need a path parameter to contain a file path, which includes slashes (`/`). You can tell FastAPI to capture a path using a special syntax in the decorator: `{param_name:path}`.

```python icon=logos:python title=main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

Now, a request to `/files/home/johndoe/myfile.txt` will work, and the `file_path` parameter will contain the full string `home/johndoe/myfile.txt`.

## Query Parameters

Query parameters are a set of key-value pairs that appear after the `?` in a URL. They are more flexible than path parameters and are often used for filtering, sorting, or pagination.

Any function parameter that is not part of the path is automatically interpreted as a query parameter.

### Default Values

You can provide default values for query parameters just as you would for any Python function argument.

```python icon=logos:python title=main.py
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

*   A request to `/items/` will use the defaults: `skip=0` and `limit=10`.
*   A request to `/items/?skip=20` will use `skip=20` and `limit=10`.
*   A request to `/items/?skip=0&limit=5` will override both defaults.

### Optional Parameters

To make a query parameter optional, you can declare it with a default value of `None`.

```python icon=logos:python title=main.py
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Union[str, None] = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

Here, `q` is an optional query parameter. If the client sends a request to `/items/foo-item?q=somequery`, the response will include `q`. If they request `/items/foo-item`, the `q` parameter will be `None`, and the `if q:` block will not execute.

### Advanced Validation

FastAPI allows for much more powerful validation on query parameters using the `Query` function. You can declare metadata like maximum length, regular expressions, and more.

```python icon=logos:python title=main.py
from typing import Union

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Union[str, None] = Query(default=None, max_length=50)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

In this example, the `q` parameter is still optional, but if it is provided, it cannot be longer than 50 characters. This kind of declarative validation keeps your application logic clean and your API secure.

## Recap

You've now seen how to handle the most common types of URL parameters. FastAPI's use of modern Python features makes your API robust, easy to write, and self-documenting.

*   **Path Parameters** are declared in the path string with `{}`.
*   **Query Parameters** are declared as function arguments that are not in the path.
*   **Type hints** provide automatic data conversion and validation for both.

Next, we'll explore how to handle data sent in the request body, which is essential for creating or updating items in your API.

[Next: Request Body](./tutorials-request-body.md)