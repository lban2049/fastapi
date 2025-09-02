# API 参考

本节提供 FastAPI 公共 API 的详细技术文档，包括其类、函数和实用工具。它专为寻求快速访问特定技术细节的开发者而设计，并假定读者对 FastAPI 的核心概念有基本了解。如需分步入门介绍，请参阅[用户指南](./user-guide.md)。

API 参考按功能进行组织，以便您高效地查找所需信息。

<x-cards data-columns="2">
  <x-card data-title="FastAPI Application" data-icon="lucide:box" data-href="/api-reference/fastapi-app">
    关于主 `FastAPI` 应用类、其配置选项及方法的全面参考。
  </x-card>
  <x-card data-title="Routing" data-icon="lucide:network" data-href="/api-reference/routing">
    `APIRouter` 和 `APIRoute` 的深度指南，用于结构化路径操作和创建模块化应用。
  </x-card>
  <x-card data-title="Parameters" data-icon="lucide:sliders-horizontal" data-href="/api-reference/parameters">
    关于 `Path`、`Query`、`Header`、`Cookie`、`Body` 和 `Form` 等参数定义函数的详细文档。
  </x-card>
  <x-card data-title="Dependencies" data-icon="lucide:git-pull-request-draft" data-href="/api-reference/dependencies">
    关于依赖注入系统（包括 `Depends` 和 `Security`）的参考。
  </x-card>
  <x-card data-title="Responses" data-icon="lucide:file-output" data-href="/api-reference/responses">
    所有可用响应类的完整参考，包括 `JSONResponse`、`HTMLResponse` 和 `StreamingResponse`。
  </x-card>
  <x-card data-title="Security Utilities" data-icon="lucide:shield" data-href="/api-reference/security">
    所有安全相关实用工具的参考，包括 OAuth2、HTTP Basic/Bearer/Digest 和 API 密钥。
  </x-card>
</x-cards>