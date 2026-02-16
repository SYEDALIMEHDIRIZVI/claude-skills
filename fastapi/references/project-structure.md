# FastAPI Professional Project Structure

## Recommended Layout

```
project-name/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI app instance, include routers
│   ├── config.py             # Settings via pydantic-settings
│   ├── dependencies.py       # Shared dependencies (auth, db session)
│   ├── database.py           # Engine, SessionLocal, Base
│   ├── models/               # SQLAlchemy / DB models
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── item.py
│   ├── schemas/              # Pydantic request/response schemas
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── item.py
│   ├── routers/              # APIRouter modules
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── users.py
│   │   └── items.py
│   ├── services/             # Business logic layer
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── item.py
│   └── utils/                # Helper functions
│       ├── __init__.py
│       └── security.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py           # Fixtures (test client, test db)
│   └── test_users.py
├── alembic/                  # DB migrations (if using Alembic)
│   ├── versions/
│   └── env.py
├── requirements.txt
├── .env.example
├── .gitignore
├── Dockerfile
└── alembic.ini
```

## Key Files

### main.py

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from .routers import auth, users, items
from .database import engine, Base

Base.metadata.create_all(bind=engine)

app = FastAPI(title="My API", version="1.0.0")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(auth.router)
app.include_router(users.router)
app.include_router(items.router)
```

### config.py

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str = "sqlite:///./app.db"
    secret_key: str = "change-me"
    access_token_expire_minutes: int = 30
    debug: bool = False

    model_config = {"env_file": ".env"}

settings = Settings()
```

### routers/users.py

```python
from fastapi import APIRouter, HTTPException
from ..schemas.user import UserCreate, UserResponse
from ..services.user import UserService
from ..dependencies import DbDep, CurrentUserDep

router = APIRouter(prefix="/users", tags=["users"])

@router.get("/me", response_model=UserResponse)
async def get_me(current_user: CurrentUserDep):
    return current_user

@router.get("/{user_id}", response_model=UserResponse)
async def get_user(user_id: int, db: DbDep):
    user = UserService.get_by_id(db, user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

## APIRouter Key Features

```python
router = APIRouter(
    prefix="/items",          # URL prefix for all routes
    tags=["items"],           # Group in API docs
    dependencies=[Depends(get_current_user)],  # Auth for all routes
    responses={404: {"description": "Not found"}},
)
```

- **prefix** avoids repeating `/items` in every decorator
- **tags** groups endpoints in Swagger UI
- **dependencies** applies auth/validation to all routes in this router
- **responses** adds common response docs

## Including Routers

```python
# Basic
app.include_router(users.router)

# Override/add prefix and tags at include time
app.include_router(admin.router, prefix="/admin", tags=["admin"])

# Same router, different versions
app.include_router(items.router, prefix="/api/v1")
app.include_router(items.router, prefix="/api/v2")
```

## Testing Setup

### conftest.py

```python
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.main import app
from app.database import Base
from app.dependencies import get_db

SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

@pytest.fixture
def db():
    Base.metadata.create_all(bind=engine)
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()
        Base.metadata.drop_all(bind=engine)

@pytest.fixture
def client(db):
    def override_get_db():
        yield db
    app.dependency_overrides[get_db] = override_get_db
    with TestClient(app) as c:
        yield c
    app.dependency_overrides.clear()
```

### test_users.py

```python
def test_create_user(client):
    response = client.post("/users/", json={"email": "test@example.com", "password": "secret"})
    assert response.status_code == 200
    data = response.json()
    assert data["email"] == "test@example.com"
    assert "id" in data
```

## Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["fastapi", "run", "app/main.py", "--host", "0.0.0.0", "--port", "8000"]
```
