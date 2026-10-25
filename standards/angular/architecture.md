# Angular Architecture

## Project Structure

### Standard Layout (Feature-Based)
```
src/
├── app/
│   ├── core/                      # Singleton services, guards, interceptors
│   │   ├── guards/
│   │   │   └── auth.guard.ts
│   │   ├── interceptors/
│   │   │   ├── auth.interceptor.ts
│   │   │   └── error.interceptor.ts
│   │   ├── services/
│   │   │   └── auth.service.ts
│   │   └── core.module.ts
│   ├── shared/                    # Reusable components, directives, pipes
│   │   ├── components/
│   │   │   ├── header/
│   │   │   │   ├── header.component.ts
│   │   │   │   ├── header.component.html
│   │   │   │   ├── header.component.scss
│   │   │   │   └── header.component.spec.ts
│   │   │   └── loading-spinner/
│   │   ├── directives/
│   │   ├── pipes/
│   │   └── shared.module.ts
│   ├── features/                  # Feature modules (lazy loaded)
│   │   ├── auth/
│   │   │   ├── components/
│   │   │   │   ├── login/
│   │   │   │   └── register/
│   │   │   ├── services/
│   │   │   ├── auth.module.ts
│   │   │   └── auth-routing.module.ts
│   │   ├── users/
│   │   │   ├── components/
│   │   │   ├── services/
│   │   │   ├── models/
│   │   │   ├── users.module.ts
│   │   │   └── users-routing.module.ts
│   │   └── dashboard/
│   ├── layout/                    # Layout components
│   │   ├── main-layout/
│   │   └── admin-layout/
│   ├── models/                    # Global interfaces/types
│   │   ├── user.model.ts
│   │   └── api-response.model.ts
│   ├── app.component.ts
│   ├── app.component.html
│   ├── app.component.scss
│   ├── app.module.ts
│   └── app-routing.module.ts
├── assets/
├── environments/
│   ├── environment.ts
│   └── environment.prod.ts
└── styles.scss
```

## Module Organization

### Feature Module
```typescript
// features/users/users.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ReactiveFormsModule } from '@angular/forms';
import { UsersRoutingModule } from './users-routing.module';
import { SharedModule } from '../../shared/shared.module';

import { UserListComponent } from './components/user-list/user-list.component';
import { UserDetailComponent } from './components/user-detail/user-detail.component';
import { UserFormComponent } from './components/user-form/user-form.component';

@NgModule({
  declarations: [
    UserListComponent,
    UserDetailComponent,
    UserFormComponent
  ],
  imports: [
    CommonModule,
    ReactiveFormsModule,
    UsersRoutingModule,
    SharedModule
  ]
})
export class UsersModule { }
```

### Standalone Components (Angular 14+)
```typescript
// features/users/components/user-list/user-list.component.ts
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { UserService } from '../../services/user.service';
import { User } from '../../models/user.model';

@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './user-list.component.html',
  styleUrls: ['./user-list.component.scss']
})
export class UserListComponent implements OnInit {
  users: User[] = [];
  
  constructor(private userService: UserService) {}
  
  ngOnInit(): void {
    this.loadUsers();
  }
  
  loadUsers(): void {
    this.userService.getUsers().subscribe(users => {
      this.users = users;
    });
  }
}
```

### Routing Module
```typescript
// features/users/users-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { UserListComponent } from './components/user-list/user-list.component';
import { UserDetailComponent } from './components/user-detail/user-detail.component';

const routes: Routes = [
  { path: '', component: UserListComponent },
  { path: ':id', component: UserDetailComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class UsersRoutingModule { }
```

### Lazy Loading
```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./features/users/users.module')
      .then(m => m.UsersModule)
  },
  {
    path: 'dashboard',
    loadChildren: () => import('./features/dashboard/dashboard.module')
      .then(m => m.DashboardModule)
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

## Service Layer

### Service Pattern
```typescript
// features/users/services/user.service.ts
import { Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';
import { environment } from '../../../environments/environment';
import { User, CreateUserDto, UpdateUserDto } from '../models/user.model';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private apiUrl = `${environment.apiUrl}/users`;

  constructor(private http: HttpClient) {}

  getUsers(page = 1, pageSize = 20): Observable<User[]> {
    const params = new HttpParams()
      .set('page', page.toString())
      .set('pageSize', pageSize.toString());
    
    return this.http.get<User[]>(this.apiUrl, { params });
  }

  getUserById(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }

  createUser(user: CreateUserDto): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }

  updateUser(id: number, user: UpdateUserDto): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }

  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

## State Management

### Using Services with BehaviorSubject
```typescript
// features/users/services/user-state.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';
import { User } from '../models/user.model';

@Injectable({
  providedIn: 'root'
})
export class UserStateService {
  private usersSubject = new BehaviorSubject<User[]>([]);
  private selectedUserSubject = new BehaviorSubject<User | null>(null);

  users$ = this.usersSubject.asObservable();
  selectedUser$ = this.selectedUserSubject.asObservable();

  setUsers(users: User[]): void {
    this.usersSubject.next(users);
  }

  selectUser(user: User): void {
    this.selectedUserSubject.next(user);
  }

  clearSelectedUser(): void {
    this.selectedUserSubject.next(null);
  }
}
```

### Using NgRx (For Complex State)
```typescript
// store/users/users.actions.ts
import { createAction, props } from '@ngrx/store';
import { User } from '../../models/user.model';

export const loadUsers = createAction('[Users] Load Users');
export const loadUsersSuccess = createAction(
  '[Users] Load Users Success',
  props<{ users: User[] }>()
);
export const loadUsersFailure = createAction(
  '[Users] Load Users Failure',
  props<{ error: any }>()
);
```

## Interceptors

### HTTP Interceptor
```typescript
// core/interceptors/auth.interceptor.ts
import { Injectable } from '@angular/core';
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor
} from '@angular/common/http';
import { Observable } from 'rxjs';
import { AuthService } from '../services/auth.service';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}

  intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    
    if (token) {
      request = request.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`
        }
      });
    }
    
    return next.handle(request);
  }
}
```

### Error Interceptor
```typescript
// core/interceptors/error.interceptor.ts
import { Injectable } from '@angular/core';
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor,
  HttpErrorResponse
} from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';
import { Router } from '@angular/router';

@Injectable()
export class ErrorInterceptor implements HttpInterceptor {
  constructor(private router: Router) {}

  intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(request).pipe(
      catchError((error: HttpErrorResponse) => {
        if (error.status === 401) {
          this.router.navigate(['/login']);
        }
        
        const errorMessage = error.error?.message || 'An error occurred';
        return throwError(() => error);
      })
    );
  }
}
```

## Best Practices

### Do
- Use feature-based folder structure
- Use standalone components (Angular 14+)
- Lazy load feature modules
- Use services for business logic and API calls
- Use reactive forms for complex forms
- Use signals for reactive state (Angular 16+)
- Use async pipe for observables in templates
- Implement proper error handling
- Use HTTP interceptors for auth and errors
- Write unit tests for services and components
- Use trackBy in *ngFor loops
- Use OnPush change detection strategy

### Don't
- Don't put business logic in components
- Don't subscribe manually (use async pipe)
- Don't use template-driven forms for complex forms
- Don't forget to unsubscribe (use async pipe or takeUntil)
- Don't use direct DOM manipulation (use Renderer2)
- Don't mix feature and shared modules
- Don't hardcode API URLs (use environment files)
- Don't expose sensitive data in client-side code
- Don't use deprecated APIs
- Don't skip error handling
