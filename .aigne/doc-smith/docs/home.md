# Home

FastAPI is a modern, fast (high-performance), web framework for building APIs with Python based on standard Python type hints.

It is built on the shoulders of giants:
- **[Starlette](https://www.starlette.io/)** for the web parts.
- **[Pydantic](https://docs.pydantic.dev/)** for the data parts.

## Key Features

<x-cards data-columns="2">
  <x-card data-title="Fast" data-icon="lucide:rocket">
    Very high performance, on par with NodeJS and Go. It is one of the fastest Python frameworks available.
  </x-card>
  <x-card data-title="Fast to Code" data-icon="lucide:file-code-2">
    Increases development speed by 200% to 300%.
  </x-card>
  <x-card data-title="Fewer Bugs" data-icon="lucide:bug-off">
    Reduces about 40% of human-induced errors.
  </x-card>
  <x-card data-title="Intuitive" data-icon="lucide:lightbulb">
    Great editor support with autocompletion everywhere, minimizing debugging time.
  </x-card>
  <x-card data-title="Easy" data-icon="lucide:graduation-cap">
    Designed to be easy to use and learn, so you spend less time reading docs.
  </x-card>
  <x-card data-title="Robust" data-icon="lucide:shield-check">
    Get production-ready code with automatic interactive documentation.
  </x-card>
  <x-card data-title="Short" data-icon="lucide:file-minus-2">
    Minimize code duplication. Get multiple features from each parameter declaration.
  </x-card>
  <x-card data-title="Standards-based" data-icon="lucide:book-check">
    Based on and fully compatible with the open standards for APIs: OpenAPI and JSON Schema.
  </x-card>
</x-cards>

## A Simple Example

Let's walk through a basic example to see FastAPI in action.

### 1. Create the Code

Create a file named `main.py`:

```python main.py icon=logos:python
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

### 2. Run the Server

Execute the following command in your terminal:

```console Terminal icon=mdi:console
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

### 3. Check the API

Open your browser and navigate to [http://127.0.0.1:8000/items/5?q=somequery](http://127.0.0.1:8000/items/5?q=somequery). You will see the following JSON response:

```json Response icon=logos:json
{
  "item_id": 5,
  "q": "somequery"
}
```

With just these few lines of code, you have created an API that:
*   Receives HTTP requests at `/` and `/items/{item_id}`.
*   Validates that `item_id` is an integer.
*   Handles an optional string query parameter `q`.
*   Converts data to and from JSON automatically.

### 4. Interactive API Documentation

FastAPI automatically generates interactive API documentation based on the OpenAPI standard.

#### Swagger UI

Navigate to [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) to see the Swagger UI documentation. You can view your endpoints and interact with your API directly from the browser.

![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

#### ReDoc

For an alternative documentation style, visit [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc).

![ReDoc](https://fastapi.tiangolo.com/img/index/index-02-redoc-simple.png)

## Performance

Independent TechEmpower benchmarks show FastAPI applications running under Uvicorn as [one of the fastest Python frameworks available](https://www.techempower.com/benchmarks/#section=test&runid=7464e520-0dc2-473d-bd34-dbdfd7e85911&hw=ph&test=query&l=zijzen-7), thanks to its foundation on Starlette for web handling and Pydantic for data validation and serialization.

## Next Steps

This was just a quick glimpse of what FastAPI can do. To truly understand its power and features, dive into our step-by-step guide.

<x-card data-title="Tutorial: First Steps" data-href="/tutorials/first-steps" data-icon="lucide:play-circle" data-cta="Start Tutorial">
  Begin the tutorial to install FastAPI and build your first application from the ground up.
</x-card>