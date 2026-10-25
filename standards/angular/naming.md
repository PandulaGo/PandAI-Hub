# Angular Naming Conventions

## Files and Folders

### Component Files
- **kebab-case**: `user-list.component.ts`, `user-detail.component.html`
- Pattern: `[name].[type].ts`
  ```
  user-list/
  ├── user-list.component.ts
  ├── user-list.component.html
  ├── user-list.component.scss
  └── user-list.component.spec.ts
  ```

### Service Files
- **kebab-case**: `user.service.ts`, `auth.service.ts`
- Pattern: `[name].service.ts`

### Module Files
- **kebab-case**: `users.module.ts`, `app.module.ts`
- Pattern: `[name].module.ts`

### Folder Names
- **kebab-case**: `user-list/`, `auth/`, `shared/`
- Feature folders: `users/`, `orders/`, `dashboard/`

## Components

### Class Names
- **PascalCase**: `UserListComponent`, `UserDetailComponent`
- Pattern: `[Name][Type]Component`
  ```typescript
  // ✅ Good
  export class UserListComponent { }
  export class UserProfileComponent { }
  
  // ❌ Bad
  export class userListComponent { } // lowercase
  export class UserList { } // missing "Component"
  ```

### Selector Names
- **kebab-case** with prefix: `app-user-list`, `app-user-detail`
- Always use a prefix (e.g., `app-`, `my-app-`)
  ```typescript
  // ✅ Good
  @Component({
    selector: 'app-user-list',
    // ...
  })
  
  // ❌ Bad
  @Component({
    selector: 'userList', // camelCase
    // ...
  })
  ```

## Services

### Class Names
- **PascalCase**: `UserService`, `AuthService`
- Pattern: `[Name]Service`
  ```typescript
  // ✅ Good
  @Injectable()
  export class UserService { }
  
  // ❌ Bad
  @Injectable()
  export class userService { }
  ```

## Directives

### Class Names
- **PascalCase**: `HighlightDirective`, `AutoFocusDirective`
- Pattern: `[Name]Directive`

### Selector Names
- **kebab-case** with prefix: `appHighlight`, `appAutoFocus`
  ```typescript
  @Directive({
    selector: '[appHighlight]'
  })
  export class HighlightDirective { }
  ```

## Pipes

### Class Names
- **PascalCase**: `TruncatePipe`, `DateFormatPipe`
- Pattern: `[Name]Pipe`

### Pipe Names
- **camelCase**: `truncate`, `dateFormat`
  ```typescript
  @Pipe({
    name: 'truncate'
  })
  export class TruncatePipe { }
  ```

## Interfaces and Models

### Interface Names
- **PascalCase**: `User`, `Order`, `ApiResponse`
- Use singular names
  ```typescript
  // ✅ Good
  export interface User {
    id: number;
    name: string;
  }
  
  // ❌ Bad
  export interface UserData { } // redundant "Data"
  export interface user { } // lowercase
  ```

### DTO Names
- Pattern: `[Action][Entity]Dto`
  ```typescript
  // ✅ Good
  export interface CreateUserDto { }
  export interface UpdateUserDto { }
  export interface UserResponseDto { }
  ```

## Variables and Properties

### Component Properties
- **camelCase**: `users`, `selectedUser`, `isLoading`
- Use descriptive names
  ```typescript
  // ✅ Good
  export class UserListComponent {
    users: User[] = [];
    selectedUser: User | null = null;
    isLoading = false;
  }
  ```

### Boolean Properties
- Prefix with `is`, `has`, `should`, `can`
  ```typescript
  // ✅ Good
  isActive = true;
  hasPermission = false;
  shouldShowModal = true;
  ```

## Methods

### Method Names
- **camelCase**: `loadUsers`, `selectUser`, `handleSubmit`
- Use verb-noun pattern
  ```typescript
  // ✅ Good
  loadUsers(): void { }
  selectUser(user: User): void { }
  handleSubmit(): void { }
  
  // ❌ Bad
  Users(): void { } // no verb
  load_users(): void { } // snake_case
  ```

### Event Handler Methods
- Prefix with `on` or `handle`
  ```typescript
  // ✅ Good
  onSubmit(): void { }
  onUserSelect(user: User): void { }
  handleSave(): void { }
  
  // ❌ Bad
  submit(): void { } // unclear if it's a handler
  ```

### Lifecycle Methods
- Use Angular lifecycle hook names exactly
  ```typescript
  // ✅ Good
  ngOnInit(): void { }
  ngOnDestroy(): void { }
  ngOnChanges(changes: SimpleChanges): void { }
  ```

## Constants

### Component Constants
- **camelCase** for component-level constants
  ```typescript
  // ✅ Good
  const pageSize = 20;
  const defaultSortOrder = 'asc';
  ```

### Global Constants
- **SCREAMING_SNAKE_CASE**
  ```typescript
  // ✅ Good
  export const API_VERSION = 'v1';
  export const MAX_RETRY_COUNT = 3;
  ```

## Templates

### Template Variables
- **camelCase**: `#userForm`, `#userNameInput`
  ```html
  <!-- ✅ Good -->
  <form #userForm="ngForm">
    <input #userNameInput type="text">
  </form>
  ```

### Template Reference Variables
- **camelCase** with descriptive name
  ```html
  <!-- ✅ Good -->
  <div *ngIf="user as userData">
    {{ userData.name }}
  </div>
  ```

## CSS Classes

### Class Names
- **kebab-case** or **BEM**: `user-list`, `user-list__item`, `user-list--active`
  ```scss
  // ✅ Good (BEM)
  .user-list { }
  .user-list__item { }
  .user-list__item--selected { }
  
  // ✅ Good (kebab-case)
  .user-list { }
  .user-list-item { }
  ```

## Best Practices

### Do
- Always use a prefix for component/directive selectors (e.g., `app-`)
- Use consistent naming patterns across the project
- Use descriptive, meaningful names
- Follow Angular's style guide
- Use singular names for interfaces
- Use plural names for arrays/collections
- Be consistent within the project

### Don't
- Don't use abbreviations (unless widely understood)
- Don't use generic names like `data`, `info`, `temp`
- Don't mix naming conventions
- Don't use prefixes that conflict with HTML elements
- Don't use names that don't describe the purpose
- Don't use uppercase for CSS classes (except BEM modifiers)
