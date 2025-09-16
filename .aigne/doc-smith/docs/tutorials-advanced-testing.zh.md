# 测试

为你的 API 编写测试对于确保其行为符合预期以及在应用程序增长时保持质量至关重要。FastAPI 通过提供一个基于 `httpx` 的 `TestClient`，使得测试变得简单。

本指南将引导你完成 FastAPI 应用程序的测试设置，包括如何在测试期间处理和覆盖依赖项。

## 一个简单的示例

让我们从一个基础应用程序及其在单个文件中的测试开始。你需要安装 `pytest` 和 `httpx`：

```bash
pip install pytest httpx
```

现在，你可以创建一个文件，例如 `test_app.py`，其中包含你的 FastAPI 应用程序和相应的测试。

```python title="test_app.py"
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}


client = TestClient(app)


def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```

以上代码的解释如下：

1.  **导入 `TestClient`**：我们从 `fastapi.testclient` 导入它。
2.  **创建一个 `TestClient` 实例**：我们将 FastAPI `app` 传递给它。
3.  **编写一个测试函数**：我们定义一个标准的 `pytest` 函数，例如 `test_read_main`。
4.  **发出请求**：我们使用 `client` 对象向我们的应用程序发出请求。它提供了与 HTTP 方法相对应的 `.get()`、`.post()`、`.put()` 等方法。
5.  **添加断言**：我们使用标准的 `assert` 语句来检查响应状态码和 JSON 主体。

要运行测试，请保存文件并在终端中执行 `pytest`。`pytest` 将自动发现并运行测试函数。

## 在实际应用程序中组织测试

在一个真实世界的项目中，你通常会将应用程序代码与测试代码分开。

假设你的应用程序有一个 `main.py` 文件：

```python title="main.py" icon=logos:python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}
```

你的测试将放在一个单独的文件中，例如 `test_main.py`：

```python title="test_main.py" icon=logos:python
from fastapi.testclient import TestClient

from .main import app

client = TestClient(app)


def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```

这种结构使你的项目保持井然有序。关键是将你的 `app` 实例导入到测试文件中，并将其传递给 `TestClient`。

## 使用覆盖测试依赖项

FastAPI 最强大的功能之一是其依赖注入系统。该系统也使得在测试期间覆盖依赖项变得容易，这对于隔离你想要测试的代码非常有用。

例如，你可以将连接到数据库的依赖项替换为一个返回固定数据的模拟对象。

考虑这个带有共享依赖项 `common_parameters` 的应用程序：

```python title="dependency_testing_app.py" icon=logos:python
from typing import Union

from fastapi import Depends, FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


async def common_parameters(
    q: Union[str, None] = None, skip: int = 0, limit: int = 100
):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return {"message": "Hello Items!", "params": commons}


@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return {"message": "Hello Users!", "params": commons}

# --- 测试部分 ---

client = TestClient(app)

# 定义依赖项覆盖
async def override_dependency(q: Union[str, None] = None):
    return {"q": q, "skip": 5, "limit": 10}

# 将覆盖应用于应用程序
app.dependency_overrides[common_parameters] = override_dependency

def test_override_in_items():
    response = client.get("/items/")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": None, "skip": 5, "limit": 10},
    }

def test_override_in_items_with_q():
    response = client.get("/items/?q=foo")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": "foo", "skip": 5, "limit": 10},
    }

def test_override_in_items_with_params():
    response = client.get("/items/?q=foo&skip=100&limit=200")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": "foo", "skip": 5, "limit": 10},
    }
```

在上面的示例中：

1.  我们定义了一个 `override_dependency` 函数。在测试期间，这个函数将替代 `common_parameters` 被使用。
2.  我们更新了 `app.dependency_overrides` 字典。这个字典告诉 FastAPI：“每当看到 `common_parameters` 时，就改用 `override_dependency`。”
3.  测试接着调用 `/items/` 端点。正如你在断言中看到的，无论请求 URL 中发送了什么查询参数，响应始终包含我们覆盖中的固定值（`"skip": 5`、`"limit": 10`）。

这项技术对于为依赖外部系统的组件创建隔离且可预测的测试至关重要。

现在你已经掌握了为 FastAPI 应用程序编写全面测试的工具。对于更复杂的项目，你可能想学习如何组织[更大型的应用程序](./tutorials-advanced-bigger-applications.md)。