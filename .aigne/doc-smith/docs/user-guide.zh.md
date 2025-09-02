# 用户指南

欢迎来到 FastAPI 用户指南。本指南将通过动手实践的方式，带你逐步探索构建稳健、高效 API 所需的基本概念。每个章节都设计成一个实用的教程，并附有可供你参考构建的工作代码示例。

如果你还没有设置好你的第一个应用程序，我们建议先从 [快速入门](./getting-started.md) 教程开始。

## 核心概念

以下教程涵盖了所有 FastAPI 应用程序的基本构建模块，讲解了如何处理传入数据、进行处理，以及控制返回给客户端的响应。

```d2
direction: right

Client: 浏览器或应用

FastAPI: { 
  style: {
    stroke-width: 4
  }
  "路径和查询参数": "从 URL 中提取"
  "请求体": "解析传入数据"
  "依赖项": "处理共享逻辑（例如，身份验证、数据库）"
  "你的逻辑": "处理请求"
  "响应处理": "格式化输出"
}

Client -> FastAPI."路径和查询参数": "1. 请求（例如，GET /items/5?q=foo）"
FastAPI."路径和查询参数" -> FastAPI."请求体"
FastAPI."请求体" -> FastAPI."依赖项"
FastAPI."依赖项" -> FastAPI."你的逻辑"
FastAPI."你的逻辑" -> FastAPI."响应处理"
FastAPI."响应处理" -> Client: "2. 响应（例如，JSON）"

```

探索每个主题，了解这些部分是如何协同工作的。

<x-cards data-columns="2">
  <x-card data-title="路径参数" data-href="/user-guide/path-parameters" data-icon="lucide:route">
    学习如何声明和验证嵌入在 URL 路径中的参数，包括类型提示和数值验证。
  </x-card>
  <x-card data-title="查询参数" data-href="/user-guide/query-parameters" data-icon="lucide:list-filter">
    了解如何定义、验证和记录查询字符串中的参数，包括默认值和别名。
  </x-card>
  <x-card data-title="请求体" data-href="/user-guide/request-body" data-icon="lucide:box-select">
    学习如何使用 Pydantic 模型接收和验证来自请求体的复杂数据结构。
  </x-card>
  <x-card data-title="处理响应" data-href="/user-guide/handling-responses" data-icon="lucide:arrow-left-from-line">
    通过定义响应模型、更改状态码以及设置自定义标头和 Cookie 来控制 API 响应。
  </x-card>
  <x-card data-title="依赖注入" data-href="/user-guide/dependency-injection" data-icon="lucide:syringe">
    掌握 FastAPI 强大的依赖注入系统，以管理依赖项、共享逻辑以及处理身份验证或数据库连接。
  </x-card>
</x-cards>

## 后续步骤

掌握这些核心概念后，你将完全有能力使用 FastAPI 构建可用于生产环境的 API。

当你准备好探索更复杂的功能时，请继续阅读我们的 [高级主题](./advanced.md) 部分。