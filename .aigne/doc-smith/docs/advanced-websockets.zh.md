# WebSockets

FastAPI 完全支持 WebSocket，可实现客户端与服务器之间的实时双向通信。这对于聊天服务、实时通知和协作编辑工具等应用非常有用。

FastAPI 的 WebSocket 功能底层由 Starlette 提供支持。

WebSocket 连接的基本流程如下：

```d2
direction: down

Client: { 
  shape: c4-person 
}

Server: { 
  shape: rectangle 
}

Client -> Server: "1. 带有 'Upgrade: websocket' 头的 HTTP GET 请求"
Server -> Client: "2. HTTP 101 切换协议响应"
Client <-> Server: "3. 持久双向通信通道" {
  style {
    stroke-dash: 4
  }
}
```

## 第一步

让我们从一个简单的示例开始：服务器将回显从客户端接收到的任何消息。

首先，你需要一个 WebSocket 端点，可以使用 `@app.websocket()` 装饰器来创建。

以下是一个完整的应用程序：

```python title="tutorial001.py"
from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse

app = FastAPI()

html = """
<!DOCTYPE html>
<html>
    <head>
        <title>聊天</title>
    </head>
    <body>
        <h1>WebSocket 聊天</h1>
        <form action="" onsubmit="sendMessage(event)">
            <input type="text" id="messageText" autocomplete="off"/>
            <button>发送</button>
        </form>
        <ul id='messages'>
        </ul>
        <script>
            var ws = new WebSocket("ws://localhost:8000/ws");
            ws.onmessage = function(event) {
                var messages = document.getElementById('messages')
                var message = document.createElement('li')
                var content = document.createTextNode(event.data)
                message.appendChild(content)
                messages.appendChild(message)
            };
            function sendMessage(event) {
                var input = document.getElementById("messageText")
                ws.send(input.value)
                input.value = ''
                event.preventDefault()
            }
        </script>
    </body>
</html>
"""


@app.get("/")
async def get():
    return HTMLResponse(html)


@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"消息文本是：{data}")

```

### 代码分解

1.  **`HTML 前端`**：在根路径 `/` 提供一个简单的网页服务。其中包含的 JavaScript 会建立到 `ws://localhost:8000/ws` 的 WebSocket 连接。
2.  **`@app.websocket("/ws")`**：此装饰器用于声明一个 WebSocket 端点。
3.  **`websocket: WebSocket`**：该函数接收一个 `WebSocket` 对象作为参数。
4.  **`await websocket.accept()`**：这很关键。你必须先 `accept` 连接，然后才能发送或接收消息。
5.  **`while True:`**：连接在此循环中保持打开状态，从而允许连续的消息交换。
6.  **`await websocket.receive_text()`**：等待客户端发送消息并以文本形式读取。
7.  **`await websocket.send_text(...)`**：向客户端回发一条消息。

如果客户端断开连接，`websocket.receive_text()` 将会抛出 `WebSocketDisconnect` 异常，这将中断循环并结束函数，从而有效地关闭服务器端的连接。

## 使用 Depends 及其他参数

WebSocket *路径操作函数* 可以接受与常规 HTTP *路径操作函数* 相同的参数和依赖项，包括路径参数、查询参数、Cookie、标头以及使用 `Depends` 的依赖项。

这对于身份验证尤其有用。你可以创建一个依赖项来检查查询参数中的令牌或会话 Cookie。

下面的示例展示了如何保护一个 WebSocket 端点，该端点要求提供会话 `Cookie` 或 `token` 查询参数。

```python title="tutorial002.py"
from typing import Union

from fastapi import (
    Cookie,
    Depends,
    FastAPI,
    Query,
    WebSocket,
    WebSocketException,
    status,
)
from fastapi.responses import HTMLResponse

app = FastAPI()

# ... (为简洁起见，省略了 HTML，它与前一个类似
# 但包含了项目 ID 和令牌的字段)

async def get_cookie_or_token(
    websocket: WebSocket,
    session: Union[str, None] = Cookie(default=None),
    token: Union[str, None] = Query(default=None),
):
    if session is None and token is None:
        raise WebSocketException(code=status.WS_1008_POLICY_VIOLATION)
    return session or token


@app.websocket("/items/{item_id}/ws")
async def websocket_endpoint(
    websocket: WebSocket,
    item_id: str,
    q: Union[int, None] = None,
    cookie_or_token: str = Depends(get_cookie_or_token),
):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(
            f"会话 Cookie 或查询令牌的值是：{cookie_or_token}"
        )
        if q is not None:
            await websocket.send_text(f"查询参数 q 是：{q}")
        await websocket.send_text(f"消息文本是：{data}，项目 ID 是：{item_id}")

```

在此示例中：

*   WebSocket URL 包含一个路径参数 `item_id` 和一个可选的查询参数 `q`。
*   使用 `Depends` 注入了 `get_cookie_or_token` 依赖项。它会尝试获取 `session` Cookie 或 `token` 查询参数。
*   如果两者都不存在，它会抛出 `WebSocketException`。这将向客户端发送一个关闭码并干净地终止连接，从而阻止端点的任何进一步执行。

## 处理多个客户端：聊天应用

对于聊天室等应用，你需要管理多个连接的客户端并向所有客户端广播消息。一个简单的方法是创建一个管理类来跟踪活动连接。

```d2
direction: down

Manager: {
  shape: class
  label: "ConnectionManager"
}

Clients: {
  shape: rectangle
  grid-columns: 3
  "客户端 A": { shape: c4-person }
  "客户端 B": { shape: c4-person }
  "客户端 C": { shape: c4-person }
}

Clients <-> Manager: "connect() / disconnect()"

"客户端 A" -> Manager: "send_text('你好')"

Manager -> "客户端 A": "send_personal_message('你写道：你好')"
Manager -> Clients: "broadcast('客户端 A 说：你好')"
```

以下是 `ConnectionManager` 的实现及其在聊天应用中的集成：

```python title="tutorial003.py"
from typing import List

from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.responses import HTMLResponse

app = FastAPI()

# ... (为简洁起见，省略了 HTML)

class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def send_personal_message(self, message: str, websocket: WebSocket):
        await websocket.send_text(message)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            await connection.send_text(message)


manager = ConnectionManager()


@app.get("/")
async def get():
    return HTMLResponse(html)


@app.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: int):
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.send_personal_message(f"你写道：{data}", websocket)
            await manager.broadcast(f"客户端 #{client_id} 说：{data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"客户端 #{client_id} 离开了聊天")

```

### 聊天应用中的关键概念

*   **`ConnectionManager`**：一个维护 `active_connections` 列表的简单类。
*   **`connect(websocket)`**：接受一个新连接并将其添加到列表中。
*   **`disconnect(websocket)`**：从活动连接列表中移除一个 WebSocket。
*   **`broadcast(message)`**：遍历所有活动连接并向它们发送消息。
*   **`try...except WebSocketDisconnect`**：这是处理客户端断开连接的标准方式。当客户端关闭连接时，`receive_text()` 会抛出 `WebSocketDisconnect`。`except` 块会捕获此异常，让你能够执行清理操作，例如将客户端从连接管理器中移除并通知其他用户。

现在你就拥有了一个功能齐全的多客户端聊天应用。要了解如何将其组织成一个更大的项目，可以继续阅读下一节。

接下来，让我们探讨如何构建[更大的应用](./advanced-bigger-applications.md)。
