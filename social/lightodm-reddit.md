# LightODM Reddit Posts

## r/Python - Main Post

**Title:** [P] LightODM: Minimal MongoDB ODM with both sync & async support

I love Beanie for MongoDB, but I kept running into projects where I needed both sync and async operations (FastAPI + background workers), and Beanie is async-only. So I built LightODM.

**The pitch:** Pydantic v2 models for MongoDB with zero magic. ~500 lines of code, 3 dependencies.

```python
from lightodm import MongoBaseModel

class User(MongoBaseModel):
    class Settings:
        name = "users"

    name: str
    email: str
    age: int

# Sync
user = User(name="Alice", email="alice@example.com", age=28)
user.save()
found = User.get(user.id)

# Async (same model!)
await user.asave()
found = await User.aget(user.id)
```

**Key differences from Beanie:**
- ✅ Both sync (PyMongo) and async (Motor) with the same model
- ✅ Uses MongoDB's native query syntax (no DSL to learn)
- ✅ Direct access to PyMongo/Motor collections when you need them
- ✅ ~500 lines vs ~5000+ lines
- ❌ No built-in relationships (use IDs manually)
- ❌ No migrations (use PyMongo directly)

**Why not just use Beanie?** You should, if you only need async! LightODM is for when you need both sync and async, or when you want less abstraction.

GitHub: https://github.com/Aprova-GmbH/lightodm
PyPI: `pip install lightodm`
Blog: https://vykhand.github.io/LightODM-Minimalistic-MongoDB-ODM/

Apache 2.0, Python 3.11+, >95% test coverage.

Includes migration guide from Beanie if you're interested.

---

## r/mongodb - Shorter Post

**Title:** LightODM: Python ODM with both sync and async support

Built this as a lighter alternative to Beanie for projects that need both synchronous and asynchronous MongoDB operations.

Main features:
- Pydantic v2 models with full validation
- Both sync (PyMongo) and async (Motor) with the same model
- MongoDB's native query syntax (no query builder DSL)
- Direct collection access for aggregation pipelines
- ~500 lines of code, minimal abstractions

Example:
```python
class User(MongoBaseModel):
    class Settings:
        name = "users"
    name: str
    email: str

# Use sync or async based on your context
user.save()          # Sync
await user.asave()   # Async
```

Not trying to replace Beanie - it's great for async-only apps. This is for simpler projects or when you need both sync and async.

GitHub: https://github.com/Aprova-GmbH/lightodm

---

## r/FastAPI - FastAPI-focused Post

**Title:** MongoDB ODM with sync/async support for FastAPI + background workers

If you're using FastAPI with MongoDB and have background workers (Celery, RQ, etc.), you might have run into the sync/async problem with Beanie (async-only).

Built LightODM to solve this - same Pydantic model works with both:

```python
from fastapi import FastAPI
from lightodm import MongoBaseModel

class User(MongoBaseModel):
    class Settings:
        name = "users"
    name: str
    email: str

app = FastAPI()

@app.post("/users/")
async def create_user(user: User):
    await user.asave()  # Async in endpoints
    return user

# In your background worker (sync context)
def process_user(user_id: str):
    user = User.get(user_id)  # Sync in workers
    # do work
```

Connection via environment variables (MONGO_URL, MONGO_DB_NAME) - no manual init required.

Apache 2.0, Python 3.11+
GitHub: https://github.com/Aprova-GmbH/lightodm
PyPI: `pip install lightodm`

---

## r/madeinpython - Show & Tell Style

**Title:** Made LightODM - MongoDB ODM that works both sync AND async

Hey! Published my MongoDB ODM after getting frustrated with the sync/async divide.

**The backstory:** Love Beanie, but it's async-only. When you have FastAPI (async) + background workers (sync), you're stuck using two different MongoDB libraries or doing weird async-to-sync conversions.

**My solution:**

```python
from lightodm import MongoBaseModel

class User(MongoBaseModel):
    class Settings:
        name = "users"

    name: str
    email: str

# Use sync or async - same model!
user.save()          # Sync (PyMongo)
await user.asave()   # Async (Motor)

User.find({"age": {"$gte": 18}})      # Sync
await User.afind({"age": {"$gte": 18}})  # Async
```

**Why you might like it:**
- Every method has both sync and async versions
- Uses MongoDB's native query syntax (no DSL)
- ~500 lines of code (easy to understand)
- Direct access to PyMongo/Motor collections
- Zero magic, maximum control

**Not for everyone:** If you only need async, use Beanie. If you need relationships and migrations, use Beanie. This is for simpler projects or mixed sync/async codebases.

**Status:**
- 0.1.0 on PyPI (`pip install lightodm`)
- Apache 2.0 licensed
- Python 3.11+
- Using it in production

GitHub: https://github.com/Aprova-GmbH/lightodm

Happy to answer questions!