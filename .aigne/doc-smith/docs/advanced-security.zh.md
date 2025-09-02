# 安全性

实现身份认证和授权是大多数 Web API 的关键组成部分。FastAPI 提供了多种工具，可以轻松地通过 OAuth2、HTTP Basic/Bearer/Digest Auth 和 API 密钥等标准协议来处理安全性。

本指南将通过实际的分步示例，引导你完成这些安全方案的实现。

## 认证流程概述

大多数基于令牌的安全流程都遵循类似的模式：客户端使用凭据（如用户名和密码）进行一次性认证以获取令牌，然后在所有后续请求中使用该令牌。

下面是一个说明常见 OAuth2 登录和请求流程的图表：

```d2
shape: sequence_diagram

Client: 客户端应用程序
API: FastAPI 服务器

Client->API: 1. 使用用户名和密码请求 /token
API->API: 2. 验证凭据并生成 JWT 令牌
API->Client: 3. 返回访问令牌

Client->API: 4. 使用 'Authorization: Bearer <token>' 请求 /users/me
API->API: 5. 解码并验证 JWT 令牌，识别用户
API->Client: 6. 返回用户数据
```

## 使用密码和不记名令牌的 OAuth2

OAuth2 是一种用于授权的标准协议。“密码流”是一种常见的模式，用户通过它直接向你的应用程序提供凭据，然后应用程序用这些凭据换取不记名令牌。

### 第一步

首先，你需要一个 `OAuth2PasswordBearer` 的实例。这个类是一个依赖项，它提供一个 `tokenUrl`。客户端将向此 URL 发送用户名和密码以获取令牌。

```python
# 来源：docs_src/security/tutorial001.py
from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


@app.get("/items/")
async def read_items(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

在此示例中，`Depends(oauth2_scheme)` 声明 `/items/` 端点依赖于 OAuth2 方案。FastAPI 会知道它需要查找带有不记名令牌的 `Authorization` 请求头，并将该令牌作为 `token` 参数提供给你的函数。

如果你打开位于 `/docs` 的交互式文档，你会看到一个“Authorize”按钮。点击它后，会弹出一个窗口让你输入用户名和密码。但是，我们还没有创建 `/token` 端点，所以这还无法正常工作。这里的关键在于，该依赖项已经与你的 API 文档集成在一起了。

### 获取当前用户

仅仅获取令牌字符串并没有太大用处。你需要使用该令牌来识别发出请求的用户。我们可以创建一个依赖项 `get_current_user` 来处理此事。

该依赖项将：
1.  从 `oauth2_scheme` 依赖项中获取令牌。
2.  解码令牌以获取用户信息。
3.  返回用户对象。

```python
# 来源：docs_src/security/tutorial002.py
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

现在，`read_users_me` 路径操作依赖于 `get_current_user`。当请求到达时，FastAPI 会调用 `get_current_user`，而后者又会从 `oauth2_scheme` 获取令牌。然后，用户信息就可以通过 `current_user` 参数获取。

### 密码流和令牌端点

现在我们来实现 `/token` 端点。该端点将以表单数据的形式接收用户名和密码，进行验证，然后返回一个访问令牌。

FastAPI 提供了 `OAuth2PasswordRequestForm` 依赖项来处理接收表单数据。

我们还添加了一个依赖项 `get_current_active_user`，它建立在 `get_current_user` 的基础上，用于检查用户是否处于活动状态。

```python
# 来源：docs_src/security/tutorial003.py
from typing import Union

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel

# --- 这是一个简化示例。为简洁起见，模型和伪数据库未完全显示 --- 
# 有关 fake_users_db、User、UserInDB 等的详细信息，请参阅完整的源文件。

fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "fakehashedsecret",
        "disabled": False,
    },
}

class User(BaseModel):
    username: str
    email: Union[str, None] = None
    full_name: Union[str, None] = None
    disabled: Union[bool, None] = None

class UserInDB(User):
    hashed_password: str

def fake_hash_password(password: str):
    return "fakehashed" + password

app = FastAPI()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

# ... (get_user 和 fake_decode_token 函数) ...

async def get_current_user(token: str = Depends(oauth2_scheme)):
    # 在实际应用中，这里会解码令牌并获取用户
    user_dict = fake_users_db.get(token)
    if not user_dict:
         raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
    return UserInDB(**user_dict)

async def get_current_active_user(current_user: User = Depends(get_current_user)):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    user = UserInDB(**user_dict)
    hashed_password = fake_hash_password(form_data.password)
    if not hashed_password == user.hashed_password:
        raise HTTPException(status_code=400, detail="Incorrect username or password")

    return {"access_token": user.username, "token_type": "bearer"}


@app.get("/users/me")
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    return current_user
```

`/token` 端点会验证用户名和密码。如果正确，它会返回一个包含 `access_token` 和 `token_type` 的 JSON 对象。在此示例中，我们仅使用用户名作为令牌，这并不安全。下一步将解决此问题。

### 使用 JWT 作为令牌

我们不应该直接返回用户名作为令牌，而是应该使用像 JSON Web Tokens (JWT) 这样的标准。JWT 包含经过签名的数据，其完整性可以被验证。

这包括：
1.  **密码哈希**：使用像 `passlib` 这样的库来安全地哈希和验证密码。
2.  **JWT 创建**：创建一个包含用户标识符（`sub` 声明）和过期时间（`exp`）的 JWT。
3.  **JWT 解码**：在 `get_current_user` 依赖项中，从 `Authorization` 请求头解码 JWT 以获取用户身份。

```python
# 来源：docs_src/security/tutorial004.py
from datetime import datetime, timedelta, timezone
from typing import Union

import jwt
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jwt.exceptions import InvalidTokenError
from passlib.context import CryptContext
from pydantic import BaseModel

# --- 常量 ---
SECRET_KEY = "一个非常机密的密钥，应放在环境变量中"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# --- Pydantic 模型 ---
class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Union[str, None] = None

# ... (User 和 UserInDB 模型) ...

# --- 安全工具 ---
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
app = FastAPI()

def create_access_token(data: dict, expires_delta: Union[timedelta, None] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

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
    # 在实际应用中，你会从数据库中获取用户
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user

# --- 路径操作 ---
@app.post("/token")
async def login_for_access_token(form_data: OAuth2PasswordRequestForm = Depends()) -> Token:
    # 在实际应用中，authenticate_user 会将密码与数据库中哈希过的密码进行比对
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

# ... (其他端点：/users/me/、/users/me/items/) ...
```

现在，`/token` 端点会返回一个真正的 JWT，而我们的 `get_current_user` 依赖项可以安全地验证它并提取用户信息。

### 用于授权的 OAuth2 范围

身份认证用于识别用户，而授权则决定用户可以做什么。OAuth2 使用“范围（scopes）”来实现这一目的。

你可以使用 `Security` 依赖项为路径操作要求特定的范围。

```python
# 来源：docs_src/security/tutorial005.py
from fastapi import Security
from fastapi.security import SecurityScopes

oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={"me": "Read information about the current user.", "items": "Read items."},
)

async def get_current_user(
    security_scopes: SecurityScopes, token: str = Depends(oauth2_scheme)
):
    # ... (JWT 解码同前) ...
    # 现在，检查令牌负载中的范围
    token_data = # ... 解码令牌并获取范围
    for scope in security_scopes.scopes:
        if scope not in token_data.scopes:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Not enough permissions",
                headers={"WWW-Authenticate": f'Bearer scope="{security_scopes.scope_str}"'},
            )
    return user

async def get_current_active_user(
    current_user: User = Security(get_current_user, scopes=["me"]),
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

@app.get("/users/me/items/")
async def read_own_items(
    current_user: User = Security(get_current_active_user, scopes=["items"]),
):
    return [{"item_id": "Foo", "owner": current_user.username}]
```

在此示例中：
-   `OAuth2PasswordBearer` 通过一个包含可用范围的字典进行初始化。
-   `get_current_user` 依赖项现在接受一个 `SecurityScopes` 参数，并检查令牌是否包含所需的范围。
-   路径操作 `/users/me/items/` 通过使用 `Security(get_current_active_user, scopes=["items"])` 来要求 `items` 范围。

## HTTP 基本认证

HTTP 基本认证是一种更简单的方案，其中用户名和密码经过 Base64 编码后直接包含在 `Authorization` 请求头中。虽然它比 OAuth2 简单，但通常安全性较低，因为凭据会随着每个请求发送。

FastAPI 提供了 `HTTPBasic` 安全方案。

```python
# 来源：docs_src/security/tutorial007.py
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

在这里，`get_current_username` 依赖项使用 `HTTPBasic` 来提取凭据。为了防止时序攻击，使用 `secrets.compare_digest` 来比较凭据非常重要。

## API 密钥

API 密钥是验证服务器间请求或识别项目的一种常用方式。FastAPI 支持从请求头、查询参数或 cookie 中接收 API 密钥。

<x-cards data-columns="3">
  <x-card data-title="请求头中的 API 密钥" data-icon="lucide:arrow-right-from-line">
    使用 APIKeyHeader 类从自定义请求头中提取密钥。这是一种常用且推荐的方法。
  </x-card>
  <x-card data-title="查询参数中的 API 密钥" data-icon="lucide:at-sign">
    使用 APIKeyQuery 类从 URL 查询参数中提取密钥。这对于简单的脚本或浏览器测试很有用。
  </x-card>
  <x-card data-title="Cookie 中的 API 密钥" data-icon="lucide:cookie">
    使用 APIKeyCookie 类从浏览器 cookie 中提取密钥。这通常用于管理 Web 会话。
  </x-card>
</x-cards>

这些类都可作为依赖项来保护你的端点。你需要创建一个依赖函数，用它来根据数据库或机密列表验证收到的密钥，如此示例中使用 `APIKeyHeader` 一样：

```python
from fastapi import Depends, FastAPI, HTTPException, Security
from fastapi.security import APIKeyHeader

API_KEY = "my-super-secret-api-key"
API_KEY_NAME = "X-API-KEY"

api_key_header_scheme = APIKeyHeader(name=API_KEY_NAME)

app = FastAPI()

async def get_api_key(api_key_header: str = Security(api_key_header_scheme)):
    if api_key_header == API_KEY:
        return api_key_header
    else:
        raise HTTPException(
            status_code=403,
            detail="Could not validate credentials"
        )

@app.get("/items/")
async def read_items(api_key: str = Depends(get_api_key)):
    return [{"item": "Foo"}, {"item": "Bar"}]
```

---

本指南涵盖了 FastAPI 中可用的主要安全工具。现在你可以为你的应用程序实现强大的身份认证和授权。有关特定类及其参数的更多详细信息，请参阅 [安全工具 API 参考](./api-reference-security.md)。

接下来，你可能想学习有关 [中间件](./advanced-middleware.md) 的知识，以便对每个传入的请求执行操作。
