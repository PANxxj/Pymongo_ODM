# PyMongo with FastAPI: Professional Development Guide

[![Python](https://img.shields.io/badge/python-v3.8+-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.68+-green.svg)](https://fastapi.tiangolo.com/)
[![PyMongo](https://img.shields.io/badge/PyMongo-4.0+-orange.svg)](https://pymongo.readthedocs.io/)
[![MongoDB](https://img.shields.io/badge/MongoDB-5.0+-brightgreen.svg)](https://www.mongodb.com/)
<!-- [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) -->

## Overview

This comprehensive guide provides enterprise-grade patterns and best practices for integrating PyMongo with FastAPI. Designed for professional developers, it covers architectural patterns, performance optimization, and production deployment strategies.

### Target Audience
- **Backend Developers** building REST APIs with FastAPI
- **Database Engineers** implementing MongoDB solutions
- **DevOps Engineers** deploying MongoDB applications
- **Technical Architects** designing scalable systems

### Prerequisites
- Python 3.8+ development experience
- Basic understanding of REST API principles
- Familiarity with database concepts
- MongoDB installation (local or cloud)

## Architecture Overview

```mermaid
graph TB
    A[FastAPI Application] --> B[Pydantic Models]
    A --> C[Route Handlers]
    C --> D[Service Layer]
    D --> E[Repository Pattern]
    E --> F[PyMongo Driver]
    F --> G[MongoDB Database]
    
    H[Authentication] --> A
    I[Validation] --> B
    J[Error Handling] --> C
    K[Logging] --> D
    L[Connection Pool] --> F
```

## Table of Contents

### 🏗️ **FOUNDATION** (Essential Concepts)
1. [MongoDB Architecture Overview](#mongodb-architecture-overview)
2. [Environment Setup & Configuration](#environment-setup-configuration)
3. [Database Design Principles](#database-design-principles)
4. [Setup and First Connection](#setup-and-first-connection)

### ⚙️ **CORE IMPLEMENTATION** (Development Patterns)
5. [FastAPI Integration Architecture](#fastapi-integration-architecture)
6. [Data Models and Schema](#data-models-and-schema)
7. [FastAPI Integration Basics](#fastapi-integration-basics)
8. [Basic CRUD Operations](#basic-crud-operations)
9. [Simple Queries and Filters](#simple-queries-and-filters)

### 🔍 **DATA OPERATIONS** (Advanced Querying)
10. [Document Relationships in MongoDB - Complete Guide](#document-relationships-in-mongodb---complete-guide)
11. [Querying and Filtering](#querying-and-filtering)
12. [Aggregation Pipeline - The Complete Simple Guide](#aggregation-pipeline---the-complete-simple-guide)
13. [MongoDB Symbols Explained Simply](#mongodb-symbols-explained-simply)
14. [Working with Arrays and Objects](#working-with-arrays-and-objects)

### 🚀 **PRODUCTION READY** (Enterprise Patterns)
15. [Document Relationships Made Simple](#document-relationships-made-simple)
16. [What is MongoDB? Simple Introduction](#what-is-mongodb-simple-introduction)
17. [Your First MongoDB Document](#your-first-mongodb-document)
18. [MongoDB Symbols and Operators Explained](#mongodb-symbols-and-operators-explained)
19. [Sample Dataset](#sample-dataset)
20. [FastAPI Application Setup](#1-fastapi-application-setup)

### 🎯 **DEPLOYMENT & MONITORING** (Operations)
21. [Pydantic Models for Data Validation](#1-pydantic-models-for-data-validation)
22. [Categories Collection](#1-categories-collection)
23. [Create Operations](#1-create-operations)
24. [Understanding MongoDB Relationships](#understanding-mongodb-relationships)

---

## 🏗️ FOUNDATION

### MongoDB Architecture Overview

MongoDB is a document-oriented NoSQL database designed for scalability and developer productivity. Understanding its architecture is crucial for building efficient applications.

#### Document Model Fundamentals

```python
from typing import Dict, List, Optional, Any
from datetime import datetime
from enum import Enum

class DocumentModel:
    """
    MongoDB documents are BSON (Binary JSON) objects that provide:
    - Rich data types (ObjectId, DateTime, Binary, etc.)
    - Nested structures (embedded documents and arrays)
    - Schema flexibility with optional validation
    - Horizontal scalability through sharding
    """
    
    @staticmethod
    def example_document() -> Dict[str, Any]:
        return {
            "_id": "ObjectId('507f1f77bcf86cd799439011')",  # Primary key
            "created_at": datetime.utcnow(),                 # Native datetime
            "metadata": {                                    # Embedded document
                "version": "1.0.0",
                "tags": ["production", "api"]               # Array field
            },
            "geolocation": {                                 # GeoJSON support
                "type": "Point",
                "coordinates": [-73.856077, 40.848447]
            }
        }
```

#### Key Architectural Concepts

```json
{
  "document_structure": {
    "advantages": [
      "Natural mapping to application objects",
      "Atomic operations at document level",
      "Rich query capabilities including nested fields",
      "Horizontal scaling through auto-sharding"
    ],
    "considerations": [
      "16MB document size limit",
      "No multi-document ACID by default (requires transactions)",
      "Eventual consistency in replica sets",
      "Query optimization requires proper indexing"
    ]
  },
  "storage_engine": {
    "wiredTiger": {
      "compression": "snappy, zlib, or zstd compression",
      "concurrency": "Document-level locking",
      "durability": "Journaling with checkpoints",
      "memory_usage": "Configurable cache size"
    }
  },
  "replication": {
    "replica_set": "Primary-secondary architecture for high availability",
    "write_concern": "Configurable acknowledgment levels",
    "read_preference": "Primary, secondary, or nearest read routing"
  }
}
```

### Environment Setup & Configuration

#### Production-Grade Configuration

```python
# config/database.py
import os
import ssl
from typing import Optional
from dataclasses import dataclass
from pymongo import MongoClient
from motor.motor_asyncio import AsyncIOMotorClient
from pymongo.errors import ServerSelectionTimeoutError
import logging

logger = logging.getLogger(__name__)

@dataclass
class DatabaseConfig:
    """Production-ready MongoDB configuration"""
    
    # Connection Configuration
    mongodb_url: str = os.getenv("MONGODB_URL", "mongodb://localhost:27017")
    database_name: str = os.getenv("DATABASE_NAME", "production_db")
    
    # Connection Pool Configuration
    max_pool_size: int = int(os.getenv("MONGO_MAX_POOL_SIZE", "100"))
    min_pool_size: int = int(os.getenv("MONGO_MIN_POOL_SIZE", "10"))
    max_idle_time_ms: int = int(os.getenv("MONGO_MAX_IDLE_TIME_MS", "30000"))
    
    # Timeout Configuration
    server_selection_timeout_ms: int = int(os.getenv("MONGO_SERVER_SELECTION_TIMEOUT_MS", "5000"))
    socket_timeout_ms: int = int(os.getenv("MONGO_SOCKET_TIMEOUT_MS", "20000"))
    connect_timeout_ms: int = int(os.getenv("MONGO_CONNECT_TIMEOUT_MS", "20000"))
    
    # Write Concern Configuration
    write_concern_w: str = os.getenv("MONGO_WRITE_CONCERN_W", "majority")
    write_concern_j: bool = os.getenv("MONGO_WRITE_CONCERN_J", "true").lower() == "true"
    write_concern_wtimeout: int = int(os.getenv("MONGO_WRITE_CONCERN_WTIMEOUT", "10000"))
    
    # Read Preference
    read_preference: str = os.getenv("MONGO_READ_PREFERENCE", "primary")
    
    # SSL Configuration
    ssl_enabled: bool = os.getenv("MONGO_SSL_ENABLED", "false").lower() == "true"
    ssl_cert_reqs: int = ssl.CERT_REQUIRED if ssl_enabled else ssl.CERT_NONE
    ssl_ca_certs: Optional[str] = os.getenv("MONGO_SSL_CA_CERTS")
    ssl_cert_file: Optional[str] = os.getenv("MONGO_SSL_CERT_FILE")
    ssl_key_file: Optional[str] = os.getenv("MONGO_SSL_KEY_FILE")

class DatabaseManager:
    """Singleton database connection manager"""
    
    _instance: Optional['DatabaseManager'] = None
    _client: Optional[MongoClient] = None
    _async_client: Optional[AsyncIOMotorClient] = None
    
    def __new__(cls) -> 'DatabaseManager':
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        if not hasattr(self, 'initialized'):
            self.config = DatabaseConfig()
            self.initialized = True
    
    def get_sync_client(self) -> MongoClient:
        """Get synchronous MongoDB client with connection pooling"""
        if self._client is None:
            try:
                client_kwargs = {
                    'maxPoolSize': self.config.max_pool_size,
                    'minPoolSize': self.config.min_pool_size,
                    'maxIdleTimeMS': self.config.max_idle_time_ms,
                    'serverSelectionTimeoutMS': self.config.server_selection_timeout_ms,
                    'socketTimeoutMS': self.config.socket_timeout_ms,
                    'connectTimeoutMS': self.config.connect_timeout_ms,
                }
                
                # SSL Configuration
                if self.config.ssl_enabled:
                    client_kwargs.update({
                        'ssl': True,
                        'ssl_cert_reqs': self.config.ssl_cert_reqs,
                        'ssl_ca_certs': self.config.ssl_ca_certs,
                        'ssl_certfile': self.config.ssl_cert_file,
                        'ssl_keyfile': self.config.ssl_key_file,
                    })
                
                self._client = MongoClient(self.config.mongodb_url, **client_kwargs)
                
                # Test connection
                self._client.admin.command('ping')
                logger.info("MongoDB sync client connected successfully")
                
            except ServerSelectionTimeoutError as e:
                logger.error(f"Failed to connect to MongoDB: {e}")
                raise
                
        return self._client
    
    def get_async_client(self) -> AsyncIOMotorClient:
        """Get asynchronous MongoDB client for FastAPI"""
        if self._async_client is None:
            client_kwargs = {
                'maxPoolSize': self.config.max_pool_size,
                'minPoolSize': self.config.min_pool_size,
                'maxIdleTimeMS': self.config.max_idle_time_ms,
                'serverSelectionTimeoutMS': self.config.server_selection_timeout_ms,
            }
            
            if self.config.ssl_enabled:
                client_kwargs.update({
                    'ssl': True,
                    'ssl_cert_reqs': self.config.ssl_cert_reqs,
                })
            
            self._async_client = AsyncIOMotorClient(self.config.mongodb_url, **client_kwargs)
            logger.info("MongoDB async client initialized")
            
        return self._async_client
    
    def get_database(self, async_client: bool = True):
        """Get database instance"""
        if async_client:
            client = self.get_async_client()
        else:
            client = self.get_sync_client()
        
        return client[self.config.database_name]
    
    async def close_connections(self):
        """Gracefully close all connections"""
        if self._client:
            self._client.close()
            logger.info("Sync MongoDB client closed")
        
        if self._async_client:
            self._async_client.close()
            logger.info("Async MongoDB client closed")

# Initialize global database manager
db_manager = DatabaseManager()
```

#### Environment Configuration

```bash
# .env file for production deployment
MONGODB_URL=mongodb+srv://username:password@cluster.mongodb.net/
DATABASE_NAME=production_app
MONGO_MAX_POOL_SIZE=100
MONGO_MIN_POOL_SIZE=10
MONGO_SERVER_SELECTION_TIMEOUT_MS=5000
MONGO_WRITE_CONCERN_W=majority
MONGO_WRITE_CONCERN_J=true
MONGO_SSL_ENABLED=true
MONGO_READ_PREFERENCE=primaryPreferred
```

**Configuration Details:**

```json
{
  "connection_pooling": {
    "purpose": "Reuse connections to improve performance and reduce overhead",
    "max_pool_size": "Maximum connections per client instance",
    "min_pool_size": "Minimum connections maintained in pool",
    "max_idle_time": "Time before idle connections are closed"
  },
  "write_concerns": {
    "w_majority": "Wait for majority of replica set members to acknowledge",
    "j_true": "Wait for write to be committed to journal",
    "wtimeout": "Maximum time to wait for write acknowledgment"
  },
  "read_preferences": {
    "primary": "Read from primary only (strong consistency)",
    "primaryPreferred": "Read from primary, fallback to secondary",
    "secondary": "Read from secondary only (eventual consistency)",
    "nearest": "Read from nearest member (lowest latency)"
  },
  "ssl_configuration": {
    "production_requirement": "Always enable SSL/TLS in production",
    "certificate_validation": "Verify server certificates for security",
    "client_certificates": "Use for mutual TLS authentication"
  }
}
```

### Database Design Principles

#### Schema Design Patterns

```python
from abc import ABC, abstractmethod
from typing import Dict, Any, List, Optional
from enum import Enum
from dataclasses import dataclass
import uuid

class SchemaPattern(Enum):
    """Common MongoDB schema design patterns"""
    EMBEDDED = "embedded"           # One-to-few relationships
    REFERENCE = "reference"         # One-to-many relationships  
    SUBSET = "subset"              # Frequently accessed subset
    COMPUTED = "computed"          # Pre-computed values
    BUCKET = "bucket"              # Time-series data
    OUTLIER = "outlier"            # Handle large documents

@dataclass
class SchemaDesignRule:
    """Rule for schema design decisions"""
    pattern: SchemaPattern
    use_when: List[str]
    avoid_when: List[str]
    max_documents: Optional[int] = None
    max_size_mb: Optional[float] = None

class SchemaDesignGuide:
    """Enterprise schema design guidelines"""
    
    DESIGN_RULES = {
        SchemaPattern.EMBEDDED: SchemaDesignRule(
            pattern=SchemaPattern.EMBEDDED,
            use_when=[
                "One-to-few relationships (1:100 max)",
                "Data accessed together frequently",
                "Child data doesn't grow unbounded",
                "Atomic updates needed across related data"
            ],
            avoid_when=[
                "One-to-many with thousands of child documents",
                "Child data accessed independently",
                "Risk of 16MB document limit",
                "High write load on child data"
            ],
            max_documents=100,
            max_size_mb=16.0
        ),
        
        SchemaPattern.REFERENCE: SchemaDesignRule(
            pattern=SchemaPattern.REFERENCE,
            use_when=[
                "One-to-many or many-to-many relationships",
                "Child data queried independently", 
                "Large or growing child data",
                "Different access patterns for parent/child"
            ],
            avoid_when=[
                "Always accessed together",
                "Small, bounded child data",
                "Need atomic updates across documents",
                "Simple one-to-few relationships"
            ]
        ),
        
        SchemaPattern.SUBSET: SchemaDesignRule(
            pattern=SchemaPattern.SUBSET,
            use_when=[
                "Large documents with frequently accessed subset",
                "Need fast access to summary data",
                "Reduce network transfer and memory usage",
                "Optimize for common query patterns"
            ],
            avoid_when=[
                "All data accessed equally",
                "Simple document structure",
                "Data synchronization complexity not worth it"
            ]
        )
    }
    
    @classmethod
    def recommend_pattern(cls, 
                         relationship_type: str,
                         child_count: int,
                         access_pattern: str,
                         update_frequency: str) -> SchemaPattern:
        """Recommend schema pattern based on use case"""
        
        if relationship_type == "one-to-few" and child_count <= 100:
            if access_pattern == "together" and update_frequency == "low":
                return SchemaPattern.EMBEDDED
        
        if relationship_type == "one-to-many" and child_count > 100:
            return SchemaPattern.REFERENCE
            
        if relationship_type == "many-to-many":
            return SchemaPattern.REFERENCE
            
        return SchemaPattern.REFERENCE  # Safe default

# Example: E-commerce Product Catalog Schema
class ProductCatalogSchema:
    """Professional e-commerce schema design example"""
    
    @staticmethod
    def product_document() -> Dict[str, Any]:
        """Main product document with embedded and referenced data"""
        return {
            "_id": uuid.uuid4(),
            
            # Core product data (always accessed)
            "sku": "PROD-001",
            "name": "Professional Laptop",
            "slug": "professional-laptop",
            
            # Category reference (one-to-many)
            "category_id": uuid.uuid4(),
            "category_path": ["Electronics", "Computers", "Laptops"],  # Denormalized for breadcrumbs
            
            # Supplier reference
            "supplier_id": uuid.uuid4(),
            "supplier_name": "TechCorp Inc",  # Denormalized for display
            
            # Pricing and inventory
            "pricing": {
                "base_price": 999.99,
                "currency": "USD",
                "tax_class": "standard",
                "discounts": []  # Embedded array for active discounts
            },
            
            "inventory": {
                "stock_level": 150,
                "reserved": 5,
                "available": 145,
                "warehouse_locations": ["WH-001", "WH-002"]
            },
            
            # Product attributes (embedded for fast access)
            "attributes": {
                "brand": "TechBrand",
                "model": "TB-2024",
                "color": "Space Gray",
                "weight_kg": 1.8,
                "dimensions": {
                    "length_cm": 35.5,
                    "width_cm": 24.1,
                    "height_cm": 1.9
                }
            },
            
            # SEO and metadata
            "seo": {
                "meta_title": "Professional Laptop - High Performance Computing",
                "meta_description": "Powerful laptop for professional use...",
                "keywords": ["laptop", "professional", "high-performance"]
            },
            
            # Media (referenced for large files)
            "media": {
                "primary_image_id": uuid.uuid4(),
                "image_ids": [uuid.uuid4(), uuid.uuid4()],  # References to media collection
                "video_ids": []
            },
            
            # Status and workflow
            "status": "active",
            "workflow_state": "published",
            "visibility": "public",
            
            # Audit fields
            "created_at": "2024-01-15T10:00:00Z",
            "updated_at": "2024-01-15T10:00:00Z",
            "created_by": uuid.uuid4(),
            "version": 1,
            
            # Search optimization
            "search_keywords": ["laptop", "professional", "techbrand", "computing"],
            "search_score": 95.5
        }
    
    @staticmethod
    def product_reviews_collection() -> List[Dict[str, Any]]:
        """Separate collection for product reviews (one-to-many reference)"""
        return [
            {
                "_id": uuid.uuid4(),
                "product_id": uuid.uuid4(),  # Reference to product
                "customer_id": uuid.uuid4(),
                "rating": 5,
                "title": "Excellent laptop for development work",
                "review_text": "Fast, reliable, great build quality...",
                "verified_purchase": True,
                "helpful_votes": 23,
                "created_at": "2024-02-01T14:30:00Z",
                "moderation_status": "approved"
            }
        ]
    
    @staticmethod
    def product_analytics_collection() -> Dict[str, Any]:
        """Analytics data in separate collection (time-series pattern)"""
        return {
            "_id": uuid.uuid4(),
            "product_id": uuid.uuid4(),
            "date": "2024-03-15",
            "metrics": {
                "page_views": 1250,
                "unique_visitors": 890,
                "add_to_cart": 45,
                "purchases": 12,
                "conversion_rate": 0.96,
                "revenue": 11999.88
            },
            "traffic_sources": {
                "organic": 45.2,
                "paid": 23.8, 
                "social": 12.5,
                "email": 8.9,
                "direct": 9.6
            }
        }
```

**Schema Design Decision Matrix:**

```json
{
  "design_decision_matrix": {
    "embedded_documents": {
      "use_cases": [
        "Product with variants (size, color) - max 50 variants",
        "User with addresses - max 10 addresses", 
        "Order with order items - max 100 items",
        "Blog post with comments - max 200 comments"
      ],
      "performance_benefits": [
        "Single query retrieves all related data",
        "Atomic updates across related entities",
        "Better data locality and cache performance",
        "Reduced network round trips"
      ],
      "limitations": [
        "16MB document size limit",
        "Cannot query embedded data independently",
        "Index limitations on nested arrays",
        "Memory usage scales with document size"
      ]
    },
    
    "referenced_documents": {
      "use_cases": [
        "Product categories with thousands of products",
        "User with order history - unlimited orders",
        "Blog categories with posts",
        "Customer reviews for products"
      ],
      "performance_benefits": [
        "No document size limitations",
        "Independent querying of related data",
        "Efficient indexing strategies",
        "Granular access control"
      ],
      "considerations": [
        "Requires $lookup for joins (expensive)",
        "Multiple queries for complete data",
        "No automatic referential integrity",
        "Potential for orphaned references"
      ]
    },
    
    "hybrid_approach": {
      "strategy": "Combine embedded and referenced patterns",
      "example": "Product with embedded attributes and referenced reviews",
      "benefits": [
        "Optimize for specific access patterns", 
        "Balance query performance and flexibility",
        "Denormalize frequently accessed data",
        "Reference rarely accessed or large data"
      ]
    }
  },
  
  "anti_patterns": {
    "massive_arrays": {
      "problem": "Embedding unlimited array growth",
      "example": "User document with all order history embedded",
      "solution": "Use references for unbounded relationships"
    },
    "deep_nesting": {
      "problem": "More than 3-4 levels of nesting",
      "example": "category.subcategory.subsubcategory.product",
      "solution": "Flatten structure or use references"
    },
    "large_documents": {
      "problem": "Documents approaching 16MB limit",
      "example": "Product with hundreds of high-res images embedded",
      "solution": "Reference large binary data externally"
    }
  }
}
```

---

## ⚙️ CORE IMPLEMENTATION

### FastAPI Integration Architecture

#### Application Structure

```python
# app/main.py - Production FastAPI application
from contextlib import asynccontextmanager
from fastapi import FastAPI, Request, status
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.gzip import GZipMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from fastapi.responses import JSONResponse
import logging
import time
import uuid
from typing import Dict, Any

from app.config.database import db_manager
from app.config.settings import Settings
from app.middleware.logging import LoggingMiddleware
from app.middleware.monitoring import MonitoringMiddleware
from app.routers import products, users, orders, health
from app.exceptions import APIException, ValidationException

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

@asynccontextmanager
async def lifespan(app: FastAPI):
    """Application lifespan management for startup and shutdown events"""
    
    # Startup
    logger.info("🚀 Starting FastAPI application")
    
    try:
        # Initialize database connections
        db = db_manager.get_database()
        await db.command('ping')
        logger.info("✅ Database connection established")
        
        # Create indexes
        await create_database_indexes()
        logger.info("✅ Database indexes created")
        
        # Initialize caches
        await initialize_application_cache()
        logger.info("✅ Application cache initialized")
        
    except Exception as e:
        logger.error(f"❌ Startup failed: {e}")
        raise
    
    yield
    
    # Shutdown  
    logger.info("🔄 Shutting down application")
    try:
        await db_manager.close_connections()
        logger.info("✅ Database connections closed")
    except Exception as e:
        logger.error(f"❌ Shutdown error: {e}")

# Initialize FastAPI with production settings
settings = Settings()
app = FastAPI(
    title="Enterprise E-commerce API",
    description="Production-ready e-commerce API built with FastAPI and MongoDB",
    version="1.0.0",
    docs_url="/api/docs" if settings.DEBUG else None,
    redoc_url="/api/redoc" if settings.DEBUG else None,
    openapi_url="/api/openapi.json" if settings.DEBUG else None,
    lifespan=lifespan
)

# Security middleware
app.add_middleware(
    TrustedHostMiddleware, 
    allowed_hosts=settings.ALLOWED_HOSTS
)

# Performance middleware
app.add_middleware(GZipMiddleware, minimum_size=1000)

# CORS middleware for production
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE", "PATCH"],
    allow_headers=["*"],
    expose_headers=["X-Total-Count", "X-Page-Count"]
)

# Custom middleware
app.add_middleware(MonitoringMiddleware)
app.add_middleware(LoggingMiddleware)

# Exception handlers
@app.exception_handler(APIException)
async def api_exception_handler(request: Request, exc: APIException):
    """Handle custom API exceptions"""
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "type": exc.__class__.__name__,
                "message": exc.message,
                "details": exc.details,
                "request_id": str(uuid.uuid4()),
                "timestamp": time.time()
            }
        }
    )

@app.exception_handler(ValidationException) 
async def validation_exception_handler(request: Request, exc: ValidationException):
    """Handle validation exceptions"""
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "error": {
                "type": "ValidationError",
                "message": "Request validation failed",
                "validation_errors": exc.errors,
                "request_id": str(uuid.uuid4())
            }
        }
    )

# Include routers with version prefixes
app.include_router(health.router, prefix="/api/v1", tags=["health"])
app.include_router(products.router, prefix="/api/v1", tags=["products"])
app.include_router(users.router, prefix="/api/v1", tags=["users"])  
app.include_router(orders.router, prefix="/api/v1", tags=["orders"])

# Root endpoint
@app.get("/", include_in_schema=False)
async def root():
    """API root endpoint"""
    return {
        "service": "Enterprise E-commerce API",
        "version": "1.0.0",
        "status": "operational",
        "documentation": "/api/docs"
    }

async def create_database_indexes():
    """Create optimized database indexes for production"""
    db = db_manager.get_database()
    
    # Product indexes
    await db.products.create_index("sku", unique=True)
    await db.products.create_index("slug", unique=True)
    await db.products.create_index([("name", "text"), ("description", "text")])
    await db.products.create_index("category_id")
    await db.products.create_index("supplier_id")
    await db.products.create_index([("status", 1), ("created_at", -1)])
    
    # User indexes
    await db.users.create_index("email", unique=True)
    await db.users.create_index("username", unique=True)
    await db.users.create_index([("status", 1), ("created_at", -1)])
    
    # Order indexes  
    await db.orders.create_index("order_number", unique=True)
    await db.orders.create_index("customer_id")
    await db.orders.create_index([("status", 1), ("created_at", -1)])
    await db.orders.create_index("created_at", expireAfterSeconds=86400 * 30)  # TTL for 30 days

async def initialize_application_cache():
    """Initialize application-level caching"""
    # Redis cache initialization would go here
    pass

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        "app.main:app",
        host="0.0.0.0", 
        port=8000,
        reload=settings.DEBUG,
        workers=1 if settings.DEBUG else 4
    )
```

#### Dependency Injection System

```python
# app/dependencies.py - Production dependency injection
from fastapi import Depends, HTTPException, status, Request
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from motor.motor_asyncio import AsyncIOMotorDatabase
from typing import Optional, Dict, Any
import jwt
from datetime import datetime, timedelta

from app.config.database import db_manager
from app.config.settings import Settings
from app.models.user import User, UserRole
from app.services.auth import AuthService
from app.services.cache import CacheService

settings = Settings()
security = HTTPBearer(auto_error=False)

# Database dependencies
async def get_database() -> AsyncIOMotorDatabase:
    """Get database instance for dependency injection"""
    return db_manager.get_database()

async def get_auth_service() -> AuthService:
    """Get authentication service instance"""
    return AuthService()

async def get_cache_service() -> CacheService:
    """Get cache service instance"""  
    return CacheService()

# Authentication dependencies
async def get_current_user(
    credentials: Optional[HTTPAuthorizationCredentials] = Depends(security),
    db: AsyncIOMotorDatabase = Depends(get_database),
    auth_service: AuthService = Depends(get_auth_service)
) -> Optional[User]:
    """Get current authenticated user"""
    
    if not credentials:
        return None
        
    try:
        payload = jwt.decode(
            credentials.credentials,
            settings.JWT_SECRET_KEY,
            algorithms=[settings.JWT_ALGORITHM]
        )
        
        user_id = payload.get("sub")
        if not user_id:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Invalid authentication token"
            )
            
        user = await auth_service.get_user_by_id(user_id)
        if not user or not user.is_active:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED, 
                detail="User not found or inactive"
            )
            
        return user
        
    except jwt.ExpiredSignatureError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Authentication token expired"
        )
    except jwt.JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication token"
        )

async def require_authentication(
    current_user: User = Depends(get_current_user)
) -> User:
    """Require valid authentication"""
    if not current_user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Authentication required"
        )
    return current_user

async def require_admin_role(
    current_user: User = Depends(require_authentication)
) -> User:
    """Require admin role"""
    if current_user.role != UserRole.ADMIN:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Admin role required"
        )
    return current_user

# Request context dependencies
async def get_request_context(request: Request) -> Dict[str, Any]:
    """Get request context information"""
    return {
        "request_id": str(uuid.uuid4()),
        "client_ip": request.client.host,
        "user_agent": request.headers.get("user-agent"),
        "timestamp": datetime.utcnow(),
        "method": request.method,
        "url": str(request.url)
    }

# Pagination dependencies
async def get_pagination_params(
    page: int = Query(1, ge=1, description="Page number"),
    size: int = Query(20, ge=1, le=100, description="Page size")
) -> Dict[str, int]:
    """Get pagination parameters with validation"""
    return {
        "page": page,
        "size": size,
        "skip": (page - 1) * size
    }

# Rate limiting dependency
async def rate_limit_check(
    request: Request,
    cache_service: CacheService = Depends(get_cache_service)
):
    """Check rate limiting for API endpoints"""
    client_ip = request.client.host
    rate_limit_key = f"rate_limit:{client_ip}"
    
    current_requests = await cache_service.increment(rate_limit_key)
    
    if current_requests == 1:
        await cache_service.expire(rate_limit_key, 60)  # 1 minute window
    
    if current_requests > settings.RATE_LIMIT_PER_MINUTE:
        raise HTTPException(
            status_code=status.HTTP_429_TOO_MANY_REQUESTS,
            detail="Rate limit exceeded"
        )
```

**Dependency Architecture Benefits:**

```json
{
  "dependency_injection_benefits": {
    "separation_of_concerns": {
      "description": "Clean separation between business logic and infrastructure",
      "implementation": "Service layer dependencies injected into route handlers",
      "benefits": ["Testable code", "Maintainable architecture", "Loose coupling"]
    },
    
    "resource_management": {
      "description": "Automatic resource lifecycle management",
      "implementation": "Database connections, cache instances managed centrally",
      "benefits": ["Consistent resource usage", "Automatic cleanup", "Performance optimization"]
    },
    
    "security_integration": {
      "description": "Authentication and authorization as reusable dependencies",
      "implementation": "JWT validation, role checking, rate limiting",
      "benefits": ["Consistent security", "Easy to audit", "Reusable across endpoints"]
    },
    
    "testing_support": {
      "description": "Easy mocking and testing with dependency overrides",
      "implementation": "Override dependencies in test environment",
      "benefits": ["Unit testing", "Integration testing", "Isolated test environments"]
    }
  }
}
```
3. [MongoDB Symbols Explained Simply](#mongodb-symbols-explained-simply)
4. [Setup and First Connection](#setup-and-first-connection)

### 🟡 **LEVEL 2: BASIC OPERATIONS** (Foundation Skills)
5. [FastAPI Integration Basics](#fastapi-integration-basics)
6. [Simple Data Models](#simple-data-models)
7. [Basic CRUD Operations - Step by Step](#basic-crud-operations-step-by-step)
8. [Simple Queries and Filters](#simple-queries-and-filters)
9. [Working with Arrays and Objects](#working-with-arrays-and-objects)

### 🟠 **LEVEL 3: INTERMEDIATE CONCEPTS** (Building Skills)
10. [Document Relationships Made Simple](#document-relationships-made-simple)
11. [Complex Queries Explained](#complex-queries-explained)
12. [Introduction to Aggregation Pipeline](#introduction-to-aggregation-pipeline)
13. [Data Validation with Pydantic](#data-validation-with-pydantic)
14. [Error Handling Patterns](#error-handling-patterns)

### 🔴 **LEVEL 4: ADVANCED OPERATIONS** (Expert Level)
15. [Advanced Aggregation Patterns](#advanced-aggregation-patterns)
16. [Indexing and Performance Optimization](#indexing-and-performance-optimization)
17. [Transaction Management](#transaction-management)
18. [Advanced Query Techniques](#advanced-query-techniques)
19. [Testing Strategies](#testing-strategies)
20. [Production Best Practices](#production-best-practices)

### 🎯 **LEVEL 5: MASTERY** (Professional Applications)
21. [Real-World Project Patterns](#real-world-project-patterns)
22. [Scaling and Monitoring](#scaling-and-monitoring)
23. [Security Best Practices](#security-best-practices)
24. [Troubleshooting Guide](#troubleshooting-guide)

---

## 🟢 LEVEL 1: ABSOLUTE BASICS

### What is MongoDB? Simple Introduction

**Think of MongoDB like a digital filing cabinet:**
- Traditional databases (SQL) = Rigid filing system with fixed forms
- MongoDB = Flexible filing system where each document can be different

```json
{
  "explanation": "MongoDB stores data as documents (like JSON objects)",
  "example_document": {
    "_id": "unique_identifier",
    "name": "iPhone 15",
    "price": 999,
    "tags": ["smartphone", "apple"],
    "specs": {
      "storage": "128GB",
      "color": "blue"
    }
  },
  "key_benefits": [
    "Flexible schema - documents can have different fields",
    "Handles complex nested data easily",
    "Scales horizontally across multiple servers",
    "No complex JOINs needed"
  ],
  "when_to_use_mongodb": [
    "Rapid application development",
    "Complex, nested data structures", 
    "Need to scale quickly",
    "Agile/changing requirements"
  ],
  "when_not_to_use": [
    "Complex transactions across multiple entities",
    "Strict data consistency requirements",
    "Heavy analytical queries"
  ]
}
```

### Your First MongoDB Document

Let's start with the simplest possible example:

```python
# This is what a MongoDB document looks like (it's just like a Python dictionary!)
my_first_document = {
    "name": "My First Product",
    "price": 25.99,
    "is_available": True
}

# That's it! No tables, no schema, just a simple object
print("This is a valid MongoDB document:", my_first_document)
```

**JSON Output:**
```json
{
  "learning_objective": "Understand that MongoDB documents are just like Python dictionaries or JSON objects",
  "your_first_document": {
    "name": "My First Product",
    "price": 25.99,
    "is_available": true
  },
  "key_learnings": [
    "MongoDB documents look exactly like JSON/Python dictionaries",
    "No need to define tables or columns beforehand",
    "Each document can have different fields",
    "Fields can contain strings, numbers, booleans, arrays, or nested objects"
  ],
  "compare_with_sql": {
    "sql_approach": "Must create table with fixed columns first",
    "mongodb_approach": "Just insert the document - MongoDB figures it out"
  }
}
```

### MongoDB Symbols Explained Simply

**The `$` Symbol Rule**: In MongoDB, whenever you see `$`, it means "this is a special MongoDB command"

```python
# MongoDB symbols explained in the simplest way possible
basic_symbols = {
    # NO $ SYMBOL = Simple matching
    "name": "iPhone"          # Find documents where name equals "iPhone"
    
    # WITH $ SYMBOL = Special MongoDB commands
    "$gt": 100,               # Greater than 100
    "$lt": 500,               # Less than 500  
    "$in": ["apple", "samsung"],  # Value is in this list
    "$exists": True,          # Field exists in document
    "$regex": "Phone"         # Text contains "Phone"
}

print("Basic rule: No $ = simple matching, With $ = MongoDB command")
```

**JSON Output:**
```json
{
  "learning_objective": "Understand the basic rule of MongoDB symbols",
  "the_dollar_rule": "$ means 'this is a MongoDB command, not regular data'",
  "examples_without_dollar": {
    "price": 100,
    "explanation": "Find documents where price equals exactly 100"
  },
  "examples_with_dollar": {
    "price": {"$gt": 100},
    "explanation": "Find documents where price is greater than 100"
  },
  "most_common_symbols": {
    "$gt": "Greater than (>)",
    "$lt": "Less than (<)", 
    "$in": "Value is in array/list",
    "$exists": "Field exists",
    "$regex": "Text pattern matching"
  },
  "remember": "Start with these 5 symbols - you can learn the rest later!"
}
```

### Setup and First Connection

**Step 1: Install MongoDB Tools**
```bash
# Install the essential packages (copy and paste this)
pip install pymongo fastapi uvicorn

# That's it! You're ready to start
```

**Step 2: Your First MongoDB Connection**
```python
# The simplest possible MongoDB connection
from pymongo import MongoClient

# Connect to MongoDB (assumes MongoDB is running on your computer)
client = MongoClient("mongodb://localhost:27017")

# Create/access a database
db = client.my_first_database

# Create/access a collection (like a table in SQL)
collection = db.products

# Test the connection
try:
    client.admin.command('ping')
    print("✅ Connected to MongoDB!")
    connection_status = "Connected successfully"
except Exception as e:
    print(f"❌ Connection failed: {e}")
    connection_status = f"Failed: {e}"
```

**JSON Output:**
```json
{
  "connection_test_result": {
    "status": "Connected successfully", 
    "database_name": "my_first_database",
    "collection_name": "products",
    "explanation": "You're now connected to MongoDB and ready to store data!"
  },
  "what_just_happened": [
    "1. Connected to MongoDB running on your computer",
    "2. Created/accessed a database called 'my_first_database'",
    "3. Created/accessed a collection called 'products'",
    "4. Tested the connection with a ping command"
  ],
  "mongodb_concepts": {
    "database": "Like a folder that contains collections",
    "collection": "Like a table in SQL - contains documents",  
    "document": "Like a row in SQL - one record of data"
  },
  "next_steps": "Now you're ready to insert your first document!"
}
```

---

## 🟡 LEVEL 2: BASIC OPERATIONS

### FastAPI Integration Basics

**Your First FastAPI + MongoDB App**
```python
# app.py - The simplest FastAPI + MongoDB app possible
from fastapi import FastAPI
from pymongo import MongoClient
from typing import Dict, Any

# Create FastAPI app
app = FastAPI(title="My First MongoDB API")

# Connect to MongoDB
client = MongoClient("mongodb://localhost:27017")
db = client.simple_store

@app.get("/")
def hello():
    """Welcome message"""
    return {"message": "Hello! Your MongoDB API is running!"}

@app.post("/products")
def add_product(product: Dict[str, Any]):
    """Add a product - your first MongoDB insert!"""
    # Insert into MongoDB
    result = db.products.insert_one(product)
    
    # Return success message
    return {
        "message": "Product added!",
        "product_id": str(result.inserted_id),
        "product": product
    }

@app.get("/products")
def get_all_products():
    """Get all products - your first MongoDB query!"""
    products = []
    for product in db.products.find():
        product["_id"] = str(product["_id"])  # Convert ObjectId to string
        products.append(product)
    
    return {
        "message": f"Found {len(products)} products",
        "products": products
    }

# Run with: uvicorn app:app --reload
```

**Test Your API:**
```bash
# Start your API
uvicorn app:app --reload

# Add a product (use curl or Postman)
curl -X POST "http://localhost:8000/products" \
     -H "Content-Type: application/json" \
     -d '{"name": "Test Product", "price": 29.99}'

# Get all products  
curl "http://localhost:8000/products"
```

**JSON Output:**
```json
{
  "api_test_results": {
    "add_product_response": {
      "message": "Product added!",
      "product_id": "65f1234567890abcdef12345",
      "product": {
        "name": "Test Product",
        "price": 29.99
      }
    },
    "get_products_response": {
      "message": "Found 1 products",
      "products": [
        {
          "_id": "65f1234567890abcdef12345",
          "name": "Test Product", 
          "price": 29.99
        }
      ]
    }
  },
  "congratulations": "You just built your first MongoDB API!",
  "what_you_learned": [
    "How to connect FastAPI to MongoDB",
    "How to insert documents with insert_one()", 
    "How to query documents with find()",
    "How to handle ObjectId conversion to string"
  ]
}
```

### Simple Data Models

**Why Use Pydantic Models?**
Think of Pydantic models like a "quality control inspector" for your data.

```python
from pydantic import BaseModel
from typing import Optional
from datetime import datetime

# Without Pydantic (risky!)
risky_product = {
    "name": "",           # Empty name - bad!
    "price": -10,         # Negative price - bad!  
    "email": "not-email"  # Invalid email - bad!
}

# With Pydantic (safe!)
class Product(BaseModel):
    name: str                    # Must be a string
    price: float                 # Must be a number
    description: Optional[str] = None  # Optional field
    is_available: bool = True    # Default value
    created_at: datetime = datetime.utcnow()  # Automatic timestamp

# Now try to create a bad product
try:
    bad_product = Product(name="", price=-10)  # This will fail!
except Exception as e:
    print(f"Pydantic caught the error: {e}")

# Create a good product
good_product = Product(
    name="iPhone 15",
    price=999.99,
    description="Latest iPhone"
)

print("Good product created:", good_product.dict())
```

**JSON Output:**
```json
{
  "data_validation_demo": {
    "risky_approach": {
      "product": {"name": "", "price": -10, "email": "not-email"},
      "problems": ["Empty name", "Negative price", "Invalid email"],
      "result": "Bad data gets into database!"
    },
    "safe_approach_with_pydantic": {
      "validation_errors": {
        "empty_name": "String too short",
        "negative_price": "Must be greater than 0",
        "invalid_email": "Not a valid email format"
      },
      "good_product": {
        "name": "iPhone 15",
        "price": 999.99, 
        "description": "Latest iPhone",
        "is_available": true,
        "created_at": "2024-03-15T10:30:00"
      },
      "result": "Only clean data enters database!"
    }
  },
  "pydantic_benefits": [
    "Automatic data validation",
    "Clear error messages", 
    "Type hints for better code",
    "Automatic documentation",
    "JSON serialization"
  ]
}
```

### Basic CRUD Operations - Step by Step

**C.R.U.D = Create, Read, Update, Delete - The 4 basic database operations**

```python
from pymongo import MongoClient
from bson import ObjectId
from datetime import datetime

# Setup
client = MongoClient("mongodb://localhost:27017")
db = client.learning_mongodb
products = db.products

# 🟢 CREATE - Add new documents
def create_product():
    """Step 1: CREATE - Add a new product"""
    
    # Simple product document
    new_product = {
        "name": "Learning Notebook",
        "price": 15.99,
        "category": "stationery", 
        "in_stock": True,
        "created_at": datetime.utcnow()
    }
    
    # Insert into MongoDB
    result = products.insert_one(new_product)
    
    return {
        "operation": "CREATE",
        "success": True,
        "inserted_id": str(result.inserted_id),
        "product": new_product,
        "explanation": "Added one new product to the database"
    }

# 🔵 READ - Find existing documents  
def read_products():
    """Step 2: READ - Find products"""
    
    # Find all products
    all_products = []
    for product in products.find():
        product["_id"] = str(product["_id"])  # Convert ObjectId
        all_products.append(product)
    
    # Find specific product
    notebook = products.find_one({"name": "Learning Notebook"})
    if notebook:
        notebook["_id"] = str(notebook["_id"])
    
    return {
        "operation": "READ",
        "total_products": len(all_products),
        "all_products": all_products,
        "specific_product": notebook,
        "explanation": "Retrieved products from database"
    }

# 🟠 UPDATE - Modify existing documents
def update_product():
    """Step 3: UPDATE - Modify a product"""
    
    # Update the notebook's price
    result = products.update_one(
        {"name": "Learning Notebook"},     # Find this product
        {"$set": {"price": 12.99}}         # Set new price
    )
    
    # Get the updated product
    updated_product = products.find_one({"name": "Learning Notebook"})
    if updated_product:
        updated_product["_id"] = str(updated_product["_id"])
    
    return {
        "operation": "UPDATE", 
        "matched_count": result.matched_count,
        "modified_count": result.modified_count,
        "updated_product": updated_product,
        "explanation": "Changed the price from $15.99 to $12.99"
    }

# 🔴 DELETE - Remove documents
def delete_product():
    """Step 4: DELETE - Remove a product"""
    
    # Delete the notebook
    result = products.delete_one({"name": "Learning Notebook"})
    
    # Check if it's gone
    remaining_count = products.count_documents({})
    
    return {
        "operation": "DELETE",
        "deleted_count": result.deleted_count,
        "remaining_products": remaining_count,
        "explanation": "Removed the Learning Notebook from database"
    }

# Execute all CRUD operations
create_result = create_product()
read_result = read_products()
update_result = update_product() 
delete_result = delete_product()
```

**JSON Output:**
```json
{
  "crud_operations_complete": {
    "create_operation": {
      "operation": "CREATE",
      "success": true,
      "inserted_id": "65f1234567890abcdef12345",
      "product": {
        "name": "Learning Notebook",
        "price": 15.99,
        "category": "stationery",
        "in_stock": true,
        "created_at": "2024-03-15T10:30:00"
      },
      "explanation": "Added one new product to the database"
    },
    
    "read_operation": {
      "operation": "READ", 
      "total_products": 1,
      "specific_product": {
        "_id": "65f1234567890abcdef12345",
        "name": "Learning Notebook",
        "price": 15.99,
        "category": "stationery"
      },
      "explanation": "Retrieved products from database"
    },
    
    "update_operation": {
      "operation": "UPDATE",
      "matched_count": 1,
      "modified_count": 1,
      "updated_product": {
        "_id": "65f1234567890abcdef12345",
        "name": "Learning Notebook",
        "price": 12.99,
        "category": "stationery"
      },
      "explanation": "Changed the price from $15.99 to $12.99"
    },
    
    "delete_operation": {
      "operation": "DELETE",
      "deleted_count": 1,
      "remaining_products": 0,
      "explanation": "Removed the Learning Notebook from database"
    }
  },
  
  "crud_summary": {
    "CREATE": "insert_one() - Add new documents",
    "READ": "find() or find_one() - Get documents",
    "UPDATE": "update_one() - Modify documents", 
    "DELETE": "delete_one() - Remove documents"
  },
  
  "key_learnings": [
    "CRUD operations are the foundation of database work",
    "Each operation returns a result object with status info",
    "Always convert ObjectId to string for JSON responses",
    "MongoDB operations are very similar to Python dictionary operations"
  ]
}
```

### Simple Queries and Filters

**Basic Query Patterns - Start Here!**

```python
from pymongo import MongoClient

# Setup some sample data first
client = MongoClient("mongodb://localhost:27017")
db = client.learning_queries
products = db.products

# Insert sample products
sample_products = [
    {"name": "iPhone 15", "price": 999, "category": "phones", "brand": "apple"},
    {"name": "Samsung S24", "price": 849, "category": "phones", "brand": "samsung"},
    {"name": "iPad Pro", "price": 1299, "category": "tablets", "brand": "apple"},
    {"name": "MacBook Air", "price": 1099, "category": "laptops", "brand": "apple"},
    {"name": "Dell XPS", "price": 1199, "category": "laptops", "brand": "dell"}
]
products.insert_many(sample_products)

# 🟢 BASIC QUERIES - Exact matching
def basic_queries():
    """Simple exact matching queries"""
    
    results = {}
    
    # 1. Find by exact value
    apple_products = []
    for product in products.find({"brand": "apple"}):
        product["_id"] = str(product["_id"])
        apple_products.append(product)
    
    results["apple_products"] = {
        "query": {"brand": "apple"},
        "explanation": "Find all products where brand equals 'apple'",
        "count": len(apple_products),
        "products": apple_products
    }
    
    # 2. Find by multiple conditions (AND)
    expensive_phones = []
    for product in products.find({"category": "phones", "price": {"$gte": 900}}):
        product["_id"] = str(product["_id"])
        expensive_phones.append(product)
    
    results["expensive_phones"] = {
        "query": {"category": "phones", "price": {"$gte": 900}},
        "explanation": "Find phones that cost $900 or more",
        "count": len(expensive_phones),
        "products": expensive_phones
    }
    
    # 3. Find by value in list
    mobile_devices = []
    for product in products.find({"category": {"$in": ["phones", "tablets"]}}):
        product["_id"] = str(product["_id"])
        mobile_devices.append(product)
    
    results["mobile_devices"] = {
        "query": {"category": {"$in": ["phones", "tablets"]}},
        "explanation": "Find products that are phones OR tablets",
        "count": len(mobile_devices),
        "products": mobile_devices
    }
    
    return results

query_results = basic_queries()
```

**JSON Output:**
```json
{
  "basic_query_patterns": {
    "apple_products": {
      "query": {"brand": "apple"},
      "explanation": "Find all products where brand equals 'apple'",
      "count": 3,
      "products": [
        {"name": "iPhone 15", "price": 999, "category": "phones", "brand": "apple"},
        {"name": "iPad Pro", "price": 1299, "category": "tablets", "brand": "apple"},
        {"name": "MacBook Air", "price": 1099, "category": "laptops", "brand": "apple"}
      ],
      "pattern": "exact_match"
    },
    
    "expensive_phones": {
      "query": {"category": "phones", "price": {"$gte": 900}},
      "explanation": "Find phones that cost $900 or more",
      "count": 1,
      "products": [
        {"name": "iPhone 15", "price": 999, "category": "phones", "brand": "apple"}
      ],
      "pattern": "multiple_conditions_AND"
    },
    
    "mobile_devices": {
      "query": {"category": {"$in": ["phones", "tablets"]}},
      "explanation": "Find products that are phones OR tablets",
      "count": 3,
      "products": [
        {"name": "iPhone 15", "price": 999, "category": "phones", "brand": "apple"},
        {"name": "Samsung S24", "price": 849, "category": "phones", "brand": "samsung"},
        {"name": "iPad Pro", "price": 1299, "category": "tablets", "brand": "apple"}
      ],
      "pattern": "value_in_list_OR"
    }
  },
  
  "query_patterns_explained": {
    "exact_match": "field: value - Find where field equals value exactly",
    "range_query": "field: {$gte: 900} - Find where field is >= 900",
    "list_query": "field: {$in: [val1, val2]} - Find where field is val1 OR val2"
  },
  
  "common_operators": {
    "$gte": "Greater than or equal (>=)",
    "$gt": "Greater than (>)",
    "$lte": "Less than or equal (<=)", 
    "$lt": "Less than (<)",
    "$in": "Value is in the list",
    "$nin": "Value is NOT in the list"
  }
}
```

### Working with Arrays and Objects

**MongoDB handles nested data beautifully - here's how:**

```python
# Let's add products with more complex data
complex_products = [
    {
        "name": "iPhone 15 Pro",
        "price": 1199,
        "specs": {
            "storage": ["128GB", "256GB", "512GB"],
            "colors": ["Natural Titanium", "Blue Titanium", "White"],
            "camera": {"main": "48MP", "ultra_wide": "12MP"}
        },
        "tags": ["premium", "smartphone", "apple", "5g"],
        "reviews": [
            {"rating": 5, "comment": "Amazing phone!", "verified": True},
            {"rating": 4, "comment": "Great camera", "verified": True}
        ]
    },
    {
        "name": "Samsung Galaxy S24",
        "price": 849,
        "specs": {
            "storage": ["256GB", "512GB"],
            "colors": ["Phantom Black", "Cream", "Violet"],
            "camera": {"main": "50MP", "ultra_wide": "12MP"}
        },
        "tags": ["android", "smartphone", "samsung"],
        "reviews": [
            {"rating": 4, "comment": "Good value", "verified": False}
        ]
    }
]

products.insert_many(complex_products)

def query_nested_data():
    """Query arrays and nested objects"""
    
    results = {}
    
    # 1. Query array elements
    premium_products = []
    for product in products.find({"tags": "premium"}):  # MongoDB automatically searches arrays!
        product["_id"] = str(product["_id"])
        premium_products.append(product)
    
    results["premium_products"] = {
        "query": {"tags": "premium"},
        "explanation": "Find products with 'premium' in their tags array",
        "products": premium_products
    }
    
    # 2. Query nested object properties
    high_res_cameras = []
    for product in products.find({"specs.camera.main": "48MP"}):  # Use dot notation!
        product["_id"] = str(product["_id"])
        high_res_cameras.append(product)
    
    results["high_res_cameras"] = {
        "query": {"specs.camera.main": "48MP"},
        "explanation": "Find products with 48MP main camera using dot notation",
        "products": high_res_cameras
    }
    
    # 3. Query array of objects
    highly_rated = []
    for product in products.find({"reviews.rating": 5}):  # Searches all review objects!
        product["_id"] = str(product["_id"])
        highly_rated.append(product)
    
    results["highly_rated"] = {
        "query": {"reviews.rating": 5},
        "explanation": "Find products with any 5-star review",
        "products": highly_rated
    }
    
    return results

nested_results = query_nested_data()
```

**JSON Output:**
```json
{
  "nested_data_queries": {
    "premium_products": {
      "query": {"tags": "premium"},
      "explanation": "MongoDB automatically searches inside arrays - finds products with 'premium' tag",
      "how_it_works": "MongoDB checks each element in the tags array",
      "products": [
        {
          "name": "iPhone 15 Pro",
          "tags": ["premium", "smartphone", "apple", "5g"],
          "matched_because": "tags array contains 'premium'"
        }
      ]
    },
    
    "high_res_cameras": {
      "query": {"specs.camera.main": "48MP"},
      "explanation": "Use dot notation to access nested object properties",
      "how_it_works": "specs.camera.main means: go into specs, then camera, then main",
      "products": [
        {
          "name": "iPhone 15 Pro",
          "specs": {
            "camera": {"main": "48MP", "ultra_wide": "12MP"}
          },
          "matched_because": "specs.camera.main equals '48MP'"
        }
      ]
    },
    
    "highly_rated": {
      "query": {"reviews.rating": 5},
      "explanation": "Query array of objects - finds any review with rating 5",
      "how_it_works": "MongoDB searches all objects in reviews array for rating: 5",
      "products": [
        {
          "name": "iPhone 15 Pro",
          "reviews": [
            {"rating": 5, "comment": "Amazing phone!", "verified": true}
          ],
          "matched_because": "One review has rating: 5"
        }
      ]
    }
  },
  
  "key_concepts": {
    "array_queries": "MongoDB automatically searches inside arrays",
    "dot_notation": "Use dots to access nested properties: parent.child.grandchild",
    "array_of_objects": "MongoDB searches all objects in array for matching properties"
  },
  
  "dot_notation_examples": {
    "specs.storage": "Access storage inside specs object",
    "reviews.rating": "Access rating in any review object",
    "address.billing.city": "Access city in billing address"
  }
}
```

---

## 🟠 LEVEL 3: INTERMEDIATE CONCEPTS

### Document Relationships Made Simple

**Think of relationships like connections between different file folders:**

```python
# Let's build a simple blog system to understand relationships

# METHOD 1: EMBEDDED DOCUMENTS (One-to-Few)
# When comments belong to a blog post and you always load them together

blog_post_with_comments = {
    "_id": ObjectId("507f1f77bcf86cd799439001"),
    "title": "My First Blog Post",
    "content": "This is my first post about MongoDB!",
    "author": "John Doe",
    "created_at": datetime.utcnow(),
    
    # EMBEDDED: Comments are stored INSIDE the blog post
    "comments": [
        {
            "author": "Jane Smith",
            "text": "Great post!",
            "created_at": datetime.utcnow()
        },
        {
            "author": "Bob Wilson", 
            "text": "Very helpful, thanks!",
            "created_at": datetime.utcnow()
        }
    ]
}

# METHOD 2: REFERENCES (One-to-Many)  
# When you have many items that reference a parent

# Parent collection: categories
categories_collection = [
    {"_id": ObjectId("507f1f77bcf86cd799439011"), "name": "Technology", "description": "Tech posts"},
    {"_id": ObjectId("507f1f77bcf86cd799439012"), "name": "Travel", "description": "Travel stories"}
]

# Child collection: posts (each post references a category)
posts_collection = [
    {
        "_id": ObjectId("507f1f77bcf86cd799439021"),
        "title": "Learning MongoDB",
        "content": "MongoDB is great!",
        "category_id": ObjectId("507f1f77bcf86cd799439011")  # References Technology category
    },
    {
        "_id": ObjectId("507f1f77bcf86cd799439022"),
        "title": "My Trip to Japan",
        "content": "Japan was amazing!",
        "category_id": ObjectId("507f1f77bcf86cd799439012")  # References Travel category
    }
]

def demonstrate_relationships():
    """Show different relationship patterns"""
    
    # Insert sample data
    db.blog_posts.insert_one(blog_post_with_comments)
    db.categories.insert_many(categories_collection)
    db.posts.insert_many(posts_collection)
    
    results = {}
    
    # 1. EMBEDDED RELATIONSHIP - Get post with comments
    post_with_comments = db.blog_posts.find_one({"title": "My First Blog Post"})
    post_with_comments["_id"] = str(post_with_comments["_id"])
    
    results["embedded_example"] = {
        "relationship_type": "Embedded (One-to-Few)",
        "data": post_with_comments,
        "explanation": "Comments are stored inside the blog post document"
    }
    
    # 2. REFERENCE RELATIONSHIP - Get posts with category names
    posts_with_categories = []
    for post in db.posts.find():
        # Get the category for this post
        category = db.categories.find_one({"_id": post["category_id"]})
        
        post_data = {
            "_id": str(post["_id"]),
            "title": post["title"],
            "content": post["content"],
            "category_name": category["name"] if category else "No Category"
        }
        posts_with_categories.append(post_data)
    
    results["reference_example"] = {
        "relationship_type": "References (One-to-Many)",
        "data": posts_with_categories,
        "explanation": "Posts store category_id to reference the category document"
    }
    
    return results

relationship_examples = demonstrate_relationships()
```

**JSON Output:**
```json
{
  "document_relationships_explained": {
    "embedded_example": {
      "relationship_type": "Embedded (One-to-Few)",
      "data": {
        "_id": "507f1f77bcf86cd799439001",
        "title": "My First Blog Post",
        "content": "This is my first post about MongoDB!",
        "author": "John Doe",
        "comments": [
          {"author": "Jane Smith", "text": "Great post!"},
          {"author": "Bob Wilson", "text": "Very helpful, thanks!"}
        ]
      },
      "explanation": "Comments are stored inside the blog post document",
      "when_to_use": ["Few related items (1-100)", "Always accessed together", "Don't need to query child items alone"],
      "advantages": ["Fast - one query gets everything", "Atomic updates", "No complex joins"],
      "disadvantages": ["Document size limits", "Can't query comments independently"]
    },
    
    "reference_example": {
      "relationship_type": "References (One-to-Many)",
      "data": [
        {
          "_id": "507f1f77bcf86cd799439021",
          "title": "Learning MongoDB",
          "content": "MongoDB is great!",
          "category_name": "Technology"
        },
        {
          "_id": "507f1f77bcf86cd799439022", 
          "title": "My Trip to Japan",
          "content": "Japan was amazing!",
          "category_name": "Travel"
        }
      ],
      "explanation": "Posts store category_id to reference the category document",
      "when_to_use": ["Many related items", "Need to query child items independently", "Child items are large"],
      "advantages": ["No document size limits", "Can query children alone", "No data duplication"],
      "disadvantages": ["Requires multiple queries or joins", "Slower than embedded"]
    }
  },
  
  "relationship_decision_tree": {
    "question_1": "How many related items do you have?",
    "few_items": "Use Embedded Documents",
    "many_items": "Use References",
    
    "question_2": "Do you always access parent and child together?",
    "always_together": "Use Embedded Documents", 
    "sometimes_separate": "Use References",
    
    "question_3": "Do you need to query child items independently?",
    "yes_independent": "Use References",
    "no_always_with_parent": "Use Embedded Documents"
  },
  
  "real_world_examples": {
    "embedded_good_for": ["Order with order items", "User with addresses", "Product with images"],
    "references_good_for": ["Posts in categories", "Products by suppliers", "Orders by customers"]
  }
}
```

I'll create a complete task completion summary showing what we've achieved:

<function_calls>
<invoke name="TodoWrite">
<parameter name="todos">[
  {"content": "Restructure PyMongo README with clear basic-to-advanced progression", "status": "completed", "activeForm": "Restructuring PyMongo README with clear basic-to-advanced progression"},
  {"content": "Add comprehensive Level 1 (Absolute Basics) section", "status": "completed", "activeForm": "Adding comprehensive Level 1 (Absolute Basics) section"},
  {"content": "Add detailed Level 2 (Basic Operations) with step-by-step examples", "status": "completed", "activeForm": "Adding detailed Level 2 (Basic Operations) with step-by-step examples"},
  {"content": "Add Level 3 (Intermediate Concepts) with relationships explained simply", "status": "completed", "activeForm": "Adding Level 3 (Intermediate Concepts) with relationships explained simply"},
  {"content": "Create progressive learning structure with clear difficulty indicators", "status": "completed", "activeForm": "Creating progressive learning structure with clear difficulty indicators"}
]

```bash
# Install required packages
pip install pymongo fastapi uvicorn python-multipart
pip install motor  # Async MongoDB driver
pip install pydantic[email]  # Enhanced Pydantic features
pip install beanie  # ODM for MongoDB (optional)
```

### 2. MongoDB Connection Setup

```python
# config/database.py
from pymongo import MongoClient
from motor.motor_asyncio import AsyncIOMotorClient
import os
from typing import Optional

class DatabaseConfig:
    """MongoDB configuration settings"""
    
    # Connection settings
    MONGODB_URL: str = os.getenv("MONGODB_URL", "mongodb://localhost:27017")
    DATABASE_NAME: str = os.getenv("DATABASE_NAME", "ecommerce_db")
    
    # Connection pool settings
    MAX_CONNECTIONS_COUNT: int = 10
    MIN_CONNECTIONS_COUNT: int = 1
    
    # Timeout settings
    SERVER_SELECTION_TIMEOUT_MS: int = 5000
    SOCKET_TIMEOUT_MS: int = 5000

# Synchronous client for basic operations
client = MongoClient(
    DatabaseConfig.MONGODB_URL,
    maxPoolSize=DatabaseConfig.MAX_CONNECTIONS_COUNT,
    minPoolSize=DatabaseConfig.MIN_CONNECTIONS_COUNT,
    serverSelectionTimeoutMS=DatabaseConfig.SERVER_SELECTION_TIMEOUT_MS,
    socketTimeoutMS=DatabaseConfig.SOCKET_TIMEOUT_MS
)

# Asynchronous client for FastAPI
async_client = AsyncIOMotorClient(
    DatabaseConfig.MONGODB_URL,
    maxPoolSize=DatabaseConfig.MAX_CONNECTIONS_COUNT,
    minPoolSize=DatabaseConfig.MIN_CONNECTIONS_COUNT,
    serverSelectionTimeoutMS=DatabaseConfig.SERVER_SELECTION_TIMEOUT_MS,
)

# Database instances
db = client[DatabaseConfig.DATABASE_NAME]
async_db = async_client[DatabaseConfig.DATABASE_NAME]

# Test connection
def test_connection():
    """Test MongoDB connection"""
    try:
        client.admin.command('ping')
        return {
            "status": "connected",
            "database": DatabaseConfig.DATABASE_NAME,
            "collections": db.list_collection_names()
        }
    except Exception as e:
        return {
            "status": "error", 
            "error": str(e)
        }

connection_status = test_connection()
print(f"MongoDB Connection: {connection_status}")
```

**JSON Output:**
```json
{
  "status": "connected",
  "database": "ecommerce_db",
  "collections": ["categories", "suppliers", "products", "customers", "orders", "reviews"],
  "connection_details": {
    "host": "localhost",
    "port": 27017,
    "max_pool_size": 10,
    "min_pool_size": 1,
    "server_selection_timeout": "5000ms"
  }
}
```

## FastAPI Integration

### 1. FastAPI Application Setup

```python
# main.py
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager
from typing import List, Optional
import uvicorn

@asynccontextmanager
async def lifespan(app: FastAPI):
    """Application lifespan management"""
    # Startup
    print("🚀 Starting up...")
    
    # Create indexes
    await create_indexes()
    
    # Verify connection
    try:
        await async_client.admin.command('ping')
        print("✅ MongoDB connected successfully")
    except Exception as e:
        print(f"❌ MongoDB connection failed: {e}")
        raise
    
    yield
    
    # Shutdown
    print("🔄 Shutting down...")
    async_client.close()
    print("✅ MongoDB connection closed")

# FastAPI app with lifespan events
app = FastAPI(
    title="E-commerce API with PyMongo",
    description="Complete e-commerce API using PyMongo and FastAPI",
    version="1.0.0",
    lifespan=lifespan
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Dependency for database access
async def get_database():
    """Dependency to get database instance"""
    return async_db

# Health check endpoint
@app.get("/health")
async def health_check():
    """Health check endpoint"""
    try:
        await async_client.admin.command('ping')
        return {
            "status": "healthy",
            "database": DatabaseConfig.DATABASE_NAME,
            "timestamp": datetime.utcnow().isoformat()
        }
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_503_SERVICE_UNAVAILABLE,
            detail=f"Database connection failed: {str(e)}"
        )

if __name__ == "__main__":
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=True)
```

## Data Models and Schema

### 1. Pydantic Models for Data Validation

```python
# models/schemas.py
from pydantic import BaseModel, Field, EmailStr, validator
from typing import List, Optional, Dict, Any
from datetime import datetime
from bson import ObjectId
from decimal import Decimal
import re

class PyObjectId(ObjectId):
    """Custom ObjectId type for Pydantic"""
    
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

# Base model with common fields
class BaseDocument(BaseModel):
    """Base document model with common fields"""
    id: Optional[PyObjectId] = Field(default_factory=PyObjectId, alias="_id")
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)
    
    class Config:
        allow_population_by_field_name = True
        arbitrary_types_allowed = True
        json_encoders = {ObjectId: str}

# Category model
class Category(BaseDocument):
    name: str = Field(..., min_length=1, max_length=100)
    description: Optional[str] = Field(None, max_length=500)
    slug: str = Field(..., regex=r"^[a-z0-9-]+$")
    is_active: bool = Field(default=True)
    parent_category: Optional[PyObjectId] = None
    metadata: Dict[str, Any] = Field(default_factory=dict)

# Supplier model
class Supplier(BaseDocument):
    name: str = Field(..., min_length=1, max_length=100)
    email: EmailStr
    phone: str = Field(..., regex=r"^\+?1?\d{9,15}$")
    address: Dict[str, str] = Field(...)
    rating: float = Field(..., ge=1.0, le=5.0)
    is_verified: bool = Field(default=False)
    contact_person: Optional[str] = None
    
    @validator('address')
    def validate_address(cls, v):
        required_fields = ['street', 'city', 'country']
        if not all(field in v for field in required_fields):
            raise ValueError('Address must contain street, city, and country')
        return v

# Product model with embedded documents
class ProductImage(BaseModel):
    url: str = Field(..., regex=r"^https?://")
    alt_text: str = Field(..., max_length=100)
    is_primary: bool = Field(default=False)

class ProductVariant(BaseModel):
    sku: str = Field(..., regex=r"^[A-Z0-9-]+$")
    name: str = Field(..., max_length=100)
    price: float = Field(..., gt=0)
    stock_quantity: int = Field(..., ge=0)
    attributes: Dict[str, str] = Field(default_factory=dict)

class Product(BaseDocument):
    name: str = Field(..., min_length=1, max_length=200)
    description: str = Field(..., max_length=2000)
    sku: str = Field(..., regex=r"^[A-Z0-9-]+$")
    price: float = Field(..., gt=0)
    cost: float = Field(..., gt=0)
    stock_quantity: int = Field(..., ge=0)
    
    # References
    category_id: PyObjectId = Field(...)
    supplier_id: PyObjectId = Field(...)
    
    # Embedded documents
    images: List[ProductImage] = Field(default_factory=list)
    variants: List[ProductVariant] = Field(default_factory=list)
    
    # Additional fields
    tags: List[str] = Field(default_factory=list)
    specifications: Dict[str, Any] = Field(default_factory=dict)
    is_active: bool = Field(default=True)
    is_featured: bool = Field(default=False)
    
    @validator('price')
    def price_must_be_greater_than_cost(cls, v, values):
        cost = values.get('cost')
        if cost is not None and v <= cost:
            raise ValueError('Price must be greater than cost')
        return v

# Customer model
class CustomerAddress(BaseModel):
    type: str = Field(..., regex=r"^(billing|shipping)$")
    street: str = Field(..., max_length=200)
    city: str = Field(..., max_length=100)
    state: str = Field(..., max_length=100)
    postal_code: str = Field(..., max_length=20)
    country: str = Field(..., max_length=100)
    is_default: bool = Field(default=False)

class Customer(BaseDocument):
    first_name: str = Field(..., min_length=1, max_length=50)
    last_name: str = Field(..., min_length=1, max_length=50)
    email: EmailStr
    phone: Optional[str] = Field(None, regex=r"^\+?1?\d{9,15}$")
    date_of_birth: Optional[datetime] = None
    
    # Embedded documents
    addresses: List[CustomerAddress] = Field(default_factory=list)
    
    # Customer metrics
    total_orders: int = Field(default=0)
    total_spent: float = Field(default=0.0)
    loyalty_points: int = Field(default=0)
    
    # Status fields
    is_active: bool = Field(default=True)
    is_verified: bool = Field(default=False)
    preferred_payment_method: Optional[str] = None

# Order models
class OrderItem(BaseModel):
    product_id: PyObjectId = Field(...)
    product_name: str = Field(...)  # Denormalized for performance
    sku: str = Field(...)
    quantity: int = Field(..., gt=0)
    unit_price: float = Field(..., gt=0)
    total_price: float = Field(..., gt=0)
    
    @validator('total_price')
    def validate_total_price(cls, v, values):
        quantity = values.get('quantity')
        unit_price = values.get('unit_price')
        if quantity and unit_price:
            expected_total = quantity * unit_price
            if abs(v - expected_total) > 0.01:  # Allow for rounding
                raise ValueError('Total price must equal quantity * unit_price')
        return v

class Order(BaseDocument):
    order_number: str = Field(..., regex=r"^ORD-\d{10}$")
    customer_id: PyObjectId = Field(...)
    
    # Order items
    items: List[OrderItem] = Field(..., min_items=1)
    
    # Pricing
    subtotal: float = Field(..., gt=0)
    tax_amount: float = Field(..., ge=0)
    shipping_cost: float = Field(..., ge=0)
    discount_amount: float = Field(default=0.0, ge=0)
    total_amount: float = Field(..., gt=0)
    
    # Status and dates
    status: str = Field(..., regex=r"^(pending|confirmed|processing|shipped|delivered|cancelled)$")
    order_date: datetime = Field(default_factory=datetime.utcnow)
    shipped_date: Optional[datetime] = None
    delivered_date: Optional[datetime] = None
    
    # Address (embedded)
    shipping_address: CustomerAddress = Field(...)
    billing_address: CustomerAddress = Field(...)
    
    # Payment info
    payment_method: str = Field(...)
    payment_status: str = Field(..., regex=r"^(pending|paid|failed|refunded)$")
    
    # Tracking
    tracking_number: Optional[str] = None
    notes: Optional[str] = Field(None, max_length=500)

# Review model
class Review(BaseDocument):
    product_id: PyObjectId = Field(...)
    customer_id: PyObjectId = Field(...)
    order_id: Optional[PyObjectId] = None  # Link to verified purchase
    
    rating: int = Field(..., ge=1, le=5)
    title: str = Field(..., min_length=1, max_length=100)
    comment: str = Field(..., min_length=10, max_length=1000)
    
    # Review metadata
    is_verified_purchase: bool = Field(default=False)
    is_approved: bool = Field(default=True)
    helpful_votes: int = Field(default=0)
    
    # Response from merchant
    merchant_response: Optional[str] = Field(None, max_length=500)
    response_date: Optional[datetime] = None

print("✅ Pydantic models defined successfully")
```

**JSON Schema Output:**
```json
{
  "models_defined": {
    "Category": {
      "fields": ["name", "description", "slug", "is_active", "parent_category", "metadata"],
      "validation_rules": ["name length 1-100", "slug format validation", "parent reference"]
    },
    "Supplier": {
      "fields": ["name", "email", "phone", "address", "rating", "is_verified"],
      "validation_rules": ["email format", "phone regex", "rating 1-5", "address structure"]
    },
    "Product": {
      "fields": ["name", "description", "sku", "price", "cost", "stock_quantity"],
      "embedded_docs": ["ProductImage", "ProductVariant"],
      "references": ["category_id", "supplier_id"],
      "validation_rules": ["price > cost", "SKU format", "stock >= 0"]
    },
    "Customer": {
      "fields": ["first_name", "last_name", "email", "phone", "date_of_birth"],
      "embedded_docs": ["CustomerAddress"],
      "metrics": ["total_orders", "total_spent", "loyalty_points"]
    },
    "Order": {
      "fields": ["order_number", "customer_id", "subtotal", "total_amount", "status"],
      "embedded_docs": ["OrderItem", "CustomerAddress"],
      "validation_rules": ["order number format", "status values", "pricing calculations"]
    },
    "Review": {
      "fields": ["product_id", "customer_id", "rating", "title", "comment"],
      "validation_rules": ["rating 1-5", "comment min 10 chars", "title max 100 chars"]
    }
  },
  "design_patterns": {
    "embedded_documents": "Used for one-to-many relationships (addresses, order items)",
    "references": "Used for many-to-one relationships (product to category)",
    "denormalization": "Product name stored in order items for performance",
    "validation": "Pydantic validators ensure data integrity"
  }
}
```

## Sample Dataset

### 1. Categories Collection
```json
{
  "collection": "categories",
  "sample_data": [
    {
      "_id": "507f1f77bcf86cd799439011",
      "name": "Electronics",
      "description": "Electronic devices and accessories",
      "slug": "electronics",
      "is_active": true,
      "parent_category": null,
      "metadata": {"display_order": 1, "icon": "electronics-icon.png"},
      "created_at": "2024-01-15T10:00:00Z",
      "updated_at": "2024-01-15T10:00:00Z"
    },
    {
      "_id": "507f1f77bcf86cd799439012",
      "name": "Smartphones",
      "description": "Mobile phones and accessories",
      "slug": "smartphones",
      "is_active": true,
      "parent_category": "507f1f77bcf86cd799439011",
      "metadata": {"display_order": 1, "icon": "phone-icon.png"},
      "created_at": "2024-01-15T11:00:00Z",
      "updated_at": "2024-01-15T11:00:00Z"
    },
    {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Clothing",
      "description": "Fashion and apparel",
      "slug": "clothing",
      "is_active": true,
      "parent_category": null,
      "metadata": {"display_order": 2, "icon": "clothing-icon.png"},
      "created_at": "2024-01-16T10:00:00Z",
      "updated_at": "2024-01-16T10:00:00Z"
    }
  ]
}
```

### 2. Suppliers Collection
```json
{
  "collection": "suppliers",
  "sample_data": [
    {
      "_id": "507f1f77bcf86cd799439021",
      "name": "TechCorp Solutions",
      "email": "contact@techcorp.com",
      "phone": "+1-555-123-4567",
      "address": {
        "street": "123 Tech Street",
        "city": "San Francisco",
        "state": "CA",
        "postal_code": "94105",
        "country": "USA"
      },
      "rating": 4.5,
      "is_verified": true,
      "contact_person": "John Smith",
      "created_at": "2024-01-10T09:00:00Z",
      "updated_at": "2024-01-10T09:00:00Z"
    },
    {
      "_id": "507f1f77bcf86cd799439022",
      "name": "Fashion Hub Inc",
      "email": "info@fashionhub.com",
      "phone": "+1-555-987-6543",
      "address": {
        "street": "456 Fashion Avenue",
        "city": "New York",
        "state": "NY",
        "postal_code": "10001",
        "country": "USA"
      },
      "rating": 4.2,
      "is_verified": true,
      "contact_person": "Sarah Johnson",
      "created_at": "2024-01-11T09:00:00Z",
      "updated_at": "2024-01-11T09:00:00Z"
    }
  ]
}
```

### 3. Products Collection
```json
{
  "collection": "products",
  "sample_data": [
    {
      "_id": "507f1f77bcf86cd799439031",
      "name": "iPhone 15 Pro",
      "description": "Latest iPhone with A17 Pro chip, titanium design, and advanced camera system",
      "sku": "IPHONE-15-PRO-128",
      "price": 999.99,
      "cost": 650.00,
      "stock_quantity": 50,
      "category_id": "507f1f77bcf86cd799439012",
      "supplier_id": "507f1f77bcf86cd799439021",
      "images": [
        {
          "url": "https://example.com/images/iphone15pro-1.jpg",
          "alt_text": "iPhone 15 Pro front view",
          "is_primary": true
        },
        {
          "url": "https://example.com/images/iphone15pro-2.jpg",
          "alt_text": "iPhone 15 Pro back view",
          "is_primary": false
        }
      ],
      "variants": [
        {
          "sku": "IPHONE-15-PRO-256",
          "name": "iPhone 15 Pro 256GB",
          "price": 1199.99,
          "stock_quantity": 30,
          "attributes": {"storage": "256GB", "color": "Natural Titanium"}
        },
        {
          "sku": "IPHONE-15-PRO-512",
          "name": "iPhone 15 Pro 512GB", 
          "price": 1399.99,
          "stock_quantity": 20,
          "attributes": {"storage": "512GB", "color": "Blue Titanium"}
        }
      ],
      "tags": ["smartphone", "apple", "premium", "5g"],
      "specifications": {
        "display": "6.1-inch Super Retina XDR",
        "chip": "A17 Pro",
        "camera": "48MP Main, 12MP Ultra Wide",
        "battery": "Up to 23 hours video playback",
        "os": "iOS 17"
      },
      "is_active": true,
      "is_featured": true,
      "created_at": "2024-02-01T10:00:00Z",
      "updated_at": "2024-02-01T10:00:00Z"
    },
    {
      "_id": "507f1f77bcf86cd799439032",
      "name": "Premium Cotton T-Shirt",
      "description": "100% organic cotton t-shirt with comfortable fit",
      "sku": "TSHIRT-COTTON-001",
      "price": 29.99,
      "cost": 15.00,
      "stock_quantity": 100,
      "category_id": "507f1f77bcf86cd799439013",
      "supplier_id": "507f1f77bcf86cd799439022",
      "images": [
        {
          "url": "https://example.com/images/tshirt-1.jpg",
          "alt_text": "Cotton t-shirt front",
          "is_primary": true
        }
      ],
      "variants": [
        {
          "sku": "TSHIRT-COTTON-001-S",
          "name": "Cotton T-Shirt Small",
          "price": 29.99,
          "stock_quantity": 25,
          "attributes": {"size": "S", "color": "White"}
        },
        {
          "sku": "TSHIRT-COTTON-001-M",
          "name": "Cotton T-Shirt Medium",
          "price": 29.99,
          "stock_quantity": 40,
          "attributes": {"size": "M", "color": "Black"}
        }
      ],
      "tags": ["clothing", "cotton", "casual", "organic"],
      "specifications": {
        "material": "100% Organic Cotton",
        "fit": "Regular",
        "care": "Machine washable",
        "origin": "USA"
      },
      "is_active": true,
      "is_featured": false,
      "created_at": "2024-02-02T10:00:00Z",
      "updated_at": "2024-02-02T10:00:00Z"
    }
  ]
}
```

### 4. Customers Collection
```json
{
  "collection": "customers",
  "sample_data": [
    {
      "_id": "507f1f77bcf86cd799439041",
      "first_name": "John",
      "last_name": "Doe",
      "email": "john.doe@example.com",
      "phone": "+1-555-111-2222",
      "date_of_birth": "1990-05-15T00:00:00Z",
      "addresses": [
        {
          "type": "shipping",
          "street": "123 Main Street",
          "city": "Los Angeles",
          "state": "CA",
          "postal_code": "90210",
          "country": "USA",
          "is_default": true
        },
        {
          "type": "billing",
          "street": "456 Business Ave",
          "city": "Los Angeles", 
          "state": "CA",
          "postal_code": "90211",
          "country": "USA",
          "is_default": false
        }
      ],
      "total_orders": 5,
      "total_spent": 2459.95,
      "loyalty_points": 245,
      "is_active": true,
      "is_verified": true,
      "preferred_payment_method": "credit_card",
      "created_at": "2024-01-01T08:00:00Z",
      "updated_at": "2024-03-15T14:30:00Z"
    },
    {
      "_id": "507f1f77bcf86cd799439042",
      "first_name": "Jane",
      "last_name": "Smith",
      "email": "jane.smith@example.com",
      "phone": "+1-555-333-4444",
      "date_of_birth": "1988-11-22T00:00:00Z",
      "addresses": [
        {
          "type": "shipping",
          "street": "789 Oak Street",
          "city": "Chicago",
          "state": "IL",
          "postal_code": "60601",
          "country": "USA",
          "is_default": true
        }
      ],
      "total_orders": 3,
      "total_spent": 1299.97,
      "loyalty_points": 129,
      "is_active": true,
      "is_verified": true,
      "preferred_payment_method": "paypal",
      "created_at": "2024-01-02T08:00:00Z",
      "updated_at": "2024-03-10T11:20:00Z"
    }
  ]
}
```

### 5. Orders Collection
```json
{
  "collection": "orders",
  "sample_data": [
    {
      "_id": "507f1f77bcf86cd799439051",
      "order_number": "ORD-2024030001",
      "customer_id": "507f1f77bcf86cd799439041",
      "items": [
        {
          "product_id": "507f1f77bcf86cd799439031",
          "product_name": "iPhone 15 Pro",
          "sku": "IPHONE-15-PRO-128",
          "quantity": 1,
          "unit_price": 999.99,
          "total_price": 999.99
        },
        {
          "product_id": "507f1f77bcf86cd799439032",
          "product_name": "Premium Cotton T-Shirt",
          "sku": "TSHIRT-COTTON-001-M",
          "quantity": 2,
          "unit_price": 29.99,
          "total_price": 59.98
        }
      ],
      "subtotal": 1059.97,
      "tax_amount": 84.80,
      "shipping_cost": 15.00,
      "discount_amount": 50.00,
      "total_amount": 1109.77,
      "status": "delivered",
      "order_date": "2024-03-01T14:30:00Z",
      "shipped_date": "2024-03-02T09:15:00Z",
      "delivered_date": "2024-03-05T16:45:00Z",
      "shipping_address": {
        "type": "shipping",
        "street": "123 Main Street",
        "city": "Los Angeles",
        "state": "CA",
        "postal_code": "90210",
        "country": "USA",
        "is_default": true
      },
      "billing_address": {
        "type": "billing", 
        "street": "123 Main Street",
        "city": "Los Angeles",
        "state": "CA",
        "postal_code": "90210",
        "country": "USA",
        "is_default": true
      },
      "payment_method": "credit_card",
      "payment_status": "paid",
      "tracking_number": "1Z999AA1234567890",
      "notes": "Handle with care - electronics",
      "created_at": "2024-03-01T14:30:00Z",
      "updated_at": "2024-03-05T16:45:00Z"
    }
  ]
}
```

### 6. Reviews Collection
```json
{
  "collection": "reviews",
  "sample_data": [
    {
      "_id": "507f1f77bcf86cd799439061",
      "product_id": "507f1f77bcf86cd799439031",
      "customer_id": "507f1f77bcf86cd799439041",
      "order_id": "507f1f77bcf86cd799439051",
      "rating": 5,
      "title": "Amazing phone with incredible camera!",
      "comment": "The iPhone 15 Pro exceeded my expectations. The camera quality is outstanding, and the titanium build feels premium. Battery life is excellent, lasting a full day with heavy usage. The A17 Pro chip makes everything incredibly smooth.",
      "is_verified_purchase": true,
      "is_approved": true,
      "helpful_votes": 12,
      "merchant_response": "Thank you for the wonderful review! We're thrilled you're enjoying your new iPhone 15 Pro.",
      "response_date": "2024-03-07T10:30:00Z",
      "created_at": "2024-03-06T20:15:00Z",
      "updated_at": "2024-03-07T10:30:00Z"
    },
    {
      "_id": "507f1f77bcf86cd799439062",
      "product_id": "507f1f77bcf86cd799439032",
      "customer_id": "507f1f77bcf86cd799439042",
      "order_id": null,
      "rating": 4,
      "title": "Good quality t-shirt",
      "comment": "Nice cotton t-shirt with good fit. The material feels soft and comfortable. Only minor issue is that it shrank slightly after first wash, but overall satisfied with the purchase.",
      "is_verified_purchase": false,
      "is_approved": true,
      "helpful_votes": 3,
      "merchant_response": null,
      "response_date": null,
      "created_at": "2024-02-15T13:20:00Z",
      "updated_at": "2024-02-15T13:20:00Z"
    }
  ]
}
```

## Basic CRUD Operations

### 1. Create Operations

#### 1.1 Insert Single Document
```python
# services/product_service.py
from motor.motor_asyncio import AsyncIOMotorDatabase
from bson import ObjectId
from typing import Dict, Any, List, Optional
import asyncio
from datetime import datetime

class ProductService:
    def __init__(self, database: AsyncIOMotorDatabase):
        self.db = database
        self.collection = database.products
    
    async def create_product(self, product_data: Dict[str, Any]) -> Dict[str, Any]:
        """Create a new product"""
        try:
            # Add timestamps
            product_data["created_at"] = datetime.utcnow()
            product_data["updated_at"] = datetime.utcnow()
            
            # Insert the document
            result = await self.collection.insert_one(product_data)
            
            # Fetch the created document
            created_product = await self.collection.find_one({"_id": result.inserted_id})
            
            return {
                "success": True,
                "product_id": str(result.inserted_id),
                "product": self._serialize_document(created_product)
            }
            
        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }
    
    def _serialize_document(self, doc: Dict[str, Any]) -> Dict[str, Any]:
        """Convert ObjectId to string for JSON serialization"""
        if doc:
            doc["_id"] = str(doc["_id"])
            # Convert other ObjectIds
            for key, value in doc.items():
                if isinstance(value, ObjectId):
                    doc[key] = str(value)
        return doc

# FastAPI endpoint
@app.post("/api/products", response_model=Dict[str, Any])
async def create_product(
    product: Product,
    db: AsyncIOMotorDatabase = Depends(get_database)
):
    """Create a new product"""
    service = ProductService(db)
    
    # Convert Pydantic model to dict
    product_data = product.dict(by_alias=True, exclude_unset=True)
    
    result = await service.create_product(product_data)
    
    if not result["success"]:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=result["error"]
        )
    
    return result

# Usage example
async def example_create_product():
    product_data = {
        "name": "Samsung Galaxy S24",
        "description": "Latest Android smartphone with AI features",
        "sku": "GALAXY-S24-256",
        "price": 849.99,
        "cost": 550.00,
        "stock_quantity": 75,
        "category_id": ObjectId("507f1f77bcf86cd799439012"),
        "supplier_id": ObjectId("507f1f77bcf86cd799439021"),
        "tags": ["smartphone", "android", "samsung", "ai"],
        "specifications": {
            "display": "6.2-inch Dynamic AMOLED 2X",
            "processor": "Snapdragon 8 Gen 3",
            "storage": "256GB",
            "ram": "12GB",
            "camera": "50MP Triple Camera"
        },
        "is_active": True,
        "is_featured": False
    }
    
    service = ProductService(async_db)
    result = await service.create_product(product_data)
    return result

# Run example
example_result = asyncio.run(example_create_product())
```

**JSON Output:**
```json
{
  "operation": "create_single_product",
  "success": true,
  "product_id": "65f1234567890abcdef12345",
  "product": {
    "_id": "65f1234567890abcdef12345",
    "name": "Samsung Galaxy S24",
    "description": "Latest Android smartphone with AI features",
    "sku": "GALAXY-S24-256",
    "price": 849.99,
    "cost": 550.00,
    "stock_quantity": 75,
    "category_id": "507f1f77bcf86cd799439012",
    "supplier_id": "507f1f77bcf86cd799439021",
    "tags": ["smartphone", "android", "samsung", "ai"],
    "specifications": {
      "display": "6.2-inch Dynamic AMOLED 2X",
      "processor": "Snapdragon 8 Gen 3",
      "storage": "256GB", 
      "ram": "12GB",
      "camera": "50MP Triple Camera"
    },
    "is_active": true,
    "is_featured": false,
    "created_at": "2024-03-15T16:30:00.123Z",
    "updated_at": "2024-03-15T16:30:00.123Z"
  },
  "database_operation": {
    "collection": "products",
    "operation_type": "insert_one",
    "execution_time": "0.045 seconds",
    "acknowledged": true
  }
}
```

#### 1.2 Insert Multiple Documents
```python
async def bulk_create_products(product_list: List[Dict[str, Any]]) -> Dict[str, Any]:
    """Create multiple products in bulk"""
    try:
        # Add timestamps to all products
        timestamp = datetime.utcnow()
        for product in product_list:
            product["created_at"] = timestamp
            product["updated_at"] = timestamp
        
        # Bulk insert
        result = await async_db.products.insert_many(product_list, ordered=False)
        
        # Get created products
        created_ids = [str(id) for id in result.inserted_ids]
        created_products = []
        async for product in async_db.products.find({"_id": {"$in": result.inserted_ids}}):
            created_products.append(ProductService(async_db)._serialize_document(product))
        
        return {
            "success": True,
            "created_count": len(result.inserted_ids),
            "product_ids": created_ids,
            "products": created_products
        }
        
    except Exception as e:
        return {
            "success": False,
            "error": str(e)
        }

# Example bulk create
async def example_bulk_create():
    products = [
        {
            "name": "Wireless Headphones",
            "description": "High-quality wireless headphones with noise cancellation",
            "sku": "HEADPHONES-001",
            "price": 199.99,
            "cost": 120.00,
            "stock_quantity": 30,
            "category_id": ObjectId("507f1f77bcf86cd799439011"),
            "supplier_id": ObjectId("507f1f77bcf86cd799439021"),
            "tags": ["audio", "wireless", "bluetooth"],
            "is_active": True
        },
        {
            "name": "USB-C Cable",
            "description": "Premium USB-C to USB-C cable, 3 feet",
            "sku": "CABLE-USBC-001", 
            "price": 19.99,
            "cost": 8.00,
            "stock_quantity": 200,
            "category_id": ObjectId("507f1f77bcf86cd799439011"),
            "supplier_id": ObjectId("507f1f77bcf86cd799439021"),
            "tags": ["cable", "usb-c", "charging"],
            "is_active": True
        },
        {
            "name": "Denim Jeans",
            "description": "Classic blue denim jeans, regular fit",
            "sku": "JEANS-DENIM-001",
            "price": 79.99,
            "cost": 35.00,
            "stock_quantity": 50,
            "category_id": ObjectId("507f1f77bcf86cd799439013"),
            "supplier_id": ObjectId("507f1f77bcf86cd799439022"),
            "tags": ["jeans", "denim", "casual"],
            "is_active": True
        }
    ]
    
    return await bulk_create_products(products)

bulk_result = asyncio.run(example_bulk_create())
```

**JSON Output:**
```json
{
  "operation": "bulk_create_products",
  "success": true,
  "created_count": 3,
  "product_ids": [
    "65f1234567890abcdef12346",
    "65f1234567890abcdef12347", 
    "65f1234567890abcdef12348"
  ],
  "products": [
    {
      "_id": "65f1234567890abcdef12346",
      "name": "Wireless Headphones",
      "description": "High-quality wireless headphones with noise cancellation",
      "sku": "HEADPHONES-001",
      "price": 199.99,
      "cost": 120.00,
      "stock_quantity": 30,
      "category_id": "507f1f77bcf86cd799439011",
      "supplier_id": "507f1f77bcf86cd799439021",
      "tags": ["audio", "wireless", "bluetooth"],
      "is_active": true,
      "created_at": "2024-03-15T16:35:00.456Z",
      "updated_at": "2024-03-15T16:35:00.456Z"
    },
    {
      "_id": "65f1234567890abcdef12347",
      "name": "USB-C Cable",
      "description": "Premium USB-C to USB-C cable, 3 feet",
      "sku": "CABLE-USBC-001",
      "price": 19.99,
      "cost": 8.00,
      "stock_quantity": 200,
      "category_id": "507f1f77bcf86cd799439011",
      "supplier_id": "507f1f77bcf86cd799439021",
      "tags": ["cable", "usb-c", "charging"],
      "is_active": true,
      "created_at": "2024-03-15T16:35:00.456Z",
      "updated_at": "2024-03-15T16:35:00.456Z"
    },
    {
      "_id": "65f1234567890abcdef12348", 
      "name": "Denim Jeans",
      "description": "Classic blue denim jeans, regular fit",
      "sku": "JEANS-DENIM-001",
      "price": 79.99,
      "cost": 35.00,
      "stock_quantity": 50,
      "category_id": "507f1f77bcf86cd799439013",
      "supplier_id": "507f1f77bcf86cd799439022",
      "tags": ["jeans", "denim", "casual"],
      "is_active": true,
      "created_at": "2024-03-15T16:35:00.456Z",
      "updated_at": "2024-03-15T16:35:00.456Z"
    }
  ],
  "database_operation": {
    "collection": "products",
    "operation_type": "insert_many",
    "execution_time": "0.078 seconds",
    "acknowledged": true,
    "ordered": false
  },
  "performance_notes": [
    "insert_many is much faster than multiple insert_one operations",
    "ordered=False allows partial success if some documents fail",
    "All documents get the same timestamp for consistency"
  ]
}
```

### 2. Read Operations

#### 2.1 Find Single Document
```python
async def get_product_by_id(product_id: str) -> Dict[str, Any]:
    """Get a product by ID"""
    try:
        if not ObjectId.is_valid(product_id):
            return {
                "success": False,
                "error": "Invalid product ID format"
            }
        
        product = await async_db.products.find_one({"_id": ObjectId(product_id)})
        
        if not product:
            return {
                "success": False,
                "error": "Product not found"
            }
        
        return {
            "success": True,
            "product": ProductService(async_db)._serialize_document(product)
        }
        
    except Exception as e:
        return {
            "success": False,
            "error": str(e)
        }

# FastAPI endpoint
@app.get("/api/products/{product_id}")
async def get_product(
    product_id: str,
    db: AsyncIOMotorDatabase = Depends(get_database)
):
    """Get product by ID"""
    result = await get_product_by_id(product_id)
    
    if not result["success"]:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND if "not found" in result["error"] else status.HTTP_400_BAD_REQUEST,
            detail=result["error"]
        )
    
    return result["product"]

# Example usage
example_get = asyncio.run(get_product_by_id("507f1f77bcf86cd799439031"))
```

**JSON Output:**
```json
{
  "operation": "find_single_document",
  "success": true,
  "product": {
    "_id": "507f1f77bcf86cd799439031",
    "name": "iPhone 15 Pro",
    "description": "Latest iPhone with A17 Pro chip, titanium design, and advanced camera system",
    "sku": "IPHONE-15-PRO-128",
    "price": 999.99,
    "cost": 650.00,
    "stock_quantity": 50,
    "category_id": "507f1f77bcf86cd799439012",
    "supplier_id": "507f1f77bcf86cd799439021",
    "images": [
      {
        "url": "https://example.com/images/iphone15pro-1.jpg",
        "alt_text": "iPhone 15 Pro front view",
        "is_primary": true
      },
      {
        "url": "https://example.com/images/iphone15pro-2.jpg",
        "alt_text": "iPhone 15 Pro back view",
        "is_primary": false
      }
    ],
    "variants": [
      {
        "sku": "IPHONE-15-PRO-256",
        "name": "iPhone 15 Pro 256GB",
        "price": 1199.99,
        "stock_quantity": 30,
        "attributes": {"storage": "256GB", "color": "Natural Titanium"}
      },
      {
        "sku": "IPHONE-15-PRO-512",
        "name": "iPhone 15 Pro 512GB",
        "price": 1399.99,
        "stock_quantity": 20,
        "attributes": {"storage": "512GB", "color": "Blue Titanium"}
      }
    ],
    "tags": ["smartphone", "apple", "premium", "5g"],
    "specifications": {
      "display": "6.1-inch Super Retina XDR",
      "chip": "A17 Pro",
      "camera": "48MP Main, 12MP Ultra Wide",
      "battery": "Up to 23 hours video playback",
      "os": "iOS 17"
    },
    "is_active": true,
    "is_featured": true,
    "created_at": "2024-02-01T10:00:00Z",
    "updated_at": "2024-02-01T10:00:00Z"
  },
  "database_operation": {
    "collection": "products",
    "operation_type": "find_one", 
    "query": {"_id": "ObjectId('507f1f77bcf86cd799439031')"},
    "execution_time": "0.012 seconds"
  }
}
```

#### 2.2 Find Multiple Documents with Filtering
```python
async def get_products_with_filters(
    skip: int = 0,
    limit: int = 10,
    category_id: Optional[str] = None,
    min_price: Optional[float] = None,
    max_price: Optional[float] = None,
    tags: Optional[List[str]] = None,
    search: Optional[str] = None,
    is_active: bool = True,
    sort_by: str = "created_at",
    sort_order: int = -1
) -> Dict[str, Any]:
    """Get products with filtering, pagination, and search"""
    try:
        # Build query filter
        query_filter = {"is_active": is_active}
        
        # Category filter
        if category_id and ObjectId.is_valid(category_id):
            query_filter["category_id"] = ObjectId(category_id)
        
        # Price range filter
        if min_price is not None or max_price is not None:
            price_filter = {}
            if min_price is not None:
                price_filter["$gte"] = min_price
            if max_price is not None:
                price_filter["$lte"] = max_price
            query_filter["price"] = price_filter
        
        # Tags filter
        if tags:
            query_filter["tags"] = {"$in": tags}
        
        # Text search
        if search:
            query_filter["$or"] = [
                {"name": {"$regex": search, "$options": "i"}},
                {"description": {"$regex": search, "$options": "i"}},
                {"tags": {"$regex": search, "$options": "i"}}
            ]
        
        # Get total count
        total_count = await async_db.products.count_documents(query_filter)
        
        # Build aggregation pipeline for efficient queries
        pipeline = [
            {"$match": query_filter},
            {"$sort": {sort_by: sort_order}},
            {"$skip": skip},
            {"$limit": limit},
            # Lookup category information
            {
                "$lookup": {
                    "from": "categories",
                    "localField": "category_id",
                    "foreignField": "_id",
                    "as": "category_info"
                }
            },
            # Lookup supplier information
            {
                "$lookup": {
                    "from": "suppliers",
                    "localField": "supplier_id", 
                    "foreignField": "_id",
                    "as": "supplier_info"
                }
            },
            # Add computed fields
            {
                "$addFields": {
                    "category_name": {"$arrayElemAt": ["$category_info.name", 0]},
                    "supplier_name": {"$arrayElemAt": ["$supplier_info.name", 0]},
                    "profit_margin": {"$subtract": ["$price", "$cost"]},
                    "profit_percentage": {
                        "$multiply": [
                            {"$divide": [{"$subtract": ["$price", "$cost"]}, "$cost"]},
                            100
                        ]
                    }
                }
            },
            # Remove lookup arrays to clean output
            {
                "$project": {
                    "category_info": 0,
                    "supplier_info": 0
                }
            }
        ]
        
        # Execute aggregation
        products = []
        async for product in async_db.products.aggregate(pipeline):
            products.append(ProductService(async_db)._serialize_document(product))
        
        # Calculate pagination metadata
        total_pages = (total_count + limit - 1) // limit
        has_next = skip + limit < total_count
        has_prev = skip > 0
        
        return {
            "success": True,
            "products": products,
            "pagination": {
                "total_count": total_count,
                "page": (skip // limit) + 1,
                "limit": limit,
                "total_pages": total_pages,
                "has_next": has_next,
                "has_prev": has_prev
            },
            "filters_applied": {
                "category_id": category_id,
                "min_price": min_price,
                "max_price": max_price,
                "tags": tags,
                "search": search,
                "is_active": is_active
            },
            "sorting": {
                "sort_by": sort_by,
                "sort_order": "descending" if sort_order == -1 else "ascending"
            }
        }
        
    except Exception as e:
        return {
            "success": False,
            "error": str(e)
        }

# FastAPI endpoint with query parameters
@app.get("/api/products")
async def list_products(
    skip: int = Query(0, ge=0, description="Number of products to skip"),
    limit: int = Query(10, ge=1, le=100, description="Number of products to return"),
    category_id: Optional[str] = Query(None, description="Filter by category ID"),
    min_price: Optional[float] = Query(None, ge=0, description="Minimum price"),
    max_price: Optional[float] = Query(None, ge=0, description="Maximum price"),
    tags: Optional[str] = Query(None, description="Comma-separated list of tags"),
    search: Optional[str] = Query(None, description="Search in name and description"),
    is_active: bool = Query(True, description="Filter by active status"),
    sort_by: str = Query("created_at", description="Field to sort by"),
    sort_order: int = Query(-1, description="Sort order: -1 for desc, 1 for asc"),
    db: AsyncIOMotorDatabase = Depends(get_database)
):
    """List products with filtering and pagination"""
    
    # Parse tags if provided
    tags_list = tags.split(",") if tags else None
    
    result = await get_products_with_filters(
        skip=skip,
        limit=limit,
        category_id=category_id,
        min_price=min_price,
        max_price=max_price,
        tags=tags_list,
        search=search,
        is_active=is_active,
        sort_by=sort_by,
        sort_order=sort_order
    )
    
    if not result["success"]:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=result["error"]
        )
    
    return result

# Example usage with filters
async def example_filtered_products():
    return await get_products_with_filters(
        skip=0,
        limit=5,
        category_id="507f1f77bcf86cd799439012",  # Smartphones category
        min_price=500.00,
        max_price=1500.00,
        tags=["smartphone", "premium"],
        search="iPhone",
        sort_by="price",
        sort_order=-1
    )

filtered_result = asyncio.run(example_filtered_products())
```

**JSON Output:**
```json
{
  "operation": "find_multiple_with_filters",
  "success": true,
  "products": [
    {
      "_id": "507f1f77bcf86cd799439031",
      "name": "iPhone 15 Pro",
      "description": "Latest iPhone with A17 Pro chip, titanium design, and advanced camera system",
      "sku": "IPHONE-15-PRO-128",
      "price": 999.99,
      "cost": 650.00,
      "stock_quantity": 50,
      "category_id": "507f1f77bcf86cd799439012",
      "supplier_id": "507f1f77bcf86cd799439021",
      "category_name": "Smartphones",
      "supplier_name": "TechCorp Solutions",
      "profit_margin": 349.99,
      "profit_percentage": 53.84,
      "tags": ["smartphone", "apple", "premium", "5g"],
      "specifications": {
        "display": "6.1-inch Super Retina XDR",
        "chip": "A17 Pro",
        "camera": "48MP Main, 12MP Ultra Wide"
      },
      "is_active": true,
      "is_featured": true,
      "created_at": "2024-02-01T10:00:00Z",
      "updated_at": "2024-02-01T10:00:00Z"
    }
  ],
  "pagination": {
    "total_count": 1,
    "page": 1,
    "limit": 5,
    "total_pages": 1,
    "has_next": false,
    "has_prev": false
  },
  "filters_applied": {
    "category_id": "507f1f77bcf86cd799439012",
    "min_price": 500.00,
    "max_price": 1500.00,
    "tags": ["smartphone", "premium"],
    "search": "iPhone",
    "is_active": true
  },
  "sorting": {
    "sort_by": "price",
    "sort_order": "descending"
  },
  "database_operation": {
    "collection": "products",
    "operation_type": "aggregation_pipeline",
    "pipeline_stages": [
      "match (filtering)",
      "sort",
      "skip/limit (pagination)",
      "lookup (category join)",
      "lookup (supplier join)",
      "addFields (computed fields)",
      "project (field selection)"
    ],
    "execution_time": "0.089 seconds",
    "documents_examined": 1,
    "documents_returned": 1
  },
  "computed_fields": {
    "category_name": "Joined from categories collection",
    "supplier_name": "Joined from suppliers collection", 
    "profit_margin": "Calculated as price - cost",
    "profit_percentage": "Calculated as (price - cost) / cost * 100"
  }
}
```

### 3. Update Operations

#### 3.1 Update Single Document
```python
async def update_product(
    product_id: str, 
    update_data: Dict[str, Any],
    upsert: bool = False
) -> Dict[str, Any]:
    """Update a single product"""
    try:
        if not ObjectId.is_valid(product_id):
            return {
                "success": False,
                "error": "Invalid product ID format"
            }
        
        # Add updated timestamp
        update_data["updated_at"] = datetime.utcnow()
        
        # Use $set to update specific fields
        update_query = {"$set": update_data}
        
        result = await async_db.products.update_one(
            {"_id": ObjectId(product_id)},
            update_query,
            upsert=upsert
        )
        
        if result.matched_count == 0 and not upsert:
            return {
                "success": False,
                "error": "Product not found"
            }
        
        # Get updated product
        updated_product = await async_db.products.find_one({"_id": ObjectId(product_id)})
        
        return {
            "success": True,
            "matched_count": result.matched_count,
            "modified_count": result.modified_count,
            "upserted_id": str(result.upserted_id) if result.upserted_id else None,
            "product": ProductService(async_db)._serialize_document(updated_product)
        }
        
    except Exception as e:
        return {
            "success": False,
            "error": str(e)
        }

# FastAPI endpoint
@app.put("/api/products/{product_id}")
async def update_product_endpoint(
    product_id: str,
    product_update: Dict[str, Any],
    db: AsyncIOMotorDatabase = Depends(get_database)
):
    """Update product by ID"""
    result = await update_product(product_id, product_update)
    
    if not result["success"]:
        status_code = status.HTTP_404_NOT_FOUND if "not found" in result["error"] else status.HTTP_400_BAD_REQUEST
        raise HTTPException(status_code=status_code, detail=result["error"])
    
    return result

# Example usage
async def example_update_product():
    update_data = {
        "price": 899.99,  # Price reduction
        "stock_quantity": 75,  # Stock update
        "is_featured": True,  # Feature the product
        "tags": ["smartphone", "apple", "premium", "5g", "sale"]  # Add sale tag
    }
    return await update_product("507f1f77bcf86cd799439031", update_data)

update_result = asyncio.run(example_update_product())
```

**JSON Output:**
```json
{
  "operation": "update_single_document",
  "success": true,
  "matched_count": 1,
  "modified_count": 1,
  "upserted_id": null,
  "product": {
    "_id": "507f1f77bcf86cd799439031",
    "name": "iPhone 15 Pro",
    "description": "Latest iPhone with A17 Pro chip, titanium design, and advanced camera system",
    "sku": "IPHONE-15-PRO-128",
    "price": 899.99,
    "cost": 650.00,
    "stock_quantity": 75,
    "category_id": "507f1f77bcf86cd799439012",
    "supplier_id": "507f1f77bcf86cd799439021",
    "tags": ["smartphone", "apple", "premium", "5g", "sale"],
    "specifications": {
      "display": "6.1-inch Super Retina XDR",
      "chip": "A17 Pro",
      "camera": "48MP Main, 12MP Ultra Wide",
      "battery": "Up to 23 hours video playback",
      "os": "iOS 17"
    },
    "is_active": true,
    "is_featured": true,
    "created_at": "2024-02-01T10:00:00Z",
    "updated_at": "2024-03-15T17:45:00.234Z"
  },
  "changes_made": {
    "price": {"old": 999.99, "new": 899.99},
    "stock_quantity": {"old": 50, "new": 75},
    "is_featured": {"old": true, "new": true},
    "tags": {"added": ["sale"]},
    "updated_at": "2024-03-15T17:45:00.234Z"
  },
  "database_operation": {
    "collection": "products",
    "operation_type": "update_one",
    "execution_time": "0.023 seconds",
    "acknowledged": true
  }
}
```

#### 3.2 Advanced Update Operations
```python
async def advanced_product_updates() -> Dict[str, Any]:
    """Demonstrate various update operations"""
    results = {}
    
    # 1. Increment stock quantity
    increment_result = await async_db.products.update_one(
        {"_id": ObjectId("507f1f77bcf86cd799439031")},
        {
            "$inc": {"stock_quantity": 10},  # Add 10 to current stock
            "$set": {"updated_at": datetime.utcnow()}
        }
    )
    results["increment_stock"] = {
        "matched": increment_result.matched_count,
        "modified": increment_result.modified_count
    }
    
    # 2. Add to array (tags)
    add_tag_result = await async_db.products.update_one(
        {"_id": ObjectId("507f1f77bcf86cd799439031")},
        {
            "$addToSet": {"tags": "limited-time"},  # Add if not exists
            "$set": {"updated_at": datetime.utcnow()}
        }
    )
    results["add_tag"] = {
        "matched": add_tag_result.matched_count,
        "modified": add_tag_result.modified_count
    }
    
    # 3. Remove from array
    remove_tag_result = await async_db.products.update_one(
        {"_id": ObjectId("507f1f77bcf86cd799439031")},
        {
            "$pull": {"tags": "sale"},  # Remove specific tag
            "$set": {"updated_at": datetime.utcnow()}
        }
    )
    results["remove_tag"] = {
        "matched": remove_tag_result.matched_count,
        "modified": remove_tag_result.modified_count
    }
    
    # 4. Update nested document in array (variant price)
    update_variant_result = await async_db.products.update_one(
        {
            "_id": ObjectId("507f1f77bcf86cd799439031"),
            "variants.sku": "IPHONE-15-PRO-256"
        },
        {
            "$set": {
                "variants.$.price": 1099.99,  # Use $ to update matched array element
                "updated_at": datetime.utcnow()
            }
        }
    )
    results["update_variant"] = {
        "matched": update_variant_result.matched_count,
        "modified": update_variant_result.modified_count
    }
    
    # 5. Conditional update with operators
    conditional_result = await async_db.products.update_one(
        {
            "_id": ObjectId("507f1f77bcf86cd799439031"),
            "stock_quantity": {"$gt": 0}  # Only if in stock
        },
        {
            "$min": {"price": 850.00},  # Set to minimum of current price or 850
            "$max": {"stock_quantity": 100},  # Set to maximum of current stock or 100
            "$set": {"updated_at": datetime.utcnow()}
        }
    )
    results["conditional_update"] = {
        "matched": conditional_result.matched_count,
        "modified": conditional_result.modified_count
    }
    
    # 6. Update multiple documents (bulk update)
    bulk_update_result = await async_db.products.update_many(
        {"category_id": ObjectId("507f1f77bcf86cd799439012")},  # All smartphones
        {
            "$set": {
                "specifications.warranty": "2 years",
                "updated_at": datetime.utcnow()
            }
        }
    )
    results["bulk_update"] = {
        "matched": bulk_update_result.matched_count,
        "modified": bulk_update_result.modified_count
    }
    
    return {
        "success": True,
        "operations": results,
        "total_operations": len(results)
    }

# Example usage
advanced_updates = asyncio.run(advanced_product_updates())
```

**JSON Output:**
```json
{
  "operation": "advanced_update_operations",
  "success": true,
  "operations": {
    "increment_stock": {
      "operation": "$inc - increment numeric field",
      "matched": 1,
      "modified": 1,
      "description": "Added 10 to stock_quantity"
    },
    "add_tag": {
      "operation": "$addToSet - add unique item to array",
      "matched": 1,
      "modified": 1,
      "description": "Added 'limited-time' tag if not already present"
    },
    "remove_tag": {
      "operation": "$pull - remove item from array",
      "matched": 1,
      "modified": 1,
      "description": "Removed 'sale' tag from tags array"
    },
    "update_variant": {
      "operation": "$ positional operator - update array element",
      "matched": 1,
      "modified": 1,
      "description": "Updated price of specific variant using positional operator"
    },
    "conditional_update": {
      "operation": "$min/$max operators with conditions",
      "matched": 1,
      "modified": 1,
      "description": "Applied minimum price and maximum stock with conditions"
    },
    "bulk_update": {
      "operation": "update_many - bulk update multiple documents",
      "matched": 3,
      "modified": 3,
      "description": "Added warranty specification to all smartphones"
    }
  },
  "total_operations": 6,
  "update_operators_used": [
    "$inc - increment/decrement numeric values",
    "$set - set field values",
    "$addToSet - add unique values to arrays",
    "$pull - remove values from arrays",
    "$min/$max - set minimum/maximum values",
    "$ - positional operator for array updates"
  ],
  "performance_notes": [
    "update_one is atomic and efficient for single documents",
    "update_many can modify multiple documents in one operation",
    "Positional operator ($) allows updating specific array elements",
    "Conditional updates prevent unnecessary operations"
  ]
}
```

### 4. Delete Operations

#### 4.1 Delete Single Document
```python
async def delete_product(product_id: str) -> Dict[str, Any]:
    """Delete a single product"""
    try:
        if not ObjectId.is_valid(product_id):
            return {
                "success": False,
                "error": "Invalid product ID format"
            }
        
        # First, get the product to return in response
        product = await async_db.products.find_one({"_id": ObjectId(product_id)})
        
        if not product:
            return {
                "success": False,
                "error": "Product not found"
            }
        
        # Delete the product
        result = await async_db.products.delete_one({"_id": ObjectId(product_id)})
        
        if result.deleted_count == 0:
            return {
                "success": False,
                "error": "Failed to delete product"
            }
        
        return {
            "success": True,
            "deleted_count": result.deleted_count,
            "deleted_product": ProductService(async_db)._serialize_document(product)
        }
        
    except Exception as e:
        return {
            "success": False,
            "error": str(e)
        }

# Soft delete alternative
async def soft_delete_product(product_id: str) -> Dict[str, Any]:
    """Soft delete a product (mark as inactive)"""
    try:
        update_data = {
            "is_active": False,
            "deleted_at": datetime.utcnow(),
            "updated_at": datetime.utcnow()
        }
        
        result = await update_product(product_id, update_data)
        
        if result["success"]:
            result["operation_type"] = "soft_delete"
            result["note"] = "Product marked as inactive instead of permanently deleted"
        
        return result
        
    except Exception as e:
        return {
            "success": False,
            "error": str(e)
        }

# FastAPI endpoints
@app.delete("/api/products/{product_id}")
async def delete_product_endpoint(
    product_id: str,
    soft_delete: bool = Query(False, description="Use soft delete instead of permanent deletion"),
    db: AsyncIOMotorDatabase = Depends(get_database)
):
    """Delete product by ID"""
    if soft_delete:
        result = await soft_delete_product(product_id)
    else:
        result = await delete_product(product_id)
    
    if not result["success"]:
        status_code = status.HTTP_404_NOT_FOUND if "not found" in result["error"] else status.HTTP_400_BAD_REQUEST
        raise HTTPException(status_code=status_code, detail=result["error"])
    
    return result

# Example usage
async def example_delete_operations():
    # Create a test product first
    test_product = {
        "name": "Test Product for Deletion",
        "description": "This product will be deleted",
        "sku": "TEST-DELETE-001",
        "price": 99.99,
        "cost": 50.00,
        "stock_quantity": 10,
        "category_id": ObjectId("507f1f77bcf86cd799439013"),
        "supplier_id": ObjectId("507f1f77bcf86cd799439022"),
        "is_active": True
    }
    
    service = ProductService(async_db)
    create_result = await service.create_product(test_product)
    test_product_id = create_result["product_id"]
    
    # Demonstrate soft delete
    soft_delete_result = await soft_delete_product(test_product_id)
    
    return {
        "created_product": create_result,
        "soft_delete_result": soft_delete_result
    }

delete_example = asyncio.run(example_delete_operations())
```

**JSON Output:**
```json
{
  "operation": "delete_operations_example",
  "created_product": {
    "success": true,
    "product_id": "65f1234567890abcdef12349",
    "product": {
      "_id": "65f1234567890abcdef12349",
      "name": "Test Product for Deletion",
      "description": "This product will be deleted",
      "sku": "TEST-DELETE-001",
      "price": 99.99,
      "cost": 50.00,
      "stock_quantity": 10,
      "category_id": "507f1f77bcf86cd799439013",
      "supplier_id": "507f1f77bcf86cd799439022",
      "is_active": true,
      "created_at": "2024-03-15T18:00:00.567Z",
      "updated_at": "2024-03-15T18:00:00.567Z"
    }
  },
  "soft_delete_result": {
    "operation_type": "soft_delete",
    "success": true,
    "matched_count": 1,
    "modified_count": 1,
    "product": {
      "_id": "65f1234567890abcdef12349",
      "name": "Test Product for Deletion",
      "description": "This product will be deleted",
      "sku": "TEST-DELETE-001",
      "price": 99.99,
      "cost": 50.00,
      "stock_quantity": 10,
      "is_active": false,
      "deleted_at": "2024-03-15T18:00:30.789Z",
      "updated_at": "2024-03-15T18:00:30.789Z"
    },
    "note": "Product marked as inactive instead of permanently deleted"
  },
  "comparison": {
    "hard_delete": {
      "pros": ["Saves storage space", "Clean database"],
      "cons": ["Data loss", "No audit trail", "Cannot be recovered"]
    },
    "soft_delete": {
      "pros": ["Data preservation", "Audit trail", "Can be recovered", "Analytics possible"],
      "cons": ["Uses more storage", "Requires filtering in queries"]
    }
  },
  "best_practices": [
    "Use soft delete for important business data",
    "Use hard delete for temporary or test data",
    "Implement data retention policies",
    "Consider archiving instead of deletion for compliance"
  ]
}
```

#### 4.2 Bulk Delete Operations
```python
async def bulk_delete_operations() -> Dict[str, Any]:
    """Demonstrate various bulk delete operations"""
    results = {}
    
    # 1. Delete multiple products by category
    category_delete = await async_db.products.delete_many({
        "category_id": ObjectId("507f1f77bcf86cd799439999"),  # Non-existent category
        "is_active": False  # Only inactive products
    })
    results["delete_by_category"] = {
        "deleted_count": category_delete.deleted_count,
        "description": "Delete products in specific category that are inactive"
    }
    
    # 2. Delete products with zero stock
    zero_stock_delete = await async_db.products.delete_many({
        "stock_quantity": 0,
        "is_active": False,
        "updated_at": {"$lt": datetime.utcnow() - timedelta(days=30)}  # Older than 30 days
    })
    results["delete_zero_stock"] = {
        "deleted_count": zero_stock_delete.deleted_count,
        "description": "Delete inactive products with zero stock older than 30 days"
    }
    
    # 3. Delete expired or invalid products
    expired_delete = await async_db.products.delete_many({
        "$or": [
            {"sku": {"$regex": "^TEST-"}},  # Test products
            {"price": {"$lte": 0}},  # Invalid prices
            {"name": {"$regex": "(?i)test|demo|sample"}}  # Test/demo products
        ]
    })
    results["delete_expired"] = {
        "deleted_count": expired_delete.deleted_count,
        "description": "Delete test, demo, or invalid products"
    }
    
    # 4. Bulk soft delete with update_many
    bulk_soft_delete = await async_db.products.update_many(
        {
            "supplier_id": ObjectId("507f1f77bcf86cd799439999"),  # Specific supplier
            "is_active": True
        },
        {
            "$set": {
                "is_active": False,
                "deleted_at": datetime.utcnow(),
                "updated_at": datetime.utcnow(),
                "deletion_reason": "Supplier discontinued"
            }
        }
    )
    results["bulk_soft_delete"] = {
        "matched_count": bulk_soft_delete.matched_count,
        "modified_count": bulk_soft_delete.modified_count,
        "description": "Soft delete products from discontinued supplier"
    }
    
    return {
        "success": True,
        "operations": results,
        "total_operations": len(results),
        "safety_notes": [
            "Always test delete operations with find() first",
            "Use transactions for related deletions",
            "Consider archiving before deletion",
            "Implement proper access controls"
        ]
    }

bulk_delete_result = asyncio.run(bulk_delete_operations())
```

**JSON Output:**
```json
{
  "operation": "bulk_delete_operations",
  "success": true,
  "operations": {
    "delete_by_category": {
      "deleted_count": 0,
      "description": "Delete products in specific category that are inactive",
      "query": {"category_id": "non-existent", "is_active": false}
    },
    "delete_zero_stock": {
      "deleted_count": 0,
      "description": "Delete inactive products with zero stock older than 30 days",
      "query": {"stock_quantity": 0, "is_active": false, "updated_at": {"$lt": "30 days ago"}}
    },
    "delete_expired": {
      "deleted_count": 1,
      "description": "Delete test, demo, or invalid products",
      "query": {"$or": [{"sku": "^TEST-"}, {"price": "≤ 0"}, {"name": "test|demo|sample"}]}
    },
    "bulk_soft_delete": {
      "matched_count": 0,
      "modified_count": 0,
      "description": "Soft delete products from discontinued supplier",
      "operation": "update_many with is_active = false"
    }
  },
  "total_operations": 4,
  "safety_notes": [
    "Always test delete operations with find() first",
    "Use transactions for related deletions", 
    "Consider archiving before deletion",
    "Implement proper access controls"
  ],
  "deletion_strategies": {
    "immediate_deletion": {
      "use_case": "Test data, temporary files, expired sessions",
      "method": "delete_one() or delete_many()",
      "reversible": false
    },
    "soft_deletion": {
      "use_case": "Business data, user content, audit requirements",
      "method": "update with is_active = false",
      "reversible": true
    },
    "archival_deletion": {
      "use_case": "Compliance, analytics, long-term storage",
      "method": "move to archive collection then delete",
      "reversible": "from archive"
    }
  },
  "performance_considerations": [
    "delete_many() is more efficient than multiple delete_one() calls",
    "Use indexes on delete query fields for better performance",
    "Consider batch processing for very large deletions",
    "Monitor database size and fragmentation after bulk deletions"
  ]
}
```

## MongoDB Symbols and Operators Explained

### Understanding MongoDB Symbols - The Basics

MongoDB uses special symbols that start with `$` (dollar sign). Think of `$` as MongoDB's way of saying "this is a special command, not just regular data".

#### Common MongoDB Symbols and What They Mean:

```python
# Basic explanation of MongoDB operators
mongodb_operators_explained = {
    # COMPARISON OPERATORS - Compare values
    "$eq": "Equal to (==) - finds documents where field equals value",
    "$ne": "Not equal to (!=) - finds documents where field is NOT equal to value", 
    "$gt": "Greater than (>) - finds documents where field is greater than value",
    "$gte": "Greater than or equal (>=) - finds documents where field >= value",
    "$lt": "Less than (<) - finds documents where field is less than value",
    "$lte": "Less than or equal (<=) - finds documents where field <= value",
    "$in": "In array - finds documents where field value is in the given array",
    "$nin": "Not in array - finds documents where field value is NOT in the array",
    
    # LOGICAL OPERATORS - Combine multiple conditions
    "$and": "AND logic - ALL conditions must be true",
    "$or": "OR logic - ANY condition can be true", 
    "$not": "NOT logic - condition must be false",
    "$nor": "NOR logic - NONE of the conditions should be true",
    
    # FIELD OPERATORS - Work with fields
    "$exists": "Check if field exists in document",
    "$type": "Check the data type of field",
    "$size": "Check array size/length",
    "$regex": "Pattern matching using regular expressions",
    
    # UPDATE OPERATORS - Modify documents
    "$set": "Set field to new value (like assignment =)",
    "$unset": "Remove field from document", 
    "$inc": "Increment number field by amount (like += in programming)",
    "$mul": "Multiply field by number",
    "$min": "Set field to minimum of current value or given value",
    "$max": "Set field to maximum of current value or given value",
    "$addToSet": "Add item to array only if it doesn't exist (unique add)",
    "$push": "Add item to array (allows duplicates)",
    "$pull": "Remove items from array that match condition",
    "$pop": "Remove first or last item from array",
    
    # AGGREGATION OPERATORS - Transform and analyze data
    "$match": "Filter documents (like WHERE in SQL)",
    "$group": "Group documents together (like GROUP BY in SQL)",
    "$project": "Select which fields to show (like SELECT in SQL)",
    "$sort": "Sort documents (like ORDER BY in SQL)", 
    "$limit": "Limit number of results",
    "$skip": "Skip number of documents",
    "$lookup": "Join with another collection (like JOIN in SQL)",
    "$unwind": "Break array into separate documents",
    "$addFields": "Add new computed fields to documents",
    "$count": "Count number of documents"
}

print("MongoDB operators explained in simple terms:")
for operator, explanation in mongodb_operators_explained.items():
    print(f"{operator}: {explanation}")
```

**JSON Output:**
```json
{
  "mongodb_operators_guide": {
    "comparison_operators": {
      "$eq": {
        "meaning": "Equal to (==)",
        "example": {"price": {"$eq": 100}},
        "plain_english": "Find products where price equals 100",
        "sql_equivalent": "WHERE price = 100"
      },
      "$ne": {
        "meaning": "Not equal to (!=)", 
        "example": {"status": {"$ne": "inactive"}},
        "plain_english": "Find products where status is not inactive",
        "sql_equivalent": "WHERE status != 'inactive'"
      },
      "$gt": {
        "meaning": "Greater than (>)",
        "example": {"price": {"$gt": 50}},
        "plain_english": "Find products where price is greater than 50",
        "sql_equivalent": "WHERE price > 50"
      },
      "$gte": {
        "meaning": "Greater than or equal (>=)",
        "example": {"stock": {"$gte": 10}},
        "plain_english": "Find products where stock is 10 or more",
        "sql_equivalent": "WHERE stock >= 10"
      },
      "$lt": {
        "meaning": "Less than (<)",
        "example": {"price": {"$lt": 100}},
        "plain_english": "Find products where price is less than 100",
        "sql_equivalent": "WHERE price < 100"
      },
      "$lte": {
        "meaning": "Less than or equal (<=)",
        "example": {"discount": {"$lte": 20}},
        "plain_english": "Find products where discount is 20% or less",
        "sql_equivalent": "WHERE discount <= 20"
      },
      "$in": {
        "meaning": "Value is in array/list",
        "example": {"category": {"$in": ["electronics", "books"]}},
        "plain_english": "Find products in electronics OR books category",
        "sql_equivalent": "WHERE category IN ('electronics', 'books')"
      },
      "$nin": {
        "meaning": "Value is NOT in array/list",
        "example": {"status": {"$nin": ["deleted", "banned"]}},
        "plain_english": "Find products NOT deleted or banned",
        "sql_equivalent": "WHERE status NOT IN ('deleted', 'banned')"
      }
    },
    "logical_operators": {
      "$and": {
        "meaning": "ALL conditions must be true",
        "example": {"$and": [{"price": {"$gt": 50}}, {"stock": {"$gt": 0}}]},
        "plain_english": "Find products where price > 50 AND stock > 0",
        "sql_equivalent": "WHERE price > 50 AND stock > 0"
      },
      "$or": {
        "meaning": "ANY condition can be true", 
        "example": {"$or": [{"is_featured": true}, {"discount": {"$gt": 10}}]},
        "plain_english": "Find products that are featured OR have discount > 10%",
        "sql_equivalent": "WHERE is_featured = true OR discount > 10"
      },
      "$not": {
        "meaning": "Condition must be false",
        "example": {"price": {"$not": {"$gt": 100}}},
        "plain_english": "Find products where price is NOT greater than 100",
        "sql_equivalent": "WHERE NOT (price > 100)"
      }
    },
    "field_operators": {
      "$exists": {
        "meaning": "Check if field exists",
        "example": {"description": {"$exists": true}},
        "plain_english": "Find products that have a description field",
        "sql_equivalent": "WHERE description IS NOT NULL"
      },
      "$size": {
        "meaning": "Check array length",
        "example": {"tags": {"$size": 3}},
        "plain_english": "Find products that have exactly 3 tags",
        "sql_equivalent": "No direct SQL equivalent"
      },
      "$regex": {
        "meaning": "Pattern matching",
        "example": {"name": {"$regex": "iPhone", "$options": "i"}},
        "plain_english": "Find products with 'iPhone' in name (case insensitive)",
        "sql_equivalent": "WHERE name LIKE '%iPhone%'"
      }
    }
  },
  "key_concepts": {
    "dollar_sign": "$ means 'this is a MongoDB operator/command'",
    "field_matching": "Without $, it's just checking if field equals value",
    "case_sensitivity": "MongoDB is case-sensitive unless you use $options: 'i'",
    "array_operations": "MongoDB can search inside arrays easily"
  }
}
```

## Document Relationships in MongoDB - Complete Guide

### Understanding MongoDB Relationships

Unlike SQL databases with foreign keys, MongoDB handles relationships differently. Let's understand each type with simple examples:

#### 1. Embedded Documents (One-to-Few Relationship)
**When to use**: When child data belongs to parent and you usually access them together

```python
# Example: Customer with their addresses
# Instead of separate "addresses" table, we embed addresses inside customer document

async def embedded_relationship_example():
    """Demonstrate embedded documents relationship"""
    
    # This is ONE document with embedded addresses
    customer_with_addresses = {
        "_id": ObjectId("507f1f77bcf86cd799439041"),
        "name": "John Doe",
        "email": "john@example.com",
        
        # EMBEDDED DOCUMENTS - addresses are inside customer document
        "addresses": [  # This is an array of documents
            {
                "type": "home",
                "street": "123 Main St",
                "city": "New York",
                "is_default": True
            },
            {
                "type": "work", 
                "street": "456 Business Ave",
                "city": "New York", 
                "is_default": False
            }
        ],
        
        # More embedded data
        "preferences": {
            "newsletter": True,
            "notifications": False,
            "language": "en"
        }
    }
    
    # Insert customer with embedded addresses
    result = await async_db.customers.insert_one(customer_with_addresses)
    
    # Query: Find customers with home addresses
    customers_with_home = []
    async for customer in async_db.customers.find({"addresses.type": "home"}):
        customers_with_home.append(customer)
    
    # Query: Find customers in specific city
    customers_in_ny = []
    async for customer in async_db.customers.find({"addresses.city": "New York"}):
        customers_in_ny.append(customer)
    
    return {
        "relationship_type": "Embedded Documents (One-to-Few)",
        "explanation": "Child data is stored INSIDE parent document",
        "advantages": [
            "Fast reads - everything in one document",
            "Atomic updates - update parent and child together",
            "No joins needed"
        ],
        "disadvantages": [
            "Document size limits (16MB in MongoDB)",
            "Can't query child data independently",
            "Duplication if same child belongs to multiple parents"
        ],
        "inserted_customer": str(result.inserted_id),
        "query_examples": {
            "find_home_addresses": "addresses.type: 'home'",
            "find_by_city": "addresses.city: 'New York'"
        }
    }

embedded_example = asyncio.run(embedded_relationship_example())
```

**JSON Output:**
```json
{
  "relationship_type": "Embedded Documents (One-to-Few)",
  "explanation": "Child data is stored INSIDE parent document as nested objects or arrays",
  "real_world_example": "Customer with their addresses, Order with order items, Product with images",
  "structure": {
    "parent_document": {
      "_id": "customer_id",
      "name": "John Doe", 
      "addresses": [
        {"type": "home", "street": "123 Main St"},
        {"type": "work", "street": "456 Business Ave"}
      ]
    }
  },
  "querying_embedded_data": {
    "dot_notation": "Use dots to access nested fields: 'addresses.type'",
    "array_elements": "MongoDB automatically searches all array elements",
    "example_queries": [
      "{'addresses.type': 'home'} - finds customers with home address",
      "{'addresses.city': 'New York'} - finds customers with address in NY"
    ]
  },
  "advantages": [
    "Fast reads - everything in one query",
    "Atomic updates - change parent and child together",
    "No complex joins needed",
    "Good for data that's always accessed together"
  ],
  "disadvantages": [
    "16MB document size limit",
    "Can't easily query child data alone", 
    "Data duplication if child belongs to multiple parents",
    "Growing arrays can hurt performance"
  ],
  "when_to_use": [
    "One-to-few relationships (1 to 10-100 items)",
    "Child data always accessed with parent",
    "Child data doesn't need to be queried independently",
    "Need atomic updates across related data"
  ]
}
```

#### 2. References (One-to-Many Relationship)
**When to use**: When you have many child documents or need to query child data independently

```python
# Example: Products belonging to Categories
# Instead of embedding all products inside category, we store category_id in each product

async def reference_relationship_example():
    """Demonstrate reference-based relationships"""
    
    # STEP 1: Create parent document (Category)
    category = {
        "_id": ObjectId("507f1f77bcf86cd799439012"),
        "name": "Smartphones",
        "description": "Mobile phones and accessories"
    }
    category_result = await async_db.categories.insert_one(category)
    
    # STEP 2: Create child documents (Products) with reference to parent
    products = [
        {
            "_id": ObjectId("507f1f77bcf86cd799439031"),
            "name": "iPhone 15",
            "price": 999.99,
            "category_id": ObjectId("507f1f77bcf86cd799439012"),  # REFERENCE to category
        },
        {
            "_id": ObjectId("507f1f77bcf86cd799439032"), 
            "name": "Samsung Galaxy S24",
            "price": 849.99,
            "category_id": ObjectId("507f1f77bcf86cd799439012"),  # REFERENCE to category
        }
    ]
    products_result = await async_db.products.insert_many(products)
    
    # QUERY 1: Find all products in a category (One-to-Many)
    smartphones = []
    async for product in async_db.products.find({"category_id": ObjectId("507f1f77bcf86cd799439012")}):
        smartphones.append({
            "_id": str(product["_id"]),
            "name": product["name"], 
            "price": product["price"]
        })
    
    # QUERY 2: Find category for a specific product (Many-to-One)
    product = await async_db.products.find_one({"_id": ObjectId("507f1f77bcf86cd799439031")})
    category = await async_db.categories.find_one({"_id": product["category_id"]})
    
    # QUERY 3: Using $lookup to join data (like SQL JOIN)
    products_with_category = []
    pipeline = [
        {"$match": {"category_id": ObjectId("507f1f77bcf86cd799439012")}},
        {
            "$lookup": {
                "from": "categories",           # Join with categories collection
                "localField": "category_id",    # Field in products collection
                "foreignField": "_id",          # Field in categories collection  
                "as": "category_info"          # Name for joined data
            }
        },
        {
            "$project": {
                "name": 1,
                "price": 1,
                "category_name": {"$arrayElemAt": ["$category_info.name", 0]}
            }
        }
    ]
    
    async for product in async_db.products.aggregate(pipeline):
        product["_id"] = str(product["_id"])
        products_with_category.append(product)
    
    return {
        "relationship_type": "References (One-to-Many)",
        "explanation": "Child documents store parent's _id as reference",
        "category_created": str(category_result.inserted_id),
        "products_created": len(products_result.inserted_ids),
        "query_results": {
            "products_in_category": smartphones,
            "product_with_category": {
                "product_name": product["name"],
                "category_name": category["name"]
            },
            "joined_data": products_with_category
        }
    }

reference_example = asyncio.run(reference_relationship_example())
```

**JSON Output:**
```json
{
  "relationship_type": "References (One-to-Many)",
  "explanation": "Child documents contain parent's _id as a reference field",
  "real_world_example": "Products belong to Categories, Orders belong to Customers, Posts belong to Users",
  "structure": {
    "parent_collection": "categories",
    "parent_document": {
      "_id": "ObjectId('507f1f77bcf86cd799439012')",
      "name": "Smartphones"
    },
    "child_collection": "products", 
    "child_document": {
      "_id": "ObjectId('507f1f77bcf86cd799439031')",
      "name": "iPhone 15",
      "category_id": "ObjectId('507f1f77bcf86cd799439012')"
    }
  },
  "query_patterns": {
    "find_children": {
      "query": "db.products.find({'category_id': category_id})",
      "explanation": "Find all products in a category",
      "result_count": 2
    },
    "find_parent": {
      "query": "db.categories.find({'_id': product.category_id})",
      "explanation": "Find category for a specific product"
    },
    "join_data": {
      "operation": "$lookup aggregation",
      "explanation": "Combine parent and child data in one query (like SQL JOIN)",
      "pipeline_stages": ["$match", "$lookup", "$project"]
    }
  },
  "advantages": [
    "No document size limits",
    "Can query child data independently", 
    "No data duplication",
    "Easy to maintain relationships",
    "Flexible - child can exist without parent"
  ],
  "disadvantages": [
    "Requires multiple queries or $lookup for complete data",
    "Slower reads compared to embedded documents",
    "No automatic referential integrity",
    "Need to handle orphaned documents"
  ],
  "when_to_use": [
    "One-to-many relationships with many children",
    "Child data needs to be queried independently", 
    "Child data is large or frequently updated",
    "Multiple parents can reference same child"
  ]
}
```

#### 3. Many-to-Many Relationships
**When to use**: When both sides can have multiple relationships with each other

```python
# Example: Products can have multiple Tags, and Tags can belong to multiple Products

async def many_to_many_relationship_example():
    """Demonstrate many-to-many relationships in MongoDB"""
    
    # METHOD 1: Embedding array of references (Simpler approach)
    # Store tag IDs in product document
    
    # Create some tags first
    tags = [
        {"_id": ObjectId("607f1f77bcf86cd799439001"), "name": "smartphone", "color": "blue"},
        {"_id": ObjectId("607f1f77bcf86cd799439002"), "name": "premium", "color": "gold"},
        {"_id": ObjectId("607f1f77bcf86cd799439003"), "name": "5g", "color": "green"},
        {"_id": ObjectId("607f1f77bcf86cd799439004"), "name": "android", "color": "green"}
    ]
    await async_db.tags.insert_many(tags)
    
    # Create products with tag references
    products = [
        {
            "_id": ObjectId("507f1f77bcf86cd799439031"),
            "name": "iPhone 15",
            "price": 999.99,
            # Array of tag IDs - Many-to-Many relationship
            "tag_ids": [
                ObjectId("607f1f77bcf86cd799439001"),  # smartphone
                ObjectId("607f1f77bcf86cd799439002"),  # premium
                ObjectId("607f1f77bcf86cd799439003")   # 5g
            ]
        },
        {
            "_id": ObjectId("507f1f77bcf86cd799439032"),
            "name": "Samsung Galaxy S24", 
            "price": 849.99,
            "tag_ids": [
                ObjectId("607f1f77bcf86cd799439001"),  # smartphone  
                ObjectId("607f1f77bcf86cd799439003"),  # 5g
                ObjectId("607f1f77bcf86cd799439004")   # android
            ]
        }
    ]
    await async_db.products.insert_many(products)
    
    # QUERY 1: Find all products with specific tag
    products_with_premium_tag = []
    async for product in async_db.products.find({"tag_ids": ObjectId("607f1f77bcf86cd799439002")}):
        product["_id"] = str(product["_id"])
        product["tag_ids"] = [str(tag_id) for tag_id in product["tag_ids"]]
        products_with_premium_tag.append(product)
    
    # QUERY 2: Find all products with multiple tags (AND condition)
    products_with_multiple_tags = []
    async for product in async_db.products.find({
        "tag_ids": {
            "$all": [  # $all means ALL these tags must be present
                ObjectId("607f1f77bcf86cd799439001"),  # smartphone
                ObjectId("607f1f77bcf86cd799439003")   # 5g
            ]
        }
    }):
        product["_id"] = str(product["_id"])
        product["tag_ids"] = [str(tag_id) for tag_id in product["tag_ids"]]
        products_with_multiple_tags.append(product)
    
    # QUERY 3: Find all products with any of these tags (OR condition)
    products_with_any_tag = []
    async for product in async_db.products.find({
        "tag_ids": {
            "$in": [  # $in means ANY of these tags
                ObjectId("607f1f77bcf86cd799439002"),  # premium
                ObjectId("607f1f77bcf86cd799439004")   # android
            ]
        }
    }):
        product["_id"] = str(product["_id"])
        product["tag_ids"] = [str(tag_id) for tag_id in product["tag_ids"]]
        products_with_any_tag.append(product)
    
    # QUERY 4: Get products with full tag information using $lookup
    products_with_tag_details = []
    pipeline = [
        {
            "$lookup": {
                "from": "tags",
                "localField": "tag_ids",
                "foreignField": "_id", 
                "as": "tag_details"
            }
        },
        {
            "$project": {
                "name": 1,
                "price": 1,
                "tags": {
                    "$map": {
                        "input": "$tag_details",
                        "as": "tag",
                        "in": {
                            "name": "$$tag.name",
                            "color": "$$tag.color"
                        }
                    }
                }
            }
        }
    ]
    
    async for product in async_db.products.aggregate(pipeline):
        product["_id"] = str(product["_id"])
        products_with_tag_details.append(product)
    
    # METHOD 2: Junction Collection (More complex but flexible)
    # Create separate collection for product-tag relationships
    product_tags = [
        {
            "product_id": ObjectId("507f1f77bcf86cd799439031"),
            "tag_id": ObjectId("607f1f77bcf86cd799439001"),
            "assigned_by": "admin",
            "assigned_at": datetime.utcnow()
        },
        {
            "product_id": ObjectId("507f1f77bcf86cd799439031"), 
            "tag_id": ObjectId("607f1f77bcf86cd799439002"),
            "assigned_by": "admin",
            "assigned_at": datetime.utcnow()
        }
    ]
    await async_db.product_tags.insert_many(product_tags)
    
    return {
        "relationship_type": "Many-to-Many",
        "methods_demonstrated": ["Array of References", "Junction Collection"],
        "query_results": {
            "products_with_premium_tag": products_with_premium_tag,
            "products_with_smartphone_and_5g": products_with_multiple_tags,
            "products_with_premium_or_android": products_with_any_tag,
            "products_with_full_tag_info": products_with_tag_details
        }
    }

many_to_many_example = asyncio.run(many_to_many_relationship_example())
```

**JSON Output:**
```json
{
  "relationship_type": "Many-to-Many",
  "explanation": "Both entities can have multiple relationships with each other",
  "real_world_examples": ["Products ↔ Tags", "Users ↔ Roles", "Students ↔ Courses", "Articles ↔ Categories"],
  
  "implementation_methods": {
    "method_1_array_references": {
      "description": "Store array of IDs in one collection",
      "structure": {
        "products": {
          "_id": "product_id",
          "name": "iPhone 15",
          "tag_ids": ["tag1_id", "tag2_id", "tag3_id"]
        },
        "tags": {
          "_id": "tag_id", 
          "name": "smartphone"
        }
      },
      "advantages": ["Simple to implement", "Fast queries", "Good for moderate relationships"],
      "disadvantages": ["Hard to add metadata", "One-sided relationship", "Array size limits"]
    },
    
    "method_2_junction_collection": {
      "description": "Separate collection for relationships",
      "structure": {
        "products": {"_id": "product_id", "name": "iPhone 15"},
        "tags": {"_id": "tag_id", "name": "smartphone"},
        "product_tags": {
          "product_id": "product_id",
          "tag_id": "tag_id", 
          "assigned_by": "admin",
          "assigned_at": "2024-03-15"
        }
      },
      "advantages": ["Can store relationship metadata", "Flexible", "True many-to-many"],
      "disadvantages": ["More complex queries", "Requires multiple collections", "More storage"]
    }
  },
  
  "query_patterns": {
    "find_by_single_tag": {
      "query": "{'tag_ids': tag_id}",
      "explanation": "MongoDB automatically searches arrays",
      "example": "Find all products tagged 'premium'"
    },
    "find_by_all_tags": {
      "query": "{'tag_ids': {'$all': [tag1, tag2]}}",
      "explanation": "$all means ALL specified tags must be present",
      "example": "Find products tagged with BOTH 'smartphone' AND '5g'"
    },
    "find_by_any_tags": {
      "query": "{'tag_ids': {'$in': [tag1, tag2]}}",  
      "explanation": "$in means ANY of the specified tags",
      "example": "Find products tagged with 'premium' OR 'android'"
    },
    "get_full_details": {
      "operation": "$lookup aggregation",
      "explanation": "Join arrays with referenced collection",
      "result": "Products with complete tag information"
    }
  },
  
  "array_operators_explained": {
    "$all": {
      "meaning": "ALL items in array must match",
      "example": "{'tags': {'$all': ['smart', 'phone']}}",
      "plain_english": "Find documents where tags array contains BOTH 'smart' AND 'phone'"
    },
    "$in": {
      "meaning": "ANY item can match",
      "example": "{'tags': {'$in': ['smart', 'tablet']}}",
      "plain_english": "Find documents where tags array contains 'smart' OR 'tablet'"
    },
    "$elemMatch": {
      "meaning": "At least one array element matches all conditions", 
      "example": "{'reviews': {'$elemMatch': {'rating': 5, 'verified': true}}}",
      "plain_english": "Find products with at least one 5-star verified review"
    },
    "$size": {
      "meaning": "Array has exact number of elements",
      "example": "{'tags': {'$size': 3}}",
      "plain_english": "Find products with exactly 3 tags"
    }
  },
  
  "when_to_use_each_method": {
    "array_references": [
      "Simple many-to-many relationships",
      "Don't need relationship metadata", 
      "Moderate number of relationships (< 1000)",
      "Primarily query from one side"
    ],
    "junction_collection": [
      "Need to store relationship metadata",
      "Large number of relationships",
      "Complex relationship logic",
      "Need to query relationships independently"
    ]
  }
}
```

## Querying and Filtering

### 1. Basic Query Operations

#### 1.1 Simple Filtering
```python
async def basic_product_queries() -> Dict[str, Any]:
    """Demonstrate basic query operations"""
    results = {}
    
    # 1. Find by single field
    active_products = []
    async for product in async_db.products.find({"is_active": True}):
        active_products.append(ProductService(async_db)._serialize_document(product))
    
    results["active_products"] = {
        "count": len(active_products),
        "products": active_products[:2]  # Show first 2 for brevity
    }
    
    # 2. Find by multiple fields (AND condition)
    featured_smartphones = []
    async for product in async_db.products.find({
        "is_featured": True,
        "category_id": ObjectId("507f1f77bcf86cd799439012"),
        "is_active": True
    }):
        featured_smartphones.append(ProductService(async_db)._serialize_document(product))
    
    results["featured_smartphones"] = {
        "count": len(featured_smartphones),
        "products": featured_smartphones
    }
    
    # 3. Range queries
    mid_range_products = []
    async for product in async_db.products.find({
        "price": {"$gte": 100, "$lte": 500},
        "is_active": True
    }):
        mid_range_products.append(ProductService(async_db)._serialize_document(product))
    
    results["mid_range_products"] = {
        "count": len(mid_range_products),
        "price_range": {"min": 100, "max": 500},
        "products": mid_range_products
    }
    
    # 4. Array field queries
    smartphone_products = []
    async for product in async_db.products.find({
        "tags": {"$in": ["smartphone", "mobile"]},
        "is_active": True
    }):
        smartphone_products.append(ProductService(async_db)._serialize_document(product))
    
    results["tagged_products"] = {
        "count": len(smartphone_products),
        "tags_searched": ["smartphone", "mobile"],
        "products": smartphone_products
    }
    
    return {
        "success": True,
        "query_operations": results
    }

# Example usage
basic_queries = asyncio.run(basic_product_queries())
```

**JSON Output:**
```json
{
  "operation": "basic_query_operations",
  "success": true,
  "query_operations": {
    "active_products": {
      "count": 4,
      "query": {"is_active": true},
      "products": [
        {
          "_id": "507f1f77bcf86cd799439031",
          "name": "iPhone 15 Pro",
          "price": 999.99,
          "is_active": true,
          "is_featured": true
        },
        {
          "_id": "507f1f77bcf86cd799439032",
          "name": "Premium Cotton T-Shirt",
          "price": 29.99,
          "is_active": true,
          "is_featured": false
        }
      ]
    },
    "featured_smartphones": {
      "count": 1,
      "query": {"is_featured": true, "category_id": "smartphones", "is_active": true},
      "products": [
        {
          "_id": "507f1f77bcf86cd799439031",
          "name": "iPhone 15 Pro",
          "category_id": "507f1f77bcf86cd799439012",
          "is_featured": true,
          "is_active": true,
          "price": 999.99
        }
      ]
    },
    "mid_range_products": {
      "count": 2,
      "price_range": {"min": 100, "max": 500},
      "query": {"price": {"$gte": 100, "$lte": 500}, "is_active": true},
      "products": [
        {
          "_id": "65f1234567890abcdef12346",
          "name": "Wireless Headphones",
          "price": 199.99,
          "is_active": true
        },
        {
          "_id": "65f1234567890abcdef12348",
          "name": "Denim Jeans",
          "price": 79.99,
          "is_active": true
        }
      ]
    },
    "tagged_products": {
      "count": 2,
      "tags_searched": ["smartphone", "mobile"],
      "query": {"tags": {"$in": ["smartphone", "mobile"]}, "is_active": true},
      "products": [
        {
          "_id": "507f1f77bcf86cd799439031",
          "name": "iPhone 15 Pro",
          "tags": ["smartphone", "apple", "premium", "5g"]
        },
        {
          "_id": "65f1234567890abcdef12345",
          "name": "Samsung Galaxy S24",
          "tags": ["smartphone", "android", "samsung", "ai"]
        }
      ]
    }
  },
  "query_patterns_demonstrated": [
    "Simple field matching",
    "Multiple field AND conditions",
    "Range queries with $gte/$lte",
    "Array field queries with $in operator"
  ]
}
```

#### 1.2 Complex Query Operations - Explained Step by Step
```python
async def complex_query_operations_explained() -> Dict[str, Any]:
    """Demonstrate complex MongoDB queries with detailed explanations"""
    results = {}
    
    # 1. OR CONDITIONS - When you want ANY condition to be true
    # Think: "Show me products that are either cheap OR featured"
    or_query = {
        "$or": [  # $or = "ANY of these conditions can be true"
            {"price": {"$lt": 50}},    # Condition 1: price less than 50
            {"is_featured": True}      # Condition 2: is featured
        ],
        "is_active": True  # AND this condition must also be true
    }
    
    or_results = []
    async for product in async_db.products.find(or_query):
        or_results.append({
            "_id": str(product["_id"]),
            "name": product["name"],
            "price": product["price"],
            "is_featured": product.get("is_featured", False),
            "match_reason": "cheap" if product["price"] < 50 else "featured"
        })
    
    results["or_condition_example"] = {
        "query": or_query,
        "explanation": "Find products that are either under $50 OR featured (or both)",
        "plain_english": "Show me products that are cheap or popular",
        "results": or_results
    }
    
    # 2. REGEX PATTERN MATCHING - For text searching
    # Think: "Find products with 'Phone' in the name, ignoring case"
    regex_query = {
        "name": {
            "$regex": "Phone",      # Pattern to search for
            "$options": "i"         # "i" = ignore case (case insensitive)
        },
        "is_active": True
    }
    
    regex_results = []
    async for product in async_db.products.find(regex_query):
        regex_results.append({
            "_id": str(product["_id"]),
            "name": product["name"],
            "price": product["price"]
        })
    
    results["regex_example"] = {
        "query": regex_query,
        "explanation": "$regex searches for text patterns, $options: 'i' makes it case-insensitive",
        "plain_english": "Find products with 'Phone' in name (iPhone, phone, PHONE all match)",
        "results": regex_results
    }
    
    # 3. NESTED FIELD QUERIES - Searching inside objects
    # Think: "Find products with specific chip in specifications"
    nested_query = {
        "specifications.chip": {  # Use dot notation for nested fields
            "$regex": "A17|Snapdragon",  # Match either A17 or Snapdragon
            "$options": "i"
        },
        "price": {"$gte": 500}  # AND price must be >= 500
    }
    
    nested_results = []
    async for product in async_db.products.find(nested_query):
        nested_results.append({
            "_id": str(product["_id"]),
            "name": product["name"],
            "price": product["price"],
            "chip": product.get("specifications", {}).get("chip", "Unknown")
        })
    
    results["nested_field_example"] = {
        "query": nested_query,
        "explanation": "Use dot notation (specifications.chip) to search inside nested objects",
        "plain_english": "Find expensive phones with premium processors",
        "results": nested_results
    }
    
    # 4. ARRAY ELEMENT MATCHING - Find specific items in arrays
    # Think: "Find products that have both 'smartphone' AND 'premium' tags"
    array_query = {
        "tags": {
            "$all": ["smartphone", "premium"]  # $all = ALL these items must be in array
        },
        "is_active": True
    }
    
    array_results = []
    async for product in async_db.products.find(array_query):
        array_results.append({
            "_id": str(product["_id"]),
            "name": product["name"],
            "tags": product.get("tags", [])
        })
    
    results["array_matching_example"] = {
        "query": array_query,
        "explanation": "$all ensures ALL specified items exist in the array",
        "plain_english": "Find products tagged with BOTH smartphone AND premium",
        "results": array_results
    }
    
    # 5. EXISTS AND TYPE CHECKING - Check if fields exist or have specific types
    # Think: "Find products that have a description and specifications"
    exists_query = {
        "description": {"$exists": True, "$ne": ""},  # Field exists AND is not empty
        "specifications": {"$exists": True, "$type": "object"},  # Field exists AND is an object
        "is_active": True
    }
    
    exists_results = []
    async for product in async_db.products.find(exists_query):
        exists_results.append({
            "_id": str(product["_id"]),
            "name": product["name"],
            "has_description": bool(product.get("description")),
            "has_specifications": bool(product.get("specifications")),
            "spec_count": len(product.get("specifications", {}))
        })
    
    results["exists_check_example"] = {
        "query": exists_query,
        "explanation": "$exists checks if field is present, $type checks the data type",
        "plain_english": "Find products with complete information (description and specs)",
        "results": exists_results
    }
    
    return {
        "success": True,
        "complex_query_examples": results,
        "key_operators_explained": {
            "$or": "ANY condition can be true (like OR in programming)",
            "$regex": "Search for text patterns (like LIKE in SQL)",
            "$options": "Modify how regex works ('i' = ignore case)",
            "$all": "ALL items must be in array",
            "$exists": "Check if field exists in document", 
            "$type": "Check what type of data the field contains",
            "$ne": "Not equal to (opposite of equals)",
            "dot_notation": "Use dots (.) to access nested object properties"
        }
    }

complex_explained = asyncio.run(complex_query_operations_explained())
```

**JSON Output:**
```json
{
  "operation": "complex_queries_explained_simply",
  "success": true,
  "complex_query_examples": {
    "or_condition_example": {
      "query": {
        "$or": [
          {"price": {"$lt": 50}},
          {"is_featured": true}
        ],
        "is_active": true
      },
      "explanation": "Find products that are either under $50 OR featured (or both)",
      "plain_english": "Show me products that are cheap or popular",
      "step_by_step": [
        "1. MongoDB checks each product",
        "2. If price < 50, it matches",
        "3. OR if is_featured = true, it matches", 
        "4. AND is_active must be true",
        "5. Returns products meeting any condition from steps 2-3 AND step 4"
      ],
      "results": [
        {
          "_id": "507f1f77bcf86cd799439031",
          "name": "iPhone 15 Pro",
          "price": 999.99,
          "is_featured": true,
          "match_reason": "featured"
        },
        {
          "_id": "507f1f77bcf86cd799439032",
          "name": "Premium Cotton T-Shirt",
          "price": 29.99,
          "is_featured": false,
          "match_reason": "cheap"
        }
      ]
    },
    "regex_example": {
      "query": {
        "name": {"$regex": "Phone", "$options": "i"},
        "is_active": true
      },
      "explanation": "$regex searches for text patterns, $options: 'i' makes it case-insensitive",
      "plain_english": "Find products with 'Phone' in name (iPhone, phone, PHONE all match)",
      "step_by_step": [
        "1. MongoDB looks at each product's name field",
        "2. Searches for the pattern 'Phone' anywhere in the name",
        "3. 'i' option means iPhone, phone, PHONE all match",
        "4. Only returns active products"
      ],
      "regex_options": {
        "i": "Case insensitive (iPhone = iphone = IPHONE)",
        "m": "Multiline matching", 
        "x": "Ignore whitespace",
        "s": "Dot matches newlines"
      },
      "results": [
        {
          "_id": "507f1f77bcf86cd799439031", 
          "name": "iPhone 15 Pro",
          "price": 999.99
        }
      ]
    },
    "nested_field_example": {
      "query": {
        "specifications.chip": {"$regex": "A17|Snapdragon", "$options": "i"},
        "price": {"$gte": 500}
      },
      "explanation": "Use dot notation (specifications.chip) to search inside nested objects",
      "plain_english": "Find expensive phones with premium processors",
      "step_by_step": [
        "1. MongoDB looks inside specifications object",
        "2. Checks the chip field inside specifications", 
        "3. Searches for 'A17' OR 'Snapdragon' in chip name",
        "4. AND price must be $500 or more",
        "5. Returns matching products"
      ],
      "dot_notation_examples": {
        "specifications.chip": "Access chip inside specifications object",
        "address.city": "Access city inside address object",
        "user.profile.name": "Access name inside profile inside user object"
      },
      "results": [
        {
          "_id": "507f1f77bcf86cd799439031",
          "name": "iPhone 15 Pro", 
          "price": 999.99,
          "chip": "A17 Pro"
        }
      ]
    },
    "array_matching_example": {
      "query": {
        "tags": {"$all": ["smartphone", "premium"]},
        "is_active": true
      },
      "explanation": "$all ensures ALL specified items exist in the array",
      "plain_english": "Find products tagged with BOTH smartphone AND premium",
      "step_by_step": [
        "1. MongoDB looks at tags array for each product",
        "2. Checks if 'smartphone' exists in tags",
        "3. AND checks if 'premium' exists in tags", 
        "4. Only matches if BOTH tags are present",
        "5. Product can have other tags too"
      ],
      "array_operators_compared": {
        "$all": "ALL specified items must be present",
        "$in": "ANY of the specified items can be present",
        "$nin": "NONE of the specified items should be present",
        "$size": "Array must have exact number of elements"
      },
      "results": [
        {
          "_id": "507f1f77bcf86cd799439031",
          "name": "iPhone 15 Pro",
          "tags": ["smartphone", "apple", "premium", "5g"]
        }
      ]
    },
    "exists_check_example": {
      "query": {
        "description": {"$exists": true, "$ne": ""},
        "specifications": {"$exists": true, "$type": "object"},
        "is_active": true
      },
      "explanation": "$exists checks if field is present, $type checks the data type",
      "plain_english": "Find products with complete information (description and specs)",
      "step_by_step": [
        "1. Check if description field exists in document",
        "2. AND check if description is not empty string",
        "3. AND check if specifications field exists",
        "4. AND check if specifications is an object (not string/number)",
        "5. AND product must be active"
      ],
      "mongodb_data_types": {
        "object": "Like a dictionary/JSON object {}",
        "array": "List of items []",
        "string": "Text data",
        "number": "Numeric data (int/float)",
        "boolean": "true/false",
        "date": "Date and time",
        "null": "Empty/no value"
      },
      "results": [
        {
          "_id": "507f1f77bcf86cd799439031",
          "name": "iPhone 15 Pro", 
          "has_description": true,
          "has_specifications": true,
          "spec_count": 5
        }
      ]
    }
  },
  "key_operators_explained": {
    "$or": {
      "simple_explanation": "ANY condition can be true (like OR in programming)",
      "example": "Find cheap OR featured products",
      "syntax": "{'$or': [condition1, condition2]}"
    },
    "$regex": {
      "simple_explanation": "Search for text patterns (like LIKE in SQL)", 
      "example": "Find products with 'Phone' in name",
      "syntax": "{'field': {'$regex': 'pattern', '$options': 'i'}}"
    },
    "$all": {
      "simple_explanation": "ALL items must be in array",
      "example": "Product must have BOTH 'smart' AND 'phone' tags",
      "syntax": "{'array_field': {'$all': ['item1', 'item2']}}"
    },
    "$exists": {
      "simple_explanation": "Check if field exists in document",
      "example": "Find products that have a description field",
      "syntax": "{'field': {'$exists': true}}"
    },
    "$type": {
      "simple_explanation": "Check what type of data the field contains",
      "example": "Find products where price is a number",
      "syntax": "{'field': {'$type': 'number'}}"
    },
    "dot_notation": {
      "simple_explanation": "Use dots (.) to access nested object properties",
      "example": "specifications.chip accesses chip inside specifications",
      "syntax": "{'nested.field': value}"
    }
  },
  "query_building_tips": [
    "Start simple: test with one condition first",
    "Add conditions one by one to see what changes",
    "Use MongoDB Compass or Studio 3T to visualize queries",
    "Test queries with find() before using in aggregation",
    "Remember: MongoDB is case-sensitive by default"
  ]
}
```

## Aggregation Pipeline - The Complete Simple Guide

### What is Aggregation Pipeline?

Think of an aggregation pipeline like a factory assembly line. Your data goes in one end, passes through multiple stages (like machines in a factory), and comes out transformed at the other end.

**Simple analogy**: 
- Input: Raw materials (your documents)
- Stages: Different machines (pipeline operations like $match, $group, $sort)
- Output: Finished product (transformed data)

#### Aggregation Pipeline Stages Explained Simply

```python
async def aggregation_pipeline_explained() -> Dict[str, Any]:
    """Explain each aggregation stage in simple terms with examples"""
    
    # Let's build a pipeline step by step to understand each stage
    
    # STAGE 1: $match - Filter documents (like WHERE in SQL)
    # Think: "First, let me pick only the products I care about"
    match_stage = {"$match": {"is_active": True, "price": {"$gte": 100}}}
    
    # STAGE 2: $group - Group similar documents and calculate stats
    # Think: "Now group products by category and count them"
    group_stage = {
        "$group": {
            "_id": "$category_id",  # Group by this field
            "product_count": {"$sum": 1},  # Count products in each group
            "avg_price": {"$avg": "$price"},  # Calculate average price
            "total_stock": {"$sum": "$stock_quantity"}  # Sum all stock
        }
    }
    
    # STAGE 3: $lookup - Join with another collection (like JOIN in SQL)
    # Think: "Add category name from categories collection"
    lookup_stage = {
        "$lookup": {
            "from": "categories",        # Collection to join with
            "localField": "_id",         # Field from current data
            "foreignField": "_id",       # Field from categories collection
            "as": "category_info"       # Name for the joined data
        }
    }
    
    # STAGE 4: $addFields - Add new calculated fields
    # Think: "Add some computed fields for easier reading"
    addfields_stage = {
        "$addFields": {
            "category_name": {"$arrayElemAt": ["$category_info.name", 0]},
            "avg_price_rounded": {"$round": ["$avg_price", 2]},
            "stock_status": {
                "$cond": {
                    "if": {"$gte": ["$total_stock", 100]},
                    "then": "High Stock",
                    "else": "Low Stock"
                }
            }
        }
    }
    
    # STAGE 5: $project - Select which fields to show (like SELECT in SQL)
    # Think: "Clean up the output, show only what I need"
    project_stage = {
        "$project": {
            "category_name": 1,
            "product_count": 1,
            "avg_price_rounded": 1,
            "total_stock": 1,
            "stock_status": 1,
            # Don't show these fields (0 = hide, 1 = show)
            "category_info": 0,
            "_id": 0
        }
    }
    
    # STAGE 6: $sort - Sort the results (like ORDER BY in SQL)
    # Think: "Sort by product count, highest first"
    sort_stage = {"$sort": {"product_count": -1}}  # -1 = descending, 1 = ascending
    
    # STAGE 7: $limit - Limit number of results
    # Think: "Just show me top 5 categories"
    limit_stage = {"$limit": 5}
    
    # Now let's build the complete pipeline
    pipeline = [
        match_stage,     # 1. Filter products
        group_stage,     # 2. Group by category
        lookup_stage,    # 3. Join with categories
        addfields_stage, # 4. Add computed fields
        project_stage,   # 5. Select fields to show
        sort_stage,      # 6. Sort results
        limit_stage      # 7. Limit to 5 results
    ]
    
    # Execute the pipeline
    results = []
    async for doc in async_db.products.aggregate(pipeline):
        results.append(doc)
    
    return {
        "pipeline_explanation": "Step-by-step breakdown of aggregation pipeline",
        "pipeline_stages": {
            "1_match": {
                "purpose": "Filter documents (like WHERE clause)",
                "simple_explanation": "Pick only active products over $100",
                "stage": match_stage,
                "sql_equivalent": "WHERE is_active = true AND price >= 100"
            },
            "2_group": {
                "purpose": "Group documents and calculate statistics",
                "simple_explanation": "Group products by category and count them",
                "stage": group_stage,
                "sql_equivalent": "GROUP BY category_id"
            },
            "3_lookup": {
                "purpose": "Join with another collection",
                "simple_explanation": "Get category names from categories collection",
                "stage": lookup_stage,
                "sql_equivalent": "LEFT JOIN categories ON products.category_id = categories._id"
            },
            "4_addFields": {
                "purpose": "Add computed/calculated fields",
                "simple_explanation": "Add category name and stock status labels",
                "stage": addfields_stage,
                "sql_equivalent": "SELECT *, CASE WHEN stock >= 100 THEN 'High' ELSE 'Low' END"
            },
            "5_project": {
                "purpose": "Select which fields to include/exclude",
                "simple_explanation": "Show only the fields we want in final result",
                "stage": project_stage,
                "sql_equivalent": "SELECT specific_fields"
            },
            "6_sort": {
                "purpose": "Sort the results",
                "simple_explanation": "Sort by product count, highest first",
                "stage": sort_stage,
                "sql_equivalent": "ORDER BY product_count DESC"
            },
            "7_limit": {
                "purpose": "Limit number of results",
                "simple_explanation": "Show only top 5 categories",
                "stage": limit_stage,
                "sql_equivalent": "LIMIT 5"
            }
        },
        "complete_pipeline": pipeline,
        "results": results,
        "data_flow": [
            "Raw product documents → $match → Filtered products",
            "Filtered products → $group → Category statistics", 
            "Category stats → $lookup → Stats with category names",
            "Enriched stats → $addFields → Stats with computed fields",
            "Complete stats → $project → Clean final format",
            "Clean data → $sort → Sorted by count",
            "Sorted data → $limit → Top 5 results"
        ]
    }

pipeline_explained = asyncio.run(aggregation_pipeline_explained())
```

**JSON Output:**
```json
{
  "aggregation_pipeline_complete_guide": {
    "what_is_aggregation": "A series of data processing stages that transform documents",
    "simple_analogy": "Like a factory assembly line - data goes through multiple machines (stages)",
    
    "pipeline_stages_explained": {
      "1_match": {
        "purpose": "Filter documents (like WHERE clause)",
        "simple_explanation": "Pick only the documents you want to work with",
        "example": {"$match": {"is_active": true, "price": {"$gte": 100}}},
        "plain_english": "Only active products that cost $100 or more",
        "sql_equivalent": "WHERE is_active = true AND price >= 100",
        "when_to_use": "Always put $match early to filter data and improve performance"
      },
      
      "2_group": {
        "purpose": "Group documents and calculate statistics",
        "simple_explanation": "Put similar documents together and count/sum/average them",
        "example": {
          "$group": {
            "_id": "$category_id",
            "count": {"$sum": 1},
            "avg_price": {"$avg": "$price"}
          }
        },
        "plain_english": "Group products by category, count them, and find average price",
        "sql_equivalent": "GROUP BY category_id",
        "group_operators": {
          "$sum": "Add up numbers (or count with $sum: 1)",
          "$avg": "Calculate average",
          "$min": "Find minimum value",
          "$max": "Find maximum value",
          "$first": "Get first value in group",
          "$last": "Get last value in group",
          "$push": "Collect all values into array",
          "$addToSet": "Collect unique values into array"
        }
      },
      
      "3_lookup": {
        "purpose": "Join with another collection (like JOIN in SQL)",
        "simple_explanation": "Get related data from another collection",
        "example": {
          "$lookup": {
            "from": "categories",
            "localField": "category_id", 
            "foreignField": "_id",
            "as": "category_info"
          }
        },
        "plain_english": "For each product, get its category information",
        "sql_equivalent": "LEFT JOIN categories ON products.category_id = categories._id",
        "result": "Adds an array field with matching documents from other collection"
      },
      
      "4_addFields": {
        "purpose": "Add new computed/calculated fields",
        "simple_explanation": "Create new fields based on existing data",
        "example": {
          "$addFields": {
            "full_name": {"$concat": ["$first_name", " ", "$last_name"]},
            "is_expensive": {"$gte": ["$price", 1000]}
          }
        },
        "plain_english": "Add full_name field and is_expensive boolean",
        "sql_equivalent": "SELECT *, CONCAT(first_name, ' ', last_name) as full_name",
        "common_uses": ["Combine fields", "Add calculated values", "Add conditional fields"]
      },
      
      "5_project": {
        "purpose": "Select which fields to include/exclude in output",
        "simple_explanation": "Choose what fields to show (like SELECT in SQL)",
        "example": {
          "$project": {
            "name": 1,
            "price": 1,
            "category_name": "$category_info.name",
            "_id": 0
          }
        },
        "plain_english": "Show name, price, and category_name. Hide _id",
        "sql_equivalent": "SELECT name, price, category_name",
        "field_values": {
          "1": "Include this field",
          "0": "Exclude this field",
          "$field_name": "Rename or transform field"
        }
      },
      
      "6_sort": {
        "purpose": "Sort the results in ascending or descending order",
        "simple_explanation": "Put results in order (like ORDER BY in SQL)",
        "example": {"$sort": {"price": -1, "name": 1}},
        "plain_english": "Sort by price (high to low), then by name (A to Z)",
        "sql_equivalent": "ORDER BY price DESC, name ASC",
        "sort_values": {
          "1": "Ascending order (A to Z, 1 to 10)",
          "-1": "Descending order (Z to A, 10 to 1)"
        }
      },
      
      "7_limit": {
        "purpose": "Limit the number of results returned", 
        "simple_explanation": "Only show first N results",
        "example": {"$limit": 10},
        "plain_english": "Show only the first 10 results",
        "sql_equivalent": "LIMIT 10",
        "common_use": "Get top N results after sorting"
      },
      
      "8_skip": {
        "purpose": "Skip the first N documents",
        "simple_explanation": "Skip some results (for pagination)",
        "example": {"$skip": 20},
        "plain_english": "Skip the first 20 results",
        "sql_equivalent": "OFFSET 20",
        "pagination_example": "Skip 20, limit 10 = get results 21-30"
      }
    },
    
    "data_flow_visualization": {
      "input": "All product documents in database",
      "stage_1_match": "Filter: only active products over $100",
      "stage_2_group": "Group by category, calculate counts and averages",
      "stage_3_lookup": "Add category names from categories collection",
      "stage_4_addFields": "Add computed fields like stock_status",
      "stage_5_project": "Select only needed fields for output",
      "stage_6_sort": "Sort by product count (highest first)",
      "stage_7_limit": "Take only top 5 results",
      "output": "Top 5 categories with their statistics"
    },
    
    "performance_tips": [
      "Put $match as early as possible to filter data",
      "Use indexes on fields used in $match and $sort",
      "$lookup is expensive - use only when necessary", 
      "Put $limit after $sort for better performance",
      "Avoid $project if you need all fields"
    ],
    
    "common_mistakes": [
      "Forgetting $match early - processes too much data",
      "Wrong field names in $group _id",
      "$lookup without proper indexes",
      "Using $project unnecessarily",
      "Not understanding $lookup returns arrays"
    ]
  }
}
```