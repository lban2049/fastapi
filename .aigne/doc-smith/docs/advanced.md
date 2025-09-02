# Advanced Topics

Once you have a handle on the fundamentals of building APIs with FastAPI, you can move on to more advanced features. This section covers topics that are essential for developing production-ready applications, including security, custom request processing, real-time communication, and strategies for managing large codebases.

Mastering these concepts will allow you to build applications that are not only functional but also secure, scalable, and easy to maintain.

Here's a high-level view of how these advanced components fit into a typical FastAPI application:

```d2
direction: down

Client: "Web & Mobile Clients"

subsystem: {
  "Middleware (CORS, GZip, Custom)": {
    "Security (OAuth2, API Keys)": {
      "API Routers": {
        grid-columns: 2

        users: "/users"
        items: "/items"
        admin: "/admin"
      }
    }
  }
}

WebSocket: "WebSocket Endpoint"

Client -> subsystem: "HTTP Requests"
Client <-> WebSocket: "Real-time Messages" {
  style.animated: true
}

subsystem.Security.API Routers -> "Business Logic & Database"
WebSocket -> "Business Logic & Database"

```

This diagram illustrates the journey of a request through middleware and security layers to the appropriate router. It also shows how WebSockets provide a separate, direct channel for real-time communication.

Dive into the topics below to learn more.

<x-cards data-columns="2">
  <x-card data-title="Security" data-icon="lucide:lock" data-href="/advanced/security">
    Learn to secure your API with authentication and authorization. This includes implementing common schemes like OAuth2 with bearer tokens, HTTP Basic Auth, and API keys.
  </x-card>
  <x-card data-title="Middleware" data-icon="lucide:layers" data-href="/advanced/middleware">
    Discover how to intercept and process every request and response. Middleware is useful for adding custom headers, logging requests, handling CORS, and more.
  </x-card>
  <x-card data-title="WebSockets" data-icon="lucide:arrow-right-left" data-href="/advanced/websockets">
    Enable real-time, two-way communication between the client and server. WebSockets are ideal for chat applications, live notifications, and interactive dashboards.
  </x-card>
  <x-card data-title="Bigger Applications" data-icon="lucide:git-fork" data-href="/advanced/bigger-applications">
    Explore effective strategies for structuring large, complex applications. Learn how to split your API into multiple files and modules using `APIRouter` to keep your codebase clean and maintainable.
  </x-card>
  <x-card data-title="Testing" data-icon="lucide:beaker" data-href="/advanced/testing">
    Writing tests is crucial for maintaining a reliable application. Learn how to test your FastAPI endpoints, dependencies, and event handlers to ensure your code works as expected and remains secure.
  </x-card>
</x-cards>

---

By working through these guides, you'll gain the skills needed to build and deploy sophisticated, professional-grade APIs. After exploring these topics, you might be interested in the detailed [API Reference](./api-reference.md) for a deeper look at FastAPI's components.