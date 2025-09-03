# API 参考

本节为 FastAPI 的公共 API 提供了详细的技术文档。它专为需要查找特定类、函数和参数的开发者设计。与基于教程的用户指南不同，本参考资料的结构便于快速访问全面的信息。

以下是 FastAPI 主要组件交互方式的高级概述。应用程序实例是核心，负责管理路由器和路径操作。每个路径操作通过验证参数、解析依赖项和生成响应来处理传入的请求。

```d2
direction: down

"FastAPI 应用": {
  shape: rectangle
  style.fill: "#e6f7ff"

  "APIRouter": {
    shape: package
    style.fill: "#f6ffed"

    "路径操作 (@app.get 等)": {
      shape: rectangle
      style.fill: "#fffbe6"
    }
  }
}

"传入请求": {
  shape: circle
}

"参数": {
  shape: package
  grid-columns: 2
  "路径": {}
  "查询": {}
  "正文": {}
  "标头": {}
  "Cookie": {}
  "表单": {}
}

"依赖项": {
  shape: package
  grid-columns: 2
  "Depends": {}
  "安全": {}
}

"响应": {
  shape: document
}

"传入请求" -> "FastAPI 应用"
"FastAPI 应用" -> "参数": "验证"
"FastAPI 应用" -> "依赖项": "解析"
"路径操作 (@app.get 等)" -> "响应": "返回"

```

探索 API 的不同部分，以了解其具体功能和配置选项。

<x-cards data-columns="2">
  <x-card data-title="FastAPI 应用" data-icon="lucide:box" data-href="/api-reference/fastapi-app">
    关于 `FastAPI` 主类、其配置参数以及应用程序管理方法的参考。
  </x-card>
  <x-card data-title="路由" data-icon="lucide:milestone" data-href="/api-reference/routing">
    关于 `APIRouter` 和 `APIRoute` 的深入指南，用于构建路径操作和创建模块化应用。
  </x-card>
  <x-card data-title="参数" data-icon="lucide:list-ordered" data-href="/api-reference/parameters">
    关于参数定义函数（如 `Path`、`Query`、`Header`、`Cookie`、`Body` 和 `Form`）的详细文档。
  </x-card>
  <x-card data-title="依赖项" data-icon="lucide:git-pull-request-arrow" data-href="/api-reference/dependencies">
    关于依赖注入系统的参考，包括 `Depends` 和 `Security` 函数。
  </x-card>
  <x-card data-title="响应" data-icon="lucide:file-output" data-href="/api-reference/responses">
    所有可用响应类的完整参考，例如 `JSONResponse`、`HTMLResponse` 和 `StreamingResponse`。
  </x-card>
  <x-card data-title="安全工具" data-icon="lucide:shield" data-href="/api-reference/security">
    所有安全相关工具的参考，包括 OAuth2、HTTP Basic/Bearer 和 API 密钥的辅助函数。
  </x-card>
</x-cards>

本参考资料旨在做到全面详尽。如需了解如何结合使用这些组件的实用分步指南，请参阅[用户指南](./user-guide.md)。
