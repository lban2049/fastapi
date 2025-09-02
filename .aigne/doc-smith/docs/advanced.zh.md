# 高级主题

掌握了使用 FastAPI 构建 API 的基础知识后，您就可以开始学习更高级的功能。本节涵盖了开发生产就绪应用程序所必需的主题，包括安全性、自定义请求处理、实时通信以及管理大型代码库的策略。

掌握这些概念将使您能够构建出不仅功能齐全，而且安全、可扩展且易于维护的应用程序。

下图概要展示了这些高级组件如何融入一个典型的 FastAPI 应用程序：

```d2
direction: down

Client: "Web 和移动客户端"

subsystem: {
  "中间件 (CORS、GZip、自定义)": {
    "安全性 (OAuth2、API 密钥)": {
      "API 路由器": {
        grid-columns: 2

        users: "/users"
        items: "/items"
        admin: "/admin"
      }
    }
  }
}

WebSocket: "WebSocket 端点"

Client -> subsystem: "HTTP 请求"
Client <-> WebSocket: "实时消息" {
  style.animated: true
}

subsystem.Security.API Routers -> "业务逻辑和数据库"
WebSocket -> "业务逻辑和数据库"

```

该图说明了请求如何通过中间件和安全层到达相应的路由器。它还展示了 WebSockets 如何为实时通信提供一个独立的直接通道。

深入了解以下主题以了解更多信息。

<x-cards data-columns="2">
  <x-card data-title="安全性" data-icon="lucide:lock" data-href="/advanced/security">
    学习如何通过身份验证和授权来保护您的 API。这包括实现常见的方案，如使用不记名令牌的 OAuth2、HTTP 基本身份验证和 API 密钥。
  </x-card>
  <x-card data-title="中间件" data-icon="lucide:layers" data-href="/advanced/middleware">
    了解如何拦截和处理每个请求和响应。中间件可用于添加自定义标头、记录请求、处理 CORS 等。
  </x-card>
  <x-card data-title="WebSockets" data-icon="lucide:arrow-right-left" data-href="/advanced/websockets">
    在客户端和服务器之间实现实时的双向通信。WebSockets 是聊天应用、实时通知和交互式仪表盘的理想选择。
  </x-card>
  <x-card data-title="大型应用" data-icon="lucide:git-fork" data-href="/advanced/bigger-applications">
    探索构建大型复杂应用程序的有效策略。学习如何使用 `APIRouter` 将您的 API 拆分为多个文件和模块，以保持代码库的整洁和可维护性。
  </x-card>
  <x-card data-title="测试" data-icon="lucide:beaker" data-href="/advanced/testing">
    编写测试对于维护可靠的应用程序至关重要。学习如何测试您的 FastAPI 端点、依赖项和事件处理程序，以确保您的代码按预期工作并保持安全。
  </x-card>
</x-cards>

---

通过学习这些指南，您将获得构建和部署复杂、专业级 API 所需的技能。在探索了这些主题之后，您可能会对详细的 [API 参考](./api-reference.md) 感兴趣，以便更深入地了解 FastAPI 的组件。