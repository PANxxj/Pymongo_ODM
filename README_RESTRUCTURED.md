# PyMongo + FastAPI: Professional Guide

[![Python](https://img.shields.io/badge/python-v3.8+-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.68+-green.svg)](https://fastapi.tiangolo.com/)
[![PyMongo](https://img.shields.io/badge/PyMongo-4.0+-orange.svg)](https://pymongo.readthedocs.io/)

> **Enterprise-grade patterns for integrating PyMongo with FastAPI**

## 🚀 Quick Start (5 minutes)

```bash
# Install dependencies
pip install fastapi pymongo motor uvicorn

# Run example
git clone <repo>
cd pymongo-fastapi
uvicorn app.main:app --reload
```

**✅ Result:** Working API at `http://localhost:8000/docs`

## 📚 Learning Paths

Choose your path based on your experience:

### 🟢 **New to MongoDB?** → [Complete Beginner Path](docs/paths/beginner.md)
- ⏱️ **Time:** 2-3 hours
- 🎯 **Goal:** Build your first API with MongoDB
- **Covers:** Basic concepts → Simple CRUD → Deploy

### 🟡 **FastAPI Developer?** → [Integration Path](docs/paths/integration.md)  
- ⏱️ **Time:** 3-4 hours
- 🎯 **Goal:** Add MongoDB to existing FastAPI knowledge
- **Covers:** Connection patterns → Advanced queries → Best practices

### 🔴 **MongoDB Expert?** → [Advanced Path](docs/paths/advanced.md)
- ⏱️ **Time:** 4-6 hours  
- 🎯 **Goal:** Production-ready enterprise patterns
- **Covers:** Optimization → Scaling → Monitoring

## 🎯 Popular Use Cases

**Jump directly to what you need:**

| I want to... | Guide | Time |
|--------------|-------|------|
| Store user data | [User Management API](docs/examples/user-api.md) | 30 min |
| Build an e-commerce API | [Product Catalog](docs/examples/ecommerce.md) | 45 min |
| Handle file uploads | [File Storage](docs/examples/file-storage.md) | 30 min |
| Add search functionality | [Text Search](docs/examples/search.md) | 45 min |
| Optimize performance | [Performance Guide](docs/guides/performance.md) | 60 min |

## 📖 Core Documentation

### Essential Guides
- 🏗️ [Architecture Overview](docs/architecture.md) - How everything fits together
- ⚙️ [Setup & Configuration](docs/setup.md) - Production-ready setup
- 🔧 [CRUD Operations](docs/guides/crud.md) - Create, Read, Update, Delete
- 🔍 [Querying & Filtering](docs/guides/querying.md) - Find your data
- 📊 [Aggregation Pipeline](docs/guides/aggregation.md) - Complex data processing

### Advanced Topics
- 🔗 [Document Relationships](docs/advanced/relationships.md)
- 🚀 [Performance Optimization](docs/advanced/performance.md)
- 🔒 [Security Best Practices](docs/advanced/security.md)
- 🧪 [Testing Strategies](docs/advanced/testing.md)
- 📦 [Production Deployment](docs/advanced/deployment.md)

## 💡 Quick Examples

### Basic Connection
```python
from motor.motor_asyncio import AsyncIOMotorClient

client = AsyncIOMotorClient("mongodb://localhost:27017")
db = client.my_database
```

### Simple CRUD
```python
# Create
result = await db.users.insert_one({"name": "John", "email": "john@example.com"})

# Read
user = await db.users.find_one({"email": "john@example.com"})

# Update
await db.users.update_one({"_id": user["_id"]}, {"$set": {"name": "John Doe"}})

# Delete
await db.users.delete_one({"_id": user["_id"]})
```

### FastAPI Integration
```python
from fastapi import FastAPI, Depends
from motor.motor_asyncio import AsyncIOMotorDatabase

app = FastAPI()

async def get_database() -> AsyncIOMotorDatabase:
    client = AsyncIOMotorClient("mongodb://localhost:27017")
    return client.my_database

@app.post("/users/")
async def create_user(user_data: dict, db: AsyncIOMotorDatabase = Depends(get_database)):
    result = await db.users.insert_one(user_data)
    return {"id": str(result.inserted_id)}
```

## 🛠️ Project Structure

```
project/
├── app/
│   ├── main.py              # FastAPI app
│   ├── models/              # Pydantic models
│   ├── repositories/        # Database operations
│   └── routes/              # API endpoints
├── tests/
├── docs/
└── requirements.txt
```

## ❓ Need Help?

- 📚 **Learning:** Start with [Beginner Path](docs/paths/beginner.md)
- 🐛 **Issues:** Check [Troubleshooting](docs/troubleshooting.md)
- 💬 **Questions:** Open an issue
- 📋 **Examples:** Browse [examples/](examples/) folder

---

## What's Different About This Guide?

- ✅ **Production-ready** code, not toy examples
- ✅ **Progressive learning** from basic to expert
- ✅ **Real-world patterns** used in enterprise applications  
- ✅ **Complete examples** you can copy and run
- ✅ **Performance-focused** with optimization tips

**Next Step:** Choose your [learning path](#-learning-paths) above!