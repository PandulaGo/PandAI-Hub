# React Conventions

## Functional Components Only

### Basic Component
```tsx
import { useState, useEffect } from 'react';

export const UserList = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    fetchUsers();
  }, []);

  const fetchUsers = async () => {
    try {
      const data = await userService.getUsers();
      setUsers(data);
    } catch (error) {
      console.error('Failed to fetch users:', error);
    } finally {
      setIsLoading(false);
    }
  };

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
};
```

## Component Structure

### Recommended Order
```tsx
export const UserProfile = () => {
  // 1. State hooks
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  // 2. Custom hooks
  const { isAuthenticated } = useAuth();

  // 3. Refs
  const inputRef = useRef<HTMLInputElement>(null);

  // 4. Effects
  useEffect(() => {
    loadUser();
  }, []);

  // 5. Computed values
  const fullName = user ? `${user.firstName} ${user.lastName}` : '';

  // 6. Handlers
  const handleEdit = () => {
    // ...
  };

  const handleDelete = () => {
    // ...
  };

  // 7. Render
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} />;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h1>{fullName}</h1>
      <button onClick={handleEdit}>Edit</button>
      <button onClick={handleDelete}>Delete</button>
    </div>
  );
};
```

## Hooks Rules

### Only Call Hooks at Top Level
```tsx
// ✅ Good
export const UserForm = () => {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  useEffect(() => {
    // ...
  }, []);

  return <form>...</form>;
};

// ❌ Bad - hooks inside conditions
export const UserForm = () => {
  if (condition) {
    const [name, setName] = useState(''); // Wrong!
  }
};
```

### Only Call Hooks from React Functions
```tsx
// ✅ Good
export const UserList = () => {
  const { users } = useUsers();
  return <div>...</div>;
};

// ❌ Bad - hook in regular function
function regularFunction() {
  const [state, setState] = useState(); // Wrong!
}
```

## Performance Optimization

### useMemo for Expensive Calculations
```tsx
export const UserList = ({ users }: { users: User[] }) => {
  const sortedUsers = useMemo(() => {
    return users.sort((a, b) => a.name.localeCompare(b.name));
  }, [users]);

  return (
    <div>
      {sortedUsers.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
};
```

### useCallback for Event Handlers
```tsx
export const UserList = () => {
  const [users, setUsers] = useState<User[]>([]);

  const handleDelete = useCallback((id: number) => {
    setUsers(prev => prev.filter(user => user.id !== id));
  }, []);

  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} onDelete={handleDelete} />
      ))}
    </div>
  );
};
```

### React.memo for Expensive Components
```tsx
export const UserCard = memo(({ user }: { user: User }) => {
  return (
    <div>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
});
```

## Error Handling

### Error Boundaries
```tsx
import { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <div>Something went wrong</div>;
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary fallback={<ErrorMessage />}>
  <App />
</ErrorBoundary>
```

## Form Handling

### Controlled Components
```tsx
export const LoginForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);

    try {
      await authService.login({ email, password });
    } catch (error) {
      console.error('Login failed:', error);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
        required
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
        required
      />
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};
```

## List Rendering

### Always Use Unique Keys
```tsx
// ✅ Good
{users.map(user => (
  <UserCard key={user.id} user={user} />
))}

// ❌ Bad - using index
{users.map((user, index) => (
  <UserCard key={index} user={user} />
))}
```

## Conditional Rendering

### Inline If with && Operator
```tsx
export const UserList = ({ users }: { users: User[] }) => {
  return (
    <div>
      {users.length > 0 && (
        <h2>Found {users.length} users</h2>
      )}
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
};
```

### Inline If-Else with Ternary
```tsx
export const UserStatus = ({ isActive }: { isActive: boolean }) => {
  return (
    <div>
      {isActive ? (
        <span className="active">Active</span>
      ) : (
        <span className="inactive">Inactive</span>
      )}
    </div>
  );
};
```

## Best Practices

### Do
- Use TypeScript for all components
- Use functional components with hooks
- Use descriptive names for components and hooks
- Extract reusable logic into custom hooks
- Use proper loading and error states
- Handle errors with error boundaries
- Use React Query or SWR for server state
- Use context for simple global state
- Memoize expensive calculations
- Clean up effects (return cleanup function)
- Use environment variables for configuration
- Write unit tests with React Testing Library

### Don't
- Don't use class components
- Don't mutate state directly
- Don't use index as key in lists
- Don't put business logic in components
- Don't forget to handle loading states
- Don't skip error handling
- Don't use inline styles for complex layouts
- Don't forget to clean up effects
- Don't use `any` type
- Don't forget to handle edge cases (empty lists, null data)
