# 依赖项与安全

FastAPI 包含一个功能强大且直观的依赖注入系统。该系统可以声明应用程序运行所需的内容（如数据库会话、身份验证凭据或配置设置），并由 FastAPI 负责将其提供给*路径操作*。

该系统不仅有助于组织代码和共享逻辑，还是实现稳健身份验证和授权的基础。在本指南中，我们将探讨如何创建依赖项，并使用它们来保护 API 端点。

## 依赖注入系统

其核心，“依赖项”通常是一个函数（或任何其他可调用对象），FastAPI 会在调用*路径操作函数*之前调用它。然后，依赖项函数的返回值会作为参数传递给*路径操作函数*。

### 创建共享依赖项

我们从一个简单的示例开始。假设你有 `q`、`skip` 和 `limit` 等通用查询参数，且多个端点都会使用它们。你可以在依赖项中一次性定义这些参数，而无需在每个函数签名中重复定义。

```python 创建一个依赖函数 icon=logos:python
from typing import Union

from fastapi import Depends, FastAPI

app = FastAPI()


async def common_parameters(
    q: Union[str, None] = None, skip: int = 0, limit: int = 100
):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons


@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return commons
```

在这段代码中：
1.  我们定义了一个函数 `common_parameters`，它接收我们想要的查询参数。
2.  在我们的*路径操作函数*（`read_items` 和 `read_users`）中，我们声明了一个参数 `commons`。
3.  我们将其默认值设置为 `Depends(common_parameters)`。

FastAPI 看到 `Depends()` 后，便知道必须先调用 `common_parameters`。然后，它会获取 `common_parameters` 返回的字典，并将其作为 `commons` 参数传递给 `read_items` 或 `read_users`。这样可以保持路径操作逻辑的整洁，并避免代码重复。

### 类作为依赖项

你也可以使用类作为依赖项。FastAPI 会将类本身视为一个“可调用对象”，并创建它的一个实例。

这对于分组相关依赖项或当依赖项需要维护某些状态时非常有用。

```python 使用类作为依赖项 icon=logos:python
from typing import Union

from fastapi import Depends, FastAPI

app = FastAPI()


fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


class CommonQueryParams:
    def __init__(self, q: Union[str, None] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit


@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends(CommonQueryParams)):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

在这里，FastAPI 会创建一个 `CommonQueryParams` 的实例，并将查询参数 `q`、`skip` 和 `limit` 传递给 `__init__` 方法。然后，该实例会作为 `commons` 参数传递给 `read_items`。

一个常见的快捷方式是直接使用 `Depends()` 而不传递可调用对象。FastAPI 非常智能，会使用类型提示（`CommonQueryParams`）作为依赖项。

```python 类依赖项的快捷方式 icon=logos:python
@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends()):
    # ... same logic as before
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

## 身份验证和授权

现在，我们使用依赖注入系统来保护我们的 API。我们将介绍两种常用方法：HTTP 基本认证和带持有者令牌（Bearer Tokens）的 OAuth2。

### HTTP 基本认证

HTTP 基本认证是 HTTP 协议内置的一种简单认证方案。客户端在 `Authorization` 请求头中发送用户名和密码。

FastAPI 提供了安全工具，使这一过程变得简单。

```python HTTP 基本认证示例 icon=logos:python
from fastapi import Depends, FastAPI
from fastapi.security import HTTPBasic, HTTPBasicCredentials

app = FastAPI()

security = HTTPBasic()


@app.get("/users/me")
def read_current_user(credentials: HTTPBasicCredentials = Depends(security)):
    return {"username": credentials.username, "password": credentials.password}
```

在这里，`HTTPBasic()` 是一个可调用类，它会从请求中提取用户名和密码。通过使用 `Depends(security)`，我们告诉 FastAPI 运行此检查。如果 `Authorization` 请求头缺失或格式不正确，FastAPI 将自动返回 401 未授权错误。

### 使用密码和持有者令牌的 OAuth2

OAuth2 是一个更稳健、更灵活的授权框架。一个常见的流程是“密码流”，即用户用用户名和密码换取一个临时访问令牌。该令牌随后会作为“持有者令牌（Bearer Token）”随未来的请求一起发送。

以下是 JWT 身份验证流程图：

```d2 JWT 身份验证流程
direction: down
shape: sequence_diagram

User -> FastAPI-App: "1. 使用用户名和密码 POST /token"

FastAPI-App -> DB: "2. 验证用户"
DB -> FastAPI-App: "3. 用户记录"

FastAPI-App -> FastAPI-App: "4. 创建 JWT"

FastAPI-App -> User: "5. 返回 JWT 访问令牌"

User -> Protected-Endpoint: "6. 使用 'Authorization: Bearer <token>' 发起请求"

Protected-Endpoint -> Protected-Endpoint: "7. 依赖项验证 JWT"

Protected-Endpoint -> DB: "8. 从数据库获取当前用户"

Protected-Endpoint -> User: "9. 返回响应"

```

让我们一步步来构建它。

#### 第 1 步：基本安全方案

首先，我们使用 `OAuth2PasswordBearer` 定义我们的安全方案。我们必须告诉它客户端获取令牌的 URL。

```python 基本 OAuth2 设置 icon=logos:python
from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


@app.get("/items/")
async def read_items(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

这个简单的依赖项 `Depends(oauth2_scheme)` 会检查 `Authorization` 请求头，确认其包含 `Bearer` 和一个令牌，并以 `str` 形式返回该令牌。否则，它会引发 401 错误。

#### 第 2 步：获取当前用户

仅仅拥有令牌字符串作用不大。我们需要一个能够解码令牌并获取相应用户的依赖项。

```python 获取当前用户的依赖项 icon=logos:python
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

现在，我们的路径操作依赖于 `get_current_user`。这个新的依赖项又依赖于 `oauth2_scheme`。FastAPI 会自动处理这个依赖链。端点会得到一个经过完整验证的 `User` 对象。

#### 第 3 步：真实的 JWT 和密码哈希

到目前为止，我们一直使用伪造的令牌处理方式。现在，我们来使用 JSON Web Tokens (JWT) 和 `passlib` 实现真正的密码哈希。

这个示例更加完整，包含一个用于颁发令牌的 `/token` 端点。

```python 完整的 JWT 身份验证 icon=logos:python
from datetime import datetime, timedelta, timezone
from typing import Union

import jwt
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jwt.exceptions import InvalidTokenError
from passlib.context import CryptContext
from pydantic import BaseModel

# Configuration
SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Password Hashing
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Security Scheme
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

app = FastAPI()

# --- Pydantic Models --- (User, Token, TokenData, etc.)

# --- Database and Utility Functions --- (fake_users_db, get_user, verify_password, etc.)

# --- Dependency to get current user ---
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

async def get_current_active_user(current_user: User = Depends(get_current_user)):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

# --- Token Endpoint ---
@app.post("/token")
async def login_for_access_token(
    form_data: OAuth2PasswordRequestForm = Depends(),
) -> Token:
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
    return Token(access_token=access_token, token_type="bearer")

# --- Protected Endpoints ---
@app.get("/users/me/", response_model=User)
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    return current_user
```

这是一个完整、安全的实现。`/token` 端点接收表单数据（`username`、`password`），对其进行验证，并返回一个签名的 JWT。`get_current_user` 依赖项在允许访问受保护端点之前，会验证 JWT 的签名和过期时间。

#### 第 4 步：使用范围进行授权

身份验证确认用户是*谁*。授权决定他们被允许*做什么*。OAuth2 使用“范围（scopes）”来实现这一目的。

我们可以在 `OAuth2PasswordBearer` 方案中定义可用的范围，然后使用 `Security()` 为特定端点要求指定的范围。

```python 使用范围的 OAuth2 icon=logos:python
from fastapi import Depends, FastAPI, Security
from fastapi.security import OAuth2PasswordBearer, SecurityScopes
# ... other imports and setup from the previous step

# Update the security scheme with scopes
oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={"me": "Read information about the current user.", "items": "Read items."},
)

# Update the user dependency to check scopes
async def get_current_user(
    security_scopes: SecurityScopes, token: str = Depends(oauth2_scheme)
):
    if security_scopes.scopes:
        authenticate_value = f'Bearer scope="{security_scopes.scope_str}"'
    else:
        authenticate_value = "Bearer"
    # ... (JWT decoding logic as before) ...
    # After getting the user and token scopes:
    for scope in security_scopes.scopes:
        if scope not in token_data.scopes:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Not enough permissions",
                headers={"WWW-Authenticate": authenticate_value},
            )
    return user

# A dependency for an active user (no specific scopes required yet)
async def get_current_active_user(
    current_user: User = Security(get_current_user, scopes=["me"]),
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user


# Require the "items" scope for this endpoint
@app.get("/users/me/items/")
async def read_own_items(
    current_user: User = Security(get_current_active_user, scopes=["items"]),
):
    return [{"item_id": "Foo", "owner": current_user.username}]


# This endpoint has no specific scope requirements
@app.get("/status/")
async def read_system_status(current_user: User = Depends(get_current_user)):
    return {"status": "ok"}
```

主要变更：
*   `SecurityScopes` 作为参数添加到了依赖项中。FastAPI 会用端点所需的范围来填充它。
*   我们使用 `Security()` 而不是 `Depends()`。它的工作方式相同，但允许我们传递一个 `scopes` 列表。
*   依赖项会检查 JWT 中存在的范围是否足以满足端点所需的范围。

## 总结

FastAPI 的依赖注入系统是一个多功能工具，可用于分离关注点、重用逻辑，以及最重要的一点——构建安全稳健的 API。通过为通用参数、数据库连接和用户身份验证定义依赖项，你可以保持*路径操作*逻辑的整洁、专注且易于测试。

现在，你已经学会了如何创建依赖项，并使用它们来实现从简单的共享参数到带范围的完整 OAuth2 安全流程的各种功能。

[下一步：高级用户指南](./tutorials-advanced.md)
