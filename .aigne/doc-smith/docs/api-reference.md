# API Reference

This section provides detailed technical documentation of FastAPI's public API, including its classes, functions, and utilities. It is designed for developers seeking quick access to specific technical details and assumes a basic understanding of FastAPI's core concepts. For a step-by-step introduction, please see the [User Guide](./user-guide.md).

The API reference is organized by functionality to help you find the information you need efficiently.

<x-cards data-columns="2">
  <x-card data-title="FastAPI Application" data-icon="lucide:box" data-href="/api-reference/fastapi-app">
    Comprehensive reference for the main `FastAPI` application class, its configuration options, and methods.
  </x-card>
  <x-card data-title="Routing" data-icon="lucide:network" data-href="/api-reference/routing">
    In-depth guide to `APIRouter` and `APIRoute` for structuring path operations and creating modular applications.
  </x-card>
  <x-card data-title="Parameters" data-icon="lucide:sliders-horizontal" data-href="/api-reference/parameters">
    Detailed documentation for parameter-defining functions like `Path`, `Query`, `Header`, `Cookie`, `Body`, and `Form`.
  </x-card>
  <x-card data-title="Dependencies" data-icon="lucide:git-pull-request-draft" data-href="/api-reference/dependencies">
    Reference for the dependency injection system, including `Depends` and `Security`.
  </x-card>
  <x-card data-title="Responses" data-icon="lucide:file-output" data-href="/api-reference/responses">
    A complete reference for all available response classes, including `JSONResponse`, `HTMLResponse`, and `StreamingResponse`.
  </x-card>
  <x-card data-title="Security Utilities" data-icon="lucide:shield" data-href="/api-reference/security">
    Reference for all security-related utilities, including OAuth2, HTTP Basic/Bearer/Digest, and API Keys.
  </x-card>
</x-cards>