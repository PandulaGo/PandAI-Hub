# Django Conventions

## Project Structure

### Standard Django Project
```
project-root/
├── apps/
│   ├── users/
│   │   ├── migrations/
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── services.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   └── tests/
│   │       ├── __init__.py
│   │       ├── test_models.py
│   │       ├── test_services.py
│   │       └── test_views.py
│   └── orders/
├── core/
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   ├── __init__.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── manage.py
├── requirements.txt
└── .env
```

## Models

### Model Definition
```python
# apps/users/models.py
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    """Custom user model with email as username"""
    email = models.EmailField(unique=True)
    phone = models.CharField(max_length=20, blank=True)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        db_table = 'users'
        ordering = ['-created_at']
        verbose_name = 'user'
        verbose_name_plural = 'users'

    def __str__(self):
        return self.email

    @property
    def full_name(self):
        return f"{self.first_name} {self.last_name}".strip()
```

### Model with Relationships
```python
# apps/orders/models.py
from django.db import models
from apps.users.models import User

class Order(models.Model):
    class Status(models.TextChoices):
        PENDING = 'PENDING', 'Pending'
        PROCESSING = 'PROCESSING', 'Processing'
        COMPLETED = 'COMPLETED', 'Completed'
        CANCELLED = 'CANCELLED', 'Cancelled'

    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='orders')
    total = models.DecimalField(max_digits=10, decimal_places=2)
    status = models.CharField(
        max_length=20,
        choices=Status.choices,
        default=Status.PENDING
    )
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        db_table = 'orders'
        ordering = ['-created_at']

    def __str__(self):
        return f"Order #{self.id} - {self.user.email}"
```

## Services

### Service Layer Pattern
```python
# apps/users/services.py
from django.core.exceptions import ValidationError
from django.contrib.auth import get_user_model
from typing import Optional

User = get_user_model()

class UserService:
    @staticmethod
    def get_user_by_id(user_id: int) -> User:
        """Get user by ID or raise ValidationError"""
        try:
            return User.objects.get(id=user_id)
        except User.DoesNotExist:
            raise ValidationError(f"User with id {user_id} not found")

    @staticmethod
    def get_user_by_email(email: str) -> Optional[User]:
        """Get user by email, return None if not found"""
        try:
            return User.objects.get(email=email)
        except User.DoesNotExist:
            return None

    @staticmethod
    def create_user(email: str, password: str, **kwargs) -> User:
        """Create a new user"""
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
        """Update user fields"""
        user = UserService.get_user_by_id(user_id)
        
        # Check email uniqueness if email is being updated
        if 'email' in kwargs and kwargs['email'] != user.email:
            if User.objects.filter(email=kwargs['email']).exists():
                raise ValidationError("Email already exists")
        
        for key, value in kwargs.items():
            setattr(user, key, value)
        
        user.save()
        return user

    @staticmethod
    def delete_user(user_id: int) -> None:
        """Delete user"""
        user = UserService.get_user_by_id(user_id)
        user.delete()
```

## Views (Django REST Framework)

### ViewSet
```python
# apps/users/views.py
from rest_framework import viewsets, status
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated, IsAdminUser
from django.core.exceptions import ValidationError
from .models import User
from .serializers import UserSerializer, CreateUserSerializer
from .services import UserService

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [IsAuthenticated]

    def get_serializer_class(self):
        if self.action == 'create':
            return CreateUserSerializer
        return UserSerializer

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

    def update(self, request, *args, **kwargs):
        try:
            user = UserService.update_user(kwargs['pk'], **request.data)
            serializer = self.get_serializer(user)
            return Response(
                {'success': True, 'data': serializer.data}
            )
        except ValidationError as e:
            return Response(
                {'success': False, 'error': str(e)},
                status=status.HTTP_400_BAD_REQUEST
            )

    def destroy(self, request, *args, **kwargs):
        try:
            UserService.delete_user(kwargs['pk'])
            return Response(status=status.HTTP_204_NO_CONTENT)
        except ValidationError as e:
            return Response(
                {'success': False, 'error': str(e)},
                status=status.HTTP_404_NOT_FOUND
            )
```

### APIView (Alternative)
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

class UserListView(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request):
        users = User.objects.filter(is_active=True)
        serializer = UserSerializer(users, many=True)
        return Response({
            'success': True,
            'data': serializer.data
        })

    def post(self, request):
        serializer = CreateUserSerializer(data=request.data)
        if serializer.is_valid():
            user = UserService.create_user(**serializer.validated_data)
            return Response(
                {'success': True, 'data': UserSerializer(user).data},
                status=status.HTTP_201_CREATED
            )
        return Response(
            {'success': False, 'error': serializer.errors},
            status=status.HTTP_400_BAD_REQUEST
        )
```

## Serializers

### ModelSerializer
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

    def validate_email(self, value):
        if User.objects.filter(email=value).exists():
            raise serializers.ValidationError("Email already exists")
        return value

    def create(self, validated_data):
        return User.objects.create_user(**validated_data)
```

## URLs

### URL Configuration
```python
# apps/users/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import UserViewSet

router = DefaultRouter()
router.register(r'', UserViewSet, basename='user')

urlpatterns = [
    path('', include(router.urls)),
]

# core/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/users/', include('apps.users.urls')),
    path('api/v1/orders/', include('apps.orders.urls')),
]
```

## Admin

### Admin Registration
```python
# apps/users/admin.py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from .models import User

@admin.register(User)
class UserAdmin(BaseUserAdmin):
    list_display = ['email', 'first_name', 'last_name', 'is_active', 'created_at']
    list_filter = ['is_active', 'is_staff', 'created_at']
    search_fields = ['email', 'first_name', 'last_name']
    ordering = ['-created_at']
```

## Testing

### Model Tests
```python
# apps/users/tests/test_models.py
from django.test import TestCase
from apps.users.models import User

class UserModelTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            email='test@example.com',
            password='testpass123',
            first_name='Test',
            last_name='User'
        )

    def test_user_creation(self):
        self.assertEqual(self.user.email, 'test@example.com')
        self.assertTrue(self.user.is_active)

    def test_full_name(self):
        self.assertEqual(self.user.full_name, 'Test User')

    def test_str_representation(self):
        self.assertEqual(str(self.user), 'test@example.com')
```

### Service Tests
```python
# apps/users/tests/test_services.py
from django.test import TestCase
from django.core.exceptions import ValidationError
from apps.users.services import UserService
from apps.users.models import User

class UserServiceTest(TestCase):
    def test_create_user(self):
        user = UserService.create_user(
            email='test@example.com',
            password='testpass123'
        )
        self.assertEqual(user.email, 'test@example.com')
        self.assertTrue(user.check_password('testpass123'))

    def test_create_user_duplicate_email(self):
        UserService.create_user(email='test@example.com', password='testpass123')
        with self.assertRaises(ValidationError):
            UserService.create_user(email='test@example.com', password='testpass456')
```

## Best Practices

### Do
- Use service layer for business logic
- Use Django REST Framework for APIs
- Use custom user model
- Use environment variables for configuration
- Use Django's built-in testing framework
- Use migrations for database changes
- Use Django's ORM efficiently
- Use signals sparingly
- Use class-based views for standard CRUD
- Use function-based views for custom logic

### Don't
- Don't put business logic in views
- Don't query database directly from views
- Don't use raw SQL unless necessary
- Don't hardcode configuration values
- Don't skip migrations
- Don't use Django's default user model (create custom)
- Don't expose sensitive data in serializers
- Don't forget to validate input data
- Don't use N+1 queries (use select_related/prefetch_related)
