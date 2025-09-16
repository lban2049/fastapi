# API 参考

欢迎来到 FastAPI API 参考。本节为 FastAPI 的所有类、函数和实用工具提供了详细、全面的文档。它专为需要快速查找框架组件具体细节的开发者而设计。

如果您是 FastAPI 的新手，建议先从 [教程 - 用户指南](./tutorials.md) 开始。

<x-cards data-columns="2">
  <x-card data-title="FastAPI 类" data-icon="lucide:layout-template" data-href="/api-reference/fastapi-class">
    主 `FastAPI` 应用程序类、其参数和配置选项的详细文档。
  </x-card>
  <x-card data-title="APIRouter" data-icon="lucide:git-fork" data-href="/api-reference/apirouter">
    `APIRouter` 的参考文档，用于将您的应用程序组织成多个模块和文件。
  </x-card>
  <x-card data-title="参数" data-icon="lucide:list-filter" data-href="/api-reference/parameters">
    关于 `Path`、`Query`、`Header`、`Cookie`、`Body`、`Form` 和 `File` 参数函数的深入参考。
  </x-card>
  <x-card data-title="依赖项" data-icon="lucide:syringe" data-href="/api-reference/dependencies">
    依赖注入系统的完整指南，包括 `Depends` 和 `Security`。
  </x-card>
  <x-card data-title="安全" data-icon="lucide:shield" data-href="/api-reference/security">
    所有安全实用工具的参考，包括 `OAuth2PasswordBearer`、`HTTPBasic`、`APIKeyHeader` 等。
  </x-card>
  <x-card data-title="响应" data-icon="lucide:file-output" data-href="/api-reference/responses">
    关于 `Response` 类的详细信息，如 `JSONResponse`、`HTMLResponse`、`StreamingResponse`，以及如何自定义它们。
  </x-card>
  <x-card data-title="异常" data-icon="lucide:alert-triangle" data-href="/api-reference/exceptions">
    关于 `HTTPException`、`WebSocketException` 和相关异常处理程序的文档。
  </x-card>
</x-cards>