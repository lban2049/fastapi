# User Guide

Welcome to the FastAPI User Guide. This guide provides a hands-on, step-by-step exploration of the fundamental concepts you'll use to build robust and efficient APIs. Each section is designed to be a practical tutorial, complete with working code examples that you can build upon.

If you haven't set up your first application yet, we recommend starting with the [Getting Started](./getting-started.md) tutorial first.

## Core Concepts

The following tutorials cover the essential building blocks of any FastAPI application. They show you how to handle incoming data, process it, and control the response sent back to the client.

```d2
direction: right

Client: Browser or App

FastAPI: { 
  style: {
    stroke-width: 4
  }
  "Path & Query Params": "Extract from URL"
  "Request Body": "Parse incoming data"
  "Dependencies": "Handle shared logic (e.g., auth, DB)"
  "Your Logic": "Process request"
  "Response Handling": "Format output"
}

Client -> FastAPI."Path & Query Params": "1. Request (e.g., GET /items/5?q=foo)"
FastAPI."Path & Query Params" -> FastAPI."Request Body"
FastAPI."Request Body" -> FastAPI."Dependencies"
FastAPI."Dependencies" -> FastAPI."Your Logic"
FastAPI."Your Logic" -> FastAPI."Response Handling"
FastAPI."Response Handling" -> Client: "2. Response (e.g., JSON)"

```

Explore each topic to understand how these pieces fit together.

<x-cards data-columns="2">
  <x-card data-title="Path Parameters" data-href="/user-guide/path-parameters" data-icon="lucide:route">
    Learn how to declare and validate parameters embedded in the URL path, including type hints and numeric validations.
  </x-card>
  <x-card data-title="Query Parameters" data-href="/user-guide/query-parameters" data-icon="lucide:list-filter">
    Understand how to define, validate, and document parameters in the query string, including default values and aliases.
  </x-card>
  <x-card data-title="Request Body" data-href="/user-guide/request-body" data-icon="lucide:box-select">
    Learn how to receive and validate complex data structures from the request body using Pydantic models.
  </x-card>
  <x-card data-title="Handling Responses" data-href="/user-guide/handling-responses" data-icon="lucide:arrow-left-from-line">
    Control the API response by defining response models, changing status codes, and setting custom headers and cookies.
  </x-card>
  <x-card data-title="Dependency Injection" data-href="/user-guide/dependency-injection" data-icon="lucide:syringe">
    Master FastAPI's powerful dependency injection system to manage dependencies, share logic, and handle authentication or database connections.
  </x-card>
</x-cards>

## Next Steps

After mastering these core concepts, you will be well-equipped to build production-ready APIs with FastAPI.

When you're ready to explore more complex features, proceed to our [Advanced Topics](./advanced.md) section.