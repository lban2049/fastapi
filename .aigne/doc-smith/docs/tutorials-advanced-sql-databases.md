# SQL Databases

FastAPI can work with any database and any style of library to talk to the database. A common pattern is to use an ORM (Object-Relational Mapper). An ORM has tools to convert between objects in code and database tables (relations).

This tutorial will guide you through connecting your FastAPI application to a SQL database using **SQLModel**, a library designed to be the Pydantic of SQL databases. It combines the features of **SQLAlchemy** and **Pydantic** to provide a simple, intuitive, and powerful way to interact with your database.

We will build a complete CRUD (Create, Read, Update, Delete) API for managing a database of heroes.

## A First Simple API

Let's start by creating a simple but complete FastAPI application that interacts with a SQL database. This will cover the basic CRUD operations.

### Define the SQLModel Table Model

First, we define our data model. This class represents the `hero` table in our database. With SQLModel, this single class serves as both our Pydantic model for data validation and our SQLAlchemy model for database interaction.

```python The Database Model icon=logos:python
from typing import Union

from sqlmodel import Field, SQLModel


class Hero(SQLModel, table=True):
    id: Union[int, None] = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    age: Union[int, None] = Field(default=None, index=True)
    secret_name: str
```

Here's what's happening:
- `SQLModel`: The base class for our model.
- `table=True`: This tells SQLModel that this class corresponds to a database table.
- `Field`: Used to provide extra configuration for each field, such as defining the `primary_key` and creating a database `index` for faster queries.

### Create the Engine and Session Dependency

To communicate with the database, we need a SQLAlchemy engine. We'll use SQLite for this example because it's a simple file-based database that doesn't require a separate server.

We also create a dependency `get_session` to manage database sessions. This will ensure that each request gets its own session, which is closed after the request is finished.

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

### Create the Database and Tables on Startup

We use a FastAPI startup event to create the database and the `hero` table if they don't already exist. This is done only once when the application starts.

```python Application Startup Event icon=logos:python
from fastapi import FastAPI

app = FastAPI()


def create_db_and_tables():
    SQLModel.metadata.create_all(engine)


@app.on_event("startup")
def on_startup():
    create_db_and_tables()
```

### Create, Read, and Delete Operations

Now we can define the path operations for our CRUD API.

- **Create Hero (`POST /heroes/`)**: Accepts a `Hero` object, adds it to the session, commits it to the database, and refreshes the object to get the newly created ID.
- **Read Heroes (`GET /heroes/`)**: Fetches a list of heroes from the database with optional pagination (`offset` and `limit`).
- **Read Hero (`GET /heroes/{hero_id}`)**: Fetches a single hero by their ID.
- **Delete Hero (`DELETE /heroes/{hero_id}`)**: Deletes a hero by their ID.

### Full First Example

Here is the complete code for our initial application. You can save this as a file and run it with Uvicorn.

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

This works, but it has a problem: the `secret_name` field is exposed in all API responses. Anyone who can read a hero can see their secret name. Let's fix this.

## Separating Models for Security and Clarity

To solve the security issue and make our API more robust, we will use multiple Pydantic/SQLModel classes. Each class will represent a different "shape" of our data for different use cases: creating, reading, updating, and storing in the database.

### Define Multiple Models

We'll create several classes that inherit from each other to avoid code duplication.

- **`HeroBase`**: Contains the common fields that are present in all other models.

<x-field data-name="name" data-type="string" data-required="true" data-desc="The hero's public name."></x-field>
<x-field data-name="age" data-type="integer" data-required="false" data-desc="The hero's age."></x-field>

- **`Hero`**: The database model. It inherits from `HeroBase` and adds the fields that should only exist in the database, like `id` and `secret_name`.

<x-field data-name="id" data-type="integer" data-required="false" data-desc="The primary key in the database."></x-field>
<x-field data-name="name" data-type="string" data-required="true" data-desc="The hero's public name."></x-field>
<x-field data-name="age" data-type="integer" data-required="false" data-desc="The hero's age."></x-field>
<x-field data-name="secret_name" data-type="string" data-required="true" data-desc="The hero's secret identity."></x-field>

- **`HeroPublic`**: The model for data sent back to the client. It excludes `secret_name`.

<x-field data-name="id" data-type="integer" data-required="true" data-desc="The hero's ID."></x-field>
<x-field data-name="name" data-type="string" data-required="true" data-desc="The hero's public name."></x-field>
<x-field data-name="age" data-type="integer" data-required="false" data-desc="The hero's age."></x-field>

- **`HeroCreate`**: The model for data received from the client when creating a hero. The client must provide the `secret_name`.

<x-field data-name="name" data-type="string" data-required="true" data-desc="The hero's public name."></x-field>
<x-field data-name="age" data-type="integer" data-required="false" data-desc="The hero's age."></x-field>
<x-field data-name="secret_name" data-type="string" data-required="true" data-desc="The hero's secret identity."></x-field>

- **`HeroUpdate`**: The model for updating a hero. All fields are optional, so the client only needs to send the data they want to change.

<x-field data-name="name" data-type="string" data-required="false" data-desc="The hero's new public name."></x-field>
<x-field data-name="age" data-type="integer" data-required="false" data-desc="The hero's new age."></x-field>
<x-field data-name="secret_name" data-type="string" data-required="false" data-desc="The hero's new secret identity."></x-field>

### Add an Update Operation

With our new `HeroUpdate` model, we can now add a `PATCH` endpoint to update heroes. This endpoint will only update the fields that the client provides.

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

Key points:
- `hero.model_dump(exclude_unset=True)`: This creates a dictionary with only the data that was actually sent by the client.
- `hero_db.sqlmodel_update(hero_data)`: This new method from SQLModel updates the database object `hero_db` with the data from the `hero_data` dictionary.

### Full Improved Example

Here is the final, more secure, and feature-complete application code.

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

By using separate models for input, output, and database storage, we have created a more secure, flexible, and maintainable API.

## Next Steps

Now that you have a working API with a database, the next logical step is to learn how to write tests for it. You can learn more in the [Testing](./tutorials-advanced-testing.md) section.