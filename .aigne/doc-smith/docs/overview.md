# Overview

FastAPI is a modern, high-performance web framework for building APIs with Python, leveraging standard Python type hints. It is designed to be easy to use, fast to code, and ready for production environments.


### Key Features

FastAPI is built to optimize the development experience and the performance of the final application.

<x-cards data-columns="2">
  <x-card data-title="Fast Performance" data-icon="lucide:rocket">
    Achieve high performance, comparable to **NodeJS** and **Go**, thanks to its foundation on Starlette (for web parts) and Pydantic (for data parts). It is one of the fastest Python frameworks available.
  </x-card>
  <x-card data-title="Fast to Code" data-icon="lucide:zap">
    Increase development speed by 200% to 300%. The framework is designed to help you build features quickly with minimal, intuitive code.
  </x-card>
  <x-card data-title="Fewer Bugs" data-icon="lucide:bug-off">
    Reduce human-induced errors by about 40%. With type hints, you get excellent editor support and data validation, catching many errors during development.
  </x-card>
  <x-card data-title="Robust and Ready" data-icon="lucide:shield-check">
    Get production-ready code with automatic interactive documentation, data validation, and serialization based on open standards.
  </x-card>
</x-cards>

### Core Architecture

FastAPI stands on the shoulders of two giants: Starlette for all the web parts and Pydantic for all the data parts. This layered approach allows it to provide high performance and robust data handling simultaneously.

```d2
direction: right

"User Request" -> "FastAPI Engine"

"FastAPI Engine": {
  shape: cloud
  "Your API Code (with Type Hints)": {
    shape: document
  }
  "Starlette (Web Toolkit)": {
    shape: hexagon
  }
  "Pydantic (Data Validation)": {
    shape: hexagon
  }
}

"FastAPI Engine" -> "API Response (JSON)"

"Your API Code (with Type Hints)" -> "Starlette (Web Toolkit)": Uses for routing
"Your API Code (with Type Hints)" -> "Pydantic (Data Validation)": Uses for validation & serialization
```

### A Quick Example

Creating a FastAPI application is straightforward. Here’s a complete example:

**1. Create a file `main.py`:**

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "q": q}
```

**2. Run the server:**

```console
$ fastapi dev main.py

INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

**3. Check the result:**

Open your browser at [http://127.0.0.1:8000/items/5?q=somequery](http://127.0.0.1:8000/items/5?q=somequery). You will see the JSON response:

```json
{"item_id":5,"q":"somequery"}
```

### Automatic Interactive Documentation

One of FastAPI's most valued features is the automatic generation of interactive API documentation. Without any extra effort, you get two documentation UIs:

- **Swagger UI**, available at `/docs`:

  ![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

- **ReDoc**, available at `/redoc`:

  ![ReDoc](https://fastapi.tiangolo.com/img/index/index-02-redoc-simple.png)

### Trusted by Industry Leaders

FastAPI is used in production by leading tech companies for critical services.

> "[...] I'm using **FastAPI** a ton these days. [...] I'm actually planning to use it for all of my team's **ML services at Microsoft**. Some of them are getting integrated into the core **Windows** product and some **Office** products."
> <div style="text-align: right; margin-right: 10%;">Kabir Khan - <strong>Microsoft</strong></div>

> "We adopted the **FastAPI** library to spawn a **REST** server that can be queried to obtain **predictions**. [for Ludwig]"
> <div style="text-align: right; margin-right: 10%;">Piero Molino, Yaroslav Dudin, and Sai Sumanth Miryala - <strong>Uber</strong></div>

> "**Netflix** is pleased to announce the open-source release of our **crisis management** orchestration framework: **Dispatch**! [built with **FastAPI**]"
> <div style="text-align: right; margin-right: 10%;">Kevin Glisson, Marc Vilanova, Forest Monsen - <strong>Netflix</strong></div>

### Next Steps

This overview provides a glimpse into what makes FastAPI a compelling choice for API development. You've seen its key features, a simple code example, and the power of its automatic documentation.

Ready to build your first application? Head over to our [Getting Started](./getting-started.md) guide for a step-by-step tutorial.
