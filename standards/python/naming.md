# Python Naming Conventions (PEP 8)

## Variables and Functions

### Variables
- **snake_case**: `first_name`, `user_count`, `is_active`
- Booleans: prefix with `is_`, `has_`, `should_`, `can_`
  ```python
  # ✅ Good
  first_name = "John"
  user_count = 10
  is_active = True
  has_permission = False
  should_retry = True
  
  # ❌ Bad
  firstName = "John"    # camelCase
  FirstName = "John"    # PascalCase
  ```

### Functions
- **snake_case**: `get_user_by_id`, `calculate_total`, `validate_input`
- verb-noun pattern
  ```python
  # ✅ Good
  def get_user(user_id: int) -> User:
      pass
  
  def create_user(email: str, password: str) -> User:
      pass
  
  def validate_email(email: str) -> bool:
      pass
  
  # ❌ Bad
  def getUser(user_id: int):    # camelCase
      pass
  
  def get_user_byId(user_id):   # Mixed case
      pass
  ```

### Private Members
- Prefix with underscore: `_internal_method`, `_private_var`
  ```python
  class UserService:
      def __init__(self):
          self._cache = {}  # Private
        
      def get_user(self, user_id: int):  # Public
          return self._find_in_cache(user_id)
      
      def _find_in_cache(self, user_id: int):  # Private
          return self._cache.get(user_id)
  ```

## Classes

### Class Names
- **PascalCase**: `UserService`, `OrderRepository`, `UserNotFound`
- Use nouns for class names
  ```python
  # ✅ Good
  class UserService:
      pass
  
  class OrderRepository:
      pass
  
  class UserNotFoundError(Exception):
      pass
  
  # ❌ Bad
  class userService:    # lowercase
      pass
  
  class user_service:   # snake_case
      pass
  ```

## Constants

### Module-Level Constants
- **SCREAMING_SNAKE_CASE**: `MAX_RETRIES`, `DEFAULT_TIMEOUT`, `API_VERSION`
  ```python
  # ✅ Good
  MAX_RETRIES = 3
  DEFAULT_TIMEOUT = 30
  API_VERSION = "v1"
  BASE_URL = "https://api.example.com"
  
  # ❌ Bad
  maxRetries = 3        # camelCase
  MaxRetries = 3        # PascalCase
  ```

## Files and Modules

### File Names
- **snake_case**: `user_service.py`, `order_repository.py`
- Test files: `test_user_service.py`, `test_order_repository.py`
  ```
  # ✅ Good
  user_service.py
  order_repository.py
  test_user_service.py
  
  # ❌ Bad
  userService.py        # camelCase
  UserService.py        # PascalCase
  ```

### Module Names
- **snake_case**, short, lowercase
  ```python
  # ✅ Good
  import user_service
  from order_repository import OrderRepository
  
  # ❌ Bad
  import UserService    # PascalCase
  ```

## Database

### Table Names
- **snake_case**, plural: `users`, `orders`, `order_items`
  ```python
  # Django
  class Meta:
      db_table = 'users'
  
  # SQLAlchemy
  __tablename__ = 'users'
  ```

### Column Names
- **snake_case**: `first_name`, `created_at`, `is_active`
  ```python
  # ✅ Good
  first_name = Column(String(100))
  created_at = Column(DateTime)
  is_active = Column(Boolean, default=True)
  ```

## Packages and Directories

### Package Names
- **snake_case**, short, lowercase
  ```
  # ✅ Good
  apps/
  ├── users/
  │   ├── models.py
  │   ├── services.py
  │   └── views.py
  └── orders/
  
  # ❌ Bad
  Apps/
  ├── Users/
  └── Orders/
  ```

## Type Variables

### Generic Types
- Single uppercase letter: `T`, `TResult`, `TEntity`
  ```python
  from typing import TypeVar, Generic
  
  T = TypeVar('T')
  
  class Repository(Generic[T]):
      def get_by_id(self, id: int) -> T:
          pass
  ```

## Decorators

### Decorator Names
- **snake_case**: `@login_required`, `@cache_result`
  ```python
  def login_required(func):
      def wrapper(*args, **kwargs):
          # Check authentication
          return func(*args, **kwargs)
      return wrapper
  
  @login_required
  def get_user_profile(user_id: int):
      pass
  ```

## Imports

### Import Order
1. Standard library
2. Third-party packages
3. Local application modules
  ```python
  # ✅ Good
  import os
  import sys
  from datetime import datetime
  
  from fastapi import FastAPI
  from sqlalchemy import Column, Integer
  
  from src.models.user import User
  from src.services.user_service import UserService
  ```

## Best Practices

### Do
- Follow PEP 8 consistently
- Use descriptive, meaningful names
- Use snake_case for variables, functions, modules
- Use PascalCase for classes
- Use SCREAMING_SNAKE_CASE for constants
- Use type hints for function signatures
- Use plural names for collections
- Prefix booleans with `is_`, `has_`, `should_`

### Don't
- Don't use camelCase (that's JavaScript/Java style)
- Don't use single-letter variable names (except in loops: `i`, `j`, `k`)
- Don't use abbreviations (unless widely understood: `id`, `url`, `db`)
- Don't use names that don't describe the purpose
- Don't use generic names like `data`, `info`, `temp`
- Don't mix naming conventions
- Don't use reserved words as variable names
- Don't use names that are too long or too short
