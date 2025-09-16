# Exceptions

FastAPI uses exceptions to handle errors and return appropriate HTTP responses to the client. This is particularly useful for handling client-side issues like invalid data or authentication failures. This page provides a reference for FastAPI's built-in exceptions and their default handlers. For a step-by-step guide, see the tutorial on [Handling Errors](https://fastapi.tiangolo.com/tutorial/handling-errors/).

## HTTPException

You can raise `HTTPException` in your code to send a specific HTTP error response to the client. It's designed for operational errors that the client can act upon, not for unexpected server-side bugs.

### Parameters

<x-field data-name="status_code" data-type="int" data-required="true" data-desc="HTTP status code to send to the client."></x-field>
<x-field data-name="detail" data-type="Any" data-required="false" data-desc="Any JSON-encodable data to be sent to the client in the `detail` key of the JSON response."></x-field>
<x-field data-name="headers" data-type="Optional[Dict[str, str]]" data-required="false" data-desc="Any custom headers to include in the response."></x-field>

### Example

```python Raising HTTPException icon=logos:python
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

Similar to `HTTPException`, `WebSocketException` is used to send a specific closing code and reason to a WebSocket client when an error occurs. For more details, refer to the [WebSockets documentation](https://fastapi.tiangolo.com/advanced/websockets/).

### Parameters

<x-field data-name="code" data-type="int" data-required="true" data-desc="A closing code from the valid codes defined in the specification. See RFC 6455 Section 7.4.1."></x-field>
<x-field data-name="reason" data-type="Union[str, None]" data-required="false" data-desc="A UTF-8 encoded string explaining the reason for closing the connection."></x-field>

### Example

```python Raising WebSocketException icon=logos:python
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

## Validation Errors

When request data (e.g., body, query parameters) fails Pydantic validation, FastAPI automatically raises a validation exception. You typically don't need to raise these manually.

-   **`RequestValidationError`**: Raised for standard HTTP requests when validation fails. FastAPI's default handler catches this and returns a detailed JSON response with a `422 Unprocessable Entity` status.
-   **`WebSocketRequestValidationError`**: Raised for WebSocket requests when validation fails. The default handler closes the connection with code `1008` (`WS_1008_POLICY_VIOLATION`).

## Default Exception Handlers

FastAPI includes default handlers that convert the exceptions described above into appropriate client responses. You can also [override these handlers](https://fastapi.tiangolo.com/tutorial/handling-errors/#override-the-default-exception-handlers) to customize error responses globally.

### http_exception_handler

This is the default handler for `HTTPException`. It creates a `JSONResponse` with the status code, details, and headers from the exception object.

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

This is the default handler for `RequestValidationError`. It returns a `JSONResponse` with status code `422 Unprocessable Entity` and a body containing detailed information about the validation errors.

```python request_validation_exception_handler icon=logos:python
async def request_validation_exception_handler(
    request: Request, exc: RequestValidationError
) -> JSONResponse:
    return JSONResponse(
        status_code=HTTP_422_UNPROCESSABLE_ENTITY,
        content={"detail": jsonable_encoder(exc.errors())},
    )
```

An example response body for a validation error might look like this:

```json Example Validation Error Response icon=mdi:code-json
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

## Internal Exceptions

The following exceptions are generally used internally by FastAPI and are less commonly used directly in application code.

-   **`FastAPIError`**: A generic, FastAPI-specific runtime error.
-   **`ResponseValidationError`**: Raised internally when a response fails validation against its `response_model`.