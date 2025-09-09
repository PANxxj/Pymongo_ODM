# 🟢 Complete Beginner Path

> **Perfect for:** Developers new to MongoDB or NoSQL databases  
> **Time Investment:** 2-3 hours  
> **End Goal:** Build and deploy a working FastAPI + MongoDB application

## Your Learning Journey

```
START → Basic Concepts → First Connection → Simple CRUD → Deploy → FINISH
  📚        📚             ⚙️             💻          🚀      🎉
 15min     20min          15min          90min       30min
```

**Progress Tracker:** Check off as you complete each step

---

## Step 1: Understanding MongoDB Basics 📚
*⏱️ 15 minutes*

### What is MongoDB?
Think of MongoDB like a digital filing cabinet where instead of storing data in rigid tables (like Excel), you store flexible documents (like JSON files).

**Traditional Database (SQL):**
```
Users Table:
| ID | Name | Email | Phone |
|----|------|-------|-------|
| 1  | John | john@example.com | 123-456-7890 |
```

**MongoDB Document:**
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "name": "John",
  "email": "john@example.com", 
  "phone": "123-456-7890",
  "address": {
    "street": "123 Main St",
    "city": "New York"
  },
  "hobbies": ["reading", "coding", "gaming"]
}
```

### Key Benefits
- ✅ **Flexible structure:** Add fields anytime without changing the whole database
- ✅ **Natural fit:** Stores data like your application uses it (JSON-like)
- ✅ **Scalable:** Handles millions of documents easily
- ✅ **Developer friendly:** Less complex joins, more intuitive queries

**✅ Checkpoint:** Can you explain what makes MongoDB different from SQL databases?

---

## Step 2: Core Concepts You Need to Know 📚  
*⏱️ 20 minutes*

### Essential Terms

| Term | Simple Explanation | SQL Equivalent |
|------|-------------------|----------------|
| **Document** | A single record (like a JSON object) | Row |
| **Collection** | Group of documents | Table |
| **Database** | Container for collections | Database |
| **Field** | A key-value pair in document | Column |
| **_id** | Unique identifier for each document | Primary Key |

### Document Structure Rules
```javascript
{
  "_id": ObjectId("..."),           // Always present, unique identifier
  "name": "John Doe",               // String field
  "age": 30,                        // Number field
  "active": true,                   // Boolean field
  "tags": ["user", "premium"],      // Array field
  "address": {                      // Embedded document (nested object)
    "street": "123 Main St",
    "city": "New York"
  },
  "created_at": ISODate("...")      // Date field
}
```

### Common MongoDB Operations
- **Create:** `insert_one()`, `insert_many()`
- **Read:** `find_one()`, `find()`
- **Update:** `update_one()`, `update_many()` 
- **Delete:** `delete_one()`, `delete_many()`

**✅ Checkpoint:** Can you identify the different field types in a MongoDB document?

---

## Step 3: Your First Connection ⚙️
*⏱️ 15 minutes*

### Install Required Packages
```bash
pip install fastapi pymongo motor uvicorn python-dotenv
```

### Basic Connection Code
```python
# database.py
import os
from motor.motor_asyncio import AsyncIOMotorClient
from dotenv import load_dotenv

load_dotenv()

# MongoDB connection string
MONGODB_URL = os.getenv("MONGODB_URL", "mongodb://localhost:27017")
DATABASE_NAME = os.getenv("DATABASE_NAME", "my_first_app")

class Database:
    client: AsyncIOMotorClient = None
    database = None

# Global database instance
db = Database()

async def connect_to_mongo():
    """Create database connection"""
    print("Connecting to MongoDB...")
    db.client = AsyncIOMotorClient(MONGODB_URL)
    db.database = db.client[DATABASE_NAME]
    print("✅ Connected to MongoDB!")

async def close_mongo_connection():
    """Close database connection"""
    print("Closing MongoDB connection...")
    db.client.close()
    print("✅ MongoDB connection closed!")

def get_database():
    """Get database instance for dependency injection"""
    return db.database
```

### Test Your Connection
```python
# test_connection.py
import asyncio
from database import connect_to_mongo, get_database

async def test_connection():
    await connect_to_mongo()
    
    database = get_database()
    
    # Try to insert a test document
    result = await database.test_collection.insert_one({
        "message": "Hello MongoDB!",
        "test": True
    })
    
    print(f"✅ Test document inserted with ID: {result.inserted_id}")
    
    # Try to find the document
    document = await database.test_collection.find_one({"test": True})
    print(f"✅ Found document: {document}")

# Run the test
if __name__ == "__main__":
    asyncio.run(test_connection())
```

**🏃‍♂️ Action:** Run `python test_connection.py` - you should see success messages!

**✅ Checkpoint:** Can you successfully connect to MongoDB and insert a test document?

---

## Step 4: Building Your First API 💻
*⏱️ 90 minutes*

Now let's build a simple **User Management API** step by step.

### 4.1 Create Data Models (15 minutes)
```python
# models.py
from pydantic import BaseModel, EmailStr, Field
from typing import Optional
from datetime import datetime
from bson import ObjectId

class PyObjectId(ObjectId):
    """Custom ObjectId class for Pydantic"""
    @classmethod
    def __get_validators__(cls):
        yield cls.validate

    @classmethod
    def validate(cls, v):
        if not ObjectId.is_valid(v):
            raise ValueError("Invalid ObjectId")
        return ObjectId(v)

    @classmethod
    def __modify_schema__(cls, field_schema):
        field_schema.update(type="string")

class UserBase(BaseModel):
    """Base user model for input"""
    name: str = Field(..., min_length=1, max_length=100)
    email: EmailStr
    age: int = Field(..., ge=0, le=150)

class UserCreate(UserBase):
    """Model for creating users"""
    pass

class UserUpdate(BaseModel):
    """Model for updating users"""
    name: Optional[str] = Field(None, min_length=1, max_length=100)
    email: Optional[EmailStr] = None
    age: Optional[int] = Field(None, ge=0, le=150)

class UserResponse(UserBase):
    """Model for API responses"""
    id: PyObjectId = Field(default_factory=PyObjectId, alias="_id")
    created_at: datetime
    updated_at: Optional[datetime] = None

    class Config:
        allow_population_by_field_name = True
        arbitrary_types_allowed = True
        json_encoders = {ObjectId: str}
```

### 4.2 Create User Repository (25 minutes)
```python
# repositories/user_repository.py
from typing import List, Optional
from datetime import datetime
from bson import ObjectId
from motor.motor_asyncio import AsyncIOMotorDatabase
from models import UserCreate, UserUpdate, UserResponse

class UserRepository:
    def __init__(self, database: AsyncIOMotorDatabase):
        self.database = database
        self.collection = database.users

    async def create_user(self, user_data: UserCreate) -> str:
        """Create a new user"""
        user_dict = user_data.dict()
        user_dict["created_at"] = datetime.utcnow()
        
        result = await self.collection.insert_one(user_dict)
        return str(result.inserted_id)

    async def get_user_by_id(self, user_id: str) -> Optional[dict]:
        """Get user by ID"""
        if not ObjectId.is_valid(user_id):
            return None
            
        user = await self.collection.find_one({"_id": ObjectId(user_id)})
        if user:
            user["_id"] = str(user["_id"])
        return user

    async def get_user_by_email(self, email: str) -> Optional[dict]:
        """Get user by email"""
        user = await self.collection.find_one({"email": email})
        if user:
            user["_id"] = str(user["_id"])
        return user

    async def get_all_users(self, limit: int = 100) -> List[dict]:
        """Get all users with limit"""
        cursor = self.collection.find().limit(limit)
        users = []
        
        async for user in cursor:
            user["_id"] = str(user["_id"])
            users.append(user)
            
        return users

    async def update_user(self, user_id: str, user_data: UserUpdate) -> bool:
        """Update user by ID"""
        if not ObjectId.is_valid(user_id):
            return False
            
        # Only update fields that are provided
        update_data = {k: v for k, v in user_data.dict().items() if v is not None}
        
        if not update_data:
            return False
            
        update_data["updated_at"] = datetime.utcnow()
        
        result = await self.collection.update_one(
            {"_id": ObjectId(user_id)},
            {"$set": update_data}
        )
        
        return result.modified_count > 0

    async def delete_user(self, user_id: str) -> bool:
        """Delete user by ID"""
        if not ObjectId.is_valid(user_id):
            return False
            
        result = await self.collection.delete_one({"_id": ObjectId(user_id)})
        return result.deleted_count > 0
```

### 4.3 Create API Endpoints (35 minutes)
```python
# main.py
from fastapi import FastAPI, HTTPException, Depends, status
from typing import List
from database import connect_to_mongo, close_mongo_connection, get_database
from models import UserCreate, UserUpdate, UserResponse
from repositories.user_repository import UserRepository

app = FastAPI(
    title="User Management API",
    description="A simple API to manage users with MongoDB",
    version="1.0.0"
)

# Database connection events
@app.on_event("startup")
async def startup_event():
    await connect_to_mongo()

@app.on_event("shutdown") 
async def shutdown_event():
    await close_mongo_connection()

# Dependency to get user repository
def get_user_repository(database = Depends(get_database)) -> UserRepository:
    return UserRepository(database)

# API Endpoints
@app.post("/users/", response_model=dict, status_code=status.HTTP_201_CREATED)
async def create_user(
    user_data: UserCreate,
    user_repo: UserRepository = Depends(get_user_repository)
):
    """Create a new user"""
    
    # Check if user already exists
    existing_user = await user_repo.get_user_by_email(user_data.email)
    if existing_user:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="User with this email already exists"
        )
    
    user_id = await user_repo.create_user(user_data)
    return {"id": user_id, "message": "User created successfully"}

@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: str,
    user_repo: UserRepository = Depends(get_user_repository)
):
    """Get user by ID"""
    user = await user_repo.get_user_by_id(user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found"
        )
    return user

@app.get("/users/", response_model=List[UserResponse])
async def get_all_users(
    limit: int = 100,
    user_repo: UserRepository = Depends(get_user_repository)
):
    """Get all users"""
    users = await user_repo.get_all_users(limit)
    return users

@app.put("/users/{user_id}", response_model=dict)
async def update_user(
    user_id: str,
    user_data: UserUpdate,
    user_repo: UserRepository = Depends(get_user_repository)
):
    """Update user by ID"""
    success = await user_repo.update_user(user_id, user_data)
    if not success:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found or no changes made"
        )
    return {"message": "User updated successfully"}

@app.delete("/users/{user_id}", response_model=dict)
async def delete_user(
    user_id: str,
    user_repo: UserRepository = Depends(get_user_repository)
):
    """Delete user by ID"""
    success = await user_repo.delete_user(user_id)
    if not success:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found"
        )
    return {"message": "User deleted successfully"}

# Health check endpoint
@app.get("/")
async def root():
    return {"message": "User Management API is running!"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 4.4 Test Your API (15 minutes)
```bash
# Start the server
uvicorn main:app --reload

# Your API will be available at:
# - http://localhost:8000 (API)
# - http://localhost:8000/docs (Interactive documentation)
```

**🧪 Test with curl:**
```bash
# Create a user
curl -X POST "http://localhost:8000/users/" \
     -H "Content-Type: application/json" \
     -d '{"name": "John Doe", "email": "john@example.com", "age": 30}'

# Get all users  
curl "http://localhost:8000/users/"

# Get user by ID (replace with actual ID)
curl "http://localhost:8000/users/507f1f77bcf86cd799439011"
```

**✅ Checkpoint:** Can you create, read, update, and delete users through your API?

---

## Step 5: Deploy Your Application 🚀
*⏱️ 30 minutes*

### 5.1 Environment Configuration
```bash
# .env file
MONGODB_URL=mongodb://localhost:27017
DATABASE_NAME=user_management_prod
```

### 5.2 Requirements File
```txt
# requirements.txt
fastapi==0.104.1
pymongo==4.6.0
motor==3.3.2
uvicorn[standard]==0.24.0
python-dotenv==1.0.0
python-multipart==0.0.6
email-validator==2.1.0
```

### 5.3 Docker Deployment (Optional)
```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - MONGODB_URL=mongodb://mongodb:27017
      - DATABASE_NAME=user_management
    depends_on:
      - mongodb
  
  mongodb:
    image: mongo:7.0
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db

volumes:
  mongodb_data:
```

**🚀 Deploy:** `docker-compose up -d`

**✅ Checkpoint:** Is your application running and accessible at http://localhost:8000/docs?

---

## 🎉 Congratulations!

You've successfully built and deployed a complete FastAPI + MongoDB application!

### What You Accomplished:
- ✅ Understood MongoDB fundamentals  
- ✅ Connected to MongoDB from Python
- ✅ Created a full CRUD API with proper error handling
- ✅ Implemented data validation with Pydantic
- ✅ Deployed your application

### Your Project Structure:
```
your-project/
├── main.py              # FastAPI application
├── database.py          # MongoDB connection
├── models.py           # Pydantic models
├── repositories/
│   └── user_repository.py  # Database operations
├── requirements.txt     # Dependencies
├── .env                # Environment variables
└── docker-compose.yml  # Docker deployment
```

## Next Steps - Choose Your Adventure:

### 🟡 **Level Up:** [Intermediate Path](intermediate.md)
- Add authentication and authorization
- Implement advanced querying
- Add relationships between collections

### 🔍 **Dive Deeper:** [Specific Topics](../README.md#-popular-use-cases)
- [Performance Optimization](../guides/performance.md)
- [Text Search](../examples/search.md)  
- [File Upload Handling](../examples/file-storage.md)

### 💼 **Real Projects:** [Example Applications](../examples/)
- E-commerce API
- Blog Platform
- Task Management System

---

**🎯 Quick Survey:** How was this learning path?
- Too fast / Too slow?
- Missing explanations?
- Want more examples?

[Give feedback](https://github.com/your-repo/issues) to help improve this guide!