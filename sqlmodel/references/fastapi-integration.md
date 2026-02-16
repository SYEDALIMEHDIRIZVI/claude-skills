# SQLModel + FastAPI Integration — Complete Guide

## Why They Work Together

SQLModel and FastAPI are built by the same author and both use Pydantic under the hood. A SQLModel class with `table=True` works as both a database model AND an API schema.

---

## Session Dependency

The recommended pattern: create a session per request using FastAPI's dependency injection.

```python
from typing import Annotated
from fastapi import Depends, FastAPI, HTTPException, Query
from sqlmodel import Field, Session, SQLModel, create_engine, select

sqlite_url = "sqlite:///database.db"
engine = create_engine(sqlite_url)

def create_db_and_tables():
    SQLModel.metadata.create_all(engine)

app = FastAPI()

@app.on_event("startup")  # or use lifespan
def on_startup():
    create_db_and_tables()

def get_session():
    with Session(engine) as session:
        yield session

# Reusable type alias
SessionDep = Annotated[Session, Depends(get_session)]
```

Every endpoint receives a `session` parameter managed by FastAPI's lifecycle.

---

## Multiple Models Pattern

The most important pattern for production FastAPI + SQLModel apps. Separate concerns into distinct models using inheritance.

### The Problem
Using a single model for everything means:
- Clients can send an `id` (they shouldn't — DB generates it)
- Responses might expose `secret_name` or `hashed_password`
- Update endpoints can't distinguish "not provided" from "set to None"

### The Solution: Model Hierarchy

```python
# 1. BASE — shared fields (no table, no id)
class HeroBase(SQLModel):
    name: str = Field(index=True)
    secret_name: str
    age: int | None = Field(default=None, index=True)

# 2. TABLE — database model (has id, has table)
class Hero(HeroBase, table=True):
    id: int | None = Field(default=None, primary_key=True)

# 3. CREATE — what the client sends (inherits base, no id)
class HeroCreate(HeroBase):
    pass

# 4. PUBLIC — what the API returns (id is required)
class HeroPublic(HeroBase):
    id: int

# 5. UPDATE — partial updates (all fields optional)
class HeroUpdate(SQLModel):
    name: str | None = None
    secret_name: str | None = None
    age: int | None = None
```

### Why Each Model Exists

| Model | Purpose | Has `id`? | Has `table=True`? |
|-------|---------|-----------|-------------------|
| `HeroBase` | Shared field definitions | No | No |
| `Hero` | Database operations | Yes (optional) | Yes |
| `HeroCreate` | API input validation | No | No |
| `HeroPublic` | API response shape | Yes (required) | No |
| `HeroUpdate` | Partial update input | No | No |

---

## Full CRUD Endpoints

```python
from fastapi import Depends, FastAPI, HTTPException, Query
from sqlmodel import Session, select

# CREATE
@app.post("/heroes/", response_model=HeroPublic)
def create_hero(session: SessionDep, hero: HeroCreate):
    db_hero = Hero.model_validate(hero)
    session.add(db_hero)
    session.commit()
    session.refresh(db_hero)
    return db_hero

# READ ALL (with pagination)
@app.get("/heroes/", response_model=list[HeroPublic])
def read_heroes(
    session: SessionDep,
    offset: int = 0,
    limit: int = Query(default=100, le=100),
):
    heroes = session.exec(select(Hero).offset(offset).limit(limit)).all()
    return heroes

# READ ONE
@app.get("/heroes/{hero_id}", response_model=HeroPublic)
def read_hero(session: SessionDep, hero_id: int):
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    return hero

# UPDATE (partial)
@app.patch("/heroes/{hero_id}", response_model=HeroPublic)
def update_hero(session: SessionDep, hero_id: int, hero: HeroUpdate):
    db_hero = session.get(Hero, hero_id)
    if not db_hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    hero_data = hero.model_dump(exclude_unset=True)
    db_hero.sqlmodel_update(hero_data)
    session.add(db_hero)
    session.commit()
    session.refresh(db_hero)
    return db_hero

# DELETE
@app.delete("/heroes/{hero_id}")
def delete_hero(session: SessionDep, hero_id: int):
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    session.delete(hero)
    session.commit()
    return {"ok": True}
```

### Key Patterns

- `Hero.model_validate(hero_create)` — converts a Create schema to a table model
- `hero.model_dump(exclude_unset=True)` — only includes fields the client actually sent (for PATCH)
- `db_hero.sqlmodel_update(data_dict)` — applies partial updates from a dict
- `session.get(Hero, id)` — fast primary key lookup (returns `None` if not found)
- `response_model=HeroPublic` — controls what fields appear in the API response

---

## Relationships in API Responses

To include related data in responses, create response models with nested objects:

```python
class HeroPublicWithTeam(HeroPublic):
    team: TeamPublic | None = None

@app.get("/heroes/{hero_id}", response_model=HeroPublicWithTeam)
def read_hero_with_team(session: SessionDep, hero_id: int):
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    return hero
```

---

## Testing with FastAPI + SQLModel

```python
import pytest
from fastapi.testclient import TestClient
from sqlmodel import Session, SQLModel, create_engine, StaticPool

@pytest.fixture(name="session")
def session_fixture():
    engine = create_engine(
        "sqlite://",  # in-memory
        connect_args={"check_same_thread": False},
        poolclass=StaticPool,
    )
    SQLModel.metadata.create_all(engine)
    with Session(engine) as session:
        yield session

@pytest.fixture(name="client")
def client_fixture(session: Session):
    def get_session_override():
        return session

    app.dependency_overrides[get_session] = get_session_override
    client = TestClient(app)
    yield client
    app.dependency_overrides.clear()

def test_create_hero(client: TestClient):
    response = client.post("/heroes/", json={"name": "Spider-Boy", "secret_name": "Pedro"})
    assert response.status_code == 200
    data = response.json()
    assert data["name"] == "Spider-Boy"
    assert "id" in data
```

### Testing Key Points
- Use `sqlite://` (no file) for in-memory test databases
- Use `StaticPool` so all connections share the same in-memory DB
- Override `get_session` dependency to inject the test session
- Each test gets a fresh database (tables created in fixture)
- Clean up overrides with `app.dependency_overrides.clear()`
