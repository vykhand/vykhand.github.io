# LightODM: A Lightweight MongoDB ODM for Python

## Why I Built This

I've been using [Beanie](https://github.com/roman-right/beanie) for MongoDB projects, and it's genuinely a great ODM—well-designed, feature-rich, and solid for production use. But on several projects, I kept running into the same friction points:

- I needed both sync and async in the same codebase (FastAPI endpoints + background workers)
- The custom query builder felt like learning a new language when I already knew MongoDB's syntax
- I wanted to drop down to raw PyMongo/Motor when needed, without fighting abstractions
- For simpler projects, the feature set felt heavier than necessary

So I built [**LightODM**](https://github.com/Aprova-GmbH/lightodm)—not as a replacement for Beanie, but as a lighter alternative when you want Pydantic validation without giving up direct control over MongoDB.

---

## What is LightODM?

It's a minimal ODM that focuses on the essentials:
- Works with both sync (PyMongo) and async (Motor) in the same model
- Uses Pydantic v2 for validation and serialization
- Lets you use MongoDB's native query syntax (no DSL to learn)
- Exposes the actual PyMongo/Motor collections when you need them
- Stays small (~500 lines) with just 3 dependencies: pydantic, pymongo, motor

### Installation

```bash
pip install lightodm
```

Works with Python 3.8+.

---

## Quick Comparison: LightODM vs Beanie

| Feature | LightODM | Beanie |
|---------|----------|--------|
| **Sync Support** | ✅ Yes | ❌ Async only |
| **Async Support** | ✅ Yes | ✅ Yes |
| **Code Size** | ~500 lines | ~5000+ lines |
| **Query Syntax** | MongoDB native | Custom query builder |
| **Pydantic** | v2 | v2 |
| **Core Dependencies** | 3 (pydantic, pymongo, motor) | 5+ |
| **Connection** | Environment variables | init_beanie() required |

**Use LightODM when:**
- You need both sync and async operations
- You already know MongoDB query syntax and don't want to learn a DSL
- You want to access raw PyMongo/Motor collections easily
- You prefer fewer abstractions

**Use Beanie when:**
- You only need async (it's async-first by design)
- You want document relationships and links built in
- You prefer a query builder API over raw MongoDB syntax
- You need more ODM features out of the box

---

## How It Works

### Define Your Models

```python
from lightodm import MongoBaseModel
from typing import Optional

class User(MongoBaseModel):
    class Settings:
        name = "users"  # MongoDB collection name

    name: str
    email: str
    age: Optional[int] = None
```

### Connect to MongoDB

Set up your connection via environment variables:

```bash
export MONGO_URL="mongodb://localhost:27017"
export MONGO_DB_NAME="myapp"
# Optional:
export MONGO_USER="your_user"
export MONGO_PASSWORD="your_password"
```

That's it—LightODM handles the connection automatically when you use your models.

### Synchronous Operations

```python
# Create and save
user = User(name="Alice", email="alice@example.com", age=28)
user.save()

# Retrieve by ID
found = User.get(user.id)

# Find with MongoDB query syntax
adults = User.find({"age": {"$gte": 18}})

# Update
User.update_one(
    {"_id": user.id},
    {"$set": {"age": 29}}
)

# Delete
user.delete()
```

### Asynchronous Operations

Every method has an async version prefixed with `a`:

```python
import asyncio

async def main():
    # Create and save
    user = User(name="Bob", email="bob@example.com")
    await user.asave()

    # Retrieve
    found = await User.aget(user.id)

    # Find
    users = await User.afind({"age": {"$gte": 18}})

    # Update
    await User.aupdate_one(
        {"_id": user.id},
        {"$set": {"age": 30}}
    )

    # Delete
    await user.adelete()

asyncio.run(main())
```

Same model, same interface—choose sync or async based on your context.

---

## Key Features

### 1. Pydantic v2 Native Integration

Full Pydantic v2 validation and serialization:

```python
from pydantic import EmailStr, Field

class User(MongoBaseModel):
    class Settings:
        name = "users"

    name: str = Field(..., min_length=1, max_length=100)
    email: EmailStr
    age: int = Field(..., ge=0, le=150)
```

### 2. Direct MongoDB Access

No abstraction layers—use PyMongo/Motor directly:

```python
# Get the actual collection
collection = User.get_collection()

# Use any PyMongo method
collection.create_index([("email", 1)], unique=True)

# Run aggregation pipelines
pipeline = [
    {"$match": {"age": {"$gte": 18}}},
    {"$group": {"_id": "$city", "count": {"$sum": 1}}}
]
results = User.aggregate(pipeline)
```

### 3. Thread-Safe Singleton Connection

Connection management with automatic cleanup:

```python
from lightodm import MongoConnection

# Singleton pattern
conn = MongoConnection()

# Both sync and async clients
sync_client = conn.client
async_client = await conn.get_async_client()

# Automatic cleanup on exit (atexit handlers)
```

### 4. Bring Your Own Connection (BYOC)

For advanced scenarios, override connection methods:

```python
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017")
db = client.myapp

class User(MongoBaseModel):
    class Settings:
        name = "users"

    name: str

    @classmethod
    def get_collection(cls):
        return db.users  # Use your own connection
```

---

## Real-World Use Cases

### FastAPI Integration

```python
from fastapi import FastAPI, HTTPException
from lightodm import MongoBaseModel

app = FastAPI()

class User(MongoBaseModel):
    class Settings:
        name = "users"

    name: str
    email: str

@app.post("/users/")
async def create_user(user: User):
    await user.asave()
    return user

@app.get("/users/{user_id}")
async def get_user(user_id: str):
    user = await User.aget(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

Just make sure your environment variables are set before starting the app.

### Multi-Tenant Applications

```python
from contextvars import ContextVar

current_tenant: ContextVar[str] = ContextVar("current_tenant")

class TenantModel(MongoBaseModel):
    @classmethod
    def get_collection(cls):
        tenant = current_tenant.get()
        db_name = f"tenant_{tenant}"
        return client[db_name][cls.Settings.name]

class User(TenantModel):
    class Settings:
        name = "users"

    name: str
```

### Aggregation Pipelines

```python
# Revenue by date
pipeline = [
    {
        "$match": {
            "created_at": {"$gte": start_date},
            "status": "completed"
        }
    },
    {
        "$group": {
            "_id": {"$dateToString": {"format": "%Y-%m-%d", "date": "$created_at"}},
            "revenue": {"$sum": "$total"},
            "count": {"$sum": 1}
        }
    },
    {"$sort": {"_id": 1}}
]

results = await Order.aaggregate(pipeline)
```

---

## Why Not Beanie?

Beanie is excellent, but it's built for a different use case:

**Beanie's strengths:**
- Rich feature set (migrations, relationships, etc.)
- Fluent query API
- Active development and community
- Great for async-only applications

**LightODM's philosophy:**
- Minimalistic (easier to understand and debug)
- Sync + async support
- Direct MongoDB control
- Zero magic, maximum transparency
- Perfect for simple to medium complexity projects

Think of it this way:
- **Beanie** = Django ORM (comprehensive, opinionated)
- **LightODM** = SQLAlchemy Core (lightweight, flexible)

---

## Migration from Beanie

Migrating is straightforward:

```python
# Before (Beanie)
from beanie import Document

class User(Document):
    name: str
    email: str

    class Settings:
        name = "users"

await init_beanie(database=db, document_models=[User])
await user.insert()
users = await User.find(User.age >= 18).to_list()

# After (LightODM)
from lightodm import MongoBaseModel, connect

class User(MongoBaseModel):
    name: str
    email: str

    class Settings:
        name = "users"

connect(db_name="mydb")
await user.asave()  # or user.save() for sync
users = await User.afind({"age": {"$gte": 18}})
```

Main changes:
1. Change base class: `Document` → `MongoBaseModel`
2. Replace `insert()` with `save()` or `asave()`
3. Use MongoDB query syntax instead of Beanie's fluent API
4. Handle relationships manually

See the full [migration guide](https://github.com/Aprova-GmbH/lightodm/blob/main/docs/MIGRATION_FROM_BEANIE.md) for details.

---

## Project Status

- **Version:** 0.1.0
- **License:** Apache 2.0
- **Repository:** [github.com/Aprova-GmbH/lightodm](https://github.com/Aprova-GmbH/lightodm)
- **Documentation:** Coming soon on ReadTheDocs
- **Test Coverage:** >95%
- **Python Support:** 3.11, 3.12, 3.13

---

## Design Philosophy

LightODM follows these principles:

1. **Minimalistic**: Keep the codebase small (~500 lines) and focused on core ODM functionality.

2. **Explicit > Implicit**: No magic. Collection names are explicit. Query syntax is MongoDB's native syntax. Connections are transparent.

3. **Dual Support**: Both sync and async APIs with the same model. Use what fits your context.

4. **Pydantic Native**: Leverage Pydantic v2 fully without compatibility layers or wrappers.

5. **MongoDB Direct**: Expose PyMongo/Motor collections directly. No abstraction penalty.

6. **Zero Dependencies Beyond Core**: Only pydantic, pymongo, and motor. No surprises in your dependency tree.

---

## What's Missing (By Design)

LightODM intentionally omits features to stay minimal:

❌ **Migration tools** - Use PyMongo directly or write your own
❌ **Automatic relationships** - Reference documents manually via IDs
❌ **Query builder DSL** - Use MongoDB's query syntax
❌ **Caching layer** - Add your own if needed
❌ **Event hooks** - Use Pydantic validators or override methods

If you need these features, Beanie is a better choice.

---

## Try It

```bash
pip install lightodm
```

Or check out the [GitHub repository](https://github.com/Aprova-GmbH/lightodm) for examples and documentation.

If you're building Python applications with MongoDB and want:
- Both sync and async support
- Direct MongoDB control
- Pydantic v2 validation
- Minimal abstraction

...give LightODM a try. It solves one problem—MongoDB ODM—and solves it simply.

---

## Examples in the Wild

Check out the `examples/` directory in the repository:
- [Basic usage](https://github.com/Aprova-GmbH/lightodm/blob/main/examples/basic_usage.py) - Sync CRUD operations
- [Async usage](https://github.com/Aprova-GmbH/lightodm/blob/main/examples/async_usage.py) - Async operations
- [FastAPI integration](https://github.com/Aprova-GmbH/lightodm/blob/main/examples/fastapi_example.py) - Complete REST API
- [Custom connection](https://github.com/Aprova-GmbH/lightodm/blob/main/examples/custom_connection.py) - BYOC pattern

---

**Questions or feedback?** Open an issue on [GitHub](https://github.com/Aprova-GmbH/lightodm/issues) or reach out at vya@aprova.ch.

**Want to contribute?** Check out the [contributing guide](https://github.com/Aprova-GmbH/lightodm/blob/main/CONTRIBUTING.md).
