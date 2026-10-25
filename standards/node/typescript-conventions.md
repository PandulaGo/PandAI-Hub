# TypeScript Conventions

## Type Annotations

### Function Signatures
- Always annotate function parameters and return types
  ```typescript
  // ✅ Good
  const getUser = (id: number): User => { };
  const createUser = (data: CreateUserDto): Promise<User> => { };
  
  // ❌ Bad
  const getUser = (id) => { }; // Missing types
  const createUser = (data: any) => { }; // Using any
  ```

### Return Types
- Always specify return type for public functions
- Use `void` for functions that don't return anything
- Use `Promise<T>` for async functions
  ```typescript
  // ✅ Good
  function calculateTotal(items: Item[]): number {
    return items.reduce((sum, item) => sum + item.price, 0);
  }
  
  async function fetchUsers(): Promise<User[]> {
    return await db.query('SELECT * FROM users');
  }
  
  function logMessage(message: string): void {
    console.log(message);
  }
  ```

## Type Definitions

### Interfaces vs Types
- Use `interface` for object shapes that can be extended
- Use `type` for unions, intersections, and complex types
  ```typescript
  // ✅ Use interface for objects
  interface User {
    id: number;
    name: string;
    email: string;
  }
  
  interface AdminUser extends User {
    permissions: string[];
  }
  
  // ✅ Use type for unions/intersections
  type Status = 'active' | 'inactive' | 'pending';
  
  type UserWithOrders = User & { orders: Order[] };
  
  type ApiResponse<T> = {
    success: boolean;
    data: T;
    error?: string;
  };
  ```

### Avoid `any`
- Use `unknown` when type is truly unknown
- Use specific types or generics when possible
  ```typescript
  // ❌ Bad
  const data: any = fetchData();
  
  // ✅ Good
  const data: User = await fetchUser();
  const data: unknown = fetchData(); // If truly unknown
  
  // ✅ Better: Use generics
  function fetchData<T>(url: string): Promise<T> {
    // ...
  }
  ```

## Async/Await

### Always Use async/await
- Never use raw promises with `.then()` chains
- Always handle errors with try/catch
  ```typescript
  // ✅ Good
  async function getUser(id: number): Promise<User> {
    try {
      const user = await userRepository.findById(id);
      return user;
    } catch (error) {
      logger.error('Error fetching user', error);
      throw error;
    }
  }
  
  // ❌ Bad
  function getUser(id: number) {
    return userRepository.findById(id)
      .then(user => user)
      .catch(error => { throw error; });
  }
  ```

### Type Promises Explicitly
  ```typescript
  // ✅ Good
  async function fetchUsers(): Promise<User[]> {
    return await db.query('SELECT * FROM users');
  }
  
  // ❌ Unnecessary
  async function fetchUsers(): Promise<any> {
    return await db.query('SELECT * FROM users');
  }
  ```

## Imports and Exports

### ES Modules
- Use `import`/`export` syntax (not CommonJS `require`)
- Group imports: Node modules → third-party → internal
  ```typescript
  // ✅ Good
  import { Request, Response } from 'express';
  import { z } from 'zod';
  import { UserService } from '../services/user-service';
  import type { User } from '../types';
  ```

### Named Exports
- Prefer named exports over default exports
  ```typescript
  // ✅ Good (named export)
  export const userService = {
    getUser: async (id: number) => { },
    createUser: async (data: CreateUserDto) => { }
  };
  
  // ✅ Good (multiple named exports)
  export const getUser = async (id: number) => { };
  export const createUser = async (data: CreateUserDto) => { };
  
  // ❌ Avoid default exports (harder to refactor)
  export default class UserService { }
  ```

### Type-Only Imports
- Use `import type` for type-only imports
  ```typescript
  // ✅ Good
  import type { User, CreateUserDto } from '../types';
  
  // ❌ Unnecessary runtime import
  import { User } from '../types'; // User is only used as type
  ```

## Generics

### Use Descriptive Names
- Use single uppercase letter for simple generics
- Use descriptive names for multiple generics
  ```typescript
  // ✅ Good
  interface Repository<T> {
    findById(id: number): Promise<T | null>;
    findAll(): Promise<T[]>;
  }
  
  interface KeyValuePair<TKey, TValue> {
    key: TKey;
    value: TValue;
  }
  
  // ✅ Good: Constrained generics
  function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
  }
  ```

## Null and Undefined

### Use Optional Chaining
  ```typescript
  // ✅ Good
  const userName = user?.profile?.name;
  const firstOrder = user?.orders?.[0];
  
  // ❌ Bad
  const userName = user && user.profile && user.profile.name;
  ```

### Use Nullish Coalescing
  ```typescript
  // ✅ Good
  const port = process.env.PORT ?? 3000;
  const name = user.name ?? 'Anonymous';
  
  // ❌ Bad (|| treats 0, '', false as falsy)
  const port = process.env.PORT || 3000;
  ```

### Strict Null Checks
- Always enable `strict: true` in tsconfig.json
- Handle null/undefined explicitly
  ```typescript
  // ✅ Good
  async function getUser(id: number): Promise<User> {
    const user = await userRepository.findById(id);
    if (!user) {
      throw new NotFoundError('User not found');
    }
    return user;
  }
  
  // ❌ Bad (non-null assertion)
  async function getUser(id: number): Promise<User> {
    const user = await userRepository.findById(id);
    return user!; // Dangerous
  }
  ```

## Type Guards

### Use Type Guards for Type Narrowing
  ```typescript
  // ✅ Good
  function isUser(obj: any): obj is User {
    return 'id' in obj && 'name' in obj && 'email' in obj;
  }
  
  function processEntity(entity: User | Order) {
    if (isUser(entity)) {
      console.log(entity.name); // TypeScript knows it's User
    } else {
      console.log(entity.total); // TypeScript knows it's Order
    }
  }
  ```

## Enums

### Prefer Union Types Over Enums
  ```typescript
  // ✅ Good (union type)
  type Status = 'active' | 'inactive' | 'pending';
  
  const userStatus: Status = 'active';
  
  // ✅ Good (const object)
  const UserStatus = {
    ACTIVE: 'active',
    INACTIVE: 'inactive',
    PENDING: 'pending'
  } as const;
  
  type UserStatus = typeof UserStatus[keyof typeof UserStatus];
  
  // ❌ Avoid (enum has runtime overhead)
  enum Status {
    Active = 'active',
    Inactive = 'inactive',
    Pending = 'pending'
  }
  ```

## Type Assertions

### Avoid Type Assertions
- Use type guards instead when possible
  ```typescript
  // ❌ Avoid
  const user = data as User;
  
  // ✅ Better: Use type guard
  function isUser(data: unknown): data is User {
    return typeof data === 'object' && data !== null && 'id' in data;
  }
  
  if (isUser(data)) {
    const user = data; // TypeScript knows it's User
  }
  ```

## Utility Types

### Use Built-in Utility Types
  ```typescript
  // ✅ Good
  type UserPartial = Partial<User>; // All properties optional
  type UserRequired = Required<User>; // All properties required
  type UserReadonly = Readonly<User>; // All properties readonly
  type UserPick = Pick<User, 'id' | 'name'>; // Only id and name
  type UserOmit = Omit<User, 'password'>; // Everything except password
  
  // ✅ Good: Record type
  type UserMap = Record<number, User>; // { [key: number]: User }
  ```

## Configuration

### tsconfig.json
  ```json
  {
    "compilerOptions": {
      "target": "ES2020",
      "module": "ES2020",
      "lib": ["ES2020"],
      "outDir": "./dist",
      "rootDir": "./src",
      "strict": true,
      "esModuleInterop": true,
      "skipLibCheck": true,
      "forceConsistentCasingInFileNames": true,
      "resolveJsonModule": true,
      "moduleResolution": "node",
      "declaration": true,
      "declarationMap": true,
      "sourceMap": true,
      "noUnusedLocals": true,
      "noUnusedParameters": true,
      "noImplicitReturns": true,
      "noFallthroughCasesInSwitch": true
    }
  }
  ```

## Best Practices

### Do
- Always enable `strict: true` in tsconfig.json
- Use type annotations for function parameters and return types
- Use `interface` for object shapes, `type` for unions/intersections
- Use `unknown` instead of `any` when type is unknown
- Use optional chaining (`?.`) and nullish coalescing (`??`)
- Use type guards for type narrowing
- Use generics for reusable components
- Use utility types (`Partial`, `Required`, `Pick`, `Omit`)
- Use named exports over default exports
- Use `import type` for type-only imports

### Don't
- Don't use `any` type (use `unknown` or specific types)
- Don't use non-null assertions (`!`) unless absolutely necessary
- Don't use type assertions (`as`) unless necessary
- Don't use enums (use union types or const objects)
- Don't disable TypeScript errors with `@ts-ignore`
- Don't use `Object` or `{}` as types (use `Record<string, unknown>`)
- Don't mix `interface` and `type` for the same purpose
- Don't use `namespace` (use ES modules)
- Don't use `require()` (use `import`)
- Don't use `var` (use `const` or `let`)
