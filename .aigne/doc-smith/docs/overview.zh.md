# 概述

FastAPI 是一个用于通过 Python 构建 API 的现代化、高性能 Web 框架，它基于标准的 Python 类型提示。其设计旨在易于使用、编码快速，并适用于生产环境。

## 主要特性

FastAPI 专注于速度、简洁性和标准化，提供了高效且愉悦的开发体验。

<x-cards data-columns="3">
  <x-card data-title="性能卓越" data-icon="lucide:rocket">
    得益于其 Starlette 和 Pydantic 基础，其性能可与 NodeJS 和 Go 相媲美。它是现有最快的 Python 框架之一。
  </x-card>
  <x-card data-title="编码快速" data-icon="lucide:zap">
    将功能开发速度提升 200% 到 300%。最大限度地减少代码重复，用更少的代码完成更多的工作。
  </x-card>
  <x-card data-title="更少错误" data-icon="lucide:shield-check">
    将开发人员导致的错误减少约 40%。类型提示和结构化数据验证可在错误进入生产环境前将其捕获。
  </x-card>
  <x-card data-title="直观易用" data-icon="lucide:lightbulb">
    得益于出色的编辑器支持和无处不在的自动补全功能。花更少的时间调试，更多的时间用于构建。
  </x-card>
  <x-card data-title="易于学习" data-icon="lucide:book-open">
    其设计旨在易于学习和使用。文档清晰明了，让您可以专注于应用程序的逻辑。
  </x-card>
  <x-card data-title="基于标准" data-icon="lucide:file-json-2">
    完全兼容 API 的开放标准，包括 OpenAPI（前身为 Swagger）和 JSON Schema。
  </x-card>
</x-cards>

## 核心架构

FastAPI 的卓越性能和丰富功能，得益于以下这些强大的基础：

-   **[Starlette](https://www.starlette.io/)**: 用于处理所有 Web 相关部分，提供了一个轻量级、高性能的 ASGI 框架。
-   **[Pydantic](https://docs.pydantic.dev/)**: 用于处理所有数据相关部分，基于 Python 类型提示提供强大的数据验证、序列化和文档功能。

这种关注点分离的设计使得 FastAPI 在 Web 处理和数据管理两方面都表现出色。

```d2
direction: down

"您的 API 代码": {
  shape: rectangle
  style.fill: "#DDF0FF"
}

"FastAPI": {
  shape: package
  "您的 API 代码"
}

"Starlette": {
  shape: hexagon
  label: "Starlette (Web 层)"
  style.fill: "#D5E8D4"
}

"Pydantic": {
  shape: hexagon
  label: "Pydantic (数据层)"
  style.fill: "#FAD7AC"
}

"FastAPI" -> "Starlette": "用于所有 Web 部分"
"FastAPI" -> "Pydantic": "用于所有数据部分"

```

## 代码一览

了解创建一个功能齐全且带有自动文档的 API 是多么简单。

### 1. 创建文件

创建一个名为 `main.py` 的文件，内容如下：

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

INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

### 3. 查看交互式文档

FastAPI 会根据您的代码自动生成交互式 API 文档。只需在浏览器中打开 [`http://127.0.0.1:8000/docs`](http://127.0.0.1:8000/docs) 即可查看实际效果。

![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

## 深受行业领导者信赖

全球众多领先公司已在生产环境中使用 FastAPI。

> “ [...] 我最近在大量使用 **FastAPI**。[...] 我实际上正计划将其用于我团队在 **微软** 的所有 **机器学习服务**。其中一些服务正在被集成到核心的 **Windows** 产品和一些 **Office** 产品中。”
> 
> **Kabir Khan - 微软**

> “我们采用了 **FastAPI** 库来生成一个 **REST** 服务器，该服务器可被查询以获取 **预测结果**。[针对 Ludwig]”
> 
> **Piero Molino, Yaroslav Dudin, and Sai Sumanth Miryala - Uber**

> “**Netflix** 很高兴地宣布，我们开源了我们的 **危机管理** 编排框架：**Dispatch**！[使用 **FastAPI** 构建]”
> 
> **Kevin Glisson, Marc Vilanova, Forest Monsen - Netflix**

## 后续步骤

本概述介绍了 FastAPI 的主要优点。要开始构建您的第一个 API，请参阅我们的分步教程。

<x-card data-title="入门指南" data-icon="lucide:play-circle" data-href="/getting-started" data-cta="开始教程">
  一份关于如何安装 FastAPI 并创建您的第一个应用程序的分步指南。
</x-card>