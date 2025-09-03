# WebSockets

FastAPI provides first-class support for WebSockets, enabling real-time, bidirectional communication between the client and the server. This is useful for applications like chat services, live notifications, and collaborative editing tools.

Under the hood, FastAPI's WebSocket functionality is powered by Starlette.

The basic flow of a WebSocket connection is as follows:

```d2
direction: down

"Client": { shape: person }
"Server": { shape: rectangle }

"Client" -> "Server": "1. HTTP GET Request with 'Upgrade: websocket' header"
"Server" -> "Client": "2. HTTP 101 Switching Protocols Response"
"Client" <-> "Server": "3. Persistent Bidirectional Communication Channel" {
  style {
    stroke-dash: 4
  }
}
```

## First Steps

Let's start with a simple example where the server echoes back any message it receives from a client.

First, you need a `WebSocket` endpoint. You create it using the `@app.websocket()` decorator.

Here's a complete application:

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

### Code Breakdown

1.  **HTML Frontend**: A simple webpage is served at the root `/`. It contains JavaScript to establish a WebSocket connection to `ws://localhost:8000/ws`.
2.  **`@app.websocket("/ws")`**: This decorator declares a WebSocket endpoint.
3.  **`websocket: WebSocket`**: The function receives a `WebSocket` object as a parameter.
4.  **`await websocket.accept()`**: This is crucial. You must `accept` the connection before you can send or receive messages.
5.  **`while True:`**: The connection remains open in this loop, allowing for continuous message exchange.
6.  **`await websocket.receive_text()`**: This waits for a message from the client and reads it as text.
7.  **`await websocket.send_text(...)`**: This sends a message back to the client.

If the client disconnects, `websocket.receive_text()` will raise a `WebSocketDisconnect` exception, which will break the loop and end the function, effectively closing the server-side connection.

## Using `Depends` and Other Parameters

WebSocket *path operation functions* can accept the same parameters and dependencies as regular HTTP *path operation functions*. This includes path parameters, query parameters, cookies, headers, and dependencies with `Depends`.

This is particularly useful for authentication. You can create a dependency that checks for a token in a query parameter or a session cookie.

Here's an example that secures a WebSocket endpoint, requiring either a session `Cookie` or a `token` query parameter.

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

In this example:

*   The WebSocket URL includes a path parameter `item_id` and an optional query parameter `q`.
*   The `get_cookie_or_token` dependency is injected using `Depends`. It tries to get a `session` cookie or a `token` query parameter.
*   If neither is present, it raises a `WebSocketException`. This will send a close code to the client and cleanly terminate the connection, preventing any further execution of the endpoint.

## Handling Multiple Clients: A Chat App

For applications like a chat room, you need to manage multiple connected clients and broadcast messages to all of them. A simple way to achieve this is by creating a manager class that keeps track of active connections.

```d2
direction: down

"Manager": {
  shape: class
  label: "ConnectionManager"
}

"Clients": {
  shape: package
  grid-columns: 3
  "Client A": { shape: person }
  "Client B": { shape: person }
  "Client C": { shape: person }
}

"Clients" <-> "Manager": "connect() / disconnect()"

"Client A" -> "Manager": "send_text('Hello')"

"Manager" -> "Client A": "send_personal_message('You wrote: Hello')"
"Manager" -> "Clients": "broadcast('Client A says: Hello')"
```

Here is the implementation of a `ConnectionManager` and its integration into a chat application:

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

### Key Concepts in the Chat App

*   **`ConnectionManager`**: A simple class that maintains a list of `active_connections`.
*   **`connect(websocket)`**: Accepts a new connection and adds it to the list.
*   **`disconnect(websocket)`**: Removes a WebSocket from the list of active connections.
*   **`broadcast(message)`**: Iterates through all active connections and sends them a message.
*   **`try...except WebSocketDisconnect`**: This is the standard way to handle client disconnections. When a client closes the connection, `receive_text()` will raise `WebSocketDisconnect`. The `except` block catches this, allowing you to perform cleanup actions, like removing the client from the connection manager and notifying other users.

Now you have a fully functional multi-client chat application. To learn how to organize this into a larger project, you can proceed to the next section.

Next, let's explore how to structure [Bigger Applications](./advanced-bigger-applications.md).
