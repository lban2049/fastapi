# Request Body

When you need to send data from a client (like a browser) to your API, you send it as a **request body**. A request body is data sent by the client to your API. A **response body** is the data your API sends to the client.

Your API almost always has to send a response body. But clients don't necessarily need to send request bodies all the time. To declare a request body, you use Pydantic models with all their power and benefits.

## Create your data model

First, you need to import `BaseModel` from `pydantic`.

Then, you declare your data model as a class that inherits from `BaseModel`. Use standard Python types for all the attributes.

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

When a model attribute has a default value, it is not required. Otherwise, it is required. Use `None` to make it just optional.

For example, in the model above, `name` and `price` are required, while `description` and `tax` are optional.

## Declare it as a parameter

To add it to your *path operation*, declare it the same way you declared path and query parameters:

```python
@app.post("/items/")
async def create_item(item: Item):
    return item
```

...and declare its type as the model you created, `Item`.

With just that Python type declaration, **FastAPI** will:

*   Read the body of the request as JSON.
*   Convert the corresponding types (if needed).
*   Validate the data. If the data is invalid, it will return a nice and clear error, indicating the exact position and description of the incorrect data.
*   Give you the received data in the parameter `item`.
*   Generate JSON Schema definitions for your model, you can also use them anywhere else in your project if it makes sense.
*   Those schemas will be part of the generated OpenAPI schema, and used by the automatic documentation UIs.

## Use the model

Inside of the function, you can access all the attributes of the model object directly:

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

## Request body + path parameters

You can declare path parameters and a request body at the same time. **FastAPI** will recognize that the function parameters that match path parameters should be taken from the path, and that function parameters that are declared to be Pydantic models should be taken from the request body.

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
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.dict()}
```

## Request body + path + query parameters

You can also declare **body**, **path** and **query** parameters, all at the same time.

**FastAPI** will recognize each of them and take the data from the correct place.

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

The function parameters will be recognized as follows:

*   If the parameter is also declared in the **path**, it will be used as a path parameter.
*   If the parameter is of a **singular type** (like `int`, `float`, `str`, `bool`, etc) it will be interpreted as a **query** parameter.
*   If the parameter is declared to be of a **Pydantic model** type, it will be interpreted as a request **body**.

## Mix multiple parameters

You can mix `Path`, `Query` and request body declarations in your *path operation function* and FastAPI will handle all of them.

```python
from typing import Union

from fastapi import FastAPI, Path
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


@app.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int = Path(title="The ID of the item to get", ge=0, le=1000),
    q: Union[str, None] = None,
    item: Union[Item, None] = None,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    if item:
        results.update({"item": item})
    return results
```

## Nested Models

You can define complex, nested JSON objects in your request bodies by nesting Pydantic models.

For example, an item can have a list of tags. For this, you can define the `tags` attribute as a list.

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

For better type safety and editor support, you can be more specific about the items in the list, for example, `tags: list[str] = []`. You can also have lists of other Pydantic models to create deeper levels of nesting.

## Embed a single body parameter

By default, if you declare a single Pydantic model in your function, its content is expected as the direct body of the request. However, you can instruct FastAPI to expect a JSON object with a specific key. You can achieve this by using `Body`.

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

In this case, FastAPI will expect a body like:

```json
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    }
}
```

Instead of:

```json
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2
}
```

This also demonstrates using `Field` to add extra validation and metadata to your Pydantic model attributes.

---

Now that you know how to handle data sent from the client, let's explore how to control what you send back.

Next up, learn how to configure the [Handling Responses](./user-guide-handling-responses.md).