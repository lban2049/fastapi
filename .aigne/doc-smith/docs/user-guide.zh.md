# 用户指南

欢迎阅读 FastAPI 用户指南。本节旨在通过简单易懂的实用示例，带您逐步了解 FastAPI 的核心功能。我们将涵盖从处理传入的请求数据到管理依赖项的所有内容。

如果您尚未阅读，我们建议您从 [快速入门](./getting-started.md) 教程开始，设置您的第一个应用程序。

本指南探讨了典型的请求-响应生命周期，以及 FastAPI 的各项功能如何映射到每个阶段。

```d2
direction: down

Client-Request: {
  label: "客户端请求"
  shape: circle
}

FastAPI-Application: {
  label: "FastAPI 应用程序"
  shape: rectangle
  grid-columns: 1

  Parameter-Handling: {
    label: "参数处理"
    shape: rectangle
    grid-columns: 2

    Path-Parameters: {
      label: "路径参数\n/items/{item_id}"
    }
    Query-Parameters: {
      label: "查询参数\n/items/?skip=0"
    }
  }

  Data-Validation: {
    label: "数据校验"
    shape: rectangle
    Request-Body: "Pydantic 模型"
  }

  Shared-Logic: {
    label: "共享逻辑"
    shape: rectangle
    Dependency-Injection: "可重用组件"
  }
}

API-Response: {
  label: "API 响应"
  shape: circle
}

Client-Request -> FastAPI-Application.Parameter-Handling: "接收请求"
FastAPI-Application.Parameter-Handling -> FastAPI-Application.Data-Validation: "提取数据"
FastAPI-Application.Data-Validation -> FastAPI-Application.Shared-Logic: "运行依赖"
FastAPI-Application.Shared-Logic -> API-Response: "发送响应"
```

深入了解核心概念：

<x-cards data-columns="2">
  <x-card data-title="路径参数" data-icon="lucide:braces" data-href="/user-guide/path-parameters">
    学习如何在 API 端点中声明和校验路径参数，包括类型提示和数值校验。
  </x-card>
  <x-card data-title="查询参数" data-icon="lucide:help-circle" data-href="/user-guide/query-parameters">
    了解如何定义、校验和记录查询参数，包括字符串校验、默认值和别名。
  </x-card>
  <x-card data-title="请求体" data-icon="lucide:file-text" data-href="/user-guide/request-body">
    学习如何使用 Pydantic 模型接收和校验请求体中的数据，包括嵌套数据结构和多个请求体参数。
  </x-card>
  <x-card data-title="处理响应" data-icon="lucide:arrow-left-from-line" data-href="/user-guide/handling-responses">
    通过定义响应模型、更改状态码以及设置自定义响应头和 Cookie 来控制 API 响应。
  </x-card>
  <x-card data-title="依赖注入" data-icon="lucide:share-2" data-href="/user-guide/dependency-injection">
    掌握 FastAPI 强大的依赖注入系统，以管理依赖项、共享逻辑以及处理身份验证和数据库连接。
  </x-card>
</x-cards>

掌握这些核心概念后，您将能够游刃有余地构建健壮、高效的 API。

准备好学习更多内容了吗？请深入阅读我们的 [高级主题](./advanced.md) 部分，了解安全性、中间件、WebSocket 以及如何组织大型应用程序。