# React Naming Conventions

## Files and Folders

### Component Files
- **PascalCase**: `UserList.tsx`, `UserProfile.tsx`, `LoginForm.tsx`
- One component per file
- File name matches component name
  ```
  features/users/components/
  ├── UserList.tsx
  ├── UserCard.tsx
  └── UserProfile.tsx
  ```

### Hook Files
- **camelCase** with "use" prefix: `useUsers.ts`, `useAuth.ts`
  ```
  features/users/hooks/
  ├── useUsers.ts
  └── useUser.ts
  ```

### Service Files
- **kebab-case**: `user-service.ts`, `auth-service.ts`
  ```
  features/users/services/
  └── user-service.ts
  ```

### Folder Names
- **kebab-case**: `features/`, `shared/`, `components/`, `hooks/`
- Feature folders: **kebab-case**: `users/`, `auth/`, `dashboard/`

## Components

### Component Names
- **PascalCase**: `UserList`, `UserProfile`, `LoginForm`
  ```tsx
  // ✅ Good
  export const UserList = () => { };
  export const UserProfile = () => { };
  
  // ❌ Bad
  export const userList = () => { }; // lowercase
  export const user_list = () => { }; // snake_case
  ```

### Component Props
- **PascalCase** for interface: `UserCardProps`, `LoginFormProps`
  ```tsx
  // ✅ Good
  interface UserCardProps {
    user: User;
    onEdit?: (id: number) => void;
  }
  
  export const UserCard = ({ user, onEdit }: UserCardProps) => { };
  ```

## Hooks

### Custom Hook Names
- **camelCase** with "use" prefix: `useUsers`, `useAuth`, `useDebounce`
  ```tsx
  // ✅ Good
  export const useUsers = () => { };
  export const useAuth = () => { };
  
  // ❌ Bad
  export const UseUsers = () => { }; // PascalCase
  export const users = () => { }; // no "use" prefix
  ```

## Variables and State

### State Variables
- **camelCase**: `users`, `isLoading`, `selectedUser`
  ```tsx
  // ✅ Good
  const [users, setUsers] = useState<User[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [selectedUser, setSelectedUser] = useState<User | null>(null);
  ```

### Boolean State
- Prefix with `is`, `has`, `should`, `can`
  ```tsx
  // ✅ Good
  const [isLoading, setIsLoading] = useState(false);
  const [hasError, setHasError] = useState(false);
  const [isVisible, setIsVisible] = useState(false);
  ```

## Functions

### Event Handlers
- **camelCase** with "handle" prefix: `handleSubmit`, `handleClick`
- Or "on" prefix for props: `onEdit`, `onDelete`
  ```tsx
  // ✅ Good
  const handleSubmit = (e: React.FormEvent) => { };
  const handleClick = () => { };
  
  // Props
  interface Props {
    onEdit: (id: number) => void;
    onDelete: (id: number) => void;
  }
  ```

### Async Functions
- **camelCase**: `fetchUsers`, `createUser`, `loadData`
  ```tsx
  // ✅ Good
  const fetchUsers = async () => { };
  const createUser = async (data: CreateUserDto) => { };
  ```

## Types and Interfaces

### Interface Names
- **PascalCase**: `User`, `Order`, `ApiResponse`
  ```tsx
  // ✅ Good
  interface User {
    id: number;
    name: string;
    email: string;
  }
  ```

### Type Names
- **PascalCase**: `Status`, `UserRole`
  ```tsx
  // ✅ Good
  type Status = 'active' | 'inactive' | 'pending';
  type UserRole = 'admin' | 'user' | 'guest';
  ```

## CSS Classes

### Class Names
- **kebab-case** or **camelCase** (if using CSS modules)
  ```tsx
  // ✅ Good (plain CSS)
  <div className="user-list">
    <div className="user-card">
  
  // ✅ Good (CSS modules)
  <div className={styles.userList}>
    <div className={styles.userCard}>
  ```

## Environment Variables

### Variable Names
- **SCREAMING_SNAKE_CASE** with prefix: `VITE_API_URL`, `REACT_APP_API_URL`
  ```
  VITE_API_URL=http://localhost:3000
  VITE_APP_TITLE=My App
  ```

## Best Practices

### Do
- Use PascalCase for component names
- Use camelCase for variables, functions, and hooks
- Use "use" prefix for custom hooks
- Use "handle" prefix for event handlers in components
- Use "on" prefix for event handler props
- Use descriptive, meaningful names
- Use singular names for single items, plural for collections
- Be consistent within the project

### Don't
- Don't use abbreviations (unless widely understood)
- Don't use generic names like `data`, `info`, `temp`
- Don't use single-letter variable names (except in short callbacks)
- Don't mix naming conventions
- Don't use names that don't describe the purpose
- Don't use `$` prefix (that's jQuery style)
- Don't use `_` prefix (that's Python style)
