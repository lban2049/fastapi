# 安全实用工具

FastAPI 提供了一套简单而强大的工具来处理安全和身份验证。这些实用工具构建于依赖注入系统之上，可让你轻松实现各种安全方案，如 OAuth2、HTTP 基本/持有者/摘要式认证和 API 密钥。它们直接与自动生成的 OpenAPI 文档集成，使你的 API 的安全要求清晰明了且具有交互性。

本参考指南为 `fastapi.security` 中可用的每个安全类和实用工具模型提供了详细文档。

```d2
direction: down

Security-Utilities: {
  label: "Security Utilities"
  shape: package
  grid-columns: 2

  API-Key-Auth: {
    label: "API Key Auth"
    shape: rectangle
    APIKeyQuery: {
      label: "From Query Param"
    }
    APIKeyHeader: {
      label: "From Header"
    }
    APIKeyCookie: {
      label: "From Cookie"
    }
  }

  HTTP-Auth: {
    label: "HTTP Auth"
    shape: rectangle
    HTTPBasic: {}
    HTTPBearer: {}
    HTTPDigest: {}
    HTTPBasicCredentials: {
      shape: document
    }
    HTTPAuthorizationCredentials: {
      shape: document
    }
  }

  OAuth2: {
    shape: rectangle
    OAuth2PasswordBearer: {}
    OAuth2AuthorizationCodeBearer: {}
    OAuth2PasswordRequestForm: {
      shape: document
    }
    OAuth2PasswordRequestFormStrict: {
      shape: document
    }
    SecurityScopes: {
      shape: document
    }
  }

  OpenID-Connect: {
    label: "OpenID Connect"
    shape: rectangle
    OpenIdConnect: {}
  }
}
```

## API 密钥认证

API 密钥认证可以来源于查询参数、标头或 Cookie。

### APIKeyQuery

从查询参数中提取 API 密钥。你可以创建一个实例并将其用作依赖项。

**参数**

| Parameter | Type | Description |
|---|---|---|
| `name` | `str` | 用于 API 密钥的查询参数的名称。 |
| `scheme_name` | `Optional[str]` | 安全方案的名称，在 OpenAPI 文档中可见。 |
| `description` | `Optional[str]` | OpenAPI 文档中安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在密钥缺失时引发 HTTP 403 错误。如果为 `False`，依赖项将返回 `None`。 |

**示例**

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyQuery

app = FastAPI()

query_scheme = APIKeyQuery(name="api_key")


@app.get("/items/")
async def read_items(api_key: str = Depends(query_scheme)):
    return {"api_key": api_key}
```

### APIKeyHeader

从 HTTP 标头中提取 API 密钥。

**参数**

| Parameter | Type | Description |
|---|---|---|
| `name` | `str` | 用于 API 密钥的 HTTP 标头的名称。 |
| `scheme_name` | `Optional[str]` | 安全方案的名称，在 OpenAPI 文档中可见。 |
| `description` | `Optional[str]` | OpenAPI 文档中安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在密钥缺失时引发 HTTP 403 错误。如果为 `False`，依赖项将返回 `None`。 |

**示例**

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyHeader

app = FastAPI()

header_scheme = APIKeyHeader(name="x-key")


@app.get("/items/")
async def read_items(key: str = Depends(header_scheme)):
    return {"key": key}
```

### APIKeyCookie

从请求 Cookie 中提取 API 密钥。

**参数**

| Parameter | Type | Description |
|---|---|---|
| `name` | `str` | 用于 API 密钥的 Cookie 的名称。 |
| `scheme_name` | `Optional[str]` | 安全方案的名称，在 OpenAPI 文档中可见。 |
| `description` | `Optional[str]` | OpenAPI 文档中安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在密钥缺失时引发 HTTP 403 错误。如果为 `False`，依赖项将返回 `None`。 |

**示例**

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyCookie

app = FastAPI()

cookie_scheme = APIKeyCookie(name="session")


@app.get("/items/")
async def read_items(session: str = Depends(cookie_scheme)):
    return {"session": session}
```

## HTTP 认证

实现标准的 HTTP 认证方案。

### HTTPBasic

处理 HTTP 基本认证。依赖项的结果是一个 `HTTPBasicCredentials` 对象。

**参数**

| Parameter | Type | Description |
|---|---|---|
| `scheme_name` | `Optional[str]` | 安全方案的名称，在 OpenAPI 文档中可见。 |
| `realm` | `Optional[str]` | HTTP 基本认证领域。 |
| `description` | `Optional[str]` | OpenAPI 文档中安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在未提供认证时引发错误。如果为 `False`，则返回 `None`。 |

**示例**

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import HTTPBasic, HTTPBasicCredentials

app = FastAPI()

security = HTTPBasic()


@app.get("/users/me")
def read_current_user(credentials: Annotated[HTTPBasicCredentials, Depends(security)]):
    return {"username": credentials.username, "password": credentials.password}
```

### HTTPBearer

处理 HTTP Bearer 令牌认证。依赖项的结果是一个 `HTTPAuthorizationCredentials` 对象。

**参数**

| Parameter | Type | Description |
|---|---|---|
| `bearerFormat` | `Optional[str]` | 持有者令牌的格式（例如，'JWT'），在 OpenAPI 文档中可见。 |
| `scheme_name` | `Optional[str]` | 安全方案的名称。 |
| `description` | `Optional[str]` | 安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在令牌缺失时引发错误。如果为 `False`，则返回 `None`。 |

**示例**

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import HTTPAuthorizationCredentials, HTTPBearer

app = FastAPI()

security = HTTPBearer()


@app.get("/users/me")
def read_current_user(
    credentials: Annotated[HTTPAuthorizationCredentials, Depends(security)]
):
    return {"scheme": credentials.scheme, "credentials": credentials.credentials}
```

### HTTPDigest

处理 HTTP 摘要式认证。依赖项的结果是一个 `HTTPAuthorizationCredentials` 对象。

**参数**

| Parameter | Type | Description |
|---|---|---|
| `scheme_name` | `Optional[str]` | 安全方案的名称，在 OpenAPI 文档中可见。 |
| `description` | `Optional[str]` | 安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在摘要缺失时引发错误。如果为 `False`，则返回 `None`。 |

**示例**

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import HTTPAuthorizationCredentials, HTTPDigest

app = FastAPI()

security = HTTPDigest()


@app.get("/users/me")
def read_current_user(
    credentials: Annotated[HTTPAuthorizationCredentials, Depends(security)]
):
    return {"scheme": credentials.scheme, "credentials": credentials.credentials}
```

### HTTPBasicCredentials

一个包含来自 HTTP 基本认证的用户名和密码的数据模型。

**属性**

| Attribute | Type | Description |
|---|---|---|
| `username` | `str` | HTTP 基本认证的用户名。 |
| `password` | `str` | HTTP 基本认证的密码。 |

### HTTPAuthorizationCredentials

一个包含来自 `Authorization` 标头的方案和凭据的数据模型。

**属性**

| Attribute | Type | Description |
|---|---|---|
| `scheme` | `str` | 授权方案（例如，'Bearer'、'Digest'）。 |
| `credentials` | `str` | 标头值中的凭据部分。 |

## OAuth2

用于实现 OAuth2 流的实用工具。

### OAuth2PasswordBearer

定义一个 OAuth2 密码持有者流。它从 `Authorization` 标头中提取令牌。

**参数**

| Parameter | Type | Description |
|---|---|---|
| `tokenUrl` | `str` | 提供令牌的路径操作的 URL（例如，`/token`）。 |
| `scheme_name` | `Optional[str]` | OpenAPI 的安全方案名称。 |
| `scopes` | `Optional[Dict[str, str]]` | 可用范围及其描述的字典。 |
| `description` | `Optional[str]` | 安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在令牌缺失时引发错误。如果为 `False`，则返回 `None`。 |
| `refreshUrl` | `Optional[str]` | 刷新令牌的 URL。 |

### OAuth2AuthorizationCodeBearer

定义一个 OAuth2 授权码持有者流。它从 `Authorization` 标头中提取令牌。

**参数**

| Parameter | Type | Description |
|---|---|---|
| `authorizationUrl` | `str` | 授权步骤的 URL。 |
| `tokenUrl` | `str` | 获取令牌的 URL。 |
| `refreshUrl` | `Optional[str]` | 刷新令牌的 URL。 |
| `scheme_name` | `Optional[str]` | OpenAPI 的安全方案名称。 |
| `scopes` | `Optional[Dict[str, str]]` | 可用范围及其描述的字典。 |
| `description` | `Optional[str]` | 安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在令牌缺失时引发错误。如果为 `False`，则返回 `None`。 |

### OAuth2PasswordRequestForm

一个从请求中捕获 OAuth2 密码流表单数据的依赖项类。

**属性**

| Attribute | Type | Description |
|---|---|---|
| `grant_type` | `Optional[str]` | 必须是 'password'。此为宽容模式，允许 `None`。 |
| `username` | `str` | 来自表单数据的用户名。 |
| `password` | `str` | 来自表单数据的密码。 |
| `scopes` | `List[str]` | 请求的范围列表，从一个以空格分隔的字符串中解析得出。 |
| `client_id` | `Optional[str]` | 客户端 ID，如果在表单中提供。 |
| `client_secret` | `Optional[str]` | 客户端密钥，如果在表单中提供。 |

### OAuth2PasswordRequestFormStrict

`OAuth2PasswordRequestForm` 的一个更严格的版本，它要求 `grant_type` 表单字段必须存在且值为 `'password'`，这是 OAuth2 规范所强制要求的。

### SecurityScopes

一个特殊的依赖项类，用于获取同一*路径操作*中其他依赖项所需的安全范围。

**属性**

| Attribute | Type | Description |
|---|---|---|
| `scopes` | `List[str]` | 所有依赖项所需的范围列表。 |
| `scope_str` | `str` | 一个包含所有范围的字符串，以空格分隔。 |

## OpenID Connect

### OpenIdConnect

定义 OpenID Connect 认证。它从 `Authorization` 标头中提取令牌。

**参数**

| Parameter | Type | Description |
|---|---|---|
| `openIdConnectUrl` | `str` | OpenID Connect 发现 URL。 |
| `scheme_name` | `Optional[str]` | OpenAPI 的安全方案名称。 |
| `description` | `Optional[str]` | 安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在令牌缺失时引发错误。如果为 `False`，则返回 `None`。 |
