# 安全

实现安全是任何 API 的一个关键方面。FastAPI 提供了一套与标准安全协议集成的工具和依赖项，可以轻松地为您的应用程序添加强大的身份验证和授权功能。这是通过 FastAPI 强大的依赖注入系统来处理的。

本指南涵盖了多种常见的安全方案，包括使用 JWT 令牌的 OAuth2、HTTP 基本身份验证和 API 密钥。

## 安全流程概述

一个典型的身份验证流程，例如使用不记名令牌的 OAuth2，涉及客户端首先使用凭据进行身份验证以获取令牌，然后使用该令牌访问受保护的资源。

```d2
direction: down

"User": { shape: person }
"API Server": {
  shape: package
  grid-columns: 1
  "/token": {label: "令牌端点"}
  "/users/me": {label: "受保护的端点"}
}

"User" -> "API Server"."/token": "1. 使用凭据进行身份验证" {
  label: "POST /token\n(username, password)"
}
"API Server"."/token" -> "User": "2. 接收访问令牌 (JWT)"

"User" -> "API Server"."/users/me": "3. 使用令牌请求受保护的数据" {
  label: "GET /users/me\n(Authorization: Bearer <token>)"
}
"API Server"."/users/me" -> "User": "4. 接收受保护的数据"
```

## 使用密码和不记名令牌的 OAuth2

OAuth2 是一种广泛使用的授权协议。“密码”流是用户直接提供凭据以换取访问令牌的常用方式。然后，该令牌作为“不记名”令牌在 `Authorization` 标头中发送，用于后续请求。

### 第一步：创建安全方案

首先，您需要一个 `OAuth2PasswordBearer` 的实例。该对象是一个依赖项，它将在 `Authorization` 标头中要求一个不记名令牌。

`tokenUrl` 参数指向客户端将用于获取令牌的 URL（我们稍后将创建它）。

```python
from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


@app.get("/items/")
async def read_items(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

这样，`/items/` 端点将需要一个 `Authorization` 标头，其值类似于 `Bearer your-token-here`。该依赖项将以 `str` 形式返回令牌。

### 获取当前用户

仅仅拥有令牌字符串是不够的；您需要验证它并获取相应的用户数据。您可以创建第二个依赖项 `get_current_user`，它依赖于 `oauth2_scheme`。

这个新的依赖项将接收令牌，对其进行解码，并返回用户的数据。

```python
from typing import Union

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


class User(BaseModel):
    username: str
    email: Union[str, None] = None
    full_name: Union[str, None] = None
    disabled: Union[bool, None] = None


def fake_decode_token(token):
    return User(
        username=token + "fakedecoded", email="john@example.com", full_name="John Doe"
    )


async def get_current_user(token: str = Depends(oauth2_scheme)):
    user = fake_decode_token(token)
    return user


@app.get("/users/me")
async def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user
```

现在，`read_users_me` 路径操作依赖于 `get_current_user`。FastAPI 将按顺序调用依赖项：首先是 `oauth2_scheme`，然后是使用其结果调用 `get_current_user`。最终结果是一个 `User` 对象。

### 令牌端点

接下来，您需要创建 `/token` 路径操作，以便客户端可以发送用户名和密码来获取令牌。

FastAPI 提供了 `OAuth2PasswordRequestForm` 来处理传入的表单数据。

```python
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm

# ... (来自先前示例的用户模型和 fake_users_db)

app = FastAPI()

# ... (get_current_user 等)

@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    user = UserInDB(**user_dict)
    # 目前，我们不检查密码，只检查用户名
    # 令牌也只是用户名
    return {"access_token": user.username, "token_type": "bearer"}


@app.get("/users/me")
async def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user
```

此端点从一个伪数据库中验证用户，并返回一个包含 `access_token` 和 `token_type` 的对象。

### 使用 JWT 作为令牌

在实际应用程序中，您应该使用像 JSON Web Tokens (JWT) 这样的加密签名令牌，而不是简单的字符串。这可以确保令牌数据未被篡改。

您需要安装 `passlib` 用于密码哈希，以及 `python-jwt` 用于创建和验证 JWT。

```bash
pip install "passlib[bcrypt]" python-jwt
```

以下是一个包含密码哈希和 JWT 创建的更完整的示例：

```python
from datetime import datetime, timedelta, timezone
from typing import Union

import jwt
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jwt.exceptions import InvalidTokenError
from passlib.context import CryptContext
from pydantic import BaseModel

# --- 配置 ---
SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# --- 模型 ---
class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Union[str, None] = None

# ... (User, UserInDB 模型)

# --- 哈希与数据库 ---
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
# ... (包含哈希密码的 fake_users_db, verify_password, get_user)

# --- JWT 创建 ---
def create_access_token(data: dict, expires_delta: Union[timedelta, None] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

# --- 获取当前用户的依赖项 ---
async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except InvalidTokenError:
        raise credentials_exception
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user

# --- 令牌端点 ---
@app.post("/token", response_model=Token)
async def login_for_access_token(
    form_data: OAuth2PasswordRequestForm = Depends(),
):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}

# --- 受保护的端点 ---
@app.get("/users/me/", response_model=User)
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    return current_user
```

此实现正确地验证了用户，生成了一个安全的 JWT，并在受保护的端点上对其进行验证。

### 用于授权的 OAuth2 范围

范围（Scopes）用于向客户端授予特定权限。您可以在 `OAuth2PasswordBearer` 中定义可用的范围，然后使用 `Security` 依赖项为某些端点要求特定的范围。

1.  **定义范围：**

    ```python
    oauth2_scheme = OAuth2PasswordBearer(
        tokenUrl="token",
        scopes={"me": "Read information about the current user.", "items": "Read items."},
    )
    ```

2.  **在令牌端点中请求范围：** 客户端可以在 `/token` 请求中请求特定的范围。您应该将这些范围包含在 JWT 中。

    ```python
    # 在您的 /token 端点中
    access_token = create_access_token(
        data={"sub": user.username, "scope": " ".join(form_data.scopes)},
        expires_delta=access_token_expires,
    )
    ```

3.  **在依赖项中检查范围：** `get_current_user` 依赖项必须更新，以检查令牌是否包含正在访问的端点所需的范围。

    ```python
    from fastapi.security import SecurityScopes

    async def get_current_user(
        security_scopes: SecurityScopes, token: str = Depends(oauth2_scheme)
    ):
        # ... (JWT 解码)
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        scope_str: str = payload.get("scope", "")
        token_scopes = scope_str.split(" ")
        # ... (用户获取)
        for scope in security_scopes.scopes:
            if scope not in token_scopes:
                raise HTTPException(
                    status_code=status.HTTP_401_UNAUTHORIZED,
                    detail="Not enough permissions",
                    # ...
                )
        return user
    ```

4.  **在路径操作中要求范围：** 使用 `Security` 而不是 `Depends` 来指定所需的范围。

    ```python
    from fastapi import Security

    @app.get("/users/me/items/")
    async def read_own_items(
        current_user: User = Security(get_current_active_user, scopes=["items"]),
    ):
        return [{"item_id": "Foo", "owner": current_user.username}]
    ```

## HTTP 基本身份验证

HTTP 基本身份验证是一种更简单的方案，其中用户名和密码包含在 `Authorization` 标头中，并经过 Base64 编码。虽然简单，但它只应在 HTTPS 上使用，因为凭据未加密。

FastAPI 为此提供了 `HTTPBasic` 和 `HTTPBasicCredentials`。

```python
import secrets

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import HTTPBasic, HTTPBasicCredentials

app = FastAPI()

security = HTTPBasic()


def get_current_username(credentials: HTTPBasicCredentials = Depends(security)):
    current_username_bytes = credentials.username.encode("utf8")
    correct_username_bytes = b"stanleyjobson"
    is_correct_username = secrets.compare_digest(
        current_username_bytes, correct_username_bytes
    )
    current_password_bytes = credentials.password.encode("utf8")
    correct_password_bytes = b"swordfish"
    is_correct_password = secrets.compare_digest(
        current_password_bytes, correct_password_bytes
    )
    if not (is_correct_username and is_correct_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Basic"},
        )
    return credentials.username


@app.get("/users/me")
def read_current_user(username: str = Depends(get_current_username)):
    return {"username": username}
```

此示例创建了一个依赖项 `get_current_username`，它接收凭据，使用 `secrets.compare_digest` 将它们与已知值进行比较以防止时序攻击，如果凭据不匹配则引发异常。

## API 密钥身份验证

API 密钥是向特定客户端或服务授予访问权限的常用方法。密钥是客户端在每个请求中发送的单个令牌。FastAPI 提供了从不同位置提取 API 密钥的帮助程序。

<x-cards data-columns="3">
  <x-card data-title="查询中的 API 密钥" data-icon="lucide:file-question">
    API 密钥作为 URL 中的查询参数传递。
  </x-card>
  <x-card data-title="标头中的 API 密钥" data-icon="lucide:file-terminal">
    API 密钥在自定义 HTTP 标头中传递。
  </x-card>
  <x-card data-title="Cookie 中的 API 密钥" data-icon="lucide:cookie">
    API 密钥在请求 Cookie 中传递。
  </x-card>
</x-cards>

### 查询中的 API 密钥

使用 `APIKeyQuery` 来期望在查询参数中获取 API 密钥。

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyQuery

app = FastAPI()

api_key_query = APIKeyQuery(name="api-key", auto_error=False)


@app.get("/items/")
async def read_items(api_key: str = Depends(api_key_query)):
    if api_key:
        return {"api_key": api_key}
    else:
        return {"message": "No API Key provided."}
```

### 标头中的 API 密钥

使用 `APIKeyHeader` 来期望在自定义标头（例如，`X-API-Key`）中获取 API 密钥。

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyHeader

app = FastAPI()

api_key_header = APIKeyHeader(name="X-API-Key")


@app.get("/items/")
async def read_items(api_key: str = Depends(api_key_header)):
    return {"api_key": api_key}
```

### Cookie 中的 API 密钥

使用 `APIKeyCookie` 来期望在 Cookie 中获取 API 密钥。

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyCookie

app = FastAPI()

cookie_scheme = APIKeyCookie(name="session")


@app.get("/items/")
async def read_items(session: str = Depends(cookie_scheme)):
    return {"session": session}
```

## 安全策略

我们非常重视安全。我们鼓励您及时更新 FastAPI 版本，以受益于最新的功能、错误修复和安全修复。

### 报告漏洞

如果您认为自己发现了安全漏洞，请通过发送电子邮件至 **security@tiangolo.com** 私下报告。请提供尽可能详细的信息，包括重现问题的步骤。在找到解决方案之前，请不要公开讨论潜在的漏洞。

---

借助这些工具，您可以在 FastAPI 应用程序中实现强大且标准的安全实践。有关安全实用程序的更多详细信息，您可以查阅 [API 参考](./api-reference-security.md)。要了解在请求到达您的路径操作之前如何处理请求，请参阅下一章关于 [中间件](./advanced-middleware.md) 的内容。
