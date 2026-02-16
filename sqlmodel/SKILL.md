---
name: sqlmodel
description: SQLModel project builder and learning skill. Use when the user wants to build a project with SQLModel, learn SQLModel concepts, define database models, set up relationships, perform CRUD operations, integrate with FastAPI, handle migrations, debug SQLModel issues, or level up from hello world to professional database-driven applications. Covers model definition, sessions, queries, relationships, multiple models pattern, testing, and best practices.
argument-hint: [action] [details]
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, WebSearch, WebFetch, Task
---

# SQLModel Mastery Skill — Hello World to Professional

You are an expert SQLModel developer and mentor. Help the user learn, build, and master SQLModel-powered applications progressively. Adapt depth based on their current level.

## What is SQLModel?

SQLModel is a Python library by the creator of FastAPI. It combines **SQLAlchemy** (database engine) and **Pydantic** (data validation) into one. A single class definition gives you both a database table and a validated data model. Install: `pip install sqlmodel`

## How to Interpret Arguments

- `/sqlmodel new <description>` — Scaffold a new SQLModel project (e.g., `/sqlmodel new inventory system`, `/sqlmodel new blog database`)
- `/sqlmodel learn <topic>` — Teach a concept (e.g., `/sqlmodel learn relationships`, `/sqlmodel learn sessions`)
- `/sqlmodel model <name>` — Create a SQLModel table model (e.g., `/sqlmodel model User`, `/sqlmodel model Product`)
- `/sqlmodel crud <resource>` — Generate full CRUD operations for a resource (e.g., `/sqlmodel crud users`)
- `/sqlmodel relationship <type>` — Set up relationships (e.g., `/sqlmodel relationship one-to-many`, `/sqlmodel relationship many-to-many`)
- `/sqlmodel query <description>` — Help build a query (e.g., `/sqlmodel query find users older than 30`)
- `/sqlmodel fastapi` — Integrate SQLModel with a FastAPI project (session dependency, multiple models)
- `/sqlmodel migrate` — Set up Alembic migrations for the project
- `/sqlmodel test` — Add or run tests for the current project
- `/sqlmodel structure` — Refactor into a professional multi-file layout
- `/sqlmodel debug` — Diagnose SQLModel issues in the current project
- `/sqlmodel review` — Review current SQLModel code for best practices
- `/sqlmodel roadmap` — Show the learning roadmap and assess current level

If no action is given, ask the user what they'd like to do and show available actions.

---

## Learning Roadmap

Guide users through these progressive levels:

### Level 1 — Hello World
- Install: `pip install sqlmodel`
- Define a model with `class Hero(SQLModel, table=True)`
- Create an engine with `create_engine("sqlite:///database.db")`
- Create tables with `SQLModel.metadata.create_all(engine)`
- Insert a row using `Session`, `add()`, `commit()`
- Read it back using `select()`, `session.exec()`
- Understand: model, engine, session, table=True vs plain SQLModel

### Level 2 — Fields & Types
- `Field(primary_key=True)` for primary keys
- Required fields (`name: str`) vs optional (`age: int | None = None`)
- `Field(index=True)` for indexed columns
- `Field(default=...)` and `Field(default_factory=...)`
- Column constraints: `Field(max_length=50)`, `Field(ge=0)`, `Field(unique=True)`
- `Field(foreign_key="table.column")` for references

### Level 3 — CRUD Operations
- **Create**: `session.add(obj)` + `session.commit()`
- **Read**: `select(Model)`, `.where()`, `.exec()`, `.all()`, `.first()`, `.one()`
- **Update**: fetch → modify attributes → `session.add()` → `session.commit()`
- **Delete**: fetch → `session.delete(obj)` → `session.commit()`
- `session.refresh(obj)` to reload from DB (get auto-generated `id`)
- `session.get(Model, id)` for direct primary key lookup

### Level 4 — Filtering & Queries
- Comparison operators: `==`, `!=`, `>`, `<`, `>=`, `<=`
- AND: chain `.where()` or pass multiple args
- OR: `or_(condition1, condition2)`
- `col()` wrapper for type-safe column expressions
- `.offset(n)` and `.limit(n)` for pagination
- `.order_by(Model.field)` for sorting

### Level 5 — Relationships (One-to-Many)
- `Field(foreign_key="table.id")` on the "many" side
- `Relationship(back_populates="field_name")` on both sides
- Parent: `children: list["Child"] = Relationship(back_populates="parent")`
- Child: `parent: Parent | None = Relationship(back_populates="children")`
- Accessing related objects: `hero.team`, `team.heroes`

### Level 6 — Relationships (Many-to-Many)
- Link table model with composite primary key
- `Relationship(back_populates="...", link_model=LinkModel)` on both sides
- Creating and querying linked records
- Understanding the link table pattern

### Level 7 — Multiple Models Pattern (FastAPI Integration)
- `ModelBase(SQLModel)` — shared fields, no `table=True`
- `Model(ModelBase, table=True)` — database table with `id`
- `ModelCreate(ModelBase)` — input schema (no `id`)
- `ModelPublic(ModelBase)` — response schema (`id: int` required)
- `ModelUpdate(SQLModel)` — partial update (all fields optional)
- `Model.model_validate(create_obj)` to convert between models
- Session as FastAPI dependency with `yield`

### Level 8 — Professional Project Structure
- Separate files: models, database, crud/services, schemas
- Engine and session management
- Configuration with environment variables
- Alembic migrations setup
- Testing with in-memory SQLite

### Level 9 — Advanced Patterns
- Cascade deletes with `sa_relationship_kwargs`
- Hybrid properties
- Custom validators
- Async sessions with `create_async_engine`
- Connection pooling
- Raw SQL when needed

### Level 10 — Production & Deployment
- PostgreSQL/MySQL connection strings
- Migration workflows with Alembic
- Database seeding
- Performance: indexes, eager/lazy loading
- Testing strategies with fixtures and rollback

---

## Project Scaffolding

When creating a new project (`/sqlmodel new`), follow this workflow:

### Step 1 — Ask Requirements
1. What does the app do? (brief description)
2. Standalone database layer or with FastAPI?
3. Which database? (SQLite for dev / PostgreSQL for prod)
4. What are the main entities/models?
5. Any relationships between them?

### Step 2 — Create Project Structure

**Simple (learning/standalone):**
```
project-name/
├── main.py
├── models.py
├── database.py
├── requirements.txt
└── .gitignore
```

**Professional (with FastAPI):**
```
project-name/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   ├── models/
│   │   ├── __init__.py
│   │   └── user.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   └── user.py
│   ├── routers/
│   │   ├── __init__.py
│   │   └── users.py
│   └── services/
│       ├── __init__.py
│       └── user.py
├── migrations/
│   ├── env.py
│   └── versions/
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   └── test_users.py
├── alembic.ini
├── requirements.txt
├── .env.example
├── .gitignore
└── Dockerfile
```

### Step 3 — Implement
- Set up virtual environment: `python -m venv venv && source venv/bin/activate`
- Install: `pip install sqlmodel` (add `fastapi[standard]` if using FastAPI)
- Write models first, then database setup, then CRUD operations
- Test each operation after creation

---

## Core Code Patterns

### Model Definition
```python
from sqlmodel import Field, SQLModel

class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    secret_name: str
    age: int | None = Field(default=None, index=True)
```

### Engine & Table Creation
```python
from sqlmodel import create_engine, SQLModel

engine = create_engine("sqlite:///database.db", echo=True)
SQLModel.metadata.create_all(engine)
```

### Session CRUD
```python
from sqlmodel import Session, select

# CREATE
with Session(engine) as session:
    hero = Hero(name="Spider-Boy", secret_name="Pedro Parqueador")
    session.add(hero)
    session.commit()
    session.refresh(hero)  # hero.id is now set

# READ
with Session(engine) as session:
    heroes = session.exec(select(Hero)).all()
    hero = session.exec(select(Hero).where(Hero.name == "Spider-Boy")).first()
    hero = session.get(Hero, 1)  # by primary key

# UPDATE
with Session(engine) as session:
    hero = session.exec(select(Hero).where(Hero.name == "Spider-Boy")).one()
    hero.age = 16
    session.add(hero)
    session.commit()

# DELETE
with Session(engine) as session:
    hero = session.exec(select(Hero).where(Hero.name == "Spider-Boy")).one()
    session.delete(hero)
    session.commit()
```

### One-to-Many Relationship
```python
from sqlmodel import Field, Relationship, SQLModel

class Team(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    headquarters: str
    heroes: list["Hero"] = Relationship(back_populates="team")

class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    secret_name: str
    age: int | None = Field(default=None, index=True)
    team_id: int | None = Field(default=None, foreign_key="team.id")
    team: Team | None = Relationship(back_populates="heroes")
```

### Many-to-Many Relationship
```python
class HeroTeamLink(SQLModel, table=True):
    hero_id: int | None = Field(default=None, foreign_key="hero.id", primary_key=True)
    team_id: int | None = Field(default=None, foreign_key="team.id", primary_key=True)

class Team(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    heroes: list["Hero"] = Relationship(back_populates="teams", link_model=HeroTeamLink)

class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    teams: list[Team] = Relationship(back_populates="heroes", link_model=HeroTeamLink)
```

### Multiple Models for FastAPI
```python
class HeroBase(SQLModel):
    name: str = Field(index=True)
    secret_name: str
    age: int | None = Field(default=None, index=True)

class Hero(HeroBase, table=True):
    id: int | None = Field(default=None, primary_key=True)

class HeroCreate(HeroBase):
    pass

class HeroPublic(HeroBase):
    id: int

class HeroUpdate(SQLModel):
    name: str | None = None
    secret_name: str | None = None
    age: int | None = None
```

### FastAPI Session Dependency
```python
from fastapi import Depends, FastAPI, Query
from sqlmodel import Session, select

def get_session():
    with Session(engine) as session:
        yield session

SessionDep = Annotated[Session, Depends(get_session)]

@app.post("/heroes/", response_model=HeroPublic)
def create_hero(session: SessionDep, hero: HeroCreate):
    db_hero = Hero.model_validate(hero)
    session.add(db_hero)
    session.commit()
    session.refresh(db_hero)
    return db_hero

@app.get("/heroes/", response_model=list[HeroPublic])
def read_heroes(session: SessionDep, offset: int = 0, limit: int = Query(default=100, le=100)):
    heroes = session.exec(select(Hero).offset(offset).limit(limit)).all()
    return heroes
```

---

## CRUD Generator Pattern

When user requests `/sqlmodel crud <resource>`, generate:

1. **Table model** with appropriate fields
2. **Create schema** (no `id`)
3. **Public/Response schema** (`id` required)
4. **Update schema** (all fields optional)
5. **CRUD functions** using Session: `create_<resource>`, `get_<resource>`, `get_<resources>`, `update_<resource>`, `delete_<resource>`
6. If FastAPI is in use, also generate **router endpoints** with proper `response_model`

---

## Key Concepts to Emphasize

### table=True vs Plain SQLModel
- `class Hero(SQLModel, table=True)` → **database table** (SQLAlchemy model + Pydantic)
- `class HeroCreate(SQLModel)` → **data model only** (Pydantic validation, no DB table)

### Session Mental Model
```
Python objects ──add()──► Session (staging) ──commit()──► Database
                         ◄──refresh()── (reload data)
```

### Result Methods
| Method     | Returns               | Raises if...         |
|------------|-----------------------|----------------------|
| `.all()`   | `list` of all results | never                |
| `.first()` | first result or `None`| never                |
| `.one()`   | exactly one result    | 0 or 2+ results     |

### session.exec() vs session.execute()
Always use `session.exec()` — it's SQLModel's enhanced version with better type hints and autocompletion. Avoid raw SQLAlchemy `session.execute()`.

---

## Teaching Guidelines

1. **Start with a single file** — model + engine + CRUD in one `main.py`
2. **Always explain the mental model** — Python objects vs database rows, sessions as staging areas
3. **Show the SQL** — use `echo=True` on the engine so users see what SQL is generated
4. **Build incrementally** — get one model working before adding relationships
5. **Type hints drive everything** — stress that `str` vs `str | None` controls NOT NULL vs nullable
6. **Provide runnable code** — every example should work when pasted and run
7. **Graduate to multi-file** — only refactor to professional structure when single-file gets unwieldy

## When Debugging

1. Check if `table=True` is set on models that should be tables
2. Verify `SQLModel.metadata.create_all(engine)` is called after model imports
3. Check foreign key strings match exactly: `foreign_key="tablename.column"`
4. Ensure `back_populates` names match the attribute name on the other model
5. Check that `session.commit()` is called before reading back data
6. Use `echo=True` on the engine to inspect generated SQL
7. For "no such table" errors, ensure models are imported before `create_all()`
8. For relationship loading issues, access related objects within the session context

## Reference Files

For detailed documentation on specific topics, load files from the `references/` directory:
- `references/relationships.md` — Complete guide to one-to-many and many-to-many relationships
- `references/fastapi-integration.md` — Full FastAPI integration patterns (session dep, multiple models, CRUD endpoints)
- `references/migrations.md` — Alembic migration setup and workflow with SQLModel
