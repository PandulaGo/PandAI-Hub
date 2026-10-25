# React Architecture

## Project Structure

### Feature-Based Layout
```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   ├── RegisterForm.tsx
│   │   │   └── ProtectedRoute.tsx
│   │   ├── hooks/
│   │   │   ├── useAuth.ts
│   │   │   └── useLogin.ts
│   │   ├── services/
│   │   │   └── auth-service.ts
│   │   ├── types/
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── users/
│   │   ├── components/
│   │   │   ├── UserList.tsx
│   │   │   ├── UserCard.tsx
│   │   │   └── UserProfile.tsx
│   │   ├── hooks/
│   │   │   ├── useUsers.ts
│   │   │   └── useUser.ts
│   │   ├── services/
│   │   │   └── user-service.ts
│   │   ├── types/
│   │   │   └── index.ts
│   │   └── index.ts
│   └── dashboard/
├── shared/
│   ├── components/
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Modal.tsx
│   │   └── LoadingSpinner.tsx
│   ├── hooks/
│   │   ├── useDebounce.ts
│   │   └── useLocalStorage.ts
│   ├── utils/
│   │   ├── api.ts
│   │   └── format.ts
│   └── types/
│       └── index.ts
├── layouts/
│   ├── MainLayout.tsx
│   └── AuthLayout.tsx
├── pages/
│   ├── HomePage.tsx
│   ├── UsersPage.tsx
│   └── NotFoundPage.tsx
├── App.tsx
└── main.tsx
```

## Component Patterns

### Functional Components Only
```tsx
// features/users/components/UserList.tsx
import { useState, useEffect } from 'react';
import { User } from '../types';
import { userService } from '../services/user-service';
import { UserCard } from './UserCard';
import { LoadingSpinner } from '@/shared/components/LoadingSpinner';

export const UserList = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const loadUsers = async () => {
      try {
        setIsLoading(true);
        const data = await userService.getUsers();
        setUsers(data);
      } catch (err) {
        setError('Failed to load users');
      } finally {
        setIsLoading(false);
      }
    };

    loadUsers();
  }, []);

  if (isLoading) return <LoadingSpinner />;
  if (error) return <div className="error">{error}</div>;

  return (
    <div className="user-list">
      <h2>Users</h2>
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
};
```

### Component with Props
```tsx
// features/users/components/UserCard.tsx
interface UserCardProps {
  user: User;
  onEdit?: (id: number) => void;
  onDelete?: (id: number) => void;
}

export const UserCard = ({ user, onEdit, onDelete }: UserCardProps) => {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <div className="actions">
        {onEdit && <button onClick={() => onEdit(user.id)}>Edit</button>}
        {onDelete && <button onClick={() => onDelete(user.id)}>Delete</button>}
      </div>
    </div>
  );
};
```

## Custom Hooks

### Data Fetching Hook
```tsx
// features/users/hooks/useUsers.ts
import { useState, useEffect } from 'react';
import { User } from '../types';
import { userService } from '../services/user-service';

interface UseUsersResult {
  users: User[];
  isLoading: boolean;
  error: string | null;
  refetch: () => void;
}

export const useUsers = (): UseUsersResult => {
  const [users, setUsers] = useState<User[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchUsers = async () => {
    try {
      setIsLoading(true);
      const data = await userService.getUsers();
      setUsers(data);
      setError(null);
    } catch (err) {
      setError('Failed to load users');
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  return { users, isLoading, error, refetch: fetchUsers };
};
```

### Usage in Component
```tsx
export const UsersPage = () => {
  const { users, isLoading, error, refetch } = useUsers();

  if (isLoading) return <LoadingSpinner />;
  if (error) return <div className="error">{error}</div>;

  return (
    <div>
      <button onClick={refetch}>Refresh</button>
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
};
```

## State Management

### React Context (Simple Global State)
```tsx
// features/auth/contexts/AuthContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';
import { User } from '../types';

interface AuthContextType {
  user: User | null;
  isAuthenticated: boolean;
  login: (user: User, token: string) => void;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider = ({ children }: { children: ReactNode }) => {
  const [user, setUser] = useState<User | null>(null);

  const login = (user: User, token: string) => {
    setUser(user);
    localStorage.setItem('token', token);
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('token');
  };

  return (
    <AuthContext.Provider value={{ user, isAuthenticated: !!user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};
```

### React Query (Server State)
```tsx
// features/users/hooks/useUsers.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { userService } from '../services/user-service';
import { CreateUserDto } from '../types';

export const useUsers = () => {
  return useQuery({
    queryKey: ['users'],
    queryFn: () => userService.getUsers(),
  });
};

export const useCreateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (dto: CreateUserDto) => userService.createUser(dto),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
};
```

## API Service Layer

### Axios-based Service
```tsx
// shared/utils/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000/api/v1',
  headers: {
    'Content-Type': 'application/json',
  },
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

### Feature Service
```tsx
// features/users/services/user-service.ts
import api from '@/shared/utils/api';
import { User, CreateUserDto, UpdateUserDto } from '../types';

export const userService = {
  async getUsers(): Promise<User[]> {
    const response = await api.get('/users');
    return response.data.data;
  },

  async getUserById(id: number): Promise<User> {
    const response = await api.get(`/users/${id}`);
    return response.data.data;
  },

  async createUser(dto: CreateUserDto): Promise<User> {
    const response = await api.post('/users', dto);
    return response.data.data;
  },

  async updateUser(id: number, dto: UpdateUserDto): Promise<User> {
    const response = await api.put(`/users/${id}`, dto);
    return response.data.data;
  },

  async deleteUser(id: number): Promise<void> {
    await api.delete(`/users/${id}`);
  },
};
```

## Routing

### React Router Setup
```tsx
// App.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { AuthProvider } from './features/auth/contexts/AuthContext';
import { MainLayout } from './layouts/MainLayout';
import { HomePage } from './pages/HomePage';
import { UsersPage } from './pages/UsersPage';
import { NotFoundPage } from './pages/NotFoundPage';
import { ProtectedRoute } from './features/auth/components/ProtectedRoute';

export const App = () => {
  return (
    <BrowserRouter>
      <AuthProvider>
        <Routes>
          <Route element={<MainLayout />}>
            <Route path="/" element={<HomePage />} />
            <Route
              path="/users"
              element={
                <ProtectedRoute>
                  <UsersPage />
                </ProtectedRoute>
              }
            />
            <Route path="*" element={<NotFoundPage />} />
          </Route>
        </Routes>
      </AuthProvider>
    </BrowserRouter>
  );
};
```

## Best Practices

### Do
- Use TypeScript for all components
- Use functional components with hooks
- Use feature-based folder structure
- Use custom hooks for reusable logic
- Use React Context for simple global state
- Use React Query or SWR for server state
- Use React Router for navigation
- Use environment variables for configuration
- Implement error boundaries
- Use proper loading and error states
- Memoize expensive computations with useMemo
- Memoize callbacks with useCallback
- Write unit tests with React Testing Library

### Don't
- Don't use class components
- Don't mutate state directly
- Don't use index as key in lists (use unique IDs)
- Don't put business logic in components (use services/hooks)
- Don't forget to clean up effects (return cleanup function)
- Don't use inline styles for complex layouts
- Don't skip error handling
- Don't expose API keys in client code
- Don't use `any` type (use proper TypeScript types)
- Don't forget to handle loading states
