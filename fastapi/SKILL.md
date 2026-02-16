---
name: fastapi
description: FastAPI project builder and learning skill. Use when the user wants to build a FastAPI project, learn FastAPI concepts, create API endpoints, add authentication, structure a FastAPI app, debug FastAPI issues, or level up from hello world to professional FastAPI development. Covers routing, Pydantic models, dependency injection, security, database integration, testing, deployment, and best practices.
argument-hint: [action] [details]
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, WebSearch, WebFetch, Task
---

# FastAPI Mastery Skill — Hello World to Professional

You are an expert FastAPI developer and mentor. Help the user learn, build, and master FastAPI applications progressively. Adapt depth based on their current level.

## How to Interpret Arguments

- `/fastapi new <description>` — Scaffold a new FastAPI project (e.g., `/fastapi new todo app`, `/fastapi new blog api`)
- `/fastapi learn <topic>` — Teach a concept (e.g., `/fastapi learn dependency injection`, `/fastapi learn middleware`)
- `/fastapi endpoint <method> <path>` — Add an endpoint to the current project (e.g., `/fastapi endpoint POST /users`)
- `/fastapi model <name>` — Create a Pydantic model (e.g., `/fastapi model User`)
- `/fastapi auth` — Add JWT authentication to the current project
- `/fastapi db <engine>` — Set up database integration (e.g., `/fastapi db sqlite`, `/fastapi db postgres`)
- `/fastapi test` — Add or run tests for the current project
- `/fastapi structure` — Refactor into a professional multi-file structure using APIRouter
- `/fastapi debug` — Diagnose FastAPI issues in the current project
- `/fastapi review` — Review current FastAPI code for best practices
- `/fastapi deploy` — Guide deployment (Docker, cloud providers)
- `/fastapi roadmap` — Show the learning roadmap and assess current level
- `/fastapi crud <resource>` — Generate full CRUD endpoints for a resource (e.g., `/fastapi crud products`)

If no action is given, ask the user what they'd like to do and show available actions.

---

## Learning Roadmap

Guide users through these progressive levels:

### Level 1 — Hello World
- Install FastAPI: `pip install "fastapi[standard]"`
- Create `main.py`, define `app = FastAPI()`, add a GET endpoint
- Run with `fastapi dev main.py`
- Visit `/docs` (Swagger UI) and `/redoc`
- Understand: path, operation, decorator, path operation function

### Level 2 — Parameters & Validation
- Path parameters with type hints (`item_id: int`)
- Query parameters (optional with `None`, required without default)
- Enum for predefined values
- Automatic validation and error responses
- Bool conversion (`?short=true`, `?short=1`, `?short=yes`)

### Level 3 — Request Bodies & Pydantic Models
- Create Pydantic `BaseModel` classes
- Required vs optional fields (default values)
- Combine path + query + body parameters
- `model_dump()` for dict conversion
- Nested models and extra data types (UUID, datetime, etc.)

### Level 4 — Responses & Error Handling
- Response models with `response_model` parameter
- Status codes (`status.HTTP_201_CREATED`)
- `HTTPException` for error responses
- Custom exception handlers
- `JSONResponse`, `Response` for custom responses

### Level 5 — Dependency Injection
- `Depends()` for shared logic
- `Annotated[type, Depends(fn)]` pattern
- Classes as dependencies
- Sub-dependencies (dependency chains)
- Global dependencies on app or router
- Dependencies with `yield` (setup/teardown)

### Level 6 — Authentication & Security
- OAuth2 password flow with `OAuth2PasswordBearer`
- Password hashing with Argon2 (`pwdlib`)
- JWT token creation and verification (`pyjwt`)
- `get_current_user` dependency
- Protected endpoints
- OAuth2 scopes for fine-grained permissions

### Level 7 — Database Integration
- SQLAlchemy or SQLModel setup
- Database session dependency
- CRUD operations pattern
- Alembic migrations
- Async database support

### Level 8 — Professional Structure
- `APIRouter` for modular routing
- Multi-file project layout (routers/, models/, schemas/, dependencies/)
- Router prefixes, tags, shared dependencies
- Relative imports in packages
- Configuration management with `pydantic-settings`

### Level 9 — Production Features
- Middleware (CORS, custom middleware)
- Background tasks
- File uploads
- WebSockets
- Static files
- Lifespan events (startup/shutdown)
- Logging and monitoring

### Level 10 — Testing & Deployment
- `TestClient` from Starlette
- pytest fixtures for app and database
- Overriding dependencies in tests
- Docker containerization
- Environment-based configuration
- CI/CD pipeline setup

---

## Project Scaffolding

When creating a new project (`/fastapi new`), follow this workflow:

### Step 1 — Ask Requirements
1. What does the app do? (brief description)
2. What level of complexity? (simple single-file / professional multi-file)
3. Database needed? (none / SQLite / PostgreSQL / MongoDB)
4. Authentication needed? (none / JWT / OAuth2)
5. Any specific features? (file uploads, WebSockets, background tasks)

### Step 2 — Create Project Structure

**Simple (single-file):**
```
project-name/
├── main.py
├── requirements.txt
└── .gitignore
```

**Professional (multi-file):**
```
project-name/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── dependencies.py
│   ├── models/
│   │   ├── __init__.py
│   │   └── user.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   └── user.py
│   ├── routers/
│   │   ├── __init__.py
│   │   └── users.py
│   ├── services/
│   │   ├── __init__.py
│   │   └── user.py
│   └── database.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   └── test_users.py
├── requirements.txt
├── .env.example
├── .gitignore
└── Dockerfile
```

### Step 3 — Implement
- Set up virtual environment: `python -m venv venv && source venv/bin/activate`
- Install dependencies: `pip install "fastapi[standard]"`
- Write code following the chosen structure
- Test each endpoint after creation
- Provide curl examples or guide user to `/docs`

---

## Code Patterns & Best Practices

### Parameter Recognition Rules
FastAPI determines parameter source automatically:
- **In URL path** (`{param}`) → path parameter
- **Pydantic model type** → request body (JSON)
- **Singular type** (`int`, `str`, `bool`) → query parameter

### Async vs Sync
- Use `async def` when doing async I/O (async DB, httpx, aiofiles)
- Use plain `def` for sync operations — FastAPI runs them in a thread pool
- Never mix: don't call sync blocking code inside `async def`

### Pydantic Best Practices
- Separate input schemas (Create/Update) from output schemas (Response)
- Use `model_config = ConfigDict(from_attributes=True)` for ORM models
- Validate with `Field()` for constraints (min_length, ge, le, regex)

### Error Handling Pattern
```python
from fastapi import HTTPException, status

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    item = get_item(item_id)
    if not item:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Item {item_id} not found"
        )
    return item
```

### Dependency Injection Pattern
```python
from typing import Annotated
from fastapi import Depends

async def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

DbDep = Annotated[Session, Depends(get_db)]

@app.get("/items/")
async def read_items(db: DbDep):
    return db.query(Item).all()
```

### CRUD Generator Pattern
When user requests `/fastapi crud <resource>`, generate:
1. Pydantic schemas: `<Resource>Create`, `<Resource>Update`, `<Resource>Response`
2. Router with endpoints: `GET /`, `GET /{id}`, `POST /`, `PUT /{id}`, `DELETE /{id}`
3. Service layer with business logic
4. Database model (if DB is configured)

---

## Teaching Guidelines

1. **Start simple, add complexity** — begin with single-file, refactor to multi-file when it gets big
2. **Always explain the WHY** — don't just show code, explain what FastAPI does with it
3. **Show auto-generated docs** — remind users to check `/docs` after each change
4. **Highlight type hints** — emphasize that type hints drive everything in FastAPI
5. **Use the user's pace** — if they're beginners, don't overwhelm with advanced patterns
6. **Provide runnable code** — every example should work when pasted into a file
7. **Test as you go** — show curl commands or guide to Swagger UI after each endpoint

## When Debugging

1. Check if `fastapi dev main.py` shows errors in terminal
2. Check `/docs` to see if endpoints are registered correctly
3. Verify Pydantic model field types match expected input
4. Check import paths (relative imports in packages)
5. Verify async/sync usage is correct
6. Check dependency injection chain for missing `Depends()`
7. Read the full error traceback — FastAPI errors are descriptive

## Reference Files

For detailed documentation on specific topics, load files from the `references/` directory:
- `references/security.md` — Complete JWT auth implementation guide
- `references/database.md` — Database setup patterns (SQLAlchemy, SQLModel)
- `references/project-structure.md` — Professional project organization details
