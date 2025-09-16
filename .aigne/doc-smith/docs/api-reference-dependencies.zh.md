# 依赖项

FastAPI 包含一个功能强大且易于使用的依赖注入系统。它允许你以结构化和可重用的方式管理依赖项（例如数据库会话、安全要求或共享逻辑）。FastAPI 会负责调用你的依赖项，并将其结果注入你的*路径操作函数*中。

该系统可以帮助你：
-   共享逻辑和代码。
-   共享数据库连接。
-   实现安全方案（认证和授权）。
-   以及更多……

本页是 `Depends` 和 `Security` 函数的技术参考。如需分步指南，请参阅[依赖项教程](./tutorials-dependencies-and-security.md)。

## Depends

The `Depends` 函数是在应用程序中声明依赖项的主要工具。你为其提供一个可调用对象（如函数、类等），FastAPI 将执行该对象，解析任何子依赖项，并将返回的值注入到你的函数参数中。

### 参数

<x-field data-name="dependency" data-type="Optional[Callable[..., Any]]" data-default="None" data-required="false" data-desc="一个“可依赖”的可调用对象，例如一个函数。你应该传递函数对象本身，而不是调用后的结果。FastAPI 会为你执行它。"></x-field>
<x-field data-name="use_cache" data-type="bool" data-default="true" data-required="false" data-desc="如果为 `True`（默认值），依赖项的结果会在单个请求的生命周期内被缓存。如果在同一次请求中多次需要同一个依赖项，它将只被执行一次，并复用缓存的值。"></x-field>

### 示例

下面是创建一个提供通用查询参数的依赖项的方法。该依赖项可以在多个*路径操作*中复用。

```python title="dependency_example.py" icon=logos:python
from typing import Annotated, Union

from fastapi import Depends, FastAPI

app = FastAPI()


async def common_parameters(q: Union[str, None] = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: Annotated[dict, Depends(common_parameters)]):
    return commons


@app.get("/users/")
async def read_users(commons: Annotated[dict, Depends(common_parameters)]):
    return commons
```

在此示例中，`common_parameters` 是一个依赖项。当请求访问 `/items/` 或 `/users/` 时，FastAPI 将：
1.  调用 `common_parameters` 函数。
2.  从请求的查询字符串中提取 `q`、`skip` 和 `limit`。
3.  将它们作为参数传递给 `common_parameters`。
4.  获取 `common_parameters` 返回的字典。
5.  将该字典注入到 `read_items` 或 `read_users` 的 `commons` 参数中。

## Security

`Security` 函数是 `Depends` 的一个特殊子类。它的工作方式完全相同，但增加了定义安全范围的功能，这些范围随后会集成到你的 OpenAPI 模式和交互式 API 文档中（例如，在 `/docs` 路径下）。

这对于记录认证和授权要求（尤其是在使用 OAuth2 等方案时）特别有用。

### 参数

<x-field data-name="dependency" data-type="Optional[Callable[..., Any]]" data-default="None" data-required="false" data-desc="一个实现安全方案的“可依赖”可调用对象，例如，通过验证令牌或 API 密钥并返回当前用户。"></x-field>
<x-field data-name="scopes" data-type="Optional[Sequence[str]]" data-default="None" data-required="false" data-desc="一个字符串序列（列表或元组），表示此特定依赖项所需的 OAuth2 范围。该信息将用于 OpenAPI 模式。"></x-field>
<x-field data-name="use_cache" data-type="bool" data-default="true" data-required="false" data-desc="在单个请求内缓存安全依赖项的结果。默认为 `True`。"></x-field>

### 示例

在此示例中，`Security` 用于保护一个端点。它依赖于 `get_current_active_user` 这个依赖项，并指定需要 `items` 范围。

```python title="security_dependency_example.py" icon=logos:python
from typing import Annotated, Union

from pydantic import BaseModel
from fastapi import Depends, FastAPI, Security
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

class User(BaseModel):
    username: str
    email: Union[str, None] = None

# This is a simplified example. In a real application,
# you would decode and validate the token properly.
def get_current_active_user(token: Annotated[str, Depends(oauth2_scheme)]):
    # In a real app, you would decode the token and get the user from the database
    user = User(username=token + "_faked", email="user@example.com")
    return user

@app.get("/users/me/items/")
async def read_own_items(
    current_user: Annotated[User, Security(get_current_active_user, scopes=["items"])],
):
    return [{"item_id": "Foo", "owner": current_user.username}]

```

`Security(get_current_active_user, scopes=["items"])` 的使用会告诉 FastAPI：
1.  将 `get_current_active_user` 视为一个依赖项。
2.  将其结果注入 `current_user` 参数。
3.  在 OpenAPI 文档中，将此端点标记为受 `get_current_active_user` 中定义的方案（该方案本身依赖于 `oauth2_scheme`）保护，并要求具备 `items` 范围。

如需更深入地了解安全实用程序，请参阅[安全 API 参考](./api-reference-security.md)。