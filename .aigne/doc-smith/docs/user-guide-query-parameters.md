# Query Parameters

When you declare function parameters that are not part of the path parameters, they are automatically interpreted as "query" parameters.

The query is the set of key-value pairs that appear after the `?` in a URL. For example, in the URL `http://127.0.0.1:8000/items/?skip=0&limit=10`, the query parameters are `skip` and `limit`.

They are a standard part of the URL and FastAPI knows how to extract them.

To learn about declaring parameters that are part of the URL path, see the [Path Parameters](./user-guide-path-parameters.md) documentation.

## Default Values

Query parameters can be defined with default values, just like any Python function parameter. If the client does not provide a value for a parameter with a default, that default value will be used.

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

In this example, `skip` and `limit` are query parameters. Since they have default values, you can access the endpoint like this:

*   `http://127.0.0.1:8000/items/`

This will use the defaults `skip=0` and `limit=10`. You can also provide specific values:

*   `http://127.0.0.1:8000/items/?skip=20` (uses `limit=10` by default)
*   `http://127.0.0.1:8000/items/?skip=0&limit=5`

## Optional Parameters

You can declare optional query parameters by setting their default value to `None` and using `Union` or `Optional` from Python's `typing` library.

```python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Union[str, None] = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}

```

Here, the query parameter `q` is optional. If you call `/items/foo`, `q` will be `None`. If you call `/items/foo?q=somequery`, `q` will be `"somequery"`.

## Query Parameter Type Conversion

FastAPI automatically converts the types of query parameters based on your type hints. For example, a boolean parameter will correctly interpret values like `true`, `on`, `1`, or `yes`.

```python
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

If you access the URL `/items/foo?short=true`, the `short` parameter will be converted to the boolean `True`.

## Multiple Path and Query Parameters

You can declare multiple path and query parameters in any order. FastAPI is smart enough to distinguish them based on their names and where they are declared.

```python
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

In this example, `user_id` and `item_id` are path parameters, while `q` and `short` are query parameters.

## Required Query Parameters

If you declare a query parameter without a default value, it becomes required. The client must provide a value for it in the URL.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item
```

If you try to access `/items/foo-item` without the `needy` query parameter, FastAPI will return a clear HTTP error indicating that the parameter is missing.

You can also mix required, optional, and default parameters:

```python
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

Here:
- `needy` is a required `str`.
- `skip` is an `int` with a default value of `0`.
- `limit` is an optional `int`.

## Advanced Validation and Metadata with `Query`

For more advanced validations and to add metadata to your query parameters, you can use the `Query` function. It allows you to set constraints like minimum and maximum length, regular expressions, and more.

First, import `Query` from `fastapi`:

```python
from fastapi import Query
```

### String Validations

You can apply various validations to string parameters. For example, you can enforce a minimum and maximum length.

```python
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

Now, if you provide a `q` parameter that is shorter than 3 characters or longer than 50, you'll receive an error.

You can also enforce a regular expression pattern:

```python
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
In this case, the `q` parameter must be exactly `fixedquery`.

### Alias Parameters

Sometimes you need a query parameter name that isn't a valid Python identifier (e.g., `item-query`). You can use the `alias` argument in `Query` to define an alternative name for the parameter.

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: str = Query(alias="item-query")):
    return {"q": q}
```

Now, you can call the endpoint with `http://127.0.0.1:8000/items/?item-query=somevalue`.

### Deprecating Parameters

You can mark a parameter as deprecated by setting `deprecated=True`. This will be reflected in the interactive API documentation.

```python
from typing import Union

from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Union[str, None] = Query(
        default=None,
        alias="item-query",
        title="Query string",
        description="Query string for the items to search in the database",
        min_length=3,
        deprecated=True,
    )
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

---

## Recap

You have learned how to:
- Define query parameters with default, optional, and required values.
- Let FastAPI handle automatic type conversion.
- Use `Query` to add advanced validations like `min_length`, `max_length`, and `pattern`.
- Create aliases for parameters and mark them as deprecated.

Next, you will learn how to handle data sent in the request body.

<x-card data-title="Next: Request Body" data-icon="lucide:file-json-2" data-href="/user-guide/request-body">
  Learn how to receive and validate data from the request body using Pydantic models.
</x-card>
