# 异常

FastAPI 使用异常来处理错误并向客户端返回相应的 HTTP 响应。这对于处理客户端侧的问题（例如无效数据或身份验证失败）尤其有用。本页提供了 FastAPI 内置异常及其默认处理程序的参考。有关分步指南，请参阅[处理错误](https://fastapi.tiangolo.com/tutorial/handling-errors/)教程。

## HTTPException

你可以在代码中抛出 `HTTPException`，以向客户端发送特定的 HTTP 错误响应。它专为客户端可处理的操作性错误而设计，而非用于意外的服务器端程序错误。

### 参数

<x-field data-name="status_code" data-type="int" data-required="true" data-desc="发送给客户端的 HTTP 状态码。"></x-field>
<x-field data-name="detail" data-type="Any" data-required="false" data-desc="任何可 JSON 编码的数据，将作为 JSON 响应中 `detail` 键的值发送给客户端。"></x-field>
<x-field data-name="headers" data-type="Optional[Dict[str, str]]" data-required="false" data-desc="要包含在响应中的任何自定义标头。"></x-field>

### 示例

```python 抛出 HTTPException icon=logos:python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}


@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}
```

## WebSocketException

与 `HTTPException` 类似，`WebSocketException` 用于在发生错误时向 WebSocket 客户端发送特定的关闭码和原因。更多详情，请参阅 [WebSockets 文档](https://fastapi.tiangolo.com/advanced/websockets/)。

### 参数

<x-field data-name="code" data-type="int" data-required="true" data-desc="一个来自规范中定义的有效代码的关闭码。请参阅 RFC 6455 第 7.4.1 节。"></x-field>
<x-field data-name="reason" data-type="Union[str, None]" data-required="false" data-desc="一个 UTF-8 编码的字符串，用于解释关闭连接的原因。"></x-field>

### 示例

```python 抛出 WebSocketException icon=logos:python
from typing import Annotated

from fastapi import (
    Cookie,
    FastAPI,
    WebSocket,
    WebSocketException,
    status,
)

app = FastAPI()

@app.websocket("/items/{item_id}/ws")
async def websocket_endpoint(
    *,
    websocket: WebSocket,
    session: Annotated[str | None, Cookie()] = None,
    item_id: str,
):
    if session is None:
        raise WebSocketException(code=status.WS_1008_POLICY_VIOLATION)
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Session cookie is: {session}")
        await websocket.send_text(f"Message text was: {data}, for item ID: {item_id}")
```

## 验证错误

当请求数据（如请求体、查询参数）未能通过 Pydantic 验证时，FastAPI 会自动抛出验证异常。通常情况下，你不需要手动抛出这些异常。

-   **`RequestValidationError`**：标准 HTTP 请求验证失败时抛出。FastAPI 的默认处理程序会捕获此异常，并返回一个状态码为 `422 Unprocessable Entity` 的详细 JSON 响应。
-   **`WebSocketRequestValidationError`**：WebSocket 请求验证失败时抛出。默认处理程序会以代码 `1008` (`WS_1008_POLICY_VIOLATION`) 关闭连接。

## 默认异常处理程序

FastAPI 包含默认处理程序，可将上述异常转换为适当的客户端响应。你也可以[覆盖这些处理程序](https://fastapi.tiangolo.com/tutorial/handling-errors/#override-the-default-exception-handlers)以全局自定义错误响应。

### http_exception_handler

这是 `HTTPException` 的默认处理程序。它会根据异常对象中的状态码、详细信息和标头创建一个 `JSONResponse`。

```python http_exception_handler icon=logos:python
async def http_exception_handler(request: Request, exc: HTTPException) -> Response:
    headers = getattr(exc, "headers", None)
    if not is_body_allowed_for_status_code(exc.status_code):
        return Response(status_code=exc.status_code, headers=headers)
    return JSONResponse(
        {"detail": exc.detail}, status_code=exc.status_code, headers=headers
    )
```

### request_validation_exception_handler

这是 `RequestValidationError` 的默认处理程序。它会返回一个状态码为 `422 Unprocessable Entity` 的 `JSONResponse`，其响应体包含有关验证错误的详细信息。

```python request_validation_exception_handler icon=logos:python
async def request_validation_exception_handler(
    request: Request, exc: RequestValidationError
) -> JSONResponse:
    return JSONResponse(
        status_code=HTTP_422_UNPROCESSABLE_ENTITY,
        content={"detail": jsonable_encoder(exc.errors())},
    )
```

验证错误的响应体示例如下：

```json 验证错误响应示例 icon=mdi:code-json
{
  "detail": [
    {
      "loc": [
        "body",
        "name"
      ],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```

## 内部异常

以下异常通常由 FastAPI 在内部使用，在应用程序代码中较少直接使用。

-   **`FastAPIError`**：FastAPI 特有的通用运行时错误。
-   **`ResponseValidationError`**：当响应未能通过其 `response_model` 验证时，在内部抛出。