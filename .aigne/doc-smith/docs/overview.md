# Overview

FastAPI is a modern, high-performance web framework for building APIs with Python, based on standard Python type hints. It is designed to be easy to use, fast to code, and ready for production environments.

## Key Features

FastAPI provides a development experience that is both efficient and enjoyable, focusing on speed, simplicity, and standards.

<x-cards data-columns="3">
  <x-card data-title="Fast Performance" data-icon="lucide:rocket">
    Achieve performance on par with NodeJS and Go, thanks to its Starlette and Pydantic foundation. It's one of the fastest Python frameworks available.
  </x-card>
  <x-card data-title="Fast to Code" data-icon="lucide:zap">
    Increase feature development speed by 200% to 300%. Minimize code duplication and get more done with less code.
  </x-card>
  <x-card data-title="Fewer Bugs" data-icon="lucide:shield-check">
    Reduce developer-induced errors by about 40%. Type hints and structured data validation catch bugs before they reach production.
  </x-card>
  <x-card data-title="Intuitive" data-icon="lucide:lightbulb">
    Benefit from excellent editor support with autocompletion everywhere. Spend less time debugging and more time building.
  </x-card>
  <x-card data-title="Easy to Learn" data-icon="lucide:book-open">
    Designed to be straightforward to learn and use. The documentation is clear, letting you focus on your application's logic.
  </x-card>
  <x-card data-title="Standards-Based" data-icon="lucide:file-json-2">
    Fully compatible with open standards for APIs, including OpenAPI (formerly Swagger) and JSON Schema.
  </x-card>
</x-cards>

## Core Architecture

FastAPI's performance and features are possible because it stands on the shoulders of giants:

-   **[Starlette](https://www.starlette.io/)**: Used for all the web parts, providing a lightweight and high-performance ASGI framework.
-   **[Pydantic](https://docs.pydantic.dev/)**: Handles all the data parts, offering robust data validation, serialization, and documentation based on Python type hints.

This separation of concerns allows FastAPI to excel at both web handling and data management.

```d2
direction: down

"Your-API-Code": {
  label: "Your API Code"
  shape: rectangle
  style.fill: "#DDF0FF"
}

FastAPI: {
  shape: rectangle
  "Your-API-Code"
}

Starlette: {
  shape: rectangle
  label: "Starlette (Web layer)"
  style.fill: "#D5E8D4"
}

Pydantic: {
  shape: rectangle
  label: "Pydantic (Data layer)"
  style.fill: "#FAD7AC"
}

FastAPI -> Starlette: "Uses for all web parts"
FastAPI -> Pydantic: "Uses for all data parts"

```

## A Glimpse of the Code

See how simple it is to create a fully functional API with automatic documentation.

### 1. Create a File

Create a file `main.py` with the following content:

```python title="main.py"
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

Execute this command in your terminal:

```console
$ fastapi dev main.py

INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

### 3. Check the Interactive Docs

FastAPI automatically generates interactive API documentation from your code. Just open your browser to [`http://127.0.0.1:8000/docs`](http://127.0.0.1:8000/docs) to see it in action.

![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

## Trusted by Industry Leaders

FastAPI is used in production by leading companies worldwide.

> "[...] I'm using **FastAPI** a ton these days. [...] I'm actually planning to use it for all of my team's **ML services at Microsoft**. Some of them are getting integrated into the core **Windows** product and some **Office** products."
> 
> **Kabir Khan - Microsoft**

> "We adopted the **FastAPI** library to spawn a **REST** server that can be queried to obtain **predictions**. [for Ludwig]"
> 
> **Piero Molino, Yaroslav Dudin, and Sai Sumanth Miryala - Uber**

> "**Netflix** is pleased to announce the open-source release of our **crisis management** orchestration framework: **Dispatch**! [built with **FastAPI**]"
> 
> **Kevin Glisson, Marc Vilanova, Forest Monsen - Netflix**

## Next Steps

This overview covers the high-level benefits of FastAPI. To start building your first API, head over to our step-by-step tutorial.

<x-card data-title="Getting Started" data-icon="lucide:play-circle" data-href="/getting-started" data-cta="Start the Tutorial">
  A step-by-step guide to install FastAPI and create your first application.
</x-card>