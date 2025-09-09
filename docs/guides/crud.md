# CRUD Operations Guide

> **Goal:** Master Create, Read, Update, Delete operations with PyMongo + FastAPI  
> **Time:** 45-60 minutes  
> **Level:** 🟢 Beginner to 🟡 Intermediate

## Quick Navigation
- [Create](#create-operations) - Adding new data
- [Read](#read-operations) - Finding and retrieving data  
- [Update](#update-operations) - Modifying existing data
- [Delete](#delete-operations) - Removing data
- [Best Practices](#best-practices) - Production tips

---

## Create Operations

### Basic Insert

**What you'll learn:** How to add new documents to MongoDB

```python
from motor.motor_asyncio import AsyncIOMotorDatabase
from datetime import datetime

async def create_user(db: AsyncIOMotorDatabase, user_data: dict):
    """Insert a single user document"""
    
    # Add timestamp
    user_data["created_at"] = datetime.utcnow()
    
    # Insert document
    result = await db.users.insert_one(user_data)
    
    # Return the new document ID
    return str(result.inserted_id)

# Usage
user_id = await create_user(db, {
    "name": "John Doe",
    "email": "john@example.com",
    "age": 30
})
```

**💡 Key Points:**
- `insert_one()` adds a single document
- MongoDB auto-generates `_id` if not provided
- Always add timestamps for audit trails
- Return string version of ObjectId for JSON serialization

### Bulk Insert

<details>
<summary><strong>🔍 Show Advanced: Insert Multiple Documents</strong></summary>

```python
async def create_multiple_users(db: AsyncIOMotorDatabase, users_data: list):
    """Insert multiple users efficiently"""
    
    # Add timestamps to all documents
    for user in users_data:
        user["created_at"] = datetime.utcnow()
    
    # Bulk insert
    result = await db.users.insert_many(users_data)
    
    # Return list of IDs
    return [str(id) for id in result.inserted_ids]

# Usage
user_ids = await create_multiple_users(db, [
    {"name": "Alice", "email": "alice@example.com"},
    {"name": "Bob", "email": "bob@example.com"},
    {"name": "Charlie", "email": "charlie@example.com"}
])
```

**⚡ Performance Tip:** `insert_many()` is much faster than multiple `insert_one()` calls.

</details>

### FastAPI Integration

```python
from fastapi import FastAPI, Depends, HTTPException
from pydantic import BaseModel, EmailStr

class UserCreate(BaseModel):
    name: str
    email: EmailStr
    age: int

app = FastAPI()

@app.post("/users/", response_model=dict)
async def create_user_endpoint(
    user: UserCreate, 
    db: AsyncIOMotorDatabase = Depends(get_database)
):
    try:
        user_id = await create_user(db, user.dict())
        return {"id": user_id, "message": "User created successfully"}
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))
```

---

## Read Operations  

### Find Single Document

```python
async def get_user_by_email(db: AsyncIOMotorDatabase, email: str):
    """Find a user by email address"""
    
    user = await db.users.find_one({"email": email})
    
    if user:
        # Convert ObjectId to string for JSON serialization
        user["_id"] = str(user["_id"])
        return user
    
    return None

# Usage
user = await get_user_by_email(db, "john@example.com")
if user:
    print(f"Found user: {user['name']}")
else:
    print("User not found")
```

### Find Multiple Documents

```python
async def get_users_by_age_range(db: AsyncIOMotorDatabase, min_age: int, max_age: int):
    """Find users within age range"""
    
    cursor = db.users.find({
        "age": {"$gte": min_age, "$lte": max_age}
    })
    
    users = []
    async for user in cursor:
        user["_id"] = str(user["_id"])
        users.append(user)
    
    return users

# Usage  
young_adults = await get_users_by_age_range(db, 18, 25)
```

<details>
<summary><strong>🔍 Show Advanced: Pagination & Sorting</strong></summary>

```python
async def get_users_paginated(
    db: AsyncIOMotorDatabase, 
    page: int = 1, 
    limit: int = 10,
    sort_by: str = "created_at"
):
    """Get users with pagination and sorting"""
    
    skip = (page - 1) * limit
    
    cursor = db.users.find().sort(sort_by, -1).skip(skip).limit(limit)
    
    users = []
    async for user in cursor:
        user["_id"] = str(user["_id"])
        users.append(user)
    
    # Get total count for pagination info
    total = await db.users.count_documents({})
    
    return {
        "users": users,
        "page": page,
        "limit": limit,
        "total": total,
        "pages": (total + limit - 1) // limit
    }
```

</details>

---

## Update Operations

### Update Single Document

```python
async def update_user_name(db: AsyncIOMotorDatabase, user_id: str, new_name: str):
    """Update a user's name"""
    
    from bson import ObjectId
    
    result = await db.users.update_one(
        {"_id": ObjectId(user_id)},
        {
            "$set": {
                "name": new_name,
                "updated_at": datetime.utcnow()
            }
        }
    )
    
    return result.modified_count > 0

# Usage
success = await update_user_name(db, user_id, "John Smith")
if success:
    print("User updated successfully")
```

### Increment Values

```python
async def increment_user_login_count(db: AsyncIOMotorDatabase, user_id: str):
    """Increment user's login count"""
    
    from bson import ObjectId
    
    await db.users.update_one(
        {"_id": ObjectId(user_id)},
        {
            "$inc": {"login_count": 1},
            "$set": {"last_login": datetime.utcnow()}
        }
    )
```

---

## Delete Operations

### Delete Single Document

```python
async def delete_user(db: AsyncIOMotorDatabase, user_id: str):
    """Delete a user by ID"""
    
    from bson import ObjectId
    
    result = await db.users.delete_one({"_id": ObjectId(user_id)})
    return result.deleted_count > 0
```

### Soft Delete (Recommended)

```python
async def soft_delete_user(db: AsyncIOMotorDatabase, user_id: str):
    """Mark user as deleted instead of removing"""
    
    from bson import ObjectId
    
    result = await db.users.update_one(
        {"_id": ObjectId(user_id)},
        {
            "$set": {
                "is_deleted": True,
                "deleted_at": datetime.utcnow()
            }
        }
    )
    
    return result.modified_count > 0

# Update find queries to exclude deleted users
async def get_active_users(db: AsyncIOMotorDatabase):
    """Get only non-deleted users"""
    
    cursor = db.users.find({"is_deleted": {"$ne": True}})
    
    users = []
    async for user in cursor:
        user["_id"] = str(user["_id"])
        users.append(user)
    
    return users
```

---

## Best Practices

### ✅ Do's
- **Always validate input** with Pydantic models
- **Use soft deletes** for important data
- **Add timestamps** (created_at, updated_at)  
- **Convert ObjectId to string** for JSON responses
- **Handle exceptions** gracefully
- **Use bulk operations** for multiple documents

### ❌ Don'ts  
- Don't expose raw database errors to users
- Don't forget to handle ObjectId conversion
- Don't use hard deletes for user data
- Don't skip input validation
- Don't ignore database connection errors

### 🚀 Performance Tips
- Use indexes on frequently queried fields
- Limit result sets with `.limit()`
- Use projection to return only needed fields:
  ```python
  user = await db.users.find_one(
      {"email": email}, 
      {"name": 1, "email": 1}  # Only return name and email
  )
  ```

---

## Next Steps

Now that you understand CRUD operations:

1. 📊 **Learn Querying:** [Advanced Queries & Filtering](querying.md)
2. 🔗 **Understand Relationships:** [Document Relationships](../advanced/relationships.md)  
3. 🚀 **Optimize Performance:** [Performance Guide](../advanced/performance.md)

**Questions?** Check the [troubleshooting guide](../troubleshooting.md) or open an issue.