# API Reference

This section provides detailed technical documentation for FastAPI's public API. It is designed for developers who need to look up specific classes, functions, and parameters. Unlike the [User Guide](./user-guide.md), which is tutorial-based, this reference is structured for quick access to comprehensive information.

Below is a high-level overview of how FastAPI's main components interact. The application instance is the core, managing routers and path operations. Each path operation processes incoming requests by validating parameters, resolving dependencies, and generating responses.

```d2
direction: down

Incoming-Request: {
  label: "Incoming Request"
  shape: circle
}

FastAPI-App: {
  label: "FastAPI App"
  shape: rectangle
  style.fill: "#e6f7ff"

  APIRouter: {
    label: "APIRouter"
    shape: package
    style.fill: "#f6ffed"

    Path-Operation: {
      label: "Path Operation\n(@app.get, etc.)"
      shape: rectangle
      style.fill: "#fffbe6"
    }
  }
}

Parameters: {
  shape: package
  grid-columns: 2
  Path: {}
  Query: {}
  Body: {}
  Header: {}
  Cookie: {}
  Form: {}
}

Dependencies: {
  shape: package
  grid-columns: 2
  Depends: {}
  Security: {}
}

Response: {
  shape: document
}

Incoming-Request -> FastAPI-App
FastAPI-App -> Parameters: "Validates"
FastAPI-App -> Dependencies: "Resolves"
FastAPI-App.APIRouter.Path-Operation -> Response: "Returns"
```

Explore the different parts of the API to understand their specific functionalities and configuration options.

<x-cards data-columns="2">
  <x-card data-title="FastAPI Application" data-icon="lucide:box" data-href="/api-reference/fastapi-app">
    Comprehensive reference for the main `FastAPI` application class, its configuration options, and methods.
  </x-card>
  <x-card data-title="Routing" data-icon="lucide:milestone" data-href="/api-reference/routing">
    In-depth guide to `APIRouter` and `APIRoute` for structuring path operations and creating modular applications.
  </x-card>
  <x-card data-title="Parameters" data-icon="lucide:list-ordered" data-href="/api-reference/parameters">
    Detailed documentation for parameter-defining functions like `Path`, `Query`, `Header`, `Cookie`, `Body`, and `Form`.
  </x-card>
  <x-card data-title="Dependencies" data-icon="lucide:git-pull-request-arrow" data-href="/api-reference/dependencies">
    Reference for the dependency injection system, including `Depends` and `Security`.
  </x-card>
  <x-card data-title="Responses" data-icon="lucide:file-output" data-href="/api-reference/responses">
    A complete reference for all available response classes, including `JSONResponse`, `HTMLResponse`, and `StreamingResponse`.
  </x-card>
  <x-card data-title="Security Utilities" data-icon="lucide:shield" data-href="/api-reference/security">
    Reference for all security-related utilities, including OAuth2, HTTP Basic/Bearer/Digest, and API Keys.
  </x-card>
</x-cards>

This reference material is intended to be comprehensive. For practical, step-by-step guides on how to use these components together, please see the [User Guide](./user-guide.md).