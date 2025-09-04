# 概述

FastAPI 是一个现代、高性能的 Web 框架，用于基于标准 Python 类型提示使用 Python 构建 API。它旨在做到易于使用、编码快速，并为生产环境做好准备。

## 主要特性

FastAPI 专注于速度、简洁性和标准，提供了高效且愉悦的开发体验。

<x-cards data-columns="3">
  <x-card data-title="性能卓越" data-icon="lucide:rocket">
    得益于其 Starlette 和 Pydantic 基础，实现了与 NodeJS 和 Go 相媲美的性能。它是目前最快的 Python 框架之一。
  </x-card>
  <x-card data-title="编码快速" data-icon="lucide:zap">
    将功能开发速度提升 200% 至 300%。最大限度地减少代码重复，用更少的代码完成更多工作。
  </x-card>
  <x-card data-title="更少 Bug" data-icon="lucide:shield-check">
    将由开发人员导致的错误减少约 40%。类型提示和结构化数据验证能在 Bug 进入生产环境前将其捕获。
  </x-card>
  <x-card data-title="智能直观" data-icon="lucide:lightbulb">
    强大的编辑器支持，随处可用的自动补全功能。减少调试时间，将更多精力投入到构建工作中。
  </x-card>
  <x-card data-title="易于学习" data-icon="lucide:book-open">
    旨在做到简单易学、便于使用。文档清晰明了，让您可以专注于应用程序的逻辑。
  </x-card>
  <x-card data-title="基于标准" data-icon="lucide:file-json-2">
    完全兼容 OpenAPI（前身为 Swagger）和 JSON Schema 等 API 开放标准。
  </x-card>
</x-cards>

## 核心架构

FastAPI 的高性能和特性之所以成为可能，是因为它站在巨人的肩膀上：

-   **[Starlette](https://www.starlette.io/)**：用于处理所有 Web 相关部分，提供了一个轻量级、高性能的 ASGI 框架。
-   **[Pydantic](https://docs.pydantic.dev/)**：用于处理所有数据相关部分，基于 Python 类型提示提供强大的数据验证、序列化和文档功能。

这种关注点分离的设计使得 FastAPI 在 Web 处理和数据管理两方面都表现出色。

```d2
direction: down

"Your-API-Code": {
  label: "你的 API 代码"
  shape: rectangle
  style.fill: "#DDF0FF"
}

FastAPI: {
  shape: package
  "Your-API-Code"
}

Starlette: {
  shape: hexagon
  label: "Starlette (Web 层)"
  style.fill: "#D5E8D4"
}

Pydantic: {
  shape: hexagon
  label: "Pydantic (数据层)"
  style.fill: "#FAD7AC"
}

FastAPI -> Starlette: "用于所有 Web 部分"
FastAPI -> Pydantic: "用于所有数据部分"

```

## 代码示例

以下示例展示了创建一个功能齐全且带有自动文档的 API 是多么简单。

### 1. 创建文件

创建一个 `main.py` 文件，内容如下：

```python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Union[str, None] = None):
    return {"item_id": item_id, "q": q}
```

### 2. 运行服务器

在终端中执行以下命令：

```console
$ fastapi dev main.py

INFO:     Uvicorn running on http://127.0.0.1:8000 (按 CTRL+C 退出)
```

### 3. 查看交互式文档

FastAPI 会根据你的代码自动生成交互式 API 文档。只需在浏览器中打开 [`http://127.0.0.1:8000/docs`](http://127.0.0.1:8000/docs) 即可查看。

![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

## 深受行业领导者信赖

全球众多领先公司已在生产环境中使用 FastAPI。

> “近来我大量使用 **FastAPI**。[...] 我正计划将其用于我所在**微软**团队的所有**机器学习服务**。其中一些服务正被集成到核心 **Windows** 产品和部分 **Office** 产品中。”
> 
> **Kabir Khan - 微软**

> “我们采用 **FastAPI** 库来生成一个 **REST** 服务器，通过查询该服务器可以获得**预测**结果。[用于 Ludwig]”
> 
> **Piero Molino、Yaroslav Dudin 和 Sai Sumanth Miryala - Uber**

> “**Netflix** 很高兴地宣布，我们开源了我们的**危机管理**编排框架：**Dispatch**！[基于 **FastAPI** 构建]”
> 
> **Kevin Glisson、Marc Vilanova 和 Forest Monsen - Netflix**

## 后续步骤

本概述介绍了 FastAPI 的主要优点。要开始构建你的第一个 API，请参阅我们的分步教程。

<x-card data-title="开始使用" data-icon="lucide:play-circle" data-href="/getting-started" data-cta="开始教程">
  一份关于如何安装 FastAPI 并创建你的第一个应用程序的分步指南。
</x-card>