# Angular Conventions

## Standalone Components (Angular 14+)

### Basic Standalone Component
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { UserService } from './services/user.service';
import { User } from './models/user.model';

@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule, FormsModule],
  template: `
    <div class="user-list">
      <h2>Users</h2>
      <div *ngFor="let user of users">
        {{ user.name }}
      </div>
    </div>
  `,
  styleUrls: ['./user-list.component.scss']
})
export class UserListComponent implements OnInit {
  users: User[] = [];

  constructor(private userService: UserService) {}

  ngOnInit(): void {
    this.userService.getUsers().subscribe(users => {
      this.users = users;
    });
  }
}
```

## Signals (Angular 16+)

### Using Signals for Reactive State
```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <p>Count: {{ count() }}</p>
    <p>Double: {{ doubleCount() }}</p>
    <button (click)="increment()">Increment</button>
  `
})
export class CounterComponent {
  count = signal(0);
  doubleCount = computed(() => this.count() * 2);

  increment(): void {
    this.count.update(value => value + 1);
  }
}
```

### Signals with Effects
```typescript
import { Component, signal, effect } from '@angular/core';

@Component({
  selector: 'app-user-profile',
  standalone: true,
  template: `<p>User: {{ user().name }}</p>`
})
export class UserProfileComponent {
  user = signal({ name: 'John', age: 30 });

  constructor() {
    effect(() => {
      console.log('User changed:', this.user());
    });
  }
}
```

## Reactive Forms

### Form Group
```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators, ReactiveFormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-user-form',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <input formControlName="name" placeholder="Name">
      <div *ngIf="userForm.get('name')?.invalid">Name is required</div>
      
      <input formControlName="email" placeholder="Email" type="email">
      <div *ngIf="userForm.get('email')?.invalid">Invalid email</div>
      
      <button type="submit" [disabled]="userForm.invalid">Submit</button>
    </form>
  `
})
export class UserFormComponent implements OnInit {
  userForm!: FormGroup;

  constructor(private fb: FormBuilder) {}

  ngOnInit(): void {
    this.userForm = this.fb.group({
      name: ['', [Validators.required, Validators.minLength(2)]],
      email: ['', [Validators.required, Validators.email]],
      age: [null, [Validators.min(18)]]
    });
  }

  onSubmit(): void {
    if (this.userForm.valid) {
      console.log(this.userForm.value);
    }
  }
}
```

## HTTP Client

### Service with HTTP Calls
```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';
import { User } from '../models/user.model';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private http = inject(HttpClient);
  private apiUrl = 'https://api.example.com/users';

  getUsers(page = 1, pageSize = 20): Observable<User[]> {
    const params = new HttpParams()
      .set('page', page.toString())
      .set('pageSize', pageSize.toString());
    
    return this.http.get<User[]>(this.apiUrl, { params });
  }

  getUserById(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }

  createUser(user: Partial<User>): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }

  updateUser(id: number, user: Partial<User>): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }

  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

## Routing

### Route Configuration
```typescript
import { Routes } from '@angular/router';
import { UserListComponent } from './components/user-list/user-list.component';
import { UserDetailComponent } from './components/user-detail/user-detail.component';
import { authGuard } from './guards/auth.guard';

export const routes: Routes = [
  { path: '', component: UserListComponent },
  { 
    path: ':id', 
    component: UserDetailComponent,
    canActivate: [authGuard]
  }
];
```

### Lazy Loading with Standalone Components
```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: 'users',
    loadComponent: () => import('./users/users.component')
      .then(m => m.UsersComponent)
  },
  {
    path: 'dashboard',
    loadComponent: () => import('./dashboard/dashboard.component')
      .then(m => m.DashboardComponent)
  }
];
```

## Change Detection

### OnPush Strategy
```typescript
import { Component, ChangeDetectionStrategy, Input } from '@angular/core';

@Component({
  selector: 'app-user-card',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="user-card">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
    </div>
  `
})
export class UserCardComponent {
  @Input() user!: User;
}
```

## Dependency Injection

### Using inject() Function
```typescript
import { Component, inject } from '@angular/core';
import { UserService } from './services/user.service';
import { Router } from '@angular/router';

@Component({
  selector: 'app-user-list',
  standalone: true,
  template: `...`
})
export class UserListComponent {
  private userService = inject(UserService);
  private router = inject(Router);
  
  users: User[] = [];

  ngOnInit(): void {
    this.userService.getUsers().subscribe(users => {
      this.users = users;
    });
  }
}
```

## Async Pipe

### Using Async Pipe in Templates
```typescript
import { Component } from '@angular/core';
import { Observable } from 'rxjs';
import { UserService } from './services/user.service';
import { User } from './models/user.model';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div *ngFor="let user of users$ | async">
      {{ user.name }}
    </div>
  `
})
export class UserListComponent {
  users$: Observable<User[]>;

  constructor(private userService: UserService) {
    this.users$ = this.userService.getUsers();
  }
}
```

## TrackBy for Performance

### Using trackBy in *ngFor
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div *ngFor="let user of users; trackBy: trackByUserId">
      {{ user.name }}
    </div>
  `
})
export class UserListComponent {
  users: User[] = [];

  trackByUserId(index: number, user: User): number {
    return user.id;
  }
}
```

## Error Handling

### Global Error Handler
```typescript
import { ErrorHandler, Injectable } from '@angular/core';

@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  handleError(error: any): void {
    console.error('An error occurred:', error);
    // Log to error tracking service (Sentry, etc.)
  }
}

// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { GlobalErrorHandler } from './error-handler';

export const appConfig: ApplicationConfig = {
  providers: [
    { provide: ErrorHandler, useClass: GlobalErrorHandler }
  ]
};
```

## Best Practices

### Do
- Use standalone components (Angular 14+)
- Use signals for reactive state (Angular 16+)
- Use `inject()` function instead of constructor injection
- Use async pipe for observables
- Use OnPush change detection strategy
- Use trackBy in *ngFor loops
- Use reactive forms for complex forms
- Lazy load feature modules/components
- Use HTTP interceptors for auth and errors
- Implement proper error handling
- Write unit tests for services and components
- Use environment files for configuration
- Use Angular CLI for generating components/services

### Don't
- Don't subscribe manually (use async pipe)
- Don't use template-driven forms for complex forms
- Don't forget to unsubscribe (use async pipe or takeUntil)
- Don't use direct DOM manipulation (use Renderer2)
- Don't hardcode API URLs
- Don't expose sensitive data in client-side code
- Don't use deprecated APIs
- Don't skip error handling
- Don't use default change detection for simple components
- Don't mix NgModule and standalone components unnecessarily
