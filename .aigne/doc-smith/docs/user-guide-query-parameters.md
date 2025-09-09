# Query Parameters

When you declare function parameters that are not part of the path, they are automatically interpreted as "query" parameters.

The query is the set of key-value pairs that go after the `?` in a URL, separated by `&` characters. For example, in the URL `http://1227.0.0.1:8000/items/?skip=0&limit=10`, the query parameters are `skip` and `limit`.

Since they are part of the URL, they are "naturally" strings. But when you declare them with Python types (e.g., `int`, `float`, `bool`), FastAPI automatically converts and validates them.

This guide covers how to define, validate, and document query parameters. For details on path parameters, refer to the [Path Parameters](./user-guide-path-parameters.md) guide.

## Defaults

Query parameters can have default values. If the client doesn't provide a value for a parameter with a default, FastAPI will use that default value.

Here, `skip` and `limit` have default values of `0` and `10`:

```python title="tutorial001.py"
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]

```

If you go to the URL:

`http://127.0.0.1:8000/items/`

The values of `skip` and `limit` will be `0` and `10` respectively.

If you go to:

`http://127.0.0.1:8000/items/?skip=20`

Then `skip` will be `20` and `limit` will remain `10`.

## Optional Parameters

You can declare optional query parameters by using `Union` (or `|` in Python 3.10+) and setting the default value to `None`.

```python title="tutorial002.py"
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Union[str, None] = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}

```

In this case, the parameter `q` is optional. If the client provides it, it will be used. If not, its value will be `None`.

## Query Parameter Type Conversion

FastAPI automatically converts the string values from the URL into the specified Python type. For example, a `bool` parameter will recognize values like `true`, `1`, `on`, `yes` as `True`, and `false`, `0`, `off`, `no` as `False`.

```python title="tutorial003.py"
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Union[str, None] = None, short: bool = False):
    item = {"item_id": item_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item

```

If you visit `http://127.0.0.1:8000/items/foo?short=1` or `http://127.0.0.1:8000/items/foo?short=true`, the `short` parameter will be `True`, and the long description will be omitted from the response.

## Multiple Path and Query Parameters

You can declare multiple path and query parameters in any order. FastAPI is smart enough to figure out which is which based on the path string and the function signature.

```python title="tutorial004.py"
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: Union[str, None] = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item

```

FastAPI knows that `user_id` and `item_id` are part of the path, while `q` and `short` are query parameters.

## Required Query Parameters

To make a query parameter required, simply declare it without a default value.

```python title="tutorial005.py"
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item

```

Here, the function expects a required query parameter `needy`. If you try to call `http://127.0.0.1:8000/items/foo-item` without adding the `needy` parameter, FastAPI will respond with a clear HTTP error.

A valid request would be: `http://127.0.0.1:8000/items/foo-item?needy=sooooneedy`.

## Mixing Required, Optional, and Default Parameters

You can mix required parameters, parameters with default values, and optional parameters freely.

```python title="tutorial006.py"
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_user_item(
    item_id: str, needy: str, skip: int = 0, limit: Union[int, None] = None
):
    item = {"item_id": item_id, "needy": needy, "skip": skip, "limit": limit}
    return item

```

- `item_id`: A required path parameter.
- `needy`: A required query parameter.
- `skip`: A query parameter with a default value of `0`.
- `limit`: An optional query parameter.

## Additional Validation with `Query`

For more advanced validations on query parameters, you can use the `Query` function. This allows you to set constraints like minimum/maximum length, regular expressions, and more.

First, import `Query` from `fastapi`:

```python
from fastapi import FastAPI, Query
```

### String Validations

You can set constraints like `min_length` and `max_length` for string parameters.

To use `Query`, you replace the default value of your parameter with `Query(...)`.

For example, to declare an optional `q` parameter with a minimum length of 3 and a maximum length of 50, you would do:

```python title="tutorial003.py"
from typing import Union

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Union[str, None] = Query(default=None, min_length=3, max_length=50),
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results

```

Here, the `q` parameter is optional (because `default=None`), but if it is provided, it must have a length between 3 and 50 characters.

### Regular Expression Validation

You can also enforce a regular expression pattern using the `pattern` argument.

```python title="tutorial004.py"
from typing import Union

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Union[str, None] = Query(
        default=None, min_length=3, max_length=50, pattern="^fixedquery$"
    ),
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results

```

In this example, the value of `q` must be exactly `fixedquery`.

### List / Multiple values

When you define a query parameter with a type hint of `List`, it can accept multiple values. For example, `List[str]`.

```python
from typing import List, Union

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Union[List[str], None] = Query(default=None)):
    query_items = {"q": q}
    return query_items
```

With this, you can send requests like `http://127.0.0.1:8000/items/?q=foo&q=bar`. FastAPI will correctly parse `q` as a Python list: `["foo", "bar"]`.

### Other Validations

`Query` supports many other validation parameters. Here are some of the most common ones:

| Parameter | Description |
|---|---|
| `alias` | An alternative name for the parameter in the URL (e.g., `item-query` instead of `q`). |
| `title` | A human-readable title for the parameter in the documentation. |
| `description` | A detailed description for the parameter. |
| `deprecated` | Marks the parameter as deprecated in the OpenAPI documentation. |
| `gt`, `ge`, `lt`, `le` | For numeric validations: greater than, greater than or equal to, less than, less than or equal to. |
| `include_in_schema` | Set to `False` to exclude the parameter from the generated OpenAPI schema. |


## Summary

FastAPI provides a powerful and intuitive way to handle query parameters. You can define them with types and default values directly in your function signature, and FastAPI will handle the conversion and basic validation. For more complex scenarios, `Query` offers a rich set of validation options to enforce fine-grained constraints on your API's inputs.

After handling path and query parameters, the next step is often to handle data sent in the request body. Learn more about it in the next chapter on [Request Body](./user-guide-request-body.md).
