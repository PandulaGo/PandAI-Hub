# Node.js Naming Conventions

## Files and Folders

### File Names
- **kebab-case**: `user-controller.ts`, `order-service.ts`, `auth-middleware.ts`
- **Test files**: `user-controller.test.ts`, `order-service.spec.ts`
- **Index files**: `index.ts` (for module exports)
- **Configuration**: `database.config.ts`, `env.config.ts`

### Folder Names
- **kebab-case**: `controllers/`, `services/`, `repositories/`
- **Feature folders**: `users/`, `orders/`, `products/`
- **Utility folders**: `utils/`, `helpers/`, `middleware/`

## Variables

### Naming Style
- **camelCase**: `firstName`, `lastName`, `userCount`
- **Booleans**: Prefix with `is`, `has`, `should`, `can`
  ```typescript
  const isActive = true;
  const hasPermission = false;
  const shouldRetry = true;
  const canEdit = false;
  ```

### Constants
- **SCREAMING_SNAKE_CASE**: `MAX_RETRIES`, `DEFAULT_PORT`, `API_VERSION`
  ```typescript
  const MAX_RETRIES = 3;
  const DEFAULT_PORT = 3000;
  const API_VERSION = 'v1';
  ```

### Avoid
```typescript
// ❌ Don't use single letters (except in loops)
const n = 10; // Bad
const count = 10; // Good

// ❌ Don't use abbreviations
const usr = getUser(); // Bad
const user = getUser(); // Good

// ❌ Don't use generic names
const data = fetchData(); // Bad
const users = fetchUsers(); // Good
```

## Functions

### Naming Style
- **camelCase**: `getUserById`, `calculateTotal`, `validateInput`
- **verb-noun pattern**: Start with action verb
  ```typescript
  // ✅ Good
  const getUser = (id: number) => { };
  const createUser = (data: CreateUserDto) => { };
  const validateEmail = (email: string) => { };
  const calculateTotal = (items: Item[]) => { };
  ```

### Async Functions
- No "Async" suffix needed (TypeScript handles this)
  ```typescript
  // ✅ Good
  const getUser = async (id: number) => { };
  
  // ❌ Unnecessary
  const getUserAsync = async (id: number) => { };
  ```

### Callback Functions
- Use descriptive names for callbacks
  ```typescript
  // ✅ Good
  users.forEach((user) => { });
  users.filter((user) => user.isActive);
  ```

## Classes

### Naming Style
- **PascalCase**: `UserService`, `OrderRepository`, `UserController`
- Use nouns for class names
  ```typescript
  // ✅ Good
  class UserService { }
  class OrderRepository { }
  class EmailValidator { }
  
  // ❌ Bad
  class userService { } // lowercase
  class doSomething { } // verb instead of noun
  ```

## Interfaces and Types

### Naming Style
- **PascalCase**: `User`, `CreateUserDto`, `ApiResponse`
- Descriptive names
  ```typescript
  // ✅ Good
  interface User {
    id: number;
    name: string;
    email: string;
  }
  
  type CreateUserDto = {
    name: string;
    email: string;
    password: string;
  };
  
  interface ApiResponse<T> {
    success: boolean;
    data: T;
  }
  ```

### DTO Naming
- **Create[Entity]Dto**: For creation requests
- **Update[Entity]Dto**: For update requests
- **[Entity]ResponseDto**: For responses (optional, can just use entity name)
  ```typescript
  interface CreateUserDto {
    name: string;
    email: string;
    password: string;
  }
  
  interface UpdateUserDto {
    name?: string;
    email?: string;
  }
  
  interface UserResponseDto {
    id: number;
    name: string;
    email: string;
    createdAt: Date;
  }
  ```

## Database

### Table Names
- **snake_case, plural**: `users`, `orders`, `order_items`
  ```typescript
  // ✅ Good
  const tableName = 'users';
  const tableName = 'order_items';
  
  // ❌ Bad
  const tableName = 'Users'; // PascalCase
  const tableName = 'user'; // singular
  ```

### Column Names
- **snake_case**: `first_name`, `last_name`, `created_at`
  ```typescript
  // ✅ Good
  const column = 'first_name';
  const column = 'created_at';
  
  // ❌ Bad
  const column = 'firstName'; // camelCase
  const column = 'FirstName'; // PascalCase
  ```

### Primary Keys
- Always `id` (singular)
  ```typescript
  // ✅ Good
  id: number;
  
  // ❌ Bad
  userId: number; // Don't prefix with entity name
  ```

### Foreign Keys
- **[entity]_id**: `user_id`, `order_id`
  ```typescript
  // ✅ Good
  user_id: number;
  order_id: number;
  ```

## API Endpoints

### URL Patterns
- **kebab-case**: `/api/v1/order-items`, `/api/v1/user-profiles`
- **Plural nouns**: `/api/v1/users`, `/api/v1/orders`
  ```typescript
  // ✅ Good
  app.get('/api/v1/users', getUsers);
  app.get('/api/v1/order-items', getOrderItems);
  
  // ❌ Bad
  app.get('/api/v1/getUsers', getUsers); // verb in URL
  app.get('/api/v1/user', getUsers); // singular
  ```

### Query Parameters
- **camelCase**: `?page=1&pageSize=20&sortBy=createdAt`
  ```typescript
  // ✅ Good
  const { page, pageSize, sortBy } = req.query;
  
  // ❌ Bad
  const { page_size } = req.query; // snake_case
  ```

## Environment Variables

### Naming Style
- **SCREAMING_SNAKE_CASE**: `DATABASE_URL`, `API_KEY`, `PORT`
  ```typescript
  // ✅ Good
  const DATABASE_URL = process.env.DATABASE_URL;
  const PORT = process.env.PORT || 3000;
  const JWT_SECRET = process.env.JWT_SECRET;
  
  // ❌ Bad
  const databaseUrl = process.env.databaseUrl; // camelCase
  ```

## Enums

### Naming Style
- **PascalCase** for enum name
- **PascalCase** or **SCREAMING_SNAKE_CASE** for values
  ```typescript
  // ✅ Good (PascalCase values)
  enum UserStatus {
    Active,
    Inactive,
    Pending
  }
  
  // ✅ Good (SCREAMING_SNAKE_CASE values)
  enum OrderStatus {
    PENDING = 'PENDING',
    PROCESSING = 'PROCESSING',
    COMPLETED = 'COMPLETED'
  }
  ```

## Imports

### Naming Style
- Match the exported name
  ```typescript
  // ✅ Good
  import { UserService } from './user-service';
  import { getUser } from './user-controller';
  import type { User } from './types';
  
  // ❌ Bad
  import { userService } from './user-service'; // lowercase
  import { get_user } from './user-controller'; // snake_case
  ```

## Best Practices

### Do
- Use descriptive, meaningful names
- Be consistent within the project
- Use camelCase for variables and functions
- Use PascalCase for classes, interfaces, and types
- Use kebab-case for file and folder names
- Use snake_case for database tables and columns
- Use SCREAMING_SNAKE_CASE for constants and environment variables

### Don't
- Don't use single-letter variable names (except in loops)
- Don't use abbreviations (unless widely understood: `id`, `url`, `api`)
- Don't use generic names like `data`, `info`, `temp`
- Don't mix naming conventions
- Don't use reserved words as variable names
- Don't use names that are too long or too short
- Don't use names that don't describe the purpose
