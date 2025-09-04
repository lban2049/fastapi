# 用户指南

欢迎阅读 FastAPI 用户指南。本节旨在通过实用且易于理解的示例，引导您逐步了解 FastAPI 的核心功能。我们将涵盖从处理传入的请求数据到管理依赖项的所有内容。

如果您还没有这样做，我们建议您从 [快速入门](./getting-started.md) 教程开始，设置您的第一个应用程序。

本指南探讨了典型的请求-响应生命周期，以及 FastAPI 的各项功能如何对应于生命周期的每个阶段。

```d2
direction: down

"客户端请求" {
  shape: circle
}

"FastAPI 应用" {
  shape: rectangle
  grid-columns: 1

  "参数处理" {
    shape: package
    grid-columns: 2

    "路径参数": {
      label: "路径参数\n/items/{item_id}"
    }
    "查询参数": {
      label: "查询参数\n/items/?skip=0"
    }
  }

  "数据验证" {
    shape: package
    "请求体": "Pydantic 模型"
  }

  "共享逻辑" {
    shape: package
    "依赖注入": "可复用组件"
  }
}

"API 响应" {
  shape: circle
}

"客户端请求" -> "FastAPI 应用"."参数处理": "接收请求"
"FastAPI 应用"."参数处理" -> "FastAPI 应用"."数据验证": "提取数据"
"FastAPI 应用"."数据验证" -> "FastAPI 应用"."共享逻辑": "运行依赖"
"FastAPI 应用"."共享逻辑" -> "API 响应": "发送响应"
```

详细了解以下核心概念：

<x-cards data-columns="2">
  <x-card data-title="路径参数" data-icon="lucide:braces" data-href="/user-guide/path-parameters">
    学习如何在 API 端点中声明和验证路径参数，包括类型提示和数值验证。
  </x-card>
  <x-card data-title="查询参数" data-icon="lucide:help-circle" data-href="/user-guide/query-parameters">
    了解如何定义、验证和记录查询参数，包括字符串验证、默认值和别名。
  </x-card>
  <x-card data-title="请求体" data-icon="lucide:file-text" data-href="/user-guide/request-body">
    学习如何使用 Pydantic 模型从请求体中接收和验证数据，包括嵌套数据结构和多个请求体参数。
  </x-card>
  <x-card data-title="处理响应" data-icon="lucide:arrow-left-from-line" data-href="/user-guide/handling-responses">
    通过定义响应模型、更改状态码以及设置自定义标头和 Cookie 来控制 API 响应。
  </x-card>
  <x-card data-title="依赖注入" data-icon="lucide:share-2" data-href="/user-guide/dependency-injection">
    掌握 FastAPI 强大的依赖注入系统，以管理依赖、共享逻辑以及处理身份验证和数据库连接。
  </x-card>
</x-cards>

掌握这些核心概念后，您将能很好地构建稳健且高效的 API。

准备好学习更多内容了吗？请深入阅读我们的 [高级主题](./advanced.md) 部分，了解有关安全性、中间件、WebSockets 以及构建更大型应用程序的内容。