# FastAPI Conventions

## Project Structure

### Standard FastAPI Project
```
project-root/
├── src/
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── user_router.py
│   │   └── order_router.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   └── order_service.py
│   ├── repositories/
│   │   ├── __init__.py
│   │   ├── user_repository.py
│   │   └── order_repository.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── order.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── order.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   ├── security.py
│   │   └── database.py
│   ├── middleware/
│   │   ├── __init__.py
│   │   └── auth.py
│   └── main.py
├── tests/
│   ├── __init__.py
│   ├── test_users.py
│   └── test_orders.py
├── requirements.txt
├── .env
└── alembic/
    ├── versions/
    └── env.py
```

## Models (SQLAlchemy)

### Base Model
```python
# src/core/database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from src.core.config import settings

engine = create_engine(settings.database_url)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### User Model
```python
# src/models/user.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime
from sqlalchemy.sql import func
from sqlalchemy.orm import relationship
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

    # Relationships
    orders = relationship("Order", back_populates="user", cascade="all, delete-orphan")

    def __repr__(self):
        return f"<User(id={self.id}, email={self.email})>"
```

## Schemas (Pydantic)

### Request/Response Schemas
```python
# src/schemas/user.py
from pydantic import BaseModel, EmailStr, Field, ConfigDict
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
    password: Optional[str] = Field(None, min_length=8)

class UserResponse(UserBase):
    model_config = ConfigDict(from_attributes=True)
    
    id: int
    is_active: bool
    created_at: datetime

class UserLogin(BaseModel):
    email: EmailStr
    password: str
```

## Repository Pattern

### Repository Implementation
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
        
        update_data = user_data.model_dump(exclude_unset=True)
        for key, value in update_data.items():
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

## Service Layer

### Service Implementation
```python
# src/services/user_service.py
from sqlalchemy.orm import Session
from src.repositories.user_repository import UserRepository
from src.schemas.user import UserCreate, UserUpdate
from src.core.security import hash_password, verify_password
from typing import Optional, List
from src.models.user import User

class UserService:
    def __init__(self, db: Session):
        self.repository = UserRepository(db)

    def get_user(self, user_id: int) -> User:
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
        if user_data.password:
            user_data.password = hash_password(user_data.password)
        
        user = self.repository.update(user_id, user_data)
        if not user:
            raise ValueError(f"User {user_id} not found")
        return user

    def delete_user(self, user_id: int) -> None:
        success = self.repository.delete(user_id)
        if not success:
            raise ValueError(f"User {user_id} not found")
```

## Routers

### Router Implementation
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

@router.put("/{user_id}", response_model=UserResponse)
def update_user(
    user_id: int,
    user_data: UserUpdate,
    db: Session = Depends(get_db)
):
    service = UserService(db)
    try:
        return service.update_user(user_id, user_data)
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))

@router.delete("/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_user(user_id: int, db: Session = Depends(get_db)):
    service = UserService(db)
    try:
        service.delete_user(user_id)
    except ValueError as e:
        raise HTTPException(status_code=404, detail=str(e))
```

## Main Application

### FastAPI App Setup
```python
# src/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from src.routers import user_router, order_router
from src.core.config import settings

app = FastAPI(
    title=settings.app_name,
    version="1.0.0",
    docs_url="/api/docs",
    redoc_url="/api/redoc"
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:4200", "http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Routers
app.include_router(user_router.router)
app.include_router(order_router.router)

@app.get("/health")
def health_check():
    return {"status": "ok"}
```

## Authentication

### JWT Authentication
```python
# src/core/security.py
from passlib.context import CryptContext
from jose import JWTError, jwt
from datetime import datetime, timedelta
from src.core.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def create_access_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=settings.access_token_expire_minutes)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, settings.secret_key, algorithm="HS256")

def decode_access_token(token: str) -> dict:
    return jwt.decode(token, settings.secret_key, algorithms=["HS256"])
```

### Auth Middleware
```python
# src/middleware/auth.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.orm import Session
from src.core.database import get_db
from src.core.security import decode_access_token
from src.models.user import User

security = HTTPBearer()

def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: Session = Depends(get_db)
) -> User:
    try:
        payload = decode_access_token(credentials.credentials)
        user_id = payload.get("sub")
        if user_id is None:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Invalid token"
            )
    except Exception:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid token"
        )
    
    user = db.query(User).filter(User.id == int(user_id)).first()
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User not found"
        )
    
    return user
```

## Configuration

### Settings
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

### Environment File
```bash
# .env
DATABASE_URL=postgresql://user:password@localhost/dbname
SECRET_KEY=your-secret-key-here
DEBUG=True
```

## Best Practices

### Do
- Use Pydantic schemas for request/response validation
- Use dependency injection for database sessions
- Use service layer for business logic
- Use repository pattern for data access
- Use async/await for I/O operations
- Use proper error handling with HTTPException
- Use environment variables for configuration
- Use JWT for authentication
- Use CORS middleware
- Write tests with pytest

### Don't
- Don't put business logic in routers
- Don't query database directly from routers
- Don't expose sensitive data in responses
- Don't hardcode configuration values
- Don't skip input validation
- Don't forget to close database sessions
- Don't use synchronous code for I/O operations
- Don't expose stack traces in production
- Don't forget to use HTTPS in production
- Don't commit .env files
