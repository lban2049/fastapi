# Testing

FastAPI provides a straightforward way to test your API. It is based on Starlette's `TestClient` and uses `httpx`, which makes it compatible with `async` applications. You can use it directly with testing frameworks like `pytest`.

This guide covers how to write effective tests for your application, including WebSockets, event handlers, and dependency overrides.

## A Basic Example

Let's start with a simple FastAPI application in a file, for example, `main.py`:

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}


client = TestClient(app)


def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```

To test it, you import `TestClient` and pass your FastAPI `app` object to it. You can then make requests to your application within your test functions as if you were using `httpx` or `requests`.

The test function `test_read_main` does the following:
1.  Makes a `GET` request to `/` using `client.get("/")`.
2.  Asserts that the response status code is `200` (OK).
3.  Asserts that the JSON body of the response is `{"msg": "Hello World"}`.

## Testing WebSockets

Testing WebSocket endpoints follows a similar pattern, using a context manager to establish the connection.

Consider this application with a WebSocket endpoint:

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient
from fastapi.websockets import WebSocket

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}


@app.websocket("/ws")
async def websocket(websocket: WebSocket):
    await websocket.accept()
    await websocket.send_json({"msg": "Hello WebSocket"})
    await websocket.close()


def test_websocket():
    client = TestClient(app)
    with client.websocket_connect("/ws") as websocket:
        data = websocket.receive_json()
        assert data == {"msg": "Hello WebSocket"}

```

The `client.websocket_connect("/ws")` method provides a context manager. Inside the `with` block, you can interact with the WebSocket, for example, by receiving JSON data with `websocket.receive_json()` and then making assertions on that data.

## Testing with Event Handlers

If your application uses startup or shutdown events, you should use the `TestClient` as a context manager (`with TestClient(app) as client:`) to ensure these events are triggered correctly during testing.

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()

items = {}


@app.on_event("startup")
async def startup_event():
    items["foo"] = {"name": "Fighters"}
    items["bar"] = {"name": "Tenders"}


@app.get("/items/{item_id}")
async def read_items(item_id: str):
    return items[item_id]


def test_read_items():
    with TestClient(app) as client:
        response = client.get("/items/foo")
        assert response.status_code == 200
        assert response.json() == {"name": "Fighters"}
```

In this example, the `startup_event` populates a dictionary with some data. By using the `with` statement, we ensure this event runs before any requests are made by the `client`. This allows the test to correctly access the data at `/items/foo` that was created during the application startup phase.

## Testing Dependencies with Overrides

One of the most useful features for testing is the ability to override dependencies. This allows you to replace complex dependencies, like database connections or external API clients, with mock or simplified versions for your tests. You can achieve this by modifying the `app.dependency_overrides` dictionary.

```python
from typing import Union

from fastapi import Depends, FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


async def common_parameters(
    q: Union[str, None] = None, skip: int = 0, limit: int = 100
):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return {"message": "Hello Items!", "params": commons}


client = TestClient(app)


async def override_dependency(q: Union[str, None] = None):
    return {"q": q, "skip": 5, "limit": 10}


app.dependency_overrides[common_parameters] = override_dependency


def test_override_in_items():
    response = client.get("/items/")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": None, "skip": 5, "limit": 10},
    }


def test_override_in_items_with_params():
    response = client.get("/items/?q=foo&skip=100&limit=200")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": "foo", "skip": 5, "limit": 10},
    }
```

Here's a breakdown of the process:
1.  We have a `common_parameters` dependency used by an endpoint.
2.  For our tests, we create an `override_dependency` function that returns fixed values for `skip` and `limit`.
3.  The line `app.dependency_overrides[common_parameters] = override_dependency` tells FastAPI that whenever `common_parameters` is required as a dependency, it should use `override_dependency` instead.
4.  The tests demonstrate that even when we provide different query parameters for `skip` and `limit` in the URL, the values from the overridden dependency are used, confirming the override was successful.

---

You have now seen how to write tests for your application, handle WebSockets, ensure event handlers are executed, and override dependencies for isolated testing. To learn more about organizing your project as it grows, check out the guide on [Bigger Applications](./advanced-bigger-applications.md).