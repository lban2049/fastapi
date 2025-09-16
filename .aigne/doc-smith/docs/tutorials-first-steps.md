# First Steps

This guide will walk you through creating your first FastAPI application. You will learn how to install FastAPI and its required dependencies, write a simple "Hello World" app, and run it using the development server.

## Installation

First, you need to install FastAPI. It's recommended to work within a [virtual environment](https://fastapi.tiangolo.com/virtual-environments/). You can install FastAPI and a production-ready web server like Uvicorn using the `[standard]` extras.

```console title="Install FastAPI with standard dependencies"
$ pip install "fastapi[standard]"
```

The `"fastapi[standard]"` command (the quotes are important) installs FastAPI along with `uvicorn` to serve your application and other optional dependencies that are useful for development and high performance.

## Create Your First API

Create a file named `main.py` and add the following code:

```python main.py icon=logos:python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```

### Let's break down the code

1.  **`from fastapi import FastAPI`**: This line imports the `FastAPI` class from the `fastapi` library. This class is the core of your web application.

2.  **`app = FastAPI()`**: Here, you create an instance of the `FastAPI` class. This `app` variable will be the main point of interaction for creating all your API endpoints.

3.  **`@app.get("/")`**: This is a **path operation decorator**. It tells FastAPI that the function directly below it is responsible for handling requests that go to:
    *   The path `/` (the root URL).
    *   Using a `GET` operation (also known as an HTTP method).

4.  **`async def root(): ...`**: This is your **path operation function**. It's a standard Python function that will be called by FastAPI whenever it receives a `GET` request to the URL `/`. You can define it as a normal function (`def root():`) or an `async` function (`async def root():`) if you need to use `await` inside it.

5.  **`return {"message": "Hello World"}`**: You can return a `dict`, `list`, or other standard Python data types. FastAPI will automatically convert this dictionary into a JSON response.

## Run the Development Server

Now, run your application from the terminal:

```console title="Run the server"
$ fastapi dev main.py
```

The `fastapi dev` command finds your `app` object and starts a local server using Uvicorn. It also enables auto-reload, so the server will restart automatically whenever you save changes to your code.

You should see an output similar to this:

```console Terminal Output
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

INFO:     Will watch for changes in these directories: ['.']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [12345] using WatchFiles
INFO:     Started server process [12347]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

This output tells you that your application is running on `http://127.0.0.1:8000`.

## Check It

Open your web browser and navigate to `http://127.0.0.1:8000`.

You should see the following JSON response:

```json Response icon=mdi:code-json
{
  "message": "Hello World"
}
```

## Interactive API Docs

One of the best features of FastAPI is the automatic generation of interactive API documentation. You don't need to do anything extra to enable it.

Just go to `http://127.0.0.1:8000/docs` in your browser.

You'll see the Swagger UI, which provides a rich interface to explore and test your API endpoints directly from the browser.

![Swagger UI for the First Steps App](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

FastAPI also provides an alternative documentation style using ReDoc. You can access it at `http://127.0.0.1:8000/redoc`.

## Recap

Congratulations! You have successfully:

- Installed FastAPI and a web server.
- Created a simple API with a single endpoint.
- Run the application using a development server with auto-reload.
- Viewed the JSON response and the automatic interactive documentation.

## Next Steps

Now that you have a basic application running, let's learn how to handle data coming from the client. In the next section, you'll learn about **Path and Query Parameters**.

<x-card data-title="Next: Path and Query Parameters" data-icon="lucide:arrow-right-circle" data-href="/tutorials/path-and-query-parameters" data-cta="Continue Tutorial">
  Learn how to declare parameters that are part of the URL path and query string.
</x-card>