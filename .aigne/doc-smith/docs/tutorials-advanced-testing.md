# Testing

Writing tests for your API is crucial for ensuring it behaves as expected and for maintaining quality as your application grows. FastAPI makes testing straightforward by providing a `TestClient` based on `httpx`.

This guide will walk you through setting up tests for a FastAPI application, including how to handle and override dependencies during testing.

## A Simple Example

Let's start with a basic application and its test in a single file. You'll need to install `pytest` and `httpx`:

```bash
pip install pytest httpx
```

Now, you can create a file, for example `test_app.py`, with your FastAPI application and the corresponding tests.

```python title="test_app.py"
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

Here's what's happening in the code above:

1.  **Import `TestClient`**: We import it from `fastapi.testclient`.
2.  **Create a `TestClient` instance**: We pass our FastAPI `app` to it.
3.  **Write a test function**: We define a standard `pytest` function, like `test_read_main`.
4.  **Make requests**: We use the `client` object to make requests to our app. It offers methods like `.get()`, `.post()`, `.put()`, etc., that mirror HTTP methods.
5.  **Add assertions**: We use standard `assert` statements to check the response status code and the JSON body.

To run the tests, save the file and execute `pytest` in your terminal. `pytest` will automatically discover and run the test function.

## Structuring Tests in a Real Application

In a real-world project, you'll typically separate your application code from your test code.

Let's say you have a `main.py` file for your application:

```python title="main.py" icon=logos:python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}
```

Your tests would go into a separate file, for example, `test_main.py`:

```python title="test_main.py" icon=logos:python
from fastapi.testclient import TestClient

from .main import app

client = TestClient(app)


def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```

This structure keeps your project organized. The key is to import your `app` instance into your test file and pass it to the `TestClient`.

## Testing Dependencies with Overrides

One of FastAPI's most powerful features is its Dependency Injection system. This system also makes it easy to override dependencies during testing, which is incredibly useful for isolating the code you want to test.

For example, you can replace a dependency that connects to a database with a mock that returns fixed data.

Consider this application with a shared dependency `common_parameters`:

```python title="dependency_testing_app.py" icon=logos:python
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


@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return {"message": "Hello Users!", "params": commons}

# --- Testing Section ---

client = TestClient(app)

# Define the dependency override
async def override_dependency(q: Union[str, None] = None):
    return {"q": q, "skip": 5, "limit": 10}

# Apply the override to the application
app.dependency_overrides[common_parameters] = override_dependency

def test_override_in_items():
    response = client.get("/items/")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": None, "skip": 5, "limit": 10},
    }

def test_override_in_items_with_q():
    response = client.get("/items/?q=foo")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": "foo", "skip": 5, "limit": 10},
    }

def test_override_in_items_with_params():
    response = client.get("/items/?q=foo&skip=100&limit=200")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": "foo", "skip": 5, "limit": 10},
    }
```

In the example above:

1.  We define an `override_dependency` function. This function will be used instead of `common_parameters` during tests.
2.  We update the `app.dependency_overrides` dictionary. This dictionary tells FastAPI: "whenever you see `common_parameters`, use `override_dependency` instead."
3.  The tests then call the `/items/` endpoint. As you can see in the assertions, the response always contains the fixed values from our override (`"skip": 5`, `"limit": 10`), regardless of the query parameters sent in the request URL.

This technique is essential for creating isolated and predictable tests for components that rely on external systems.

Now you have the tools to write comprehensive tests for your FastAPI applications. For more complex projects, you might want to learn how to structure [Bigger Applications](./tutorials-advanced-bigger-applications.md).