# Request Body

When you need to send data from a client (like a browser) to your API, you send it as a **request body**. A request body is the data sent by the client to your API. A **response body** is the data your API sends back to the client.

Your API almost always has to send a response body. But clients don't necessarily need to send request bodies all the time. To declare a request body, you use Pydantic models, which give you all the power of data validation, conversion, and documentation.

## Create Your Pydantic Model

First, you need to define the structure of your data as a Pydantic `BaseModel`.

```python title="main.py" icon=logos:python
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

By declaring the `item` parameter with the type hint `Item`, FastAPI will:

*   Read the body of the request as JSON.
*   Convert the corresponding types (if needed).
*   Validate the data. If the data is invalid, it will return a clear error, indicating exactly where and what the incorrect data was.
*   Give you the received data in the parameter `item`.
*   Generate JSON Schema definitions for your model, which will be used in the OpenAPI documentation.

### Parameter Details

<x-field data-name="item" data-type="Item" data-required="true" data-desc="An item object received in the request body.">
  <x-field data-name="name" data-type="string" data-required="true" data-desc="The name of the item."></x-field>
  <x-field data-name="description" data-type="string | None" data-required="false" data-desc="An optional description of the item."></x-field>
  <x-field data-name="price" data-type="float" data-required="true" data-desc="The price of the item."></x-field>
  <x-field data-name="tax" data-type="float | None" data-required="false" data-desc="An optional tax amount."></x-field>
</x-field>

### Example Response

When you send a request with a valid JSON body, the API will return it as is.

```json
{
  "name": "Sample Item",
  "description": "A sample description",
  "price": 19.99,
  "tax": 1.60
}
```

## Use the Model

Inside your *path operation function*, you can access all the attributes of the model object directly:

```python title="main.py" icon=logos:python
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

In this example, we convert the Pydantic model to a dictionary using `item.dict()`, calculate a new `price_with_tax` value if `tax` is provided, and return the updated dictionary.

### Example Response

```json
{
  "name": "Sample Item",
  "description": "A sample description",
  "price": 19.99,
  "tax": 1.60,
  "price_with_tax": 21.59
}
```

## Mix Path, Query, and Request Body

You can declare path parameters, query parameters, and request body parameters all at once. FastAPI will recognize each of them and take the data from the correct place.

```python title="main.py" icon=logos:python
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

The parameters will be recognized as follows:

*   **`item_id`**: A path parameter, as it's declared in the path.
*   **`q`**: A query parameter, as it's a singular type.
*   **`item`**: A request body parameter, as it's declared as a Pydantic model.

## Add Extra Validation with `Field`

You can declare more validations and metadata for your model attributes using Pydantic's `Field`.

```python title="main.py" icon=logos:python
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

Here, we've added:
*   A `title` and `max_length` for the `description`.
*   A `description` and a validation rule (`gt=0`, greater than 0) for the `price`.

### Embedding a Single Body Parameter

Notice the `item: Item = Body(embed=True)` in the function signature. By default, FastAPI expects the JSON body directly. However, if you use `Body(embed=True)`, it will expect the body to be embedded within a key. Instead of `{"name": "Foo", ...}`, the client must send `{"item": {"name": "Foo", ...}}`.

## Nested Models

Pydantic models can be nested. You can define a model that contains attributes that are other models, lists, etc.

```python title="main.py" icon=logos:python
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

In this example, the `Item` model includes a `tags` attribute which is a list. The client can send a JSON array for this field.

**Example Request Body**
```json
{
  "name": "T-Shirt",
  "description": "A nice cotton t-shirt",
  "price": 15.50,
  "tax": 1.24,
  "tags": ["clothing", "apparel", "summer"]
}
```

## Form Data

When you need to receive form fields instead of JSON, you can use `Form`. This is typically used for data sent with `application/x-www-form-urlencoded`.

To use forms, you first need to install `python-multipart`:

```bash
pip install python-multipart
```

Then, use `Form` in your path operation:

```python title="main.py" icon=logos:python
from fastapi import FastAPI, Form

app = FastAPI()


@app.post("/login/")
async def login(username: str = Form(), password: str = Form()):
    return {"username": username}
```

## File Uploads

FastAPI also supports file uploads using `File` and `UploadFile`. This also requires `python-multipart` to be installed.

There are two main ways to handle uploads:

1.  **As bytes**: Use `bytes = File()`. This is good for small files as it stores the entire contents in memory.
2.  **As `UploadFile`**: Use `file: UploadFile`. This is more efficient for large files because it streams the file to disk.

```python title="main.py" icon=logos:python
from fastapi import FastAPI, File, UploadFile

app = FastAPI()


@app.post("/files/")
async def create_file(file: bytes = File()):
    return {"file_size": len(file)}


@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile):
    return {"filename": file.filename}
```

An `UploadFile` object has several useful attributes and methods, including:
*   `filename`: The name of the uploaded file.
*   `content_type`: The content type (MIME type) of the file.
*   `file`: A `SpooledTemporaryFile` (a file-like object).
*   `async` methods like `read()`, `write()`, and `seek()`.

Now you have a solid understanding of how to handle various types of request bodies. You can receive simple JSON, nested structures, form data, and even file uploads.

Next, let's explore how to manage dependencies and add security to your application. You can continue to the [Dependencies and Security](./tutorials-dependencies-and-security.md) guide.