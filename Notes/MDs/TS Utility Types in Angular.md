# TypeScript Utility Types in Angular

> [!abstract] Overview TypeScript utility types are one of the most powerful tools for building type-safe Angular applications. They allow you to transform and manipulate existing types in flexible ways — reducing redundancy and improving code maintainability across services, components, and state management. This guide explores the most commonly used built-in utility types and shows you how to apply them in real Angular patterns, along with custom utility types for advanced scenarios.

---

## What Are Utility Types?

Utility types are **predefined type transformations** built into TypeScript. They let you derive new types from existing ones without rewriting them, which is especially valuable in large Angular codebases where models are shared across services, components, and API layers.

> [!tip] Why This Matters in Angular Angular applications commonly share data models across services, components, reactive forms, and HTTP responses. Utility types let you shape those models per-context without duplicating or diverging your type definitions.

---

## Table of Contents

- [[#Pick]]
- [[#Omit]]
- [[#Partial]]
- [[#Readonly]]
- [[#Record]]
- [[#Custom Utility Types]]
    - [[#Nullable]]
    - [[#WithOptional]]
    - [[#WithRequired]]

---

## Built-In Utility Types

### Pick

> **Syntax:** `Pick<Type, Keys>` Creates a new type by selecting a **subset of properties** from an existing type.

#### When to Use in Angular

Use `Pick` when a component or service only needs a slice of a larger model — for example, displaying a user's name and avatar in a nav bar without exposing sensitive data.

#### Example: Angular Component with Picked Type

```typescript
// models/user.model.ts
export interface User {
  id: number;
  name: string;
  email: string;
  age: number;
  role: string;
  passwordHash: string;
}

// We only need name and email for the contact card
export type UserContactInfo = Pick<User, 'name' | 'email'>;
```

```typescript
// user-contact-card.component.ts
import { Component, Input } from '@angular/core';
import { UserContactInfo } from '../models/user.model';

@Component({
  selector: 'app-user-contact-card',
  template: `
    <div class="contact-card">
      <p><strong>Name:</strong> {{ user.name }}</p>
      <p><strong>Email:</strong> {{ user.email }}</p>
    </div>
  `
})
export class UserContactCardComponent {
  @Input() user!: UserContactInfo;
}
```

```typescript
// parent.component.ts
import { Component } from '@angular/core';
import { UserContactInfo } from '../models/user.model';

@Component({
  selector: 'app-parent',
  template: `<app-user-contact-card [user]="contactInfo" />`
})
export class ParentComponent {
  contactInfo: UserContactInfo = {
    name: 'John Doe',
    email: 'john@example.com'
    // id, age, role, passwordHash are excluded — type-safe!
  };
}
```

> [!note] Angular Service Usage `Pick` is equally useful in services when projecting API responses into a leaner shape before passing data to components.

---

### Omit

> **Syntax:** `Omit<Type, Keys>` Creates a new type by **excluding specific properties** from an existing type.

#### When to Use in Angular

Use `Omit` when creating display models from data models — for example, hiding `id` and server-generated fields when rendering user-facing profiles.

#### Example: Angular Component with Omitted Type

```typescript
// models/user.model.ts
export interface User {
  id: number;
  name: string;
  email: string;
  age: number;
  createdAt: Date;
}

// Exclude server-managed fields for the profile display
export type UserProfile = Omit<User, 'id' | 'createdAt'>;
```

```typescript
// user-profile.component.ts
import { Component, Input } from '@angular/core';
import { UserProfile } from '../models/user.model';

@Component({
  selector: 'app-user-profile',
  template: `
    <div class="profile">
      <h2>{{ profile.name }}</h2>
      <p>Email: {{ profile.email }}</p>
      <p>Age: {{ profile.age }}</p>
    </div>
  `
})
export class UserProfileComponent {
  @Input() profile!: UserProfile;
}
```

```typescript
// user.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { map } from 'rxjs/operators';
import { User, UserProfile } from '../models/user.model';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(private http: HttpClient) {}

  getUserProfile(id: number): Observable<UserProfile> {
    return this.http.get<User>(`/api/users/${id}`).pipe(
      map(({ id, createdAt, ...profile }) => profile) // Strip server fields
    );
  }
}
```

---

### Partial

> **Syntax:** `Partial<Type>` Makes **all properties of a type optional**.

#### When to Use in Angular

`Partial` is a natural fit for Angular **Reactive Forms** — where you may not have all values until the user completes the form — and for `PATCH` HTTP requests where only changed fields are sent.

#### Example: Angular Reactive Form with Partial

```typescript
// models/user.model.ts
export interface User {
  id: number;
  name: string;
  email: string;
  bio: string;
}

export type PartialUser = Partial<User>;
```

```typescript
// edit-user.component.ts
import { Component, Input, OnInit } from '@angular/core';
import { FormBuilder, FormGroup } from '@angular/forms';
import { PartialUser } from '../models/user.model';
import { UserService } from '../services/user.service';

@Component({
  selector: 'app-edit-user',
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <label>
        Name
        <input formControlName="name" placeholder="Name" />
      </label>
      <label>
        Email
        <input formControlName="email" placeholder="Email" />
      </label>
      <label>
        Bio
        <textarea formControlName="bio" placeholder="Bio"></textarea>
      </label>
      <button type="submit">Save Changes</button>
    </form>
  `
})
export class EditUserComponent implements OnInit {
  @Input() user: PartialUser = {};

  form!: FormGroup;

  constructor(
    private fb: FormBuilder,
    private userService: UserService
  ) {}

  ngOnInit(): void {
    // Form initializes safely even if user fields are undefined
    this.form = this.fb.group({
      name: [this.user.name ?? ''],
      email: [this.user.email ?? ''],
      bio: [this.user.bio ?? '']
    });
  }

  onSubmit(): void {
    const changes: PartialUser = this.form.value;
    this.userService.patchUser(this.user.id!, changes).subscribe();
  }
}
```

```typescript
// user.service.ts (PATCH method)
patchUser(id: number, changes: PartialUser): Observable<User> {
  return this.http.patch<User>(`/api/users/${id}`, changes);
}
```

> [!tip] Partial + Reactive Forms `Partial` pairs perfectly with `patchValue()` on `FormGroup`, since `patchValue` itself accepts a partial object.

---

### Readonly

> **Syntax:** `Readonly<Type>` Makes **all properties of a type immutable** at the TypeScript level.

#### When to Use in Angular

Use `Readonly` for application configuration objects, NgRx/signal state snapshots, and component `@Input()` data that should not be mutated locally.

#### Example: App Config and Component Inputs

```typescript
// config/app.config.ts
export interface AppConfig {
  apiUrl: string;
  timeout: number;
  featureFlags: Record<string, boolean>;
}

export const APP_CONFIG: Readonly<AppConfig> = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  featureFlags: { darkMode: true, betaFeatures: false }
};

// This will cause a TypeScript compile error:
// APP_CONFIG.apiUrl = 'https://other.com'; ❌
```

```typescript
// dashboard.component.ts
import { Component, Input } from '@angular/core';
import { User } from '../models/user.model';

@Component({
  selector: 'app-dashboard',
  template: `
    <h1>Welcome, {{ currentUser.name }}</h1>
    <p>Role: {{ currentUser.role }}</p>
  `
})
export class DashboardComponent {
  // Readonly signals intent: don't mutate this input locally
  @Input() currentUser!: Readonly<User>;

  updateName(): void {
    // TypeScript error: cannot assign to 'name' because it is a read-only property
    // this.currentUser.name = 'New Name'; ❌
  }
}
```

```typescript
// state/user.state.ts — useful with NgRx or Angular Signals
import { signal } from '@angular/core';
import { User } from '../models/user.model';

// A read-only view of state prevents accidental mutation
export const userState = signal<Readonly<User>>({
  id: 1,
  name: 'Jane Doe',
  email: 'jane@example.com',
  age: 28,
  role: 'admin',
  passwordHash: ''
});
```

---

### Record

> **Syntax:** `Record<Keys, Type>` Creates an **object type with specified keys mapped to a value type**.

#### When to Use in Angular

`Record` is ideal for lookup maps, permission tables, route configuration, and tab/step structures — anywhere you map a union of known keys to structured values.

#### Example: Role-Based Permissions in a Guard

```typescript
// models/permissions.model.ts
export type Role = 'admin' | 'editor' | 'viewer';

export type Permissions = Record<Role, string[]>;

export const PERMISSIONS: Permissions = {
  admin: ['read', 'write', 'delete', 'manage_users'],
  editor: ['read', 'write'],
  viewer: ['read']
};
```

```typescript
// permissions.component.ts
import { Component, Input } from '@angular/core';
import { Role, PERMISSIONS } from '../models/permissions.model';

@Component({
  selector: 'app-permissions-display',
  template: `
    <div class="permissions">
      <h3>Permissions for {{ role }}</h3>
      <ul>
        <li *ngFor="let permission of rolePermissions">
          {{ permission }}
        </li>
      </ul>
    </div>
  `
})
export class PermissionsDisplayComponent {
  @Input() role!: Role;

  get rolePermissions(): string[] {
    return PERMISSIONS[this.role];
  }
}
```

```typescript
// auth.guard.ts
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, Router } from '@angular/router';
import { AuthService } from './auth.service';
import { Role, PERMISSIONS } from '../models/permissions.model';

// Map each route to the required permission — typed with Record
const ROUTE_PERMISSIONS: Record<string, string> = {
  '/admin': 'manage_users',
  '/editor': 'write',
  '/dashboard': 'read'
};

@Injectable({ providedIn: 'root' })
export class PermissionGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(route: ActivatedRouteSnapshot): boolean {
    const path = '/' + route.url.join('/');
    const required = ROUTE_PERMISSIONS[path];
    const userRole: Role = this.authService.getRole();
    const hasAccess = PERMISSIONS[userRole].includes(required);

    if (!hasAccess) this.router.navigate(['/unauthorized']);
    return hasAccess;
  }
}
```

---

## Custom Utility Types

Sometimes the built-in utility types don't cover your exact scenario. TypeScript's type system lets you compose your own utilities using mapped types, conditional types, and intersections.

---

### Nullable

> Makes every property of a type accept `null` as a value.

#### Implementation

```typescript
// types/nullable.type.ts
export type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};
```

#### Angular Example: API Response Handling

API responses often return `null` for missing fields. `Nullable` makes that contract explicit.

```typescript
// models/user.model.ts
export interface User {
  id: number;
  name: string;
  email: string;
  avatarUrl: string;
}

export type NullableUser = Nullable<User>;
```

```typescript
// user-display.component.ts
import { Component, Input } from '@angular/core';
import { NullableUser } from '../models/user.model';

@Component({
  selector: 'app-user-display',
  template: `
    <div class="user-card">
      <img
        [src]="user.avatarUrl ?? '/assets/default-avatar.png'"
        alt="Avatar"
      />
      <p>{{ user.name ?? 'Anonymous' }}</p>
      <p>{{ user.email ?? 'No email provided' }}</p>
    </div>
  `
})
export class UserDisplayComponent {
  @Input() user!: NullableUser;
}
```

```typescript
// user.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { NullableUser } from '../models/user.model';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(private http: HttpClient) {}

  getUser(id: number): Observable<NullableUser> {
    // API may return null fields — our type now reflects that
    return this.http.get<NullableUser>(`/api/users/${id}`);
  }
}
```

> [!note] Null vs Undefined in Angular Angular's template expressions and reactive forms handle `null` and `undefined` differently. Using `Nullable<T>` makes the null contract explicit and helps avoid silent `undefined` bugs.

---

### WithOptional

> Makes **specific properties optional** while keeping the rest required.

#### Implementation

```typescript
// types/with-optional.type.ts
export type WithOptional<T, K extends keyof T> =
  Omit<T, K> & Partial<Pick<T, K>>;
```

**How it works:**

- `Omit<T, K>` removes the keys you want to make optional
- `Partial<Pick<T, K>>` re-adds them as optional
- The intersection `&` combines both into a final type

#### Angular Example: Create vs Edit Forms

A "create user" form requires some fields and makes others optional, while a full `User` model requires everything.

```typescript
// models/user.model.ts
export interface User {
  id: number;
  name: string;
  email: string;
  age: number;
  bio: string;
}

// For the create form: `age` and `bio` are optional, the rest are required
export type CreateUserPayload = WithOptional<User, 'age' | 'bio'>;
```

```typescript
// create-user.component.ts
import { Component } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { CreateUserPayload } from '../models/user.model';
import { UserService } from '../services/user.service';

@Component({
  selector: 'app-create-user',
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div>
        <label>Name *</label>
        <input formControlName="name" />
        <small *ngIf="form.get('name')?.invalid && form.get('name')?.touched">
          Name is required
        </small>
      </div>

      <div>
        <label>Email *</label>
        <input formControlName="email" type="email" />
      </div>

      <!-- Age and Bio are optional -->
      <div>
        <label>Age (optional)</label>
        <input formControlName="age" type="number" />
      </div>

      <div>
        <label>Bio (optional)</label>
        <textarea formControlName="bio"></textarea>
      </div>

      <button type="submit" [disabled]="form.invalid">Create User</button>
    </form>
  `
})
export class CreateUserComponent {
  form: FormGroup;

  constructor(
    private fb: FormBuilder,
    private userService: UserService
  ) {
    this.form = this.fb.group({
      name: ['', Validators.required],
      email: ['', [Validators.required, Validators.email]],
      age: [null],   // optional
      bio: ['']      // optional
    });
  }

  onSubmit(): void {
    if (this.form.valid) {
      // TypeScript knows `name` and `email` are required here,
      // but `age` and `bio` may be undefined
      const payload: CreateUserPayload = {
        id: 0, // assigned by server
        name: this.form.value.name,
        email: this.form.value.email,
        ...(this.form.value.age && { age: this.form.value.age }),
        ...(this.form.value.bio && { bio: this.form.value.bio })
      };
      this.userService.createUser(payload).subscribe();
    }
  }
}
```

```typescript
// user.service.ts
createUser(payload: CreateUserPayload): Observable<User> {
  return this.http.post<User>('/api/users', payload);
}
```

> [!example] Real-World Scenario `WithOptional` is also great for **multi-step forms** — earlier steps collect required fields and later steps collect optional enrichment data.

---

### WithRequired

> Makes **specific properties required** while keeping the rest optional.

#### Implementation

```typescript
// types/with-required.type.ts
export type WithRequired<T, K extends keyof T> =
  T & { [P in K]-?: T[P] };
```

**How it works:**

- `T` preserves all existing types
- `{ [P in K]-?: T[P] }` uses the `-?` modifier to **remove optionality** from the specified keys
- The intersection enforces that those keys must be present

#### Angular Example: Resolving Data Before Navigation

Angular **Route Resolvers** guarantee certain data is present before a component activates. `WithRequired` models that guarantee in the type system.

```typescript
// models/user.model.ts
export interface User {
  id?: number;
  name?: string;
  email?: string;
  age?: number;
  role?: string;
}

// After a resolver runs, we know `id` and `name` are always present
export type ResolvedUser = WithRequired<User, 'id' | 'name'>;
```

```typescript
// user.resolver.ts
import { Injectable } from '@angular/core';
import { Resolve, ActivatedRouteSnapshot } from '@angular/router';
import { Observable } from 'rxjs';
import { UserService } from './user.service';
import { ResolvedUser } from '../models/user.model';

@Injectable({ providedIn: 'root' })
export class UserResolver implements Resolve<ResolvedUser> {
  constructor(private userService: UserService) {}

  resolve(route: ActivatedRouteSnapshot): Observable<ResolvedUser> {
    const id = Number(route.paramMap.get('id'));
    // Service guarantees `id` and `name` are present in the response
    return this.userService.getResolvedUser(id);
  }
}
```

```typescript
// user-detail.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { ResolvedUser } from '../models/user.model';

@Component({
  selector: 'app-user-detail',
  template: `
    <div class="user-detail">
      <!-- id and name are guaranteed — no null checks needed -->
      <h1>User #{{ user.id }} — {{ user.name }}</h1>
      <!-- Other fields remain optional -->
      <p *ngIf="user.email">Email: {{ user.email }}</p>
      <p *ngIf="user.age">Age: {{ user.age }}</p>
      <p *ngIf="user.role">Role: {{ user.role }}</p>
    </div>
  `
})
export class UserDetailComponent implements OnInit {
  user!: ResolvedUser;

  constructor(private route: ActivatedRoute) {}

  ngOnInit(): void {
    // Data is injected by the resolver — `id` and `name` are safe to access
    this.user = this.route.snapshot.data['user'] as ResolvedUser;
  }
}
```

```typescript
// app-routing.module.ts
const routes: Routes = [
  {
    path: 'users/:id',
    component: UserDetailComponent,
    resolve: { user: UserResolver }
  }
];
```

> [!tip] Resolver + WithRequired Using `WithRequired` with resolvers lets TypeScript enforce that a component only renders when critical data is guaranteed to be present — eliminating an entire class of null-check bugs.

---

## Summary

|Utility Type|What It Does|Angular Use Case|
|---|---|---|
|`Pick<T, K>`|Select a subset of properties|Scoped component `@Input()`|
|`Omit<T, K>`|Exclude specific properties|Display models from data models|
|`Partial<T>`|Make all properties optional|Reactive forms, PATCH requests|
|`Readonly<T>`|Make all properties immutable|Config objects, state snapshots|
|`Record<K, V>`|Map known keys to a value type|Permission tables, lookup maps|
|`Nullable<T>` _(custom)_|Allow `null` on all properties|API response typing|
|`WithOptional<T, K>` _(custom)_|Make specific properties optional|Create forms, partial payloads|
|`WithRequired<T, K>` _(custom)_|Make specific properties required|Route resolvers, post-validation|

---

## Why Custom Utility Types Are Worth It

> [!summary] Key Takeaways Custom utility types like `WithOptional` and `WithRequired` let you:
> 
> - **Reduce redundancy** — No more copy-pasting nearly-identical interfaces
> - **Improve readability** — Type names communicate intent (`ResolvedUser`, `CreateUserPayload`)
> - **Enforce contracts** — TypeScript will catch mismatches between your form, service, and API layers at compile time
> - **Scale cleanly** — When your base model changes, all derived types update automatically

---

