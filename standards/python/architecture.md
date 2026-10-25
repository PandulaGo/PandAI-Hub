# Python Architecture

## Layering

### Standard Layered Architecture
```
Controllers/Views → Services → Repositories → Database
```

- **Controllers/Views**: Handle HTTP requests/responses, routing
- **Services**: Business logic, orchestration, data transformation
- **Repositories**: Data access, database queries
- **Models**: Data structures, ORM models

## Project Structure

### Django Project
```
project-root/
├── apps/
│   ├── users/
│   │   ├── migrations/
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── services.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── admin.py
│   │   └── tests/
│   │       ├── test_models.py
│   │       ├── test_services.py
│   │       └── test_views.py
│   └── orders/
├── core/
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── manage.py
├── requirements.txt
└── .env
```

### FastAPI Project
```
project-root/
├── src/
│   ├── routers/
│   │   ├── user_router.py
│   │   └── order_router.py
│   ├── services/
│   │   ├── user_service.py
│   │   └── order_service.py
│   ├── repositories/
│   │   ├── user_repository.py
│   │   └── order_repository.py
│   ├── models/
│   │   ├── user.py
│   │   └── order.py
│   ├── schemas/
│   │   ├── user.py
│   │   └── order.py
│   ├── core/
│   │   ├── config.py
│   │   ├── security.py
│   │   └── database.py
│   ├── middleware/
│   │   └── auth.py
│   └── main.py
├── tests/
│   ├── test_users.py
│   └── test_orders.py
├── requirements.txt
└── .env
```

## Django Patterns

### Model Definition
```python
# apps/users/models.py
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    email = models.EmailField(unique=True)
    phone = models.CharField(max_length=20, blank=True)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        db_table = 'users'
        ordering = ['-created_at']

    def __str__(self):
        return self.email
```

### Service Layer
```python
# apps/users/services.py
from django.contrib.auth import get_user_model
from django.core.exceptions import ValidationError
from .models import User

User = get_user_model()

class UserService:
    @staticmethod
    def get_user_by_id(user_id: int) -> User:
        try:
            return User.objects.get(id=user_id)
        except User.DoesNotExist:
            raise ValidationError(f"User with id {user_id} not found")

    @staticmethod
    def get_user_by_email(email: str) -> User:
        try:
            return User.objects.get(email=email)
        except User.DoesNotExist:
            raise ValidationError(f"User with email {email} not found")

    @staticmethod
    def create_user(email: str, password: str, **kwargs) -> User:
        if User.objects.filter(email=email).exists():
            raise ValidationError("Email already exists")
        
        user = User.objects.create_user(
            email=email,
            password=password,
            **kwargs
        )
        return user

    @staticmethod
    def update_user(user_id: int, **kwargs) -> User:
        user = UserService.get_user_by_id(user_id)
        for key, value in kwargs.items():
            setattr(user, key, value)
        user.save()
        return user

    @staticmethod
    def delete_user(user_id: int) -> None:
        user = UserService.get_user_by_id(user_id)
        user.delete()
```

### View (DRF)
```python
# apps/users/views.py
from rest_framework import viewsets, status
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from .models import User
from .serializers import UserSerializer
from .services import UserService

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [IsAuthenticated]

    def create(self, request, *args, **kwargs):
        try:
            user = UserService.create_user(**request.data)
            serializer = self.get_serializer(user)
            return Response(
                {'success': True, 'data': serializer.data},
                status=status.HTTP_201_CREATED
            )
        except ValidationError as e:
            return Response(
                {'success': False, 'error': str(e)},
                status=status.HTTP_400_BAD_REQUEST
            )
```

### Serializer
```python
# apps/users/serializers.py
from rest_framework import serializers
from .models import User

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'email', 'first_name', 'last_name', 'is_active', 'created_at']
        read_only_fields = ['id', 'created_at']

class CreateUserSerializer(serializers.ModelSerializer):
    password = serializers.CharField(write_only=True, min_length=8)

    class Meta:
        model = User
        fields = ['email', 'password', 'first_name', 'last_name']

    def create(self, validated_data):
        return User.objects.create_user(**validated_data)
```

## FastAPI Patterns

### Model (SQLAlchemy)
```python
# src/models/user.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime
from sqlalchemy.sql import func
from src.core.database import Base

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, index=True, nullable=False)
    name = Column(String(100), nullable=False)
    password_hash = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
```

### Schema (Pydantic)
```python
# src/schemas/user.py
from pydantic import BaseModel, EmailStr, Field
from datetime import datetime
from typing import Optional

class UserBase(BaseModel):
    email: EmailStr
    name: str = Field(..., min_length=2, max_length=100)

class UserCreate(UserBase):
    password: str = Field(..., min_length=8)

class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    name: Optional[str] = Field(None, min_length=2, max_length=100)

class UserResponse(UserBase):
    id: int
    is_active: bool
    created_at: datetime

    class Config:
        from_attributes = True
```

### Repository
```python
# src/repositories/user_repository.py
from sqlalchemy.orm import Session
from src.models.user import User
from src.schemas.user import UserCreate, UserUpdate
from typing import Optional, List

class UserRepository:
    def __init__(self, db: Session):
        self.db = db

    def get_by_id(self, user_id: int) -> Optional[User]:
        return self.db.query(User).filter(User.id == user_id).first()

    def get_by_email(self, email: str) -> Optional[User]:
        return self.db.query(User).filter(User.email == email).first()

    def get_all(self, skip: int = 0, limit: int = 100) -> List[User]:
        return self.db.query(User).offset(skip).limit(limit).all()

    def create(self, user_data: UserCreate, password_hash: str) -> User:
        user = User(
            email=user_data.email,
            name=user_data.name,
            password_hash=password_hash
        )
        self.db.add(user)
        self.db.commit()
        self.db.refresh(user)
        return user

    def update(self, user_id: int, user_data: UserUpdate) -> Optional[User]:
        user = self.get_by_id(user_id)
        if not user:
            return None
        for key, value in user_data.dict(exclude_unset=True).items():
            setattr(user, key, value)
        self.db.commit()
        self.db.refresh(user)
        return user

    def delete(self, user_id: int) -> bool:
        user = self.get_by_id(user_id)
        if not user:
            return False
        self.db.delete(user)
        self.db.commit()
        return True
```

### Service
```python
# src/services/user_service.py
from sqlalchemy.orm import Session
from src.repositories.user_repository import UserRepository
from src.schemas.user import UserCreate, UserUpdate
from src.core.security import hash_password
from typing import Optional, List
from src.models.user import User

class UserService:
    def __init__(self, db: Session):
        self.repository = UserRepository(db)

    def get_user(self, user_id: int) -> Optional[User]:
        user = self.repository.get_by_id(user_id)
        if not user:
            raise ValueError(f"User {user_id} not found")
        return user

    def get_users(self, skip: int = 0, limit: int = 100) -> List[User]:
        return self.repository.get_all(skip, limit)

    def create_user(self, user_data: UserCreate) -> User:
        existing = self.repository.get_by_email(user_data.email)
        if existing:
            raise ValueError("Email already exists")
        
        password_hash = hash_password(user_data.password)
        return self.repository.create(user_data, password_hash)

    def update_user(self, user_id: int, user_data: UserUpdate) -> User:
        user = self.repository.update(user_id, user_data)
        if not user:
            raise ValueError(f"User {user_id} not found")
        return user

    def delete_user(self, user_id: int) -> None:
        success = self.repository.delete(user_id)
        if not success:
            raise ValueError(f"User {user_id} not found")
```

### Router
```python
# src/routers/user_router.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from src.core.database import get_db
from src.services.user_service import UserService
from src.schemas.user import UserCreate, UserUpdate, UserResponse
from typing import List

router = APIRouter(prefix="/api/v1/users", tags=["users"])

@router.get("/", response_model=List[UserResponse])
def get_users(
    skip: int = 0,
    limit: int = 100,
    db: Session = Depends(get_db)
):
    service = UserService(db)
    return service.get_users(skip, limit)

@router.get("/{user_id}", response_model=UserResponse)
def get_user(user_id: int, db: Session = Depends(get_db)):
    service = UserService(db)
    try:
        return service.get_user(user_id)
    except ValueError as e:
        raise HTTPException(status_code=404, detail=str(e))

@router.post("/", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
def create_user(user_data: UserCreate, db: Session = Depends(get_db)):
    service = UserService(db)
    try:
        return service.create_user(user_data)
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
```

## Configuration

### Django Settings
```python
# core/settings/base.py
import os
from pathlib import Path
from dotenv import load_dotenv

load_dotenv()

BASE_DIR = Path(__file__).resolve().parent.parent.parent

SECRET_KEY = os.getenv('SECRET_KEY')
DEBUG = os.getenv('DEBUG', 'False') == 'True'
ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', '').split(',')

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'corsheaders',
    'apps.users',
    'apps.orders',
]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('DB_NAME'),
        'USER': os.getenv('DB_USER'),
        'PASSWORD': os.getenv('DB_PASSWORD'),
        'HOST': os.getenv('DB_HOST', 'localhost'),
        'PORT': os.getenv('DB_PORT', '5432'),
    }
}
```

### FastAPI Config
```python
# src/core/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    app_name: str = "My API"
    debug: bool = False
    database_url: str
    secret_key: str
    access_token_expire_minutes: int = 30

    class Config:
        env_file = ".env"

settings = Settings()
```

## Best Practices

### Do
- Use service layer for business logic
- Use repository pattern for data access
- Use type hints everywhere
- Use Pydantic for data validation (FastAPI)
- Use Django REST Framework serializers (Django)
- Use environment variables for configuration
- Use virtual environments
- Write unit tests for services
- Write integration tests for views/routers
- Handle errors explicitly

### Don't
- Don't put business logic in views/controllers
- Don't query database directly from views
- Don't hardcode configuration values
- Don't expose sensitive data
- Don't skip error handling
- Don't use bare except clauses
- Don't mix synchronous and async code unnecessarily
- Don't forget to use database transactions for multi-step operations
- Don't commit .env files
