# Advanced User Guide

Once you have mastered the basics covered in the [Tutorials - User Guide](./tutorials-first-steps.md), you'll be ready to explore FastAPI's more powerful features. This section guides you through topics that help you build production-ready applications, handle complex use cases, and maintain large codebases.

Here you'll learn how to integrate technologies like WebSockets and SQL databases, structure larger applications for scalability, add custom middleware to hook into the request-response cycle, and write robust tests for your API.

<x-cards data-columns="2">
  <x-card data-title="WebSockets" data-icon="lucide:arrow-right-left" data-href="/tutorials/advanced/websockets">
    Learn to handle real-time, two-way interactive communication between the client and the server, perfect for chat applications and live updates.
  </x-card>
  <x-card data-title="Bigger Applications" data-icon="lucide:folder-tree" data-href="/tutorials/advanced/bigger-applications">
    Discover strategies for organizing your project as it grows, using APIRouter to split your API into multiple modules for better maintainability.
  </x-card>
  <x-card data-title="Middleware" data-icon="lucide:layers" data-href="/tutorials/advanced/middleware">
    Understand how to add middleware to process requests and responses globally. Implement custom logic, add headers, log requests, and more.
  </x-card>
  <x-card data-title="SQL (Relational) Databases" data-icon="lucide:database" data-href="/tutorials/advanced/sql-databases">
    Connect your application to relational databases using SQLModel and SQLAlchemy, including session management and full CRUD operations.
  </x-card>
  <x-card data-title="Testing" data-icon="lucide:beaker" data-href="/tutorials/advanced/testing">
    Write effective tests for your API using the TestClient. Learn how to test endpoints, override dependencies, and ensure your application is robust.
  </x-card>
</x-cards>

After exploring these advanced topics, you will have a comprehensive understanding of how to build, structure, and test complex, high-performance applications with FastAPI. For a detailed breakdown of all available classes and functions, you can proceed to the [API Reference](./api-reference.md).