# Alembic Migrations with SQLModel

## Why Migrations?

`SQLModel.metadata.create_all()` only creates tables that don't exist. It does NOT:
- Add new columns to existing tables
- Rename columns
- Change column types
- Remove columns

Alembic tracks and applies incremental database changes (migrations).

---

## Setup

### Install
```bash
pip install alembic
```

### Initialize
```bash
alembic init migrations
```

This creates:
```
project/
├── alembic.ini          # Alembic config (DB URL goes here)
└── migrations/
    ├── env.py           # Migration environment (modify this)
    ├── script.py.mako   # Template for new migrations
    └── versions/        # Migration files go here
```

### Configure `alembic.ini`

Set the database URL:
```ini
sqlalchemy.url = sqlite:///database.db
```

Or for PostgreSQL:
```ini
sqlalchemy.url = postgresql://user:password@localhost/dbname
```

### Configure `migrations/env.py`

The critical change: import your SQLModel models and set `target_metadata`.

```python
# At the top of env.py, add:
from sqlmodel import SQLModel

# Import ALL your models so they register with SQLModel.metadata
from app.models.user import User
from app.models.product import Product

# Find this line and change it:
# target_metadata = None
target_metadata = SQLModel.metadata
```

**Important:** Every model with `table=True` must be imported here, or Alembic won't detect it.

---

## Creating Migrations

### Auto-generate from Model Changes

After changing your SQLModel models:

```bash
alembic revision --autogenerate -m "add email column to user"
```

This compares your models against the current DB and generates a migration file in `migrations/versions/`.

### Review the Generated Migration

Always review before applying! Example:

```python
"""add email column to user

Revision ID: abc123
"""
from alembic import op
import sqlalchemy as sa
import sqlmodel

def upgrade() -> None:
    op.add_column('user', sa.Column('email', sa.String(), nullable=True))

def downgrade() -> None:
    op.drop_column('user', 'email')
```

### Manual Migration

For changes Alembic can't auto-detect:

```bash
alembic revision -m "seed initial data"
```

Then write the `upgrade()` and `downgrade()` functions manually.

---

## Applying Migrations

```bash
# Apply all pending migrations
alembic upgrade head

# Apply one migration forward
alembic upgrade +1

# Rollback one migration
alembic downgrade -1

# Rollback to beginning
alembic downgrade base

# Show current revision
alembic current

# Show migration history
alembic history
```

---

## Common Workflow

1. Modify your SQLModel model (add field, change type, etc.)
2. Run `alembic revision --autogenerate -m "description"`
3. Review the generated migration file
4. Run `alembic upgrade head`
5. Commit both the model changes and migration file to git

---

## SQLModel-Specific Notes

### Column Type Mapping

SQLModel uses `sqlmodel.sql.sqltypes.AutoString` instead of SQLAlchemy's `sa.String`. In migrations, you may see:

```python
import sqlmodel
sa.Column('name', sqlmodel.sql.sqltypes.AutoString(), nullable=False)
```

This is normal and works correctly.

### Handling table=True Models

Only models with `table=True` are tracked. Data-only models (no `table=True`) are invisible to Alembic.

### Initial Migration from Existing DB

If you already have tables created by `create_all()`:

```bash
# Create a migration that represents the current state
alembic revision --autogenerate -m "initial schema"

# Mark it as applied without running it (tables already exist)
alembic stamp head
```

---

## Production Tips

1. **Never use `create_all()` in production** — use migrations exclusively
2. **Always review auto-generated migrations** — they can miss renames (shows as drop + create)
3. **Test migrations against a copy** of your production database before applying
4. **Keep migrations in version control** — they're part of your codebase
5. **Use environment variables** for the database URL in `alembic.ini`:
   ```ini
   # In alembic.ini, leave blank:
   sqlalchemy.url =

   # In env.py, load from environment:
   import os
   config.set_main_option("sqlalchemy.url", os.getenv("DATABASE_URL"))
   ```
6. **Run migrations in CI/CD** as part of your deployment pipeline
