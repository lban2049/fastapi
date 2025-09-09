# API 参考

本节提供了 FastAPI 公共 API 的详细技术文档。它专为需要查找特定类、函数和参数的开发人员设计。与基于教程的[用户指南](./user-guide.md)不同，本参考文档的结构旨在帮助用户快速获取全面的信息。

下面是 FastAPI 主要组件如何交互的高级概述。应用程序实例是核心，管理着路由器和路径操作。每个路径操作通过验证参数、解析依赖项和生成响应来处理传入的请求。

```d2
direction: down

Incoming-Request: {
  label: "传入请求"
  shape: circle
}

FastAPI-App: {
  label: "FastAPI 应用"
  shape: rectangle

  APIRouter: {
    label: "APIRouter"
    shape: rectangle

    Path-Operation: {
      label: "路径操作\n(@app.get 等)"
      shape: rectangle
    }
  }
}

Parameters: {
  shape: rectangle
  grid-columns: 2
  Path: {}
  Query: {}
  Body: {}
  Header: {}
  Cookie: {}
  Form: {}
}

Dependencies: {
  shape: rectangle
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
  <x-card data-title="FastAPI 应用程序" data-icon="lucide:box" data-href="/api-reference/fastapi-app">
    主 `FastAPI` 应用程序类的全面参考，包括其配置选项和方法。
  </x-card>
  <x-card data-title="路由" data-icon="lucide:milestone" data-href="/api-reference/routing">
    关于 `APIRouter` 和 `APIRoute` 的深度指南，用于构建路径操作和创建模块化应用程序。
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
  <x-card data-title="安全实用工具" data-icon="lucide:shield" data-href="/api-reference/security">
    所有安全相关实用工具的参考，包括 OAuth2、HTTP Basic/Bearer/Digest 和 API 密钥。
  </x-card>
</x-cards>

本参考材料旨在提供全面的信息。有关如何结合使用这些组件的实用分步指南，请参阅[用户指南](./user-guide.md)。