# SQL 数据库

FastAPI 可以与任何数据库以及任何风格的数据库交互库配合使用。一种常见的模式是使用 ORM (Object-Relational Mapper)。ORM 拥有在代码中的对象和数据库表（关系）之间进行转换的工具。

本教程将指导你如何使用 **SQLModel** 将 FastAPI 应用程序连接到 SQL 数据库。SQLModel 是一个旨在成为 SQL 数据库领域的 Pydantic 的库。它结合了 **SQLAlchemy** 和 **Pydantic** 的特性，提供了一种简单、直观且强大的方式来与数据库进行交互。

我们将构建一个完整的 CRUD (Create, Read, Update, Delete) API 来管理一个英雄数据库。

## 第一个简单的 API

让我们从创建一个与 SQL 数据库交互的简单但完整的 FastAPI 应用程序开始。这将涵盖基本的 CRUD 操作。

### 定义 SQLModel 表模型

首先，我们定义数据模型。这个类代表我们数据库中的 `hero` 表。通过 SQLModel，这一个类既可以作为我们用于数据验证的 Pydantic 模型，也可以作为我们用于数据库交互的 SQLAlchemy 模型。

```python The Database Model icon=logos:python
from typing import Union

from sqlmodel import Field, SQLModel


class Hero(SQLModel, table=True):
    id: Union[int, None] = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    age: Union[int, None] = Field(default=None, index=True)
    secret_name: str
```

下面是具体说明：
- `SQLModel`：我们模型的基类。
- `table=True`：这告诉 SQLModel 这个类对应一个数据库表。
- `Field`：用于为每个字段提供额外的配置，例如定义 `primary_key` 和创建数据库 `index` 以加快查询速度。

### 创建引擎和会话依赖

为了与数据库通信，我们需要一个 SQLAlchemy 引擎。本例中我们将使用 SQLite，因为它是一个简单的基于文件的数据库，不需要单独的服务器。

我们还创建了一个依赖 `get_session` 来管理数据库会话。这将确保每个请求都有自己的会话，并在请求完成后关闭。

```python Database Setup icon=logos:python
from sqlmodel import Session, create_engine

sqlite_file_name = "database.db"
sqlite_url = f"sqlite:///{sqlite_file_name}"

connect_args = {"check_same_thread": False}
engine = create_engine(sqlite_url, connect_args=connect_args)

def get_session():
    with Session(engine) as session:
        yield session
```

### 在启动时创建数据库和表

我们使用 FastAPI 的启动事件来创建数据库和 `hero` 表（如果它们尚不存在）。这仅在应用程序启动时执行一次。

```python Application Startup Event icon=logos:python
from fastapi import FastAPI

app = FastAPI()


def create_db_and_tables():
    SQLModel.metadata.create_all(engine)


@app.on_event("startup")
def on_startup():
    create_db_and_tables()
```

### 创建、读取和删除操作

现在我们可以为我们的 CRUD API 定义路径操作。

- **创建英雄 (`POST /heroes/`)**：接受一个 `Hero` 对象，将其添加到会话中，提交到数据库，并刷新该对象以获取新创建的 ID。
- **读取英雄 (`GET /heroes/`)**：从数据库中获取英雄列表，支持可选的分页功能（`offset` 和 `limit`）。
- **读取英雄 (`GET /heroes/{hero_id}`)**：通过 ID 获取单个英雄。
- **删除英雄 (`DELETE /heroes/{hero_id}`)**：通过 ID 删除一个英雄。

### 第一个完整示例

以下是我们初始应用程序的完整代码。你可以将其保存为一个文件，并使用 Uvicorn 运行。

```python tutorial001.py icon=logos:python
from typing import List, Union

from fastapi import Depends, FastAPI, HTTPException, Query
from sqlmodel import Field, Session, SQLModel, create_engine, select


class Hero(SQLModel, table=True):
    id: Union[int, None] = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    age: Union[int, None] = Field(default=None, index=True)
    secret_name: str


sqlite_file_name = "database.db"
sqlite_url = f"sqlite:///{sqlite_file_name}"

connect_args = {"check_same_thread": False}
engine = create_engine(sqlite_url, connect_args=connect_args)


def create_db_and_tables():
    SQLModel.metadata.create_all(engine)


def get_session():
    with Session(engine) as session:
        yield session


app = FastAPI()


@app.on_event("startup")
def on_startup():
    create_db_and_tables()


@app.post("/heroes/")
def create_hero(hero: Hero, session: Session = Depends(get_session)) -> Hero:
    session.add(hero)
    session.commit()
    session.refresh(hero)
    return hero


@app.get("/heroes/")
def read_heroes(
    session: Session = Depends(get_session),
    offset: int = 0,
    limit: int = Query(default=100, le=100),
) -> List[Hero]:
    heroes = session.exec(select(Hero).offset(offset).limit(limit)).all()
    return heroes


@app.get("/heroes/{hero_id}")
def read_hero(hero_id: int, session: Session = Depends(get_session)) -> Hero:
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    return hero


@app.delete("/heroes/{hero_id}")
def delete_hero(hero_id: int, session: Session = Depends(get_session)):
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    session.delete(hero)
    session.commit()
    return {"ok": True}

```

这可以正常工作，但存在一个问题：`secret_name` 字段在所有 API 响应中都被暴露了。任何能读取英雄信息的人都可以看到他们的秘密名称。让我们来修复这个问题。

## 为安全和清晰分离模型

为了解决安全问题并使我们的 API 更加健壮，我们将使用多个 Pydantic/SQLModel 类。每个类将代表我们数据在不同用例下的不同“形态”：创建、读取、更新和存储在数据库中。

### 定义多个模型

我们将创建几个相互继承的类，以避免代码重复。

- **`HeroBase`**：包含所有其他模型中都存在的公共字段。

<x-field data-name="name" data-type="string" data-required="true" data-desc="英雄的公开名称。"></x-field>
<x-field data-name="age" data-type="integer" data-required="false" data-desc="英雄的年龄。"></x-field>

- **`Hero`**：数据库模型。它继承自 `HeroBase` 并添加了只应存在于数据库中的字段，如 `id` 和 `secret_name`。

<x-field data-name="id" data-type="integer" data-required="false" data-desc="数据库中的主键。"></x-field>
<x-field data-name="name" data-type="string" data-required="true" data-desc="英雄的公开名称。"></x-field>
<x-field data-name="age" data-type="integer" data-required="false" data-desc="英雄的年龄。"></x-field>
<x-field data-name="secret_name" data-type="string" data-required="true" data-desc="英雄的秘密身份。"></x-field>

- **`HeroPublic`**：用于返回给客户端的数据模型。它排除了 `secret_name`。

<x-field data-name="id" data-type="integer" data-required="true" data-desc="英雄的 ID。"></x-field>
<x-field data-name="name" data-type="string" data-required="true" data-desc="英雄的公开名称。"></x-field>
<x-field data-name="age" data-type="integer" data-required="false" data-desc="英雄的年龄。"></x-field>

- **`HeroCreate`**：用于在创建英雄时从客户端接收数据的模型。客户端必须提供 `secret_name`。

<x-field data-name="name" data-type="string" data-required="true" data-desc="英雄的公开名称。"></x-field>
<x-field data-name="age" data-type="integer" data-required="false" data-desc="英雄的年龄。"></x-field>
<x-field data-name="secret_name" data-type="string" data-required="true" data-desc="英雄的秘密身份。"></x-field>

- **`HeroUpdate`**：用于更新英雄的模型。所有字段都是可选的，因此客户端只需发送他们想要更改的数据。

<x-field data-name="name" data-type="string" data-required="false" data-desc="英雄的新公开名称。"></x-field>
<x-field data-name="age" data-type="integer" data-required="false" data-desc="英雄的新年龄。"></x-field>
<x-field data-name="secret_name" data-type="string" data-required="false" data-desc="英雄的新秘密身份。"></x-field>

### 添加更新操作

有了新的 `HeroUpdate` 模型，我们现在可以添加一个 `PATCH` 端点来更新英雄。该端点将只更新客户端提供的字段。

```python Update Hero Endpoint icon=logos:python
@app.patch("/heroes/{hero_id}", response_model=HeroPublic)
def update_hero(
    hero_id: int, hero: HeroUpdate, session: Session = Depends(get_session)
):
    hero_db = session.get(Hero, hero_id)
    if not hero_db:
        raise HTTPException(status_code=404, detail="Hero not found")
    hero_data = hero.model_dump(exclude_unset=True)
    hero_db.sqlmodel_update(hero_data)
    session.add(hero_db)
    session.commit()
    session.refresh(hero_db)
    return hero_db
```

关键点：
- `hero.model_dump(exclude_unset=True)`：这将创建一个只包含客户端实际发送的数据的字典。
- `hero_db.sqlmodel_update(hero_data)`：这个来自 SQLModel 的新方法使用 `hero_data` 字典中的数据来更新数据库对象 `hero_db`。

### 完整的改进示例

以下是最终的、更安全、功能更完整的应用程序代码。

```python tutorial002.py icon=logos:python
from typing import List, Union

from fastapi import Depends, FastAPI, HTTPException, Query
from sqlmodel import Field, Session, SQLModel, create_engine, select


class HeroBase(SQLModel):
    name: str = Field(index=True)
    age: Union[int, None] = Field(default=None, index=True)


class Hero(HeroBase, table=True):
    id: Union[int, None] = Field(default=None, primary_key=True)
    secret_name: str


class HeroPublic(HeroBase):
    id: int


class HeroCreate(HeroBase):
    secret_name: str


class HeroUpdate(HeroBase):
    name: Union[str, None] = None
    age: Union[int, None] = None
    secret_name: Union[str, None] = None


sqlite_file_name = "database.db"
sqlite_url = f"sqlite:///{sqlite_file_name}"

connect_args = {"check_same_thread": False}
engine = create_engine(sqlite_url, connect_args=connect_args)


def create_db_and_tables():
    SQLModel.metadata.create_all(engine)


def get_session():
    with Session(engine) as session:
        yield session


app = FastAPI()


@app.on_event("startup")
def on_startup():
    create_db_and_tables()


@app.post("/heroes/", response_model=HeroPublic)
def create_hero(hero: HeroCreate, session: Session = Depends(get_session)):
    db_hero = Hero.model_validate(hero)
    session.add(db_hero)
    session.commit()
    session.refresh(db_hero)
    return db_hero


@app.get("/heroes/", response_model=List[HeroPublic])
def read_heroes(
    session: Session = Depends(get_session),
    offset: int = 0,
    limit: int = Query(default=100, le=100),
):
    heroes = session.exec(select(Hero).offset(offset).limit(limit)).all()
    return heroes


@app.get("/heroes/{hero_id}", response_model=HeroPublic)
def read_hero(hero_id: int, session: Session = Depends(get_session)):
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    return hero


@app.patch("/heroes/{hero_id}", response_model=HeroPublic)
def update_hero(
    hero_id: int, hero: HeroUpdate, session: Session = Depends(get_session)
):
    hero_db = session.get(Hero, hero_id)
    if not hero_db:
        raise HTTPException(status_code=404, detail="Hero not found")
    hero_data = hero.model_dump(exclude_unset=True)
    hero_db.sqlmodel_update(hero_data)
    session.add(hero_db)
    session.commit()
    session.refresh(hero_db)
    return hero_db


@app.delete("/heroes/{hero_id}")
def delete_hero(hero_id: int, session: Session = Depends(get_session)):
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    session.delete(hero)
    session.commit()
    return {"ok": True}

```

通过为输入、输出和数据库存储使用独立的模型，我们创建了一个更安全、更灵活、更易于维护的 API。

## 后续步骤

既然你已经有了一个带数据库的、可以工作的 API，那么合乎逻辑的下一步就是学习如何为它编写测试。你可以在 [测试](./tutorials-advanced-testing.md) 部分了解更多信息。