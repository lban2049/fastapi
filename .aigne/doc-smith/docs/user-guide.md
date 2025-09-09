# User Guide

Welcome to the FastAPI User Guide. This section is designed to walk you through the core features of FastAPI with practical, easy-to-follow examples. We'll cover everything from handling incoming request data to managing dependencies.

If you haven't already, we recommend starting with the [Getting Started](./getting-started.md) tutorial to set up your first application.

This guide explores the typical request-response lifecycle and how FastAPI's features map to each stage.

```d2
direction: down

Client-Request: {
  label: "Client Request"
  shape: circle
}

FastAPI-Application: {
  label: "FastAPI Application"
  shape: rectangle
  grid-columns: 1

  Parameter-Handling: {
    label: "Parameter Handling"
    shape: rectangle
    grid-columns: 2

    Path-Parameters: {
      label: "Path Parameters\n/items/{item_id}"
    }
    Query-Parameters: {
      label: "Query Parameters\n/items/?skip=0"
    }
  }

  Data-Validation: {
    label: "Data Validation"
    shape: rectangle
    Request-Body: "Pydantic Models"
  }

  Shared-Logic: {
    label: "Shared Logic"
    shape: rectangle
    Dependency-Injection: "Reusable Components"
  }
}

API-Response: {
  label: "API Response"
  shape: circle
}

Client-Request -> FastAPI-Application.Parameter-Handling: "Receives Request"
FastAPI-Application.Parameter-Handling -> FastAPI-Application.Data-Validation: "Extracts Data"
FastAPI-Application.Data-Validation -> FastAPI-Application.Shared-Logic: "Runs Dependencies"
FastAPI-Application.Shared-Logic -> API-Response: "Sends Response"
```

Explore the core concepts in detail:

<x-cards data-columns="2">
  <x-card data-title="Path Parameters" data-icon="lucide:braces" data-href="/user-guide/path-parameters">
    Learn how to declare and validate path parameters in your API endpoints, including type hints and numeric validations.
  </x-card>
  <x-card data-title="Query Parameters" data-icon="lucide:help-circle" data-href="/user-guide/query-parameters">
    Understand how to define, validate, and document query parameters, including string validations, default values, and aliases.
  </x-card>
  <x-card data-title="Request Body" data-icon="lucide:file-text" data-href="/user-guide/request-body">
    Learn how to receive and validate data from the request body using Pydantic models, including nested data structures and multiple body parameters.
  </x-card>
  <x-card data-title="Handling Responses" data-icon="lucide:arrow-left-from-line" data-href="/user-guide/handling-responses">
    Control the API response by defining response models, changing status codes, and setting custom headers and cookies.
  </x-card>
  <x-card data-title="Dependency Injection" data-icon="lucide:share-2" data-href="/user-guide/dependency-injection">
    Master FastAPI's powerful dependency injection system to manage dependencies, share logic, and handle authentication and database connections.
  </x-card>
</x-cards>

After mastering these core concepts, you'll be well-equipped to build robust and efficient APIs.

Ready for more? Dive into our [Advanced Topics](./advanced.md) section to learn about security, middleware, WebSockets, and structuring larger applications.