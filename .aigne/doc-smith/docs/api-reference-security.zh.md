# 安全工具

FastAPI 提供了一系列可作为依赖项调用的类，用于在您的 API 中实现各种安全方案。这些工具负责从请求中提取凭证（如令牌、API 密钥或基本认证标头），并与 OpenAPI 文档集成。

本参考文档介绍了 `fastapi.security` 中可用的主要安全工具。

---

## API 密钥

API 密钥认证是一种常见的模式，即在请求中传递一个密钥。FastAPI 提供了从查询参数、标头或 Cookie 中提取密钥的工具。

### `APIKeyQuery`

从查询参数中提取 API 密钥。

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyQuery

app = FastAPI()

query_scheme = APIKeyQuery(name="api_key")


@app.get("/items/")
async def read_items(api_key: str = Depends(query_scheme)):
    return {"api_key": api_key}
```

**参数**

| 名称 | 类型 | 描述 |
|---|---|---|
| `name` | `str` | **必需。** 包含 API 密钥的查询参数的名称。 |
| `scheme_name` | `Optional[str]` | 安全方案的名称，显示在 OpenAPI 文档中。默认为类名。 |
| `description` | `Optional[str]` | 安全方案的描述，显示在 OpenAPI 文档中。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在找不到密钥时会引发错误。如果为 `False`，则依赖项返回 `None`。 |

### `APIKeyHeader`

从请求标头中提取 API 密钥。

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyHeader

app = FastAPI()

header_scheme = APIKeyHeader(name="x-key")


@app.get("/items/")
async def read_items(key: str = Depends(header_scheme)):
    return {"key": key}
```

**参数**

| 名称 | 类型 | 描述 |
|---|---|---|
| `name` | `str` | **必需。** 包含 API 密钥的标头的名称。 |
| `scheme_name` | `Optional[str]` | 安全方案的名称，显示在 OpenAPI 文档中。默认为类名。 |
| `description` | `Optional[str]` | 安全方案的描述，显示在 OpenAPI 文档中。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在缺少标头时会引发错误。如果为 `False`，则依赖项返回 `None`。 |

### `APIKeyCookie`

从请求 Cookie 中提取 API 密钥。

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyCookie

app = FastAPI()

cookie_scheme = APIKeyCookie(name="session")


@app.get("/items/")
async def read_items(session: str = Depends(cookie_scheme)):
    return {"session": session}
```

**参数**

| 名称 | 类型 | 描述 |
|---|---|---|
| `name` | `str` | **必需。** 包含 API 密钥的 Cookie 的名称。 |
| `scheme_name` | `Optional[str]` | 安全方案的名称，显示在 OpenAPI 文档中。默认为类名。 |
| `description` | `Optional[str]` | 安全方案的描述，显示在 OpenAPI 文档中。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在找不到 Cookie 时会引发错误。如果为 `False`，则依赖项返回 `None`。 |

---

## HTTP 身份认证

这些工具实现了 RFC 文档中定义的标准 HTTP 身份认证方案，例如 Basic、Bearer 和 Digest。

### `HTTPBasic`

实现 HTTP 基本认证。它会提取 `Authorization` 标头，解码 Base64 凭证，并返回一个 `HTTPBasicCredentials` 对象。

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

**参数**

| 名称 | 类型 | 描述 |
|---|---|---|
| `scheme_name` | `Optional[str]` | OpenAPI 的安全方案名称。默认为类名。 |
| `realm` | `Optional[str]` | HTTP 基本认证领域，包含在 `WWW-Authenticate` 标头中。 |
| `description` | `Optional[str]` | OpenAPI 中安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在标头无效或缺失时会引发错误。如果为 `False`，则依赖项返回 `None`。 |

### `HTTPBearer`

实现 HTTP 持有者令牌认证。它会验证 `Authorization` 标头是否以“Bearer ”开头，并返回一个 `HTTPAuthorizationCredentials` 对象。

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

**参数**

| 名称 | 类型 | 描述 |
|---|---|---|
| `bearerFormat` | `Optional[str]` | 持有者令牌的预期格式（例如“JWT”），用于 OpenAPI 文档。 |
| `scheme_name` | `Optional[str]` | OpenAPI 的安全方案名称。默认为类名。 |
| `description` | `Optional[str]` | OpenAPI 中安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在标头无效或缺失时会引发错误。如果为 `False`，则依赖项返回 `None`。 |

### `HTTPDigest`

实现 HTTP 摘要认证。它会验证 `Authorization` 标头是否以“Digest ”开头，并返回一个 `HTTPAuthorizationCredentials` 对象。

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

**参数**

| 名称 | 类型 | 描述 |
|---|---|---|
| `scheme_name` | `Optional[str]` | OpenAPI 的安全方案名称。默认为类名。 |
| `description` | `Optional[str]` | OpenAPI 中安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在标头无效或缺失时会引发错误。如果为 `False`，则依赖项返回 `None`。 |

### 凭证模型

- **`HTTPBasicCredentials`**：使用 `HTTPBasic` 的结果。它有两个属性：
  - `username` (str)：提供的用户名。
  - `password` (str)：提供的密码。

- **`HTTPAuthorizationCredentials`**：`HTTPBearer` 或 `HTTPDigest` 的结果。它有两个属性：
  - `scheme` (str)：身份认证方案（例如，“Bearer”）。
  - `credentials` (str)：凭证字符串（例如，令牌）。

---

## OAuth2

FastAPI 提供了实现 OAuth2 流程的全面工具。

### `OAuth2PasswordBearer`

用于 OAuth2 密码持有者流程的依赖项类。它会检查是否存在有效的 `Authorization: Bearer <token>` 标头，并以字符串形式返回令牌。

**参数**

| 名称 | 类型 | 描述 |
|---|---|---|
| `tokenUrl` | `str` | **必需。** 颁发令牌的端点的 URL（例如 `/token`）。 |
| `scheme_name` | `Optional[str]` | OpenAPI 的安全方案名称。 |
| `scopes` | `Optional[Dict[str, str]]` | 一个包含可用作用域及其描述的字典，用于 OpenAPI。 |
| `description` | `Optional[str]` | OpenAPI 中安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在令牌无效或缺失时会引发错误。如果为 `False`，则返回 `None`。 |
| `refreshUrl` | `Optional[str]` | 用于刷新过期令牌的 URL。 |

### `OAuth2AuthorizationCodeBearer`

用于 OAuth2 授权码流程的依赖项类。它也需要一个 `Authorization: Bearer <token>` 标头。

**参数**

| 名称 | 类型 | 描述 |
|---|---|---|
| `authorizationUrl` | `str` | **必需。** 授权端点的 URL。 |
| `tokenUrl` | `str` | **必需。** 令牌交换端点的 URL。 |
| `refreshUrl` | `Optional[str]` | 用于刷新过期令牌的 URL。 |
| `scheme_name` | `Optional[str]` | OpenAPI 的安全方案名称。 |
| `scopes` | `Optional[Dict[str, str]]` | 一个包含可用作用域及其描述的字典，用于 OpenAPI。 |
| `description` | `Optional[str]` | OpenAPI 中安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在令牌无效或缺失时会引发错误。如果为 `False`，则返回 `None`。 |

### `OAuth2PasswordRequestForm` 和 `OAuth2PasswordRequestFormStrict`

这些是在令牌颁发端点中使用的依赖项类，用于接收表单数据（`application/x-www-form-urlencoded`）形式的凭证。

- `OAuth2PasswordRequestForm`：`grant_type` 是可选的。
- `OAuth2PasswordRequestFormStrict`：根据 OAuth2 规范，`grant_type` 必须为 `"password"`。

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordRequestForm

app = FastAPI()


@app.post("/login")
def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()])
    # 在实际应用中，您需要验证 form_data.username 和 form_data.password
    # 然后创建并返回一个令牌。
    return {"access_token": form_data.username, "token_type": "bearer"}
```

依赖项实例将具有从表单数据中提取的以下属性：
- `grant_type`：授权类型（例如，“password”）。
- `username`：用户名。
- `password`：用户密码。
- `scopes`：一个包含所请求作用域的 `list[str]`。
- `client_id`：客户端 ID（如果提供）。
- `client_secret`：客户端密钥（如果提供）。

### `SecurityScopes`

一个特殊的依赖项，用于访问同一路径操作中其他安全依赖项所需的作用域列表。

```python
from fastapi import Depends, FastAPI, Security
from fastapi.security import OAuth2PasswordBearer, SecurityScopes

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token", scopes={"me": "Read information about the current user."}) 

@app.get("/users/me")
async def read_users_me(
    security_scopes: SecurityScopes, token: str = Security(oauth2_scheme, scopes=["me"])
):
    return {"scopes": security_scopes.scopes, "token": token}
```

它提供两个属性：
- `scopes` (`List[str]`)：所需作用域的列表。
- `scope_str` (`str`)：一个包含所有作用域的字符串，各作用域之间用空格分隔。

---

## OpenID Connect

### `OpenIdConnect`

根据 OpenID Connect URL 实现身份认证。它会提取 `Authorization` 标头的值。

**参数**

| 名称 | 类型 | 描述 |
|---|---|---|
| `openIdConnectUrl` | `str` | **必需。** OpenID Connect 提供商的发现 URL。 |
| `scheme_name` | `Optional[str]` | OpenAPI 的安全方案名称。 |
| `description` | `Optional[str]` | OpenAPI 中安全方案的描述。 |
| `auto_error` | `bool` | 如果为 `True`（默认值），则在缺少 `Authorization` 标头时会引发错误。如果为 `False`，则返回 `None`。 |
