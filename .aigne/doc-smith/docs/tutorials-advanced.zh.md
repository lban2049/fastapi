# 高级用户指南

掌握 [教程 - 用户指南](./tutorials-first-steps.md) 中介绍的基础知识后，你就可以开始探索 FastAPI 更强大的功能了。本节将引导你学习有助于构建生产就绪应用程序、处理复杂用例以及维护大型代码库的主题。

在这里，你将学习如何集成 WebSockets 和 SQL 数据库等技术，构建可扩展的大型应用程序，添加自定义中间件以介入请求-响应周期，以及为你的 API 编写稳健的测试。

<x-cards data-columns="2">
  <x-card data-title="WebSockets" data-icon="lucide:arrow-right-left" data-href="/tutorials/advanced/websockets">
    学习处理客户端与服务器之间的实时双向交互通信，非常适合聊天应用和实时更新。
  </x-card>
  <x-card data-title="大型应用" data-icon="lucide:folder-tree" data-href="/tutorials/advanced/bigger-applications">
    探索随着项目规模增长而组织项目的策略，使用 APIRouter 将你的 API 拆分为多个模块，以提高可维护性。
  </x-card>
  <x-card data-title="中间件" data-icon="lucide:layers" data-href="/tutorials/advanced/middleware">
    了解如何添加中间件以全局处理请求和响应。实现自定义逻辑、添加请求头、记录请求等。
  </x-card>
  <x-card data-title="SQL (关系型) 数据库" data-icon="lucide:database" data-href="/tutorials/advanced/sql-databases">
    使用 SQLModel 和 SQLAlchemy 将你的应用连接到关系型数据库，包括会话管理和完整的 CRUD 操作。
  </x-card>
  <x-card data-title="测试" data-icon="lucide:beaker" data-href="/tutorials/advanced/testing">
    使用 TestClient 为你的 API 编写有效测试。学习如何测试端点、覆盖依赖项，并确保你的应用是稳健的。
  </x-card>
</x-cards>

探索完这些高级主题后，你将全面了解如何使用 FastAPI 构建、组织和测试复杂的高性能应用。如需了解所有可用类和函数的详细信息，可以继续阅读 [API 参考](./api-reference.md)。