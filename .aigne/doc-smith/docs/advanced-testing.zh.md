# 测试

FastAPI 提供了一种使用 `TestClient` 来测试应用的直接方法，`TestClient` 基于强大的 `httpx` 库构建。这使你无需启动真实服务器即可对应用运行测试，从而使测试快速而可靠。

## 使用 `TestClient` 进行基本测试

首先，导入 `TestClient`，并通过传入你的 FastAPI 应用来创建其实例。然后，你可以在测试函数中使用此客户端实例向你的应用发出请求。

以下是测试一个简单端点的完整示例：

```python title="tutorial001.py" icon=logos:python
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

在此测试中：

1.  我们从 `fastapi.testclient` 导入 `TestClient`。
2.  我们为 `app` 创建一个 `client` 实例。
3.  在 `test_read_main` 函数内部，`client.get("/")` 向应用的根路径发出请求。
4.  然后我们使用 `assert` 语句来验证 HTTP 状态码是否为 `200` (OK)，以及 JSON 响应体是否与预期输出匹配。

## 测试 WebSocket

你也可以使用 `TestClient` 上的 `websocket_connect()` 方法来测试 WebSocket 端点。建议将其用作上下文管理器（使用 `with` 语句），以确保连接被正确建立和关闭。

```python title="tutorial002.py" icon=logos:python
from fastapi import FastAPI
from fastapi.testclient import TestClient
from fastapi.websockets import WebSocket

app = FastAPI()


@app.websocket("/ws")
async def websocket(websocket: WebSocket):
    await websocket.accept()
    await websocket.send_json({"msg": "Hello WebSocket"})
    await websocket.close()


client = TestClient(app)


def test_websocket():
    with client.websocket_connect("/ws") as websocket:
        data = websocket.receive_json()
        assert data == {"msg": "Hello WebSocket"}
```

在此示例中，`client.websocket_connect("/ws")` 建立连接。在 `with` 块内，`websocket.receive_json()` 等待并解析来自服务器的 JSON 消息，然后我们对其进行断言。

## 使用事件处理器进行测试

如果你的应用使用 `startup` 或 `shutdown` 事件处理器，你应该将 `TestClient` 用作上下文管理器。这能确保在 `with` 块内的测试运行前后，事件处理器能被正确执行。

考虑一个在启动时初始化部分数据的应用：

```python title="tutorial003.py" icon=logos:python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()

items = {}


@app.on_event("startup")
async def startup_event():
    items["foo"] = {"name": "Fighters"}
    items["bar"] = {"name": "Tenders"}


@app.get("/items/{item_id}")
async def read_items(item_id: str):
    return items[item_id]


def test_read_items():
    with TestClient(app) as client:
        response = client.get("/items/foo")
        assert response.status_code == 200
        assert response.json() == {"name": "Fighters"}
```

通过使用 `with TestClient(app) as client:`，可以保证 `startup_event` 在任何客户端请求发出前运行，从而确保 `items` 字典已为测试填充好数据。

## 使用覆盖测试依赖项

测试中最强大的功能之一是能够覆盖依赖项。这使你能够用模拟版本替换测试中的依赖项，例如，避免进行真实的数据库或网络调用，并获得更可预测的测试结果。

你可以通过更新 `app.dependency_overrides` 字典来覆盖依赖项。字典的键是原始依赖函数，值是你想使用的新函数。

```python title="dependency_testing/tutorial001.py" icon=logos:python
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


client = TestClient(app)


async def override_dependency(q: Union[str, None] = None):
    return {"q": q, "skip": 5, "limit": 10}


app.dependency_overrides[common_parameters] = override_dependency


def test_override_in_items():
    response = client.get("/items/")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": None, "skip": 5, "limit": 10},
    }


def test_override_in_items_with_params():
    response = client.get("/items/?q=foo&skip=100&limit=200")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": "foo", "skip": 5, "limit": 10},
    }
```

在此示例中：

- 我们定义了一个 `override_dependency` 函数，其中包含固定的 `skip` 和 `limit` 值。
- 我们用我们的覆盖函数替换了原始的 `common_parameters` 依赖：`app.dependency_overrides[common_parameters] = override_dependency`。
- 测试 `test_override_in_items_with_params` 表明，即使 `skip` 和 `limit` 作为查询参数提供，也会使用被覆盖的依赖项中的值。`q` 参数仍然被处理，因为它是覆盖函数签名的一部分。