# 安全性

FastAPI 提供了一系列工具来处理 API 中的安全性和身份验证。这些实用工具被设计为在*路径操作*中作为依赖项使用，与依赖注入系统无缝集成。它们负责从请求中提取凭证，并在凭证缺失或无效时自动返回相应的 HTTP 错误。

有关实现安全性的分步指南，请参阅[依赖项和安全性教程](./tutorials-dependencies-and-security.md)。

## API 密钥认证

API 密钥认证是保护端点的常用方法。FastAPI 提供了从请求的不同部分（查询参数、标头或 Cookie）提取 API 密钥的类。

### APIKeyQuery

`APIKeyQuery` 是一个依赖类，用于通过查询参数处理 API 密钥认证。

**参数**

<x-field data-name="name" data-type="string" data-required="true" data-desc="持有 API 密钥的查询参数的名称。"></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="安全方案的可选名称，用于 OpenAPI 文档。"></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="安全方案的可选描述，在 OpenAPI 文档中可见。"></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="如果为 true，则在密钥缺失时自动发送 HTTP 403 错误。如果为 false，则依赖项返回 `None`。"></x-field>

**示例**

```python APIKeyQuery 用法 icon=logos:python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyQuery

app = FastAPI()

query_scheme = APIKeyQuery(name="api_key")


@app.get("/items/")
async def read_items(api_key: str = Depends(query_scheme)):
    return {"api_key": api_key}
```

### APIKeyHeader

`APIKeyHeader` 是一个依赖类，用于通过请求标头处理 API 密钥认证。

**参数**

<x-field data-name="name" data-type="string" data-required="true" data-desc="持有 API 密钥的标头名称（例如，'X-API-Key'）。"></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="安全方案的可选名称，用于 OpenAPI 文档。"></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="安全方案的可选描述，在 OpenAPI 文档中可见。"></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="如果为 true，则在密钥缺失时自动发送 HTTP 403 错误。如果为 false，则依赖项返回 `None`。"></x-field>

**示例**

```python APIKeyHeader 用法 icon=logos:python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyHeader

app = FastAPI()

header_scheme = APIKeyHeader(name="x-key")


@app.get("/items/")
async def read_items(key: str = Depends(header_scheme)):
    return {"key": key}
```

### APIKeyCookie

`APIKeyCookie` 是一个依赖类，用于通过请求 Cookie 处理 API 密钥认证。

**参数**

<x-field data-name="name" data-type="string" data-required="true" data-desc="持有 API 密钥的 Cookie 的名称。"></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="安全方案的可选名称，用于 OpenAPI 文档。"></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="安全方案的可选描述，在 OpenAPI 文档中可见。"></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="如果为 true，则在密钥缺失时自动发送 HTTP 403 错误。如果为 false，则依赖项返回 `None`。"></x-field>

**示例**

```python APIKeyCookie 用法 icon=logos:python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyCookie

app = FastAPI()

cookie_scheme = APIKeyCookie(name="session")


@app.get("/items/")
async def read_items(session: str = Depends(cookie_scheme)):
    return {"session": session}
```

## HTTP 认证

FastAPI 支持标准的 HTTP 认证方案，如 Basic、Bearer 和 Digest。

### HTTPBasic

`HTTPBasic` 实现了 HTTP 基本认证。它从 `Authorization` 标头中提取用户名和密码。

依赖项的结果是一个 `HTTPBasicCredentials` 对象。

**参数**

<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="安全方案的可选名称，用于 OpenAPI 文档。"></x-field>
<x-field data-name="realm" data-type="string" data-required="false" data-desc="HTTP 基本认证领域，包含在 401 响应的 'WWW-Authenticate' 标头中。"></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="安全方案的可选描述，在 OpenAPI 文档中可见。"></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="如果为 true，则在未提供凭证时自动发送 HTTP 401 错误。如果为 false，则返回 `None`。"></x-field>

**示例**

```python HTTPBasic 用法 icon=logos:python
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

`HTTPBearer` 实现了 HTTP Bearer 令牌认证，通常与 OAuth2 一起使用。

依赖项的结果是一个 `HTTPAuthorizationCredentials` 对象。

**参数**

<x-field data-name="bearerFormat" data-type="string" data-required="false" data-desc="关于 Bearer 令牌格式（例如 'JWT'）给客户端的可选提示。用于 OpenAPI 文档。"></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="安全方案的可选名称，用于 OpenAPI 文档。"></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="安全方案的可选描述，在 OpenAPI 文档中可见。"></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="如果为 true，则在未提供令牌时自动发送错误。如果为 false，则返回 `None`。"></x-field>

**示例**

```python HTTPBearer 用法 icon=logos:python
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

`HTTPDigest` 实现了 HTTP Digest 认证。

依赖项的结果是一个 `HTTPAuthorizationCredentials` 对象。

**参数**

<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="安全方案的可选名称，用于 OpenAPI 文档。"></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="安全方案的可选描述，在 OpenAPI 文档中可见。"></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="如果为 true，则在未提供摘要时自动发送错误。如果为 false，则返回 `None`。"></x-field>

**示例**

```python HTTPDigest 用法 icon=logos:python
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

### 辅助模型

#### HTTPBasicCredentials

一个数据模型，包含从 HTTP 基本认证中解码的 `username` 和 `password`。

<x-field data-name="username" data-type="string" data-required="true" data-desc="HTTP Basic 用户名。"></x-field>
<x-field data-name="password" data-type="string" data-required="true" data-desc="HTTP Basic 密码。"></x-field>

#### HTTPAuthorizationCredentials

一个数据模型，包含来自 `Authorization` 标头的 `scheme` 和 `credentials`。

<x-field data-name="scheme" data-type="string" data-required="true" data-desc="认证方案（例如 'Bearer'、'Digest'）。"></x-field>
<x-field data-name="credentials" data-type="string" data-required="true" data-desc="凭证字符串，例如令牌。"></x-field>

## OAuth2

FastAPI 为实现 OAuth2 流程提供了全面的支持。

### OAuth2PasswordBearer

这是一个依赖类，它使用 Bearer 令牌实现 OAuth2 “密码”（Password）流程。它会检查是否存在有效的 `Authorization: Bearer <token>` 标头，并以字符串形式返回令牌。

**参数**

<x-field data-name="tokenUrl" data-type="string" data-required="true" data-desc="颁发令牌的端点的 URL（例如 '/token'）。"></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="安全方案的可选名称，用于 OpenAPI。"></x-field>
<x-field data-name="scopes" data-type="Dict[str, str]" data-required="false" data-desc="作用域名称到描述的字典，用于 OpenAPI。"></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="安全方案的可选描述。"></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="如果为 true，则在 'Authorization' 标头缺失或无效时自动引发错误。"></x-field>
<x-field data-name="refreshUrl" data-type="string" data-required="false" data-desc="用于刷新令牌并获取新令牌的 URL。"></x-field>

### OAuth2AuthorizationCodeBearer

此依赖类使用 Bearer 令牌实现 OAuth2 “授权码”（Authorization Code）流程。

**参数**

<x-field data-name="authorizationUrl" data-type="string" data-required="true" data-desc="授权端点的 URL。"></x-field>
<x-field data-name="tokenUrl" data-type="string" data-required="true" data-desc="颁发令牌的端点的 URL。"></x-field>
<x-field data-name="refreshUrl" data-type="string" data-required="false" data-desc="用于刷新令牌并获取新令牌的 URL。"></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="安全方案的可选名称，用于 OpenAPI。"></x-field>
<x-field data-name="scopes" data-type="Dict[str, str]" data-required="false" data-desc="作用域名称到描述的字典，用于 OpenAPI。"></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="安全方案的可选描述。"></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="如果为 true，则在 'Authorization' 标头缺失或无效时自动引发错误。"></x-field>

### OAuth2PasswordRequestForm & OAuth2PasswordRequestFormStrict

这些是依赖类，用于解析以表单数据形式发送的 OAuth2 密码流程请求体。它们提取 `username`、`password`、`scope` 和其他字段。`OAuth2PasswordRequestFormStrict` 还要求 `grant_type` 字段必须存在且值为 `"password"`。

**表单字段**

<x-field data-name="grant_type" data-type="str" data-required="false" data-desc="`OAuth2PasswordRequestFormStrict` 要求此字段。必须为 'password'。对于 `OAuth2PasswordRequestForm` 是可选的。"></x-field>
<x-field data-name="username" data-type="str" data-required="true" data-desc="用户的用户名。"></x-field>
<x-field data-name="password" data-type="str" data-required="true" data-desc="用户的密码。"></x-field>
<x-field data-name="scope" data-type="str" data-default="" data-required="false" data-desc="由空格分隔的作用域字符串。"></x-field>
<x-field data-name="client_id" data-type="str | None" data-required="false" data-desc="客户端 ID。"></x-field>
<x-field data-name="client_secret" data-type="str | None" data-required="false" data-desc="客户端密钥。"></x-field>

**示例**

```python OAuth2PasswordRequestForm 用法 icon=logos:python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordRequestForm

app = FastAPI()


@app.post("/login")
def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()])
    # The form_data object will have attributes like username, password, scopes, etc.
    return {"username": form_data.username, "scopes": form_data.scopes}
```

### SecurityScopes

一个特殊的依赖类，用于访问同一*路径操作*中其他依赖项所需的安全作用域列表。

**属性**

<x-field data-name="scopes" data-type="List[str]" data-desc="依赖项所需的所有作用域的列表。"></x-field>
<x-field data-name="scope_str" data-type="str" data-desc="包含所有作用域的单个字符串，以空格分隔。"></x-field>

## OpenID Connect

### OpenIdConnect

`OpenIdConnect` 是一个用于处理 OpenID Connect 认证的依赖类。它主要用于在 OpenAPI 中记录安全方案。

**参数**

<x-field data-name="openIdConnectUrl" data-type="string" data-required="true" data-desc="OpenID Connect 发现 URL。"></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="安全方案的可选名称。"></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="安全方案的可选描述。"></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="如果为 true，则在 'Authorization' 标头缺失时自动引发错误。"></x-field>