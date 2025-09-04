# WebSockets

FastAPI 支持 WebSockets，可实现客户端与服务器之间的实时双向通信。这对于聊天服务、实时通知和协作编辑工具等应用非常有用。

FastAPI 的 WebSocket 功能底层由 Starlette 提供支持。

WebSocket 连接的基本流程如下：

```d2
direction: down

Client: { 
  shape: person 
}

Server: { 
  shape: rectangle 
}

Client -> Server: "1. 带有 'Upgrade: websocket' 头信息的 HTTP GET 请求"
Server -> Client: "2. HTTP 101 切换协议响应"
Client <-> Server: "3. 持久双向通信通道" {
  style {
    stroke-dash: 4
  }
}
```

## 快速入门

让我们从一个简单的示例开始，服务器会回显从客户端收到的任何消息。

首先，你需要一个 `WebSocket` 端点。可以使用 `@app.websocket()` 装饰器来创建它。

以下是一个完整的应用：

```python
from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse

app = FastAPI()

html = """
<!DOCTYPE html>
<html>
    <head>
        <title>Chat</title>
    </head>
    <body>
        <h1>WebSocket Chat</h1>
        <form action="" onsubmit="sendMessage(event)">
            <input type="text" id="messageText" autocomplete="off"/>
            <button>Send</button>
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
        await websocket.send_text(f"Message text was: {data}")

```

### 代码解析

1.  **HTML 前端**：在根路径 `/` 提供一个简单的网页。其中包含用于建立到 `ws://localhost:8000/ws` 的 WebSocket 连接的 JavaScript。
2.  **`@app.websocket("/ws")`**：此装饰器用于声明一个 WebSocket 端点。
3.  **`websocket: WebSocket`**：该函数接收一个 `WebSocket` 对象作为参数。
4.  **`await websocket.accept()`**：这步至关重要。你必须先 `accept` 连接，然后才能发送或接收消息。
5.  **`while True:`**：连接在此循环中保持打开状态，以实现持续的消息交换。
6.  **`await websocket.receive_text()`**：此方法会等待客户端的消息并将其作为文本读取。
7.  **`await websocket.send_text(...)`**：此方法会向客户端回发一条消息。

如果客户端断开连接，`websocket.receive_text()` 将引发 `WebSocketDisconnect` 异常，这将中断循环并结束函数，从而有效关闭服务器端的连接。

## 使用 `Depends` 和其他参数

WebSocket *路径操作函数* 可以接受与常规 HTTP *路径操作函数* 相同的参数和依赖项。这包括路径参数、查询参数、Cookie、请求头以及使用 `Depends` 的依赖项。

这对于身份验证特别有用。你可以创建一个依赖项来检查查询参数中的令牌或会话 Cookie。

以下示例展示了如何保护 WebSocket 端点，要求提供会话 `Cookie` 或 `token` 查询参数。

```python
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

# ... (HTML is omitted for brevity, it's similar to the previous one but with fields for item ID and token)

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
            f"Session cookie or query token value is: {cookie_or_token}"
        )
        if q is not None:
            await websocket.send_text(f"Query parameter q is: {q}")
        await websocket.send_text(f"Message text was: {data}, for item ID: {item_id}")

```

在此示例中：

*   WebSocket URL 包含一个路径参数 `item_id` 和一个可选的查询参数 `q`。
*   `get_cookie_or_token` 依赖项通过 `Depends` 注入。它会尝试获取 `session` Cookie 或 `token` 查询参数。
*   如果两者都不存在，它会引发 `WebSocketException`。这将向客户端发送一个关闭代码并干净地终止连接，从而阻止端点的任何后续执行。

## 处理多个客户端：一个聊天应用

对于聊天室之类的应用，你需要管理多个连接的客户端，并向所有客户端广播消息。实现此目的的一个简单方法是创建一个管理类来跟踪活动连接。

```d2
direction: down

Manager: {
  shape: class
  label: "ConnectionManager"
}

Clients: {
  shape: package
  grid-columns: 3
  "客户端 A": { shape: person }
  "客户端 B": { shape: person }
  "客户端 C": { shape: person }
}

Clients <-> Manager: "connect() / disconnect()"

"客户端 A" -> Manager: "send_text('Hello')"

Manager -> "客户端 A": "send_personal_message('你写道：Hello')"
Manager -> Clients: "broadcast('客户端 A 说：Hello')"
```

以下是 `ConnectionManager` 的实现及其在聊天应用中的集成：

```python
from typing import List

from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.responses import HTMLResponse

app = FastAPI()

# ... (HTML is omitted for brevity)

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
            await manager.send_personal_message(f"You wrote: {data}", websocket)
            await manager.broadcast(f"Client #{client_id} says: {data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"Client #{client_id} left the chat")

```

### 聊天应用中的关键概念

*   **`ConnectionManager`**：一个维护 `active_connections` 列表的简单类。
*   **`connect(websocket)`**：接受新连接并将其添加到列表中。
*   **`disconnect(websocket)`**：从活动连接列表中移除一个 WebSocket。
*   **`broadcast(message)`**：遍历所有活动连接并向它们发送消息。
*   **`try...except WebSocketDisconnect`**：这是处理客户端断开连接的标准方式。当客户端关闭连接时，`receive_text()` 将引发 `WebSocketDisconnect`。`except` 块会捕获此异常，让你能够执行清理操作，例如从连接管理器中移除该客户端并通知其他用户。

现在，你拥有一个功能齐全的多客户端聊天应用。要了解如何将其组织成一个更大的项目，你可以继续阅读下一节。

接下来，让我们探讨如何构建[更大型的应用](./advanced-bigger-applications.md)。