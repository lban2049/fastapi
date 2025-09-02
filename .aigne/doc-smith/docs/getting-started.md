# Getting Started

This tutorial guides you through creating and running your first FastAPI application. You will learn how to install FastAPI, write a simple API, run it locally, and explore its automatic documentation features.

## 1. Install FastAPI

First, you need to install FastAPI and a server to run it. We recommend using a virtual environment to manage your project's dependencies.

Install FastAPI along with its standard dependencies, which includes the Uvicorn server, using `pip`:

```console
$ pip install "fastapi[standard]"

---> 100%
```

**Note:** The quotes around `"fastapi[standard]"` are important to ensure the command works correctly in all terminals.

## 2. Create the Code

Create a file named `main.py` and add the following code:

```python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Union[str, None] = None):
    return {"item_id": item_id, "q": q}
```

This code defines a simple API with two endpoints:

*   `@app.get("/")`: A `GET` operation at the root path `/` that returns a simple JSON message.
*   `@app.get("/items/{item_id}")`: A `GET` operation that takes a path parameter `item_id` (which must be an integer) and an optional query parameter `q` (which can be a string).

## 3. Run the Application

Run the server from your terminal using the `fastapi` command:

```console
$ fastapi dev main.py

 ╭────────── FastAPI CLI - Development mode ───────────╮
 │                                                     │
 │  Serving at: http://127.0.0.1:8000                  │
 │                                                     │
 │  API docs: http://127.0.0.1:8000/docs               │
 │                                                     │
 │  Running in development mode, for production use:   │
 │                                                     │
 │  fastapi run                                        │
 │                                                     │
 ╰─────────────────────────────────────────────────────╯

INFO:     Will watch for changes in these directories: ['/home/user/code/awesomeapp']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [2248755] using WatchFiles
INFO:     Started server process [2248757]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

The `fastapi dev` command starts a local development server that automatically reloads when you make changes to your code.

## 4. Check It Out

Open your web browser and navigate to [http://127.0.0.1:8000/items/5?q=somequery](http://127.0.0.1:8000/items/5?q=somequery).

You will see the following JSON response:

```json
{"item_id": 5, "q": "somequery"}
```

You have successfully created an API that receives path and query parameters, validates their types, and returns a JSON response.

## 5. Explore the Interactive Docs

FastAPI automatically generates interactive documentation for your API. 

Navigate to [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) to see the Swagger UI documentation:

![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

There is also an alternative documentation interface provided by ReDoc. You can access it at [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc):

![ReDoc](https://fastapi.tiangolo.com/img/index/index-02-redoc-simple.png)

## 6. Add a Request Body

Now, let's modify the `main.py` file to handle data sent from a client in a request body. You can define the structure of the body using Pydantic models.

Update `main.py` with the following code:

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    price: float
    is_offer: Union[bool, None] = None


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Union[str, None] = None):
    return {"item_id": item_id, "q": q}


@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item):
    return {"item_name": item.name, "item_id": item_id}
```

Here's what changed:
- We imported `BaseModel` from `pydantic`.
- We created an `Item` class that inherits from `BaseModel` to define the request body's schema.
- We added a new path operation `@app.put("/items/{item_id}")` that accepts `PUT` requests and expects a request body matching the `Item` model.

## 7. See the Automatic Update

Because the server reloads automatically, your changes are already live. Refresh the documentation page at [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs). The interactive docs will update to include the new `PUT` endpoint:

![Swagger UI Updated](https://fastapi.tiangolo.com/img/index/index-03-swagger-02.png)

You can use the "Try it out" button to fill in the parameters and interact with your new endpoint directly from the browser.

## Recap and Next Steps

Congratulations! You have successfully created a FastAPI application that handles path parameters, query parameters, and request bodies, complete with automatic, interactive documentation.

Now that you understand the basics, you are ready to dive deeper into the core concepts of FastAPI.

<x-cards data-columns="3">
  <x-card data-title="Path Parameters" data-icon="lucide:milestone" data-href="/user-guide/path-parameters">
    Learn more about declaring and validating path parameters.
  </x-card>
  <x-card data-title="Query Parameters" data-icon="lucide:list-filter" data-href="/user-guide/query-parameters">
    Understand how to define and add more validations for query parameters.
  </x-card>
  <x-card data-title="Request Body" data-icon="lucide:file-code-2" data-href="/user-guide/request-body">
    Explore how to receive and validate complex data structures using Pydantic models.
  </x-card>
</x-cards>