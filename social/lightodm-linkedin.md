🚀 Introducing LightODM: A Lightweight MongoDB ODM for Python

I've been using Beanie for MongoDB projects and it's great, but I kept hitting the same friction points:
- Needed both sync and async in the same codebase
- Wanted MongoDB's native query syntax instead of learning a DSL
- Needed to drop down to raw PyMongo/Motor without fighting abstractions
- For simpler projects, the feature set felt heavier than necessary

So I built LightODM - a minimal alternative that focuses on the essentials:

✅ Works with both sync (PyMongo) and async (Motor) in the same model
✅ Uses Pydantic v2 for validation and serialization
✅ Lets you use MongoDB's native query syntax (no DSL to learn)
✅ Exposes the actual PyMongo/Motor collections when you need them
✅ Stays small (~500 lines) with just 3 dependencies: pydantic, pymongo, motor

Perfect for:
- Projects needing both sync and async operations
- Developers who already know MongoDB query syntax
- Microservices and simple to medium complexity projects
- When you want direct MongoDB control with minimal abstractions

Not trying to replace Beanie - it's excellent for complex applications with relationships and migrations. LightODM is for when you want Pydantic validation without giving up control.

📦 Available now: pip install lightodm
🔗 GitHub: https://github.com/Aprova-GmbH/lightodm
📖 Full blog post: https://vykhand.github.io/LightODM-Minimalistic-MongoDB-ODM/

#Python #MongoDB #OpenSource #BackendDevelopment #AsyncIO #FastAPI