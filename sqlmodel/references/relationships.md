# SQLModel Relationships — Complete Guide

## One-to-Many Relationships

A one-to-many relationship connects a parent record to multiple child records. Example: one Team has many Heroes.

### Setup

The "many" side holds the foreign key. Both sides use `Relationship()` with `back_populates`.

```python
from sqlmodel import Field, Relationship, SQLModel

class Team(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    headquarters: str

    # One team → many heroes
    heroes: list["Hero"] = Relationship(back_populates="team")

class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    secret_name: str
    age: int | None = Field(default=None, index=True)

    # Foreign key column (actual DB column)
    team_id: int | None = Field(default=None, foreign_key="team.id")
    # Relationship attribute (Python-only, not a DB column)
    team: Team | None = Relationship(back_populates="heroes")
```

### Key Rules
- `foreign_key="team.id"` — string must match `tablename.columnname` exactly (lowercase table name)
- `back_populates` — must match the attribute name on the OTHER model
- Parent side: `list["Child"]` (forward reference in quotes if Child is defined later)
- Child side: `Parent | None` (optional if the relationship is optional)

### Usage

```python
# Create with relationship
team = Team(name="Avengers", headquarters="NYC")
hero = Hero(name="Spider-Boy", secret_name="Pedro", team=team)

with Session(engine) as session:
    session.add(hero)  # also adds team automatically
    session.commit()

# Access related objects (must be within session)
with Session(engine) as session:
    team = session.exec(select(Team).where(Team.name == "Avengers")).one()
    for hero in team.heroes:
        print(hero.name)

    hero = session.exec(select(Hero).where(Hero.name == "Spider-Boy")).one()
    print(hero.team.name)  # "Avengers"
```

---

## Many-to-Many Relationships

A many-to-many relationship allows records on both sides to connect to multiple records on the other side. Example: Heroes can belong to multiple Teams, and Teams can have multiple Heroes.

### Link Table

Create an intermediate table with foreign keys to both sides. Both columns form a composite primary key.

```python
class HeroTeamLink(SQLModel, table=True):
    hero_id: int | None = Field(default=None, foreign_key="hero.id", primary_key=True)
    team_id: int | None = Field(default=None, foreign_key="team.id", primary_key=True)
```

### Models

```python
class Team(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    headquarters: str
    heroes: list["Hero"] = Relationship(
        back_populates="teams",
        link_model=HeroTeamLink
    )

class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    secret_name: str
    age: int | None = Field(default=None, index=True)
    teams: list[Team] = Relationship(
        back_populates="heroes",
        link_model=HeroTeamLink
    )
```

### Key Difference from One-to-Many
- No `foreign_key` field on either main model — the link table holds them
- Both sides use `list[...]` (both are "many")
- Both `Relationship()` calls include `link_model=HeroTeamLink`

### Usage

```python
with Session(engine) as session:
    team_avengers = Team(name="Avengers", headquarters="NYC")
    team_xmen = Team(name="X-Men", headquarters="Mansion")
    hero = Hero(name="Wolverine", secret_name="Logan", teams=[team_avengers, team_xmen])

    session.add(hero)
    session.commit()

# Query
with Session(engine) as session:
    hero = session.exec(select(Hero).where(Hero.name == "Wolverine")).one()
    for team in hero.teams:
        print(team.name)  # "Avengers", "X-Men"
```

---

## Link Table with Extra Data

If the link itself has data (e.g., a "role" in the team), add extra columns:

```python
class HeroTeamLink(SQLModel, table=True):
    hero_id: int | None = Field(default=None, foreign_key="hero.id", primary_key=True)
    team_id: int | None = Field(default=None, foreign_key="team.id", primary_key=True)
    role: str = "member"  # extra data on the link
    joined_at: datetime = Field(default_factory=datetime.utcnow)
```

---

## Cascade Deletes

By default, deleting a parent does NOT delete children. To enable cascade:

```python
class Team(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str
    heroes: list["Hero"] = Relationship(
        back_populates="team",
        sa_relationship_kwargs={"cascade": "all, delete-orphan"}
    )
```

This means: when a Team is deleted, all its Heroes are deleted too. Use with caution.

---

## Common Mistakes

1. **Accessing relationships outside session** — always access `.team` or `.heroes` while the session is open
2. **Mismatched back_populates** — the string must exactly match the attribute name on the other model
3. **Wrong foreign_key string** — use the actual table name (usually lowercase class name), not the class name
4. **Forgetting link_model** — many-to-many won't work without `link_model=` on both sides
5. **Circular imports** — use forward references with quotes: `list["Hero"]`
