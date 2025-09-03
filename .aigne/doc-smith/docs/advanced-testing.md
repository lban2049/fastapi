# Testing

FastAPI provides a straightforward way to test your application using `TestClient`, which is built upon the powerful `httpx` library. This allows you to run tests against your app without needing a live server, making them fast and reliable.

## Basic Testing with `TestClient`

To start, you need to import `TestClient` and create an instance of it by passing your FastAPI application.

Here's a complete example of testing a simple endpoint:

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

In this test:
1.  We import `TestClient`.
2.  We create a `client` instance for our `app`.
3.  We define a test function, `test_read_main`.
4.  Inside the test, we use `client.get("/")` to make a request to the root path.
5.  We then use `assert` statements to verify that the HTTP status code is `200` (OK) and that the JSON response body matches the expected output.

## Testing WebSockets

You can also test WebSocket endpoints using the `websocket_connect()` method on the `TestClient`. It's recommended to use it as a context manager (with a `with` statement) to ensure the connection is properly closed.

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient
from fastapi.websockets import WebSocket

app = FastAPI()


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

Here, `client.websocket_connect("/ws")` establishes the connection, and `websocket.receive_json()` waits for and parses a JSON message from the server.

## Testing with Event Handlers

If your application uses `startup` or `shutdown` event handlers, you should use the `TestClient` as a context manager. This ensures that the event handlers are executed correctly before and after the tests within the `with` block.

Consider an application that initializes some data on startup:

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

By using `with TestClient(app) as client:`, the `startup_event` is guaranteed to run before any client requests are made, ensuring `items` is populated.

## Testing with Dependency Overrides

One of the most useful features for testing is the ability to override dependencies. This allows you to replace dependencies with mock versions for your tests, for example, to avoid making real database or network calls.

You can override a dependency by updating the `app.dependency_overrides` dictionary. The key is the original dependency function, and the value is the new function you want to use.

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

In this example:
- We define an `override_dependency` function with fixed `skip` and `limit` values.
- We replace the original `common_parameters` dependency with our override: `app.dependency_overrides[common_parameters] = override_dependency`.
- The test `test_override_in_items_with_params` shows that even when `skip` and `limit` are provided as query parameters, the values from the overridden dependency are used instead. The `q` parameter is still processed as it is part of the override function's signature.