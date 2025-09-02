# Request Body

When you need to send data from a client (like a browser) to your API, you send it as a **request body**. This is typically used with operations that create or update data, such as `POST`, `PUT`, and `PATCH`.

FastAPI leverages Pydantic models to define, validate, and document these request bodies, making it easy to handle complex data structures with minimal code.

## Create Your First Request Body

First, define your data structure as a Pydantic model. This model declares the shape of the data you expect, its fields, and their types.

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    return item
```

In this example:
- We define an `Item` model with `name`, `description`, `price`, and `tax`.
- The `create_item` function accepts an `item` parameter with the type hint `Item`.

With this single type declaration, FastAPI will:
1.  Read the body of the request as JSON.
2.  Convert the types to the corresponding Python types.
3.  Validate the data. If the data is invalid, it returns a clear error indicating what was wrong.
4.  Provide the received data in the `item` parameter.
5.  Generate a JSON Schema for your model, which will be used in the OpenAPI documentation.

## Using the Model

Inside your function, you can access all the attributes of the model object directly. You can also convert the model to a dictionary if needed.

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    item_dict = item.dict()
    if item.tax is not None:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```

Here, we convert the incoming `item` to a dictionary and add a calculated `price_with_tax` field if `tax` is provided.

## Combining Path, Query, and Body Parameters

You can declare path, query, and request body parameters in the same function. FastAPI will correctly identify each one and get the data from the appropriate source.

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, q: Union[str, None] = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result
```

In this `update_item` function:
- `item_id` is a **path parameter**.
- `item` is a **request body parameter**.
- `q` is a **query parameter**.

FastAPI handles them all simultaneously.

## Multiple Body Parameters and Fields

Sometimes you might want to receive multiple body parameters or embed a single model within a JSON key. For these cases, you can use the `Body` utility.

### Embed a Single Body Parameter

If you want the request body to be a JSON object with a specific key (e.g., `"item"`) that contains the model's data, you can use `Body(embed=True)`.

```python
from typing import Union

from fastapi import Body, FastAPI
from pydantic import BaseModel, Field

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = Field(
        default=None, title="The description of the item", max_length=300
    )
    price: float = Field(gt=0, description="The price must be greater than zero")
    tax: Union[float, None] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item = Body(embed=True)):
    results = {"item_id": item_id, "item": item}
    return results
```

Instead of a request body like this:
```json
{
    "name": "Foo",
    "description": "A very nice Item",
    "price": 35.4,
    "tax": 3.2
}
```

FastAPI will now expect a body like this:
```json
{
    "item": {
        "name": "Foo",
        "description": "A very nice Item",
        "price": 35.4,
        "tax": 3.2
    }
}
```

### Add Rich Validation with `Field`

Notice in the example above, we also used Pydantic's `Field`. This allows you to add extra validation and metadata to your model's attributes, such as `title`, `description`, `max_length`, and numeric constraints like `gt` (greater than).

This extra information is also used to generate a more detailed and accurate OpenAPI schema for your API documentation.

## Nested Models

You can define complex, nested JSON objects by using Pydantic models within other Pydantic models. For example, you can have lists of sub-models, or attributes that are themselves other models.

Here's an example where an `Item` can have a list of tags.

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: list = []


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

A valid request body for this endpoint could look like:

```json
{
    "name": "Foo",
    "description": "A very nice Item",
    "price": 35.4,
    "tax": 3.2,
    "tags": ["electronics", "hardware", "computer"]
}
```

FastAPI will automatically handle the validation for these nested structures. The `tags` field could even be a `list[OtherModel]` for more deeply nested data.

Now that you know how to handle data sent from the client, let's explore how to control what you send back in the [Handling Responses](./user-guide-handling-responses.md) section.