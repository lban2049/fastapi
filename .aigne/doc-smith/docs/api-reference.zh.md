# API 参考

本节为 FastAPI 的公共 API 提供了详细的技术文档。它专为需要查找特定类、函数和参数的开发者设计。与基于教程的[用户指南](./user-guide.md)不同，本参考文档的结构化设计旨在帮助用户快速获取全面的信息。

下方是 FastAPI 主要组件交互方式的概览。应用程序实例是核心，负责管理路由器和路径操作。每个路径操作通过验证参数、解析依赖项和生成响应来处理传入的请求。

```d2
direction: down

Incoming-Request: {
  label: "传入请求"
  shape: circle
}

FastAPI-App: {
  label: "FastAPI 应用"
  shape: rectangle
  style.fill: "#e6f7ff"

  APIRouter: {
    label: "APIRouter"
    shape: package
    style.fill: "#f6ffed"

    Path-Operation: {
      label: "路径操作\n(@app.get 等)"
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
FastAPI-App -> Parameters: "验证"
FastAPI-App -> Dependencies: "解析"
FastAPI-App.APIRouter.Path-Operation -> Response: "返回"
```

探索 API 的不同部分，以了解它们的具体功能和配置选项。

<x-cards data-columns="2">
  <x-card data-title="FastAPI 应用" data-icon="lucide:box" data-href="/api-reference/fastapi-app">
    关于主 `FastAPI` 应用类、其配置选项和方法的全面参考。
  </x-card>
  <x-card data-title="路由" data-icon="lucide:milestone" data-href="/api-reference/routing">
    关于使用 `APIRouter` 和 `APIRoute` 构建路径操作和创建模块化应用的深入指南。
  </x-card>
  <x-card data-title="参数" data-icon="lucide:list-ordered" data-href="/api-reference/parameters">
    关于参数定义函数（如 `Path`、`Query`、`Header`、`Cookie`、`Body` 和 `Form`）的详细文档。
  </x-card>
  <x-card data-title="依赖项" data-icon="lucide:git-pull-request-arrow" data-href="/api-reference/dependencies">
    关于依赖注入系统的参考，包括 `Depends` 和 `Security`。
  </x-card>
  <x-card data-title="响应" data-icon="lucide:file-output" data-href="/api-reference/responses">
    所有可用响应类的完整参考，包括 `JSONResponse`、`HTMLResponse` 和 `StreamingResponse`。
  </x-card>
  <x-card data-title="安全工具" data-icon="lucide:shield" data-href="/api-reference/security">
    所有安全相关工具的参考，包括 OAuth2、HTTP Basic/Bearer/Digest 和 API 密钥。
  </x-card>
</x-cards>

本参考资料旨在提供全面的信息。如需了解如何结合使用这些组件的实用分步指南，请参阅[用户指南](./user-guide.md)。