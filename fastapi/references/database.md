# FastAPI Database Integration

## Option 1: SQLAlchemy (Most Common)

### Setup

```bash
pip install sqlalchemy
# For async: pip install sqlalchemy[asyncio] aiosqlite
```

### database.py

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase

SQLALCHEMY_DATABASE_URL = "sqlite:///./app.db"
# PostgreSQL: "postgresql://user:password@localhost/dbname"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    connect_args={"check_same_thread": False}  # SQLite only
)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

class Base(DeclarativeBase):
    pass
```

### models.py (Database Models)

```python
from sqlalchemy import Column, Integer, String, Boolean, ForeignKey
from sqlalchemy.orm import relationship
from .database import Base

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)

    items = relationship("Item", back_populates="owner")

class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    description = Column(String)
    owner_id = Column(Integer, ForeignKey("users.id"))

    owner = relationship("User", back_populates="items")
```

### schemas.py (Pydantic Models)

```python
from pydantic import BaseModel, ConfigDict

class ItemBase(BaseModel):
    title: str
    description: str | None = None

class ItemCreate(ItemBase):
    pass

class ItemResponse(ItemBase):
    id: int
    owner_id: int
    model_config = ConfigDict(from_attributes=True)

class UserBase(BaseModel):
    email: str

class UserCreate(UserBase):
    password: str

class UserResponse(UserBase):
    id: int
    is_active: bool
    items: list[ItemResponse] = []
    model_config = ConfigDict(from_attributes=True)
```

### Database Session Dependency

```python
from typing import Annotated
from fastapi import Depends
from sqlalchemy.orm import Session
from .database import SessionLocal

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

DbDep = Annotated[Session, Depends(get_db)]
```

### CRUD Operations

```python
from sqlalchemy.orm import Session
from . import models, schemas

def get_user(db: Session, user_id: int):
    return db.query(models.User).filter(models.User.id == user_id).first()

def get_users(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.User).offset(skip).limit(limit).all()

def create_user(db: Session, user: schemas.UserCreate):
    hashed_password = get_password_hash(user.password)
    db_user = models.User(email=user.email, hashed_password=hashed_password)
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user
```

### Router with Database

```python
from fastapi import APIRouter, HTTPException
from . import crud, schemas
from .dependencies import DbDep

router = APIRouter(prefix="/users", tags=["users"])

@router.post("/", response_model=schemas.UserResponse)
def create_user(user: schemas.UserCreate, db: DbDep):
    db_user = crud.get_user_by_email(db, email=user.email)
    if db_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    return crud.create_user(db=db, user=user)

@router.get("/", response_model=list[schemas.UserResponse])
def read_users(db: DbDep, skip: int = 0, limit: int = 100):
    return crud.get_users(db, skip=skip, limit=limit)
```

### Create Tables

```python
# In main.py
from .database import engine, Base

Base.metadata.create_all(bind=engine)
```

## Option 2: SQLModel (by FastAPI Creator)

SQLModel combines SQLAlchemy and Pydantic into one — a single class serves as both DB model and schema.

```bash
pip install sqlmodel
```

```python
from sqlmodel import Field, SQLModel, Session, create_engine, select

class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    secret_name: str
    age: int | None = Field(default=None, index=True)

engine = create_engine("sqlite:///database.db")
SQLModel.metadata.create_all(engine)

def get_session():
    with Session(engine) as session:
        yield session
```

## Migrations with Alembic

```bash
pip install alembic
alembic init alembic
# Edit alembic/env.py to import your Base and set target_metadata
alembic revision --autogenerate -m "initial"
alembic upgrade head
```
