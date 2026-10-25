# Python Conventions

## Type Hints

### Function Signatures
```python
# ✅ Good
def get_user(user_id: int) -> User:
    pass

def create_user(email: str, password: str, name: str) -> User:
    pass

def get_users(skip: int = 0, limit: int = 100) -> list[User]:
    pass

# ❌ Bad
def get_user(user_id):    # Missing type hints
    pass
```

### Optional Types
```python
from typing import Optional

# ✅ Good
def get_user_by_email(email: str) -> Optional[User]:
    pass

# Modern Python 3.10+
def get_user_by_email(email: str) -> User | None:
    pass
```

### Complex Types
```python
from typing import Dict, List, Tuple, Set

# ✅ Good
def get_user_orders(user_id: int) -> list[Order]:
    pass

def get_user_metadata(user_id: int) -> dict[str, Any]:
    pass

def get_coordinates() -> tuple[float, float]:
    pass
```

## Virtual Environments

### Creating Virtual Environment
```bash
# Using venv
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
.venv\Scripts\activate     # Windows

# Using Poetry
poetry init
poetry install
poetry shell
```

### Requirements
```txt
# requirements.txt (pip)
fastapi==0.104.0
sqlalchemy==2.0.23
pydantic==2.5.0
python-dotenv==1.0.0

# pyproject.toml (Poetry)
[tool.poetry.dependencies]
python = "^3.10"
fastapi = "^0.104.0"
sqlalchemy = "^2.0.23"
```

## Imports

### Import Order
```python
# 1. Standard library
import os
import sys
from datetime import datetime
from typing import Optional

# 2. Third-party packages
from fastapi import FastAPI, Depends
from sqlalchemy import Column, Integer, String
from pydantic import BaseModel

# 3. Local application
from src.models.user import User
from src.services.user_service import UserService
from src.core.config import settings
```

### Import Styles
```python
# ✅ Good - Explicit imports
from src.models.user import User
from src.services.user_service import UserService

# ❌ Bad - Wildcard imports
from src.models import *
from src.services import *
```

## Error Handling

### Custom Exceptions
```python
class AppError(Exception):
    """Base exception for application errors"""
    def __init__(self, message: str, code: str = "INTERNAL_ERROR", status_code: int = 500):
        self.message = message
        self.code = code
        self.status_code = status_code
        super().__init__(self.message)

class NotFoundError(AppError):
    def __init__(self, resource: str):
        super().__init__(
            message=f"{resource} not found",
            code="NOT_FOUND",
            status_code=404
        )

class ValidationError(AppError):
    def __init__(self, message: str):
        super().__init__(
            message=message,
            code="VALIDATION_ERROR",
            status_code=400
        )

class ConflictError(AppError):
    def __init__(self, message: str):
        super().__init__(
            message=message,
            code="CONFLICT",
            status_code=409
        )
```

### Try-Except Best Practices
```python
# ✅ Good - Catch specific exceptions
try:
    user = user_service.get_user(user_id)
except NotFoundError as e:
    logger.warning(f"User {user_id} not found")
    raise HTTPException(status_code=404, detail=str(e))
except ValidationError as e:
    logger.error(f"Validation error: {e}")
    raise HTTPException(status_code=400, detail=str(e))

# ❌ Bad - Catching generic Exception
try:
    user = user_service.get_user(user_id)
except Exception as e:
    # Too broad
    raise HTTPException(status_code=500, detail=str(e))
```

## Context Managers

### Using Context Managers
```python
# ✅ Good - Using with statement
with open('file.txt', 'r') as f:
    content = f.read()

# Database session
with Session() as session:
    user = session.query(User).first()
```

### Custom Context Manager
```python
from contextlib import contextmanager

@contextmanager
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Usage
with get_db() as db:
    user = db.query(User).first()
```

## Data Classes

### Using dataclass
```python
from dataclasses import dataclass
from datetime import datetime

@dataclass
class User:
    id: int
    email: str
    name: str
    is_active: bool = True
    created_at: datetime = None

    def __post_init__(self):
        if self.created_at is None:
            self.created_at = datetime.now()
```

## List Comprehensions

### List Comprehension
```python
# ✅ Good
user_names = [user.name for user in users]
active_users = [user for user in users if user.is_active]

# ❌ Bad - Using map/filter (less readable)
user_names = list(map(lambda u: u.name, users))
active_users = list(filter(lambda u: u.is_active, users))
```

### Dictionary Comprehension
```python
# ✅ Good
user_dict = {user.id: user for user in users}
```

## Async/Await

### Async Functions
```python
import asyncio

# ✅ Good
async def get_user(user_id: int) -> User:
    user = await db.fetch_one("SELECT * FROM users WHERE id = $1", user_id)
    return User(**user)

async def get_users() -> list[User]:
    users = await db.fetch_all("SELECT * FROM users")
    return [User(**user) for user in users]
```

### Concurrent Operations
```python
import asyncio

async def get_user_profile(user_id: int) -> dict:
    user_task = get_user(user_id)
    orders_task = get_user_orders(user_id)
    preferences_task = get_user_preferences(user_id)
    
    user, orders, preferences = await asyncio.gather(
        user_task,
        orders_task,
        preferences_task
    )
    
    return {
        "user": user,
        "orders": orders,
        "preferences": preferences
    }
```

## Logging

### Using Logging
```python
import logging

logger = logging.getLogger(__name__)

# ✅ Good
logger.info(f"User {user_id} created successfully")
logger.warning(f"User {user_id} not found")
logger.error(f"Error creating user: {e}")
logger.debug(f"Query params: {params}")

# ❌ Bad - Using print
print(f"User {user_id} created")
```

## Decorators

### Function Decorator
```python
from functools import wraps
import time

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        logger.info(f"{func.__name__} took {end - start:.2f}s")
        return result
    return wrapper

@timer
def slow_operation():
    time.sleep(2)
    return "Done"
```

## Best Practices

### Do
- Use type hints for all function signatures
- Use virtual environments for every project
- Use specific exception handling
- Use context managers for resources
- Use list comprehensions when appropriate
- Use logging instead of print
- Use dataclasses for simple data structures
- Use async/await for I/O-bound operations
- Use f-strings for string formatting
- Use environment variables for configuration

### Don't
- Don't use bare except clauses
- Don't use global variables
- Don't use mutable default arguments
- Don't mix synchronous and async code unnecessarily
- Don't use print for logging
- Don't hardcode configuration values
- Don't use wildcard imports
- Don't forget to close resources (files, connections)
- Don't use `eval()` or `exec()`
- Don't use `os.system()` (use `subprocess` instead)
