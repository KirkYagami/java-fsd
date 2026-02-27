# 🧭 Angular 21 — Routing Complete Lab

---

> **Lab Overview** This is a complete, hands-on lab for Angular 21 routing. Every file is shown in full with incremental changes clearly marked. No steps are skipped.
> 
> **What you will build:** A Product Store SPA with home, product listing, product detail, about (with named outlets), an admin section (lazy-loaded with guards), a 404 page, and programmatic navigation.
> 
> **Angular 21 Key Differences vs older Angular:**
> 
> - No `AppModule` / `NgModule` — fully standalone
> - `provideRouter()` in `app.config.ts` replaces `RouterModule.forRoot()`
> - `loadComponent()` for lazy loading (no modules needed)
> - `loadChildren()` points directly to a routes file (not a module)
> - Functional guards with `inject()` — class-based guards are deprecated
> - `withComponentInputBinding()` — route params bind directly to `input()` signals
> - `input()` signals replace `ActivatedRoute` subscriptions
> - Zoneless change detection is default — signals drive reactivity
> - `@if` / `@for` built-in control flow replaces `*ngIf` / `*ngFor`

---

## 📚 Table of Contents

- [[#Part 1 — Concepts & Mental Models]]
- [[#Part 2 — Project Setup]]
- [[#Part 3 — Project Structure]]
- [[#Part 4 — Core App Files]]
- [[#Part 5 — Feature Components]]
- [[#Part 6 — Route Configuration]]
- [[#Part 7 — Child Routes & Nested Outlets]]
- [[#Part 8 — Named Router Outlets]]
- [[#Part 9 — Route Parameters & Signal Inputs]]
- [[#Part 10 — Programmatic Navigation]]
- [[#Part 11 — Lazy Loading]]
- [[#Part 12 — Route Guards]]
- [[#Part 13 — Route Resolvers]]
- [[#Part 14 — Route Data & Page Titles]]
- [[#Part 15 — Wildcard & 404 Routes]]
- [[#Part 16 — Router Events]]
- [[#Part 17 — Complete Routes File Reference]]

---

# Part 1 — Concepts & Mental Models

## 1.1 How Angular Routing Works

```
Browser URL changes
       │
       ▼
  Angular Router
  (listens via History API)
       │
       ▼
  Matches URL against routes[]
       │
       ▼
  Runs Guards ──── false? ──▶ Cancel navigation
       │ true
       ▼
  Runs Resolvers (pre-fetch data)
       │
       ▼
  Instantiates Component
       │
       ▼
  Renders into <router-outlet>
```

## 1.2 Angular 21 Architecture — What Changed

```
BEFORE (Angular < 14):                AFTER (Angular 21):
──────────────────────────            ──────────────────────────
AppModule                             app.config.ts
  imports: [RouterModule.forRoot()]     providers: [provideRouter(routes)]

app-routing.module.ts                 app.routes.ts
  RouterModule.forRoot(routes)          export const routes: Routes = [...]

Components needed NgModule            Components are standalone
  declarations: [HomeComponent]         imports: [RouterLink] directly

Class-based Guards                    Functional Guards
  implements CanActivate                export const authGuard: CanActivateFn = () => ...
  constructor(private ...)              const service = inject(AuthService)

ActivatedRoute subscription           Signal input binding
  this.route.params.subscribe(...)      id = input<string>()  ← bound automatically
```

## 1.3 Routing Building Blocks

|Piece|File|Purpose|
|---|---|---|
|`Routes` array|`app.routes.ts`|Maps URL paths → components|
|`provideRouter()`|`app.config.ts`|Registers router with the app|
|`<router-outlet>`|`app.component.html`|Where matched component renders|
|`routerLink`|Any template|Navigate without page reload|
|`routerLinkActive`|Any template|Add CSS class to active link|
|Guards|`guards/*.ts`|Allow / deny navigation|
|Resolvers|`resolvers/*.ts`|Pre-load data before component shows|
|`loadComponent`|`app.routes.ts`|Lazy-load a standalone component|
|`loadChildren`|`app.routes.ts`|Lazy-load a child routes file|

---

# Part 2 — Project Setup

## 2.1 Prerequisites

```bash
# Verify you have Node.js 22+ and npm 10+
node --version   # Should be v22.x.x or higher
npm --version    # Should be 10.x.x or higher
```

## 2.2 Install Angular CLI 21

```bash
# Install or update Angular CLI globally
npm install -g @angular/cli@21

# Verify
ng version
# Should show: Angular CLI: 21.x.x
```

## 2.3 Create the Project

```bash
# Create new Angular 21 project
# Angular 21 creates standalone projects by default — no --standalone flag needed
ng new product-store --routing --style css

# When prompted:
# ✔ Would you like to enable Server-Side Rendering (SSR)?  → No (for this lab)

# Move into the project
cd product-store
```

> 💡 **What `--routing` does:** It pre-creates `app.routes.ts` and wires it into `app.config.ts` for you. In Angular 21, this is the default standalone setup.

## 2.4 Install Bootstrap (for styling)

```bash
npm install bootstrap
```

## 2.5 Configure Bootstrap

Open `angular.json` and add bootstrap to styles and scripts:

```json
// angular.json  (find "styles" and "scripts" under architect > build > options)
{
  "styles": [
    "src/styles.css",
    "node_modules/bootstrap/dist/css/bootstrap.min.css"
  ],
  "scripts": [
    "node_modules/bootstrap/dist/js/bootstrap.bundle.min.js"
  ]
}
```

## 2.6 Generate All Components

Run these commands to scaffold every component we'll use in this lab:

```bash
# Pages / route components
ng generate component pages/home          --skip-tests
ng generate component pages/about         --skip-tests
ng generate component pages/contact       --skip-tests
ng generate component pages/not-found     --skip-tests

# Product section
ng generate component pages/products/product-list    --skip-tests
ng generate component pages/products/product-detail  --skip-tests

# About page sub-sections (for named outlets)
ng generate component pages/about/about-reviews      --skip-tests
ng generate component pages/about/about-team         --skip-tests

# Admin section (will be lazy-loaded)
ng generate component pages/admin/admin-dashboard    --skip-tests
ng generate component pages/admin/admin-users        --skip-tests

# Shared layout
ng generate component shared/navbar   --skip-tests

# Services
ng generate service services/product  --skip-tests
ng generate service services/auth     --skip-tests

# Guards
ng generate guard guards/auth         --skip-tests --functional
ng generate guard guards/admin        --skip-tests --functional

# Resolvers
ng generate resolver resolvers/product-detail --skip-tests
```

> 💡 In Angular 21, all generated components are **standalone** by default. The `--functional` flag on guards generates a functional guard (the modern Angular 21 way).

---

# Part 3 — Project Structure

After running all generators, your `src/app/` folder should look like this:

```
src/
├── app/
│   ├── app.component.ts          ← root component (has router-outlet)
│   ├── app.component.html
│   ├── app.component.css
│   ├── app.config.ts             ← app bootstrap config (provideRouter here)
│   ├── app.routes.ts             ← ALL route definitions live here
│   │
│   ├── pages/
│   │   ├── home/
│   │   │   └── home.component.ts
│   │   ├── about/
│   │   │   ├── about.component.ts
│   │   │   ├── about-reviews/
│   │   │   │   └── about-reviews.component.ts
│   │   │   └── about-team/
│   │   │       └── about-team.component.ts
│   │   ├── contact/
│   │   │   └── contact.component.ts
│   │   ├── not-found/
│   │   │   └── not-found.component.ts
│   │   ├── products/
│   │   │   ├── product-list/
│   │   │   │   └── product-list.component.ts
│   │   │   └── product-detail/
│   │   │       └── product-detail.component.ts
│   │   └── admin/
│   │       ├── admin.routes.ts   ← admin's own routes file (lazy-loaded)
│   │       ├── admin-dashboard/
│   │       │   └── admin-dashboard.component.ts
│   │       └── admin-users/
│   │           └── admin-users.component.ts
│   │
│   ├── shared/
│   │   └── navbar/
│   │       └── navbar.component.ts
│   │
│   ├── services/
│   │   ├── product.service.ts
│   │   └── auth.service.ts
│   │
│   ├── guards/
│   │   ├── auth.guard.ts
│   │   └── admin.guard.ts
│   │
│   └── resolvers/
│       └── product-detail.resolver.ts
│
├── index.html
└── styles.css
```

---

# Part 4 — Core App Files

## 4.1 `src/index.html`

This file rarely needs changes. Verify the `<base href>` tag is present — Angular Router requires it for HTML5-style URL routing.

```html
<!-- src/index.html — full file -->
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Product Store</title>
  <base href="/">   <!-- ← REQUIRED: tells router where the app root is -->
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>
  <app-root></app-root>   <!-- Angular mounts the app here -->
</body>
</html>
```

## 4.2 `src/styles.css`

```css
/* src/styles.css — full file */

/* Active nav link highlight */
.nav-link.active-link {
  color: #0d6efd !important;
  font-weight: 600;
  border-bottom: 2px solid #0d6efd;
}

/* Router outlet fade animation */
router-outlet + * {
  animation: fadeIn 0.2s ease-in;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* Card hover effect */
.product-card {
  transition: box-shadow 0.2s ease, transform 0.2s ease;
  cursor: pointer;
}
.product-card:hover {
  box-shadow: 0 8px 24px rgba(0,0,0,0.12);
  transform: translateY(-2px);
}
```

## 4.3 `src/app/services/product.service.ts`

We'll create a simple product service with mock data. This will be used by our components and resolvers.

```typescript
// src/app/services/product.service.ts — full file

import { Injectable, signal } from '@angular/core';

// Step 1: Define the Product interface
export interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
  category: string;
  imageUrl: string;
  inStock: boolean;
}

@Injectable({ providedIn: 'root' })
export class ProductService {

  // Step 2: Mock product data (in a real app this would come from HttpClient)
  private readonly mockProducts: Product[] = [
    {
      id: 1,
      name: 'Wireless Headphones',
      description: 'Premium noise-cancelling wireless headphones with 40-hour battery life.',
      price: 299.99,
      category: 'Electronics',
      imageUrl: 'https://placehold.co/400x300?text=Headphones',
      inStock: true
    },
    {
      id: 2,
      name: 'Mechanical Keyboard',
      description: 'Compact TKL mechanical keyboard with RGB backlight and Cherry MX switches.',
      price: 149.99,
      category: 'Electronics',
      imageUrl: 'https://placehold.co/400x300?text=Keyboard',
      inStock: true
    },
    {
      id: 3,
      name: 'Standing Desk',
      description: 'Height-adjustable electric standing desk with memory presets.',
      price: 599.99,
      category: 'Furniture',
      imageUrl: 'https://placehold.co/400x300?text=Desk',
      inStock: false
    },
    {
      id: 4,
      name: 'Ergonomic Chair',
      description: 'Fully adjustable ergonomic office chair with lumbar support.',
      price: 449.99,
      category: 'Furniture',
      imageUrl: 'https://placehold.co/400x300?text=Chair',
      inStock: true
    },
    {
      id: 5,
      name: '4K Monitor',
      description: '27-inch 4K IPS display with USB-C connectivity and 144Hz refresh.',
      price: 699.99,
      category: 'Electronics',
      imageUrl: 'https://placehold.co/400x300?text=Monitor',
      inStock: true
    }
  ];

  // Step 3: Expose as a signal for reactive updates
  private products = signal<Product[]>(this.mockProducts);

  // Step 4: Public methods
  getAll(): Product[] {
    return this.products();
  }

  getById(id: number): Product | undefined {
    return this.products().find(p => p.id === id);
  }

  getByCategory(category: string): Product[] {
    return this.products().filter(p => p.category === category);
  }
}
```

## 4.4 `src/app/services/auth.service.ts`

A simple auth service with a signal-based login state. Our guards will use this.

```typescript
// src/app/services/auth.service.ts — full file

import { Injectable, signal, computed } from '@angular/core';
import { Router } from '@angular/router';

export interface User {
  id: number;
  name: string;
  email: string;
  role: 'user' | 'admin';
}

@Injectable({ providedIn: 'root' })
export class AuthService {

  // Step 1: Private signal holding the current user (null = not logged in)
  private currentUser = signal<User | null>(null);

  // Step 2: Public read-only signals derived from currentUser
  readonly isLoggedIn = computed(() => this.currentUser() !== null);
  readonly isAdmin    = computed(() => this.currentUser()?.role === 'admin');
  readonly user       = this.currentUser.asReadonly();

  constructor(private router: Router) {
    // Step 3: Restore session from localStorage on app start
    const saved = localStorage.getItem('user');
    if (saved) {
      this.currentUser.set(JSON.parse(saved));
    }
  }

  // Step 4: Login method (simulates an async auth call)
  login(email: string, password: string): boolean {
    // Mock: admin@store.com / admin123 → admin user
    //       user@store.com  / user123  → regular user
    if (email === 'admin@store.com' && password === 'admin123') {
      const user: User = { id: 1, name: 'Admin User', email, role: 'admin' };
      this.currentUser.set(user);
      localStorage.setItem('user', JSON.stringify(user));
      return true;
    }
    if (email === 'user@store.com' && password === 'user123') {
      const user: User = { id: 2, name: 'Regular User', email, role: 'user' };
      this.currentUser.set(user);
      localStorage.setItem('user', JSON.stringify(user));
      return true;
    }
    return false;
  }

  logout(): void {
    this.currentUser.set(null);
    localStorage.removeItem('user');
    this.router.navigate(['/home']);
  }
}
```

## 4.5 `src/app/app.config.ts`

This is the **heart of Angular 21 setup**. All providers (including router) go here.

```typescript
// src/app/app.config.ts — full file
// Step 1: Start with the generated file (after ng new --routing)

import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    // Note: Angular 21 is zoneless by default.
    // provideZoneChangeDetection is NOT needed unless you want zone.js back.
    provideRouter(routes)
  ]
};
```

We'll expand this file in later sections. For now this is our starting point.

## 4.6 `src/app/app.component.ts`

The root component. In Angular 21 routing, this is the **shell** — it contains the navbar and the primary `<router-outlet>`.

```typescript
// src/app/app.component.ts — full file

import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';   // ← IMPORTANT: must import RouterOutlet
import { NavbarComponent } from './shared/navbar/navbar.component';

@Component({
  selector: 'app-root',
  standalone: true,
  // Step 1: Import RouterOutlet (required for <router-outlet> to work in standalone)
  // Step 2: Import NavbarComponent (our shared navigation)
  imports: [RouterOutlet, NavbarComponent],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  title = 'product-store';
}
```

## 4.7 `src/app/app.component.html`

```html
<!-- src/app/app.component.html — full file -->

<!-- Navbar appears on every page -->
<app-navbar></app-navbar>

<!-- Main content area -->
<main class="container mt-4">
  <!-- router-outlet is the placeholder where routed components render -->
  <!-- When URL is /home    → HomeComponent renders here              -->
  <!-- When URL is /products → ProductListComponent renders here      -->
  <!-- etc.                                                           -->
  <router-outlet></router-outlet>
</main>

<!-- Footer -->
<footer class="bg-light text-center py-3 mt-5 border-top">
  <small class="text-muted">© 2025 Product Store — Built with Angular 21</small>
</footer>
```

---

# Part 5 — Feature Components

Let's build each page component one by one.

## 5.1 Navbar Component

The navbar uses `routerLink` for navigation and `routerLinkActive` for highlighting the active route.

```typescript
// src/app/shared/navbar/navbar.component.ts — full file

import { Component, inject } from '@angular/core';
import { RouterLink, RouterLinkActive } from '@angular/router';
// ↑ IMPORTANT: In standalone components, you must import these directives explicitly

import { AuthService } from '../../services/auth.service';

@Component({
  selector: 'app-navbar',
  standalone: true,
  // Step 1: Import both RouterLink and RouterLinkActive
  imports: [RouterLink, RouterLinkActive],
  templateUrl: './navbar.component.html',
  styleUrl: './navbar.component.css'
})
export class NavbarComponent {
  // Step 2: Inject AuthService using the modern inject() function
  protected auth = inject(AuthService);
}
```

```html
<!-- src/app/shared/navbar/navbar.component.html — full file -->

<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
  <div class="container">

    <!-- Brand / Logo — clicking navigates to /home -->
    <a class="navbar-brand" routerLink="/home">🛍️ Product Store</a>

    <!-- Mobile hamburger toggle -->
    <button class="navbar-toggler" type="button"
            data-bs-toggle="collapse" data-bs-target="#navMenu">
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="navMenu">
      <ul class="navbar-nav me-auto">

        <!-- routerLink  = where to go (replaces href) -->
        <!-- routerLinkActive = CSS class added when this route is active -->
        <!-- [routerLinkActiveOptions]="{exact: true}" = only match exact path -->

        <li class="nav-item">
          <a class="nav-link"
             routerLink="/home"
             routerLinkActive="active-link"
             [routerLinkActiveOptions]="{ exact: true }">
            Home
          </a>
        </li>

        <li class="nav-item">
          <a class="nav-link"
             routerLink="/products"
             routerLinkActive="active-link">
            Products
          </a>
        </li>

        <li class="nav-item">
          <!-- routerLink with named outlet parameters (covered in Part 8) -->
          <a class="nav-link"
             [routerLink]="['/about', { outlets: { reviews: ['reviews'], team: ['team'] } }]"
             routerLinkActive="active-link">
            About
          </a>
        </li>

        <li class="nav-item">
          <a class="nav-link"
             routerLink="/contact"
             routerLinkActive="active-link">
            Contact
          </a>
        </li>

        <!-- Admin link — only shown to admin users -->
        @if (auth.isAdmin()) {
          <li class="nav-item">
            <a class="nav-link"
               routerLink="/admin"
               routerLinkActive="active-link">
              ⚙️ Admin
            </a>
          </li>
        }

      </ul>

      <!-- Right side: auth controls -->
      <ul class="navbar-nav ms-auto">
        @if (auth.isLoggedIn()) {
          <li class="nav-item d-flex align-items-center">
            <span class="navbar-text text-light me-3">
              👤 {{ auth.user()?.name }}
            </span>
            <button class="btn btn-outline-light btn-sm"
                    (click)="auth.logout()">
              Logout
            </button>
          </li>
        } @else {
          <li class="nav-item">
            <a class="nav-link" routerLink="/login">Login</a>
          </li>
        }
      </ul>
    </div>
  </div>
</nav>
```

> 💡 **Key Points:**
> 
> - `routerLink="/home"` is equivalent to `href="/home"` but **does NOT reload the page**
> - `routerLinkActive="active-link"` adds `active-link` CSS class when the route is active
> - `[routerLinkActiveOptions]="{ exact: true }"` — without this, `/home` would also match `/home/child`
> - `@if` is Angular 21's built-in control flow (replaces `*ngIf`)

## 5.2 Home Component

```typescript
// src/app/pages/home/home.component.ts — full file

import { Component } from '@angular/core';
import { RouterLink } from '@angular/router';

@Component({
  selector: 'app-home',
  standalone: true,
  imports: [RouterLink],
  templateUrl: './home.component.html',
  styleUrl: './home.component.css'
})
export class HomeComponent {}
```

```html
<!-- src/app/pages/home/home.component.html — full file -->

<div class="px-4 py-5 text-center">
  <h1 class="display-5 fw-bold">Welcome to Product Store 🛍️</h1>
  <p class="col-lg-8 mx-auto fs-5 text-muted">
    Discover our curated collection of premium products. Quality you can trust, prices you'll love.
  </p>
  <div class="d-grid gap-2 d-sm-flex justify-content-sm-center">
    <!-- routerLink as an array for programmatic construction -->
    <a [routerLink]="['/products']" class="btn btn-primary btn-lg px-4 gap-3">
      Browse Products
    </a>
    <a [routerLink]="['/about']" class="btn btn-outline-secondary btn-lg px-4">
      Learn More
    </a>
  </div>
</div>

<!-- Featured categories -->
<div class="row g-4 mt-3">
  <div class="col-md-6">
    <div class="card text-white bg-primary">
      <div class="card-body text-center py-4">
        <h5 class="card-title">💻 Electronics</h5>
        <p class="card-text">Latest tech gadgets and accessories</p>
        <!-- routerLink with query params -->
        <a [routerLink]="['/products']"
           [queryParams]="{ category: 'Electronics' }"
           class="btn btn-light btn-sm">
          Shop Electronics
        </a>
      </div>
    </div>
  </div>
  <div class="col-md-6">
    <div class="card text-white bg-success">
      <div class="card-body text-center py-4">
        <h5 class="card-title">🪑 Furniture</h5>
        <p class="card-text">Ergonomic office and home furniture</p>
        <a [routerLink]="['/products']"
           [queryParams]="{ category: 'Furniture' }"
           class="btn btn-light btn-sm">
          Shop Furniture
        </a>
      </div>
    </div>
  </div>
</div>
```

## 5.3 Product List Component

```typescript
// src/app/pages/products/product-list/product-list.component.ts — full file

import { Component, inject, signal, computed, OnInit } from '@angular/core';
import { RouterLink, ActivatedRoute } from '@angular/router';
import { ProductService, Product } from '../../../services/product.service';

@Component({
  selector: 'app-product-list',
  standalone: true,
  imports: [RouterLink],
  templateUrl: './product-list.component.html',
  styleUrl: './product-list.component.css'
})
export class ProductListComponent implements OnInit {
  private productService = inject(ProductService);
  private route = inject(ActivatedRoute);

  // Step 1: Signal to hold displayed products
  products = signal<Product[]>([]);

  // Step 2: Signal for active category filter
  selectedCategory = signal<string | null>(null);

  // Step 3: Computed — all unique categories
  categories = computed(() =>
    [...new Set(this.productService.getAll().map(p => p.category))]
  );

  ngOnInit() {
    // Step 4: Read query params for category filter
    // e.g., /products?category=Electronics
    this.route.queryParamMap.subscribe(params => {
      const cat = params.get('category');
      this.selectedCategory.set(cat);
      this.loadProducts(cat);
    });
  }

  loadProducts(category: string | null) {
    if (category) {
      this.products.set(this.productService.getByCategory(category));
    } else {
      this.products.set(this.productService.getAll());
    }
  }

  // Step 5: Clear filter — navigate to /products (no query params)
  clearFilter() {
    this.selectedCategory.set(null);
    this.products.set(this.productService.getAll());
  }
}
```

```html
<!-- src/app/pages/products/product-list/product-list.component.html — full file -->

<div class="d-flex justify-content-between align-items-center mb-4">
  <h2>Products</h2>
  <!-- Category filter badges -->
  <div class="d-flex gap-2 flex-wrap">
    <button class="btn btn-sm"
            [class.btn-secondary]="!selectedCategory()"
            [class.btn-outline-secondary]="selectedCategory()"
            (click)="clearFilter()">
      All
    </button>
    @for (cat of categories(); track cat) {
      <a class="btn btn-sm"
         [class.btn-primary]="selectedCategory() === cat"
         [class.btn-outline-primary]="selectedCategory() !== cat"
         [routerLink]="['/products']"
         [queryParams]="{ category: cat }">
        {{ cat }}
      </a>
    }
  </div>
</div>

@if (selectedCategory()) {
  <p class="text-muted">
    Showing: <strong>{{ selectedCategory() }}</strong>
    ({{ products().length }} items)
  </p>
}

<div class="row g-4">
  @for (product of products(); track product.id) {
    <div class="col-md-4">
      <div class="card product-card h-100">
        <img [src]="product.imageUrl" class="card-img-top" [alt]="product.name">
        <div class="card-body">
          <span class="badge bg-secondary mb-2">{{ product.category }}</span>
          <h5 class="card-title">{{ product.name }}</h5>
          <p class="card-text text-muted small">{{ product.description }}</p>
        </div>
        <div class="card-footer d-flex justify-content-between align-items-center">
          <strong class="text-primary">${{ product.price }}</strong>
          <div class="d-flex gap-2 align-items-center">
            @if (!product.inStock) {
              <span class="badge bg-danger">Out of Stock</span>
            }
            <!-- Navigate to product detail: /products/1, /products/2, etc. -->
            <a [routerLink]="['/products', product.id]"
               class="btn btn-sm btn-outline-primary">
              View Details
            </a>
          </div>
        </div>
      </div>
    </div>
  } @empty {
    <div class="col-12 text-center py-5">
      <p class="text-muted">No products found in this category.</p>
      <button class="btn btn-primary" (click)="clearFilter()">Show All Products</button>
    </div>
  }
</div>
```

## 5.4 Product Detail Component

> ⭐ **Angular 21 Feature:** We use `input()` signals with `withComponentInputBinding()` to receive route params directly as signal inputs — no `ActivatedRoute` subscription needed!

```typescript
// src/app/pages/products/product-detail/product-detail.component.ts — full file

import { Component, inject, input, computed, OnInit } from '@angular/core';
import { RouterLink } from '@angular/router';
import { ProductService, Product } from '../../../services/product.service';

@Component({
  selector: 'app-product-detail',
  standalone: true,
  imports: [RouterLink],
  templateUrl: './product-detail.component.html',
  styleUrl: './product-detail.component.css'
})
export class ProductDetailComponent {
  private productService = inject(ProductService);

  // ✨ Angular 21: Route parameter 'id' is injected directly as an input signal!
  // This works because we enable withComponentInputBinding() in app.config.ts
  // The route param name MUST match the input() name.
  // Route definition: { path: 'products/:id', ... }
  //                                      ↑ this must match ↓
  id = input<string>('');   // ← signal input, auto-populated from :id param

  // computed() derives the product from the current id signal value
  product = computed(() => {
    const numId = Number(this.id());
    return this.productService.getById(numId);
  });
}
```

```html
<!-- src/app/pages/products/product-detail/product-detail.component.html — full file -->

<!-- Back button -->
<div class="mb-3">
  <a routerLink="/products" class="btn btn-outline-secondary btn-sm">
    ← Back to Products
  </a>
</div>

@if (product(); as p) {
  <div class="row g-4">
    <!-- Product Image -->
    <div class="col-md-5">
      <img [src]="p.imageUrl" [alt]="p.name" class="img-fluid rounded shadow">
    </div>

    <!-- Product Info -->
    <div class="col-md-7">
      <span class="badge bg-secondary mb-2">{{ p.category }}</span>
      <h1 class="mb-2">{{ p.name }}</h1>
      <p class="text-muted">{{ p.description }}</p>

      <hr>

      <div class="d-flex align-items-center gap-3 mb-4">
        <span class="fs-3 fw-bold text-primary">${{ p.price }}</span>
        @if (p.inStock) {
          <span class="badge bg-success fs-6">✓ In Stock</span>
        } @else {
          <span class="badge bg-danger fs-6">✗ Out of Stock</span>
        }
      </div>

      <button class="btn btn-primary btn-lg" [disabled]="!p.inStock">
        Add to Cart
      </button>
    </div>
  </div>
} @else {
  <!-- Product not found state -->
  <div class="text-center py-5">
    <h3 class="text-danger">Product #{{ id() }} not found</h3>
    <p class="text-muted">The product you're looking for doesn't exist.</p>
    <a routerLink="/products" class="btn btn-primary">Browse All Products</a>
  </div>
}
```

## 5.5 About Component

The About page uses **named router outlets** to show two side-by-side sections simultaneously.

```typescript
// src/app/pages/about/about.component.ts — full file

import { Component } from '@angular/core';
import { RouterOutlet, RouterLink } from '@angular/router';

@Component({
  selector: 'app-about',
  standalone: true,
  // Step 1: Import RouterOutlet (for the named outlets in template)
  // Step 2: Import RouterLink (for navigation links)
  imports: [RouterOutlet, RouterLink],
  templateUrl: './about.component.html',
  styleUrl: './about.component.css'
})
export class AboutComponent {}
```

```html
<!-- src/app/pages/about/about.component.html — full file -->

<h2 class="mb-4">About Us</h2>

<div class="card mb-4">
  <div class="card-body">
    <p class="lead">
      Product Store is your premier destination for high-quality office and tech products.
      Founded in 2020, we've helped thousands of professionals create their perfect workspace.
    </p>
  </div>
</div>

<!-- Navigation buttons for the named outlets -->
<div class="d-flex gap-2 mb-4">
  <a class="btn btn-outline-primary"
     [routerLink]="['/about', { outlets: { reviews: ['reviews'], team: ['team'] } }]">
    Show Reviews & Team
  </a>
  <a class="btn btn-outline-secondary"
     [routerLink]="['/about', { outlets: { reviews: null, team: null } }]">
    Hide Sections
  </a>
</div>

<!-- Two named router outlets side by side -->
<!-- Each renders a different child component in its own slot -->
<div class="row g-4">
  <div class="col-md-6">
    <h5 class="text-muted mb-3">Customer Reviews</h5>
    <!-- Named outlet: name="reviews" → renders AboutReviewsComponent -->
    <router-outlet name="reviews"></router-outlet>
  </div>
  <div class="col-md-6">
    <h5 class="text-muted mb-3">Our Team</h5>
    <!-- Named outlet: name="team" → renders AboutTeamComponent -->
    <router-outlet name="team"></router-outlet>
  </div>
</div>
```

## 5.6 About Reviews Component

```typescript
// src/app/pages/about/about-reviews/about-reviews.component.ts — full file

import { Component } from '@angular/core';

@Component({
  selector: 'app-about-reviews',
  standalone: true,
  imports: [],
  template: `
    <div class="card">
      <div class="card-body">
        @for (review of reviews; track review.author) {
          <div class="mb-3 pb-3 border-bottom">
            <div class="d-flex justify-content-between">
              <strong>{{ review.author }}</strong>
              <span class="text-warning">{{ review.stars }}</span>
            </div>
            <p class="text-muted mb-0 small">{{ review.text }}</p>
          </div>
        }
      </div>
    </div>
  `
})
export class AboutReviewsComponent {
  reviews = [
    { author: 'Alice M.', stars: '★★★★★', text: 'Best products I\'ve ever bought. Super fast shipping!' },
    { author: 'James K.', stars: '★★★★☆', text: 'Great quality, the standing desk transformed my work-from-home setup.' },
    { author: 'Sara L.',  stars: '★★★★★', text: 'Customer support is amazing. Highly recommend!' },
  ];
}
```

## 5.7 About Team Component

```typescript
// src/app/pages/about/about-team/about-team.component.ts — full file

import { Component } from '@angular/core';

@Component({
  selector: 'app-about-team',
  standalone: true,
  imports: [],
  template: `
    <div class="card">
      <div class="card-body">
        @for (member of team; track member.name) {
          <div class="d-flex align-items-center mb-3">
            <div class="rounded-circle bg-primary text-white d-flex align-items-center
                        justify-content-center me-3 flex-shrink-0"
                 style="width:40px;height:40px;font-size:1.2rem;">
              {{ member.emoji }}
            </div>
            <div>
              <strong>{{ member.name }}</strong>
              <div class="text-muted small">{{ member.role }}</div>
            </div>
          </div>
        }
      </div>
    </div>
  `
})
export class AboutTeamComponent {
  team = [
    { name: 'Emma Wilson',   role: 'CEO & Founder',    emoji: '👩‍💼' },
    { name: 'David Park',    role: 'Head of Products',  emoji: '👨‍💻' },
    { name: 'Priya Sharma',  role: 'Customer Success',  emoji: '👩‍🎤' },
  ];
}
```

## 5.8 Contact Component

```typescript
// src/app/pages/contact/contact.component.ts — full file

import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-contact',
  standalone: true,
  imports: [],
  template: `
    <h2>Contact Us</h2>
    <div class="row">
      <div class="col-md-6">
        <div class="card">
          <div class="card-body">
            <h5 class="card-title">Get In Touch</h5>
            <p class="text-muted">
              Have a question? We'd love to hear from you. Send us a message
              and we'll respond as soon as possible.
            </p>
            <p>📧 support&#64;productstore.com</p>
            <p>📞 +1 (800) 123-4567</p>
            <p>🏢 123 Commerce St, San Francisco, CA 94102</p>
          </div>
        </div>
      </div>
    </div>
  `
})
export class ContactComponent {}
```

## 5.9 Not Found (404) Component

```typescript
// src/app/pages/not-found/not-found.component.ts — full file

import { Component } from '@angular/core';
import { RouterLink } from '@angular/router';

@Component({
  selector: 'app-not-found',
  standalone: true,
  imports: [RouterLink],
  template: `
    <div class="text-center py-5">
      <h1 class="display-1 text-muted">404</h1>
      <h2>Page Not Found</h2>
      <p class="text-muted">
        The page you're looking for doesn't exist or has been moved.
      </p>
      <a routerLink="/home" class="btn btn-primary">← Go Home</a>
    </div>
  `
})
export class NotFoundComponent {}
```

## 5.10 Admin Dashboard Component

```typescript
// src/app/pages/admin/admin-dashboard/admin-dashboard.component.ts — full file

import { Component, inject } from '@angular/core';
import { RouterLink } from '@angular/router';
import { AuthService } from '../../../services/auth.service';
import { ProductService } from '../../../services/product.service';

@Component({
  selector: 'app-admin-dashboard',
  standalone: true,
  imports: [RouterLink],
  template: `
    <div class="d-flex justify-content-between align-items-center mb-4">
      <h2>⚙️ Admin Dashboard</h2>
      <span class="badge bg-danger">Admin Area</span>
    </div>

    <div class="row g-3 mb-4">
      <div class="col-md-4">
        <div class="card text-center border-primary">
          <div class="card-body">
            <h1 class="text-primary">{{ productCount }}</h1>
            <p class="mb-0">Total Products</p>
          </div>
        </div>
      </div>
      <div class="col-md-4">
        <div class="card text-center border-success">
          <div class="card-body">
            <h1 class="text-success">1</h1>
            <p class="mb-0">Admin Users</p>
          </div>
        </div>
      </div>
      <div class="col-md-4">
        <div class="card text-center border-warning">
          <div class="card-body">
            <h1 class="text-warning">2</h1>
            <p class="mb-0">Categories</p>
          </div>
        </div>
      </div>
    </div>

    <div class="list-group">
      <a routerLink="/admin/users"
         class="list-group-item list-group-item-action d-flex justify-content-between">
        <span>👥 Manage Users</span>
        <span>→</span>
      </a>
    </div>
  `
})
export class AdminDashboardComponent {
  private productService = inject(ProductService);
  productCount = this.productService.getAll().length;
}
```

## 5.11 Admin Users Component

```typescript
// src/app/pages/admin/admin-users/admin-users.component.ts — full file

import { Component } from '@angular/core';
import { RouterLink } from '@angular/router';

@Component({
  selector: 'app-admin-users',
  standalone: true,
  imports: [RouterLink],
  template: `
    <div class="d-flex justify-content-between align-items-center mb-4">
      <h3>👥 User Management</h3>
      <a routerLink="/admin" class="btn btn-outline-secondary btn-sm">← Back to Dashboard</a>
    </div>

    <table class="table table-hover">
      <thead class="table-dark">
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Email</th>
          <th>Role</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        @for (user of users; track user.id) {
          <tr>
            <td>{{ user.id }}</td>
            <td>{{ user.name }}</td>
            <td>{{ user.email }}</td>
            <td>
              <span class="badge"
                    [class.bg-danger]="user.role === 'admin'"
                    [class.bg-secondary]="user.role === 'user'">
                {{ user.role }}
              </span>
            </td>
            <td>
              <button class="btn btn-sm btn-outline-primary">Edit</button>
            </td>
          </tr>
        }
      </tbody>
    </table>
  `
})
export class AdminUsersComponent {
  users = [
    { id: 1, name: 'Admin User',   email: 'admin@store.com', role: 'admin' },
    { id: 2, name: 'Regular User', email: 'user@store.com',  role: 'user'  },
  ];
}
```

---

# Part 6 — Route Configuration

## 6.1 Understanding Route Objects

Every route is an object with these key properties:

```typescript
{
  path: 'products',           // URL segment (no leading slash)
  component: ProductListComponent, // component to render (eager)
  loadComponent: () => ...,   // component to lazy-load
  loadChildren: () => ...,    // child routes file to lazy-load
  children: [...],            // eager child routes
  canActivate: [guard],       // guards that protect this route
  canActivateChild: [guard],  // guards for child routes
  canDeactivate: [guard],     // guard when leaving route
  resolve: { key: resolver }, // pre-load data
  data: { title: 'Products' },// static data attached to route
  title: 'Products',          // page title (browser tab)
  redirectTo: 'home',         // redirect target
  pathMatch: 'full',          // 'full' or 'prefix'
  outlet: 'secondary',        // named outlet target
}
```

## 6.2 `src/app/app.routes.ts` — Complete Route File

Build this incrementally as follows:

### Step 1 — Basic routes

```typescript
// src/app/app.routes.ts — Step 1: Basic routes

import { Routes } from '@angular/router';

export const routes: Routes = [
  // Default redirect: '' (root) → /home
  { path: '', redirectTo: 'home', pathMatch: 'full' },

  // Eager-loaded routes (component downloaded with the initial bundle)
  { path: 'home',    component: HomeComponent },
  { path: 'contact', component: ComponentComponent },

  // Wildcard: must be LAST — catches all unmatched routes
  { path: '**', component: NotFoundComponent }
];
```

> ⚠️ `pathMatch: 'full'` on the empty `''` path is critical. Without it, `''` would match every URL (it's a prefix of everything).

### Step 2 — Add imports for eager components

```typescript
// src/app/app.routes.ts — Step 2: Add imports

import { Routes } from '@angular/router';

// Import components that are eagerly loaded
import { HomeComponent }     from './pages/home/home.component';
import { ContactComponent }  from './pages/contact/contact.component';
import { NotFoundComponent } from './pages/not-found/not-found.component';

export const routes: Routes = [
  { path: '',        redirectTo: 'home', pathMatch: 'full' },
  { path: 'home',    component: HomeComponent },
  { path: 'contact', component: ContactComponent },
  { path: '**',      component: NotFoundComponent }
];
```

### Step 3 — Add products routes (with :id parameter)

```typescript
// src/app/app.routes.ts — Step 3: Add product routes

import { Routes } from '@angular/router';
import { HomeComponent }          from './pages/home/home.component';
import { ContactComponent }       from './pages/contact/contact.component';
import { NotFoundComponent }      from './pages/not-found/not-found.component';
// ↓ NEW imports
import { ProductListComponent }   from './pages/products/product-list/product-list.component';
import { ProductDetailComponent } from './pages/products/product-detail/product-detail.component';

export const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },

  { path: 'home',    component: HomeComponent },
  { path: 'contact', component: ContactComponent },

  // ↓ NEW: product routes
  // /products         → shows the list
  // /products/1       → shows product with id=1
  // /products/2       → shows product with id=2
  { path: 'products',    component: ProductListComponent },
  { path: 'products/:id', component: ProductDetailComponent },  // :id is a route param

  { path: '**', component: NotFoundComponent }
];
```

### Step 4 — Add about route with named outlets and child routes

```typescript
// src/app/app.routes.ts — Step 4: Add about route + named outlet children

import { Routes } from '@angular/router';
import { HomeComponent }          from './pages/home/home.component';
import { ContactComponent }       from './pages/contact/contact.component';
import { NotFoundComponent }      from './pages/not-found/not-found.component';
import { ProductListComponent }   from './pages/products/product-list/product-list.component';
import { ProductDetailComponent } from './pages/products/product-detail/product-detail.component';
// ↓ NEW imports
import { AboutComponent }         from './pages/about/about.component';
import { AboutReviewsComponent }  from './pages/about/about-reviews/about-reviews.component';
import { AboutTeamComponent }     from './pages/about/about-team/about-team.component';

export const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home',    component: HomeComponent },
  { path: 'contact', component: ContactComponent },
  { path: 'products',     component: ProductListComponent },
  { path: 'products/:id', component: ProductDetailComponent },

  // ↓ NEW: About page with named outlet children
  {
    path: 'about',
    component: AboutComponent,
    children: [
      // These render in NAMED outlets inside AboutComponent's template
      // URL: /about/(reviews:reviews//team:team)
      { path: 'reviews', component: AboutReviewsComponent, outlet: 'reviews' },
      { path: 'team',    component: AboutTeamComponent,    outlet: 'team'    },
    ]
  },

  { path: '**', component: NotFoundComponent }
];
```

### Step 5 — Add lazy-loaded admin section

```typescript
// src/app/app.routes.ts — Step 5: Final version with lazy loading + guards

import { Routes } from '@angular/router';
import { HomeComponent }          from './pages/home/home.component';
import { ContactComponent }       from './pages/contact/contact.component';
import { NotFoundComponent }      from './pages/not-found/not-found.component';
import { ProductListComponent }   from './pages/products/product-list/product-list.component';
import { ProductDetailComponent } from './pages/products/product-detail/product-detail.component';
import { AboutComponent }         from './pages/about/about.component';
import { AboutReviewsComponent }  from './pages/about/about-reviews/about-reviews.component';
import { AboutTeamComponent }     from './pages/about/about-team/about-team.component';
// ↓ NEW: import guards
import { authGuard }              from './guards/auth.guard';
import { adminGuard }             from './guards/admin.guard';

export const routes: Routes = [
  // ── DEFAULT REDIRECT ──────────────────────────────────────────────
  { path: '', redirectTo: 'home', pathMatch: 'full' },

  // ── EAGER ROUTES (part of main bundle) ────────────────────────────
  {
    path: 'home',
    component: HomeComponent,
    title: 'Home — Product Store'          // ← sets browser tab title
  },
  {
    path: 'contact',
    component: ContactComponent,
    title: 'Contact Us'
  },

  // ── PRODUCT ROUTES ────────────────────────────────────────────────
  {
    path: 'products',
    component: ProductListComponent,
    title: 'Products'
  },
  {
    path: 'products/:id',
    component: ProductDetailComponent,
    title: 'Product Details'
  },

  // ── ABOUT (with named outlet children) ────────────────────────────
  {
    path: 'about',
    component: AboutComponent,
    title: 'About Us',
    children: [
      { path: 'reviews', component: AboutReviewsComponent, outlet: 'reviews' },
      { path: 'team',    component: AboutTeamComponent,    outlet: 'team'    },
    ]
  },

  // ── LAZY-LOADED ADMIN SECTION (with guards) ────────────────────────
  {
    path: 'admin',
    // loadChildren points to the admin routes file — code-split automatically!
    loadChildren: () => import('./pages/admin/admin.routes')
                          .then(m => m.adminRoutes),
    canActivate: [authGuard, adminGuard],  // both guards must pass
    title: 'Admin'
  },

  // ── WILDCARD (must be last!) ───────────────────────────────────────
  {
    path: '**',
    component: NotFoundComponent,
    title: '404 — Not Found'
  }
];
```

---

# Part 7 — Child Routes & Nested Outlets

## 7.1 What Are Child Routes?

Child routes render inside a **parent component's** `<router-outlet>`, not the root one.

```
URL: /admin/users

Root <router-outlet>        ← renders AdminComponent (the shell)
  └─ AdminComponent
       └─ child <router-outlet>  ← renders AdminUsersComponent here
```

## 7.2 `src/app/pages/admin/admin.routes.ts`

This is the admin section's own routes file — it will be lazy-loaded.

```typescript
// src/app/pages/admin/admin.routes.ts — full file

import { Routes } from '@angular/router';

// These imports are local to the admin chunk
import { AdminDashboardComponent } from './admin-dashboard/admin-dashboard.component';
import { AdminUsersComponent }     from './admin-users/admin-users.component';

export const adminRoutes: Routes = [
  // When the URL is /admin → redirect to /admin/dashboard
  { path: '', redirectTo: 'dashboard', pathMatch: 'full' },

  // /admin/dashboard
  {
    path: 'dashboard',
    component: AdminDashboardComponent,
    title: 'Admin Dashboard'
  },

  // /admin/users
  {
    path: 'users',
    component: AdminUsersComponent,
    title: 'User Management'
  },
];
```

> 💡 **Why a separate routes file?** When you use `loadChildren`, Angular creates a **separate JavaScript chunk** for everything imported in that file. The admin section's code is only downloaded when a user first navigates to `/admin`. This is **lazy loading** — it makes the initial bundle smaller and app startup faster.

## 7.3 Create an Admin Shell Component

For child routes to work, the parent (admin) needs its own `<router-outlet>`. Let's create an admin layout:

```typescript
// src/app/pages/admin/admin.component.ts — CREATE this new file

import { Component } from '@angular/core';
import { RouterOutlet, RouterLink, RouterLinkActive } from '@angular/router';

@Component({
  selector: 'app-admin',
  standalone: true,
  imports: [RouterOutlet, RouterLink, RouterLinkActive],
  template: `
    <div class="row">
      <!-- Sidebar -->
      <div class="col-md-2">
        <div class="list-group">
          <a routerLink="/admin/dashboard"
             routerLinkActive="active"
             class="list-group-item list-group-item-action">
            📊 Dashboard
          </a>
          <a routerLink="/admin/users"
             routerLinkActive="active"
             class="list-group-item list-group-item-action">
            👥 Users
          </a>
        </div>
      </div>
      <!-- Content -->
      <div class="col-md-10">
        <!-- Child routes render here -->
        <router-outlet></router-outlet>
      </div>
    </div>
  `
})
export class AdminComponent {}
```

Now update `admin.routes.ts` to use this shell:

```typescript
// src/app/pages/admin/admin.routes.ts — UPDATED with shell component

import { Routes } from '@angular/router';
import { AdminComponent }          from './admin.component';  // ← NEW shell
import { AdminDashboardComponent } from './admin-dashboard/admin-dashboard.component';
import { AdminUsersComponent }     from './admin-users/admin-users.component';

export const adminRoutes: Routes = [
  {
    path: '',
    component: AdminComponent,  // ← shell provides the sidebar + child router-outlet
    children: [
      { path: '',          redirectTo: 'dashboard', pathMatch: 'full' },
      { path: 'dashboard', component: AdminDashboardComponent, title: 'Admin Dashboard' },
      { path: 'users',     component: AdminUsersComponent,     title: 'User Management' },
    ]
  }
];
```

---

# Part 8 — Named Router Outlets

## 8.1 What Are Named Outlets?

Named outlets let you render **multiple components simultaneously** in different regions of a parent template.

```
URL: /about/(reviews:reviews//team:team)
              ↑               ↑
              outlet "reviews" shows AboutReviewsComponent
                              outlet "team" shows AboutTeamComponent

Template:
  <router-outlet name="reviews"></router-outlet>  ← left column
  <router-outlet name="team"></router-outlet>      ← right column
```

## 8.2 URL Format for Named Outlets

```
/parent/(outletName:path//outletName2:path2)

Examples:
  /about/(reviews:reviews)                   ← only reviews outlet
  /about/(reviews:reviews//team:team)        ← both outlets active
  /about/(reviews:reviews//team:null)        ← reviews open, team closed
```

## 8.3 RouterLink Syntax for Named Outlets

```html
<!-- Activate both named outlets -->
<a [routerLink]="['/about', { outlets: { reviews: ['reviews'], team: ['team'] } }]">
  Show Both
</a>

<!-- Deactivate (clear) an outlet by setting it to null -->
<a [routerLink]="['/about', { outlets: { reviews: null, team: null } }]">
  Hide Both
</a>

<!-- Activate only one outlet -->
<a [routerLink]="['/about', { outlets: { reviews: ['reviews'] } }]">
  Show Reviews Only
</a>
```

The route config for named outlets uses `outlet: 'name'`:

```typescript
// In app.routes.ts — about children
children: [
  { path: 'reviews', component: AboutReviewsComponent, outlet: 'reviews' },
  { path: 'team',    component: AboutTeamComponent,    outlet: 'team'    },
]
```

---

# Part 9 — Route Parameters & Signal Inputs

## 9.1 Three Types of Route Data

|Type|Example URL|How to define|How to read|
|---|---|---|---|
|**Route param**|`/products/42`|`path: 'products/:id'`|`input<string>('id')` or `ActivatedRoute`|
|**Query param**|`/products?category=Electronics`|No special config|`ActivatedRoute.queryParamMap`|
|**Route data**|any URL|`data: { role: 'admin' }`|`ActivatedRoute.data`|

## 9.2 Reading Route Params — Angular 21 Way

In Angular 21, with `withComponentInputBinding()` enabled in `app.config.ts`, route params, query params, and resolver data are **automatically bound to component inputs**.

### Step 1 — Enable `withComponentInputBinding()` in app.config.ts

```typescript
// src/app/app.config.ts — UPDATED: add withComponentInputBinding

import { ApplicationConfig } from '@angular/core';
import { provideRouter, withComponentInputBinding } from '@angular/router'; // ← add withComponentInputBinding

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withComponentInputBinding()  // ← enables param → input() binding
    )
  ]
};
```

### Step 2 — Use `input()` in component (already done in ProductDetailComponent)

```typescript
// Pattern: route param name must match input() name exactly

// Route: { path: 'products/:id', component: ProductDetailComponent }
//                          ↑
// Component:
export class ProductDetailComponent {
  id = input<string>('');  // ← 'id' matches ':id' in the route path
  //   ↑ this signal is automatically populated with the current route param!
}
```

### Step 3 — Reading query params as signal inputs

```typescript
// Route: { path: 'products', component: ProductListComponent }
// URL: /products?category=Electronics&sort=price

export class ProductListComponent {
  // Query params also bind to inputs automatically!
  category = input<string | null>(null);  // ← bound from ?category=...
  sort     = input<string | null>(null);  // ← bound from ?sort=...
}
```

## 9.3 The Old Way (still valid with ActivatedRoute)

For cases where signal inputs don't work (e.g., the component isn't directly routed), you can still use `ActivatedRoute`:

```typescript
// Alternative: subscribe to ActivatedRoute params
import { ActivatedRoute } from '@angular/router';

export class SomeComponent implements OnInit {
  private route = inject(ActivatedRoute);
  productId = signal<number>(0);

  ngOnInit() {
    // Observable-based — good for components that re-use across param changes
    this.route.paramMap.subscribe(params => {
      this.productId.set(Number(params.get('id')));
    });

    // Snapshot-based — only reads once (not reactive)
    const id = this.route.snapshot.paramMap.get('id');
  }
}
```

---

# Part 10 — Programmatic Navigation

## 10.1 Using the Router Service

Sometimes you need to navigate from code (after a form submit, after a login, etc.).

```typescript
// src/app/pages/products/product-list/product-list.component.ts
// ✏️ ADD programmatic navigation to existing component

import { Component, inject, signal, computed, OnInit } from '@angular/core';
import { RouterLink, ActivatedRoute, Router } from '@angular/router'; // ← add Router
import { ProductService, Product } from '../../../services/product.service';

@Component({
  selector: 'app-product-list',
  standalone: true,
  imports: [RouterLink],
  templateUrl: './product-list.component.html',
})
export class ProductListComponent implements OnInit {
  private productService = inject(ProductService);
  private route  = inject(ActivatedRoute);
  private router = inject(Router);           // ← inject Router

  products = signal<Product[]>([]);
  selectedCategory = signal<string | null>(null);
  categories = computed(() =>
    [...new Set(this.productService.getAll().map(p => p.category))]
  );

  ngOnInit() {
    this.route.queryParamMap.subscribe(params => {
      const cat = params.get('category');
      this.selectedCategory.set(cat);
      this.loadProducts(cat);
    });
  }

  loadProducts(category: string | null) {
    if (category) {
      this.products.set(this.productService.getByCategory(category));
    } else {
      this.products.set(this.productService.getAll());
    }
  }

  clearFilter() {
    // navigate() takes the same array format as [routerLink]
    this.router.navigate(['/products']);
  }

  // ✨ NEW: Navigate to product detail on card click
  goToProduct(id: number) {
    // Absolute navigation
    this.router.navigate(['/products', id]);
  }

  // ✨ NEW: Navigate with query params
  filterByCategory(category: string) {
    this.router.navigate(['/products'], {
      queryParams: { category },
      queryParamsHandling: 'merge'  // 'merge' keeps existing params, 'preserve' keeps all
    });
  }

  // ✨ NEW: Navigate and preserve browser history
  goBack() {
    // navigateByUrl() takes a string URL
    this.router.navigateByUrl('/products');
  }

  // ✨ NEW: Navigate with replaceUrl (replaces current history entry)
  redirectToHome() {
    this.router.navigate(['/home'], { replaceUrl: true });
  }
}
```

## 10.2 Navigation Options Reference

```typescript
// All options for router.navigate() and router.navigateByUrl()

this.router.navigate(['/path', paramValue], {
  queryParams:        { key: 'value' },     // add/set query params
  queryParamsHandling: 'merge',             // 'merge' | 'preserve' | '' (default clears)
  fragment:           'section-id',         // adds #section-id to URL
  replaceUrl:         true,                 // replace current history entry (no back button)
  skipLocationChange: true,                 // navigate without changing the URL
  relativeTo:         this.route,           // navigate relative to current route
  state:              { fromPage: 'list' }  // pass hidden state (not in URL)
});

// Relative navigation example:
// Current URL: /products/5
this.router.navigate(['..', '6'], { relativeTo: this.route });
// Navigates to: /products/6
```

---

# Part 11 — Lazy Loading

## 11.1 What is Lazy Loading?

By default, all components in `app.routes.ts` are bundled in the **main chunk** downloaded at startup. With lazy loading, each section gets its own chunk downloaded **on demand**.

```
Eager loading:                  Lazy loading:
                                
main.js (2MB):                  main.js (200KB):
  HomeComponent                   HomeComponent
  ProductListComponent            ProductListComponent
  ProductDetailComponent          ProductDetailComponent
  AdminDashboardComponent         ← NOT included in main
  AdminUsersComponent             ← NOT included in main
  ...all components               
                                admin-chunk.js (50KB):
                                  AdminDashboardComponent  ← downloaded when user
                                  AdminUsersComponent         first visits /admin
```

## 11.2 `loadComponent` — Lazy Load a Single Component

```typescript
// In app.routes.ts — use loadComponent instead of component

// BEFORE (eager):
{ path: 'contact', component: ContactComponent }

// AFTER (lazy):
{
  path: 'contact',
  loadComponent: () => import('./pages/contact/contact.component')
                         .then(m => m.ContactComponent)
}

// Shorthand (Angular 21 allows default exports):
{
  path: 'contact',
  loadComponent: () => import('./pages/contact/contact.component')
  // works if ContactComponent is the default export
}
```

## 11.3 `loadChildren` — Lazy Load an Entire Feature Route Set

```typescript
// In app.routes.ts

// Lazy-loads the entire admin section when user first visits /admin
{
  path: 'admin',
  loadChildren: () => import('./pages/admin/admin.routes')
                        .then(m => m.adminRoutes)
}
```

## 11.4 Preloading Strategy

By default, lazy chunks are only loaded when needed. You can preload them in the background after the initial load:

```typescript
// src/app/app.config.ts — UPDATED: add preloading

import { ApplicationConfig } from '@angular/core';
import {
  provideRouter,
  withComponentInputBinding,
  withPreloading,
  PreloadAllModules  // ← preloads all lazy routes in the background
} from '@angular/router';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withComponentInputBinding(),
      withPreloading(PreloadAllModules)  // ← preload after first paint
    )
  ]
};
```

---

# Part 12 — Route Guards

## 12.1 Guard Types in Angular 21

|Guard|When it runs|Use case|
|---|---|---|
|`CanActivateFn`|Before entering a route|Auth check, redirect to login|
|`CanActivateChildFn`|Before entering any child route|Protect all children at once|
|`CanDeactivateFn`|Before leaving a route|Unsaved changes warning|
|`CanMatchFn`|Before matching a route (lazy context)|Show different component per role|

> ⚠️ **Angular 21:** Class-based guards (`implements CanActivate`) are **deprecated**. Always use functional guards.

## 12.2 `src/app/guards/auth.guard.ts`

```typescript
// src/app/guards/auth.guard.ts — full file

import { inject }                       from '@angular/core';
import { CanActivateFn, Router,
         ActivatedRouteSnapshot,
         RouterStateSnapshot }          from '@angular/router';
import { AuthService }                  from '../services/auth.service';

// Functional guard — just a typed function
export const authGuard: CanActivateFn = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot
) => {
  // Step 1: Use inject() to get dependencies (works in injection context)
  const authService = inject(AuthService);
  const router      = inject(Router);

  // Step 2: Check if logged in
  if (authService.isLoggedIn()) {
    return true;  // ✅ allow navigation
  }

  // Step 3: Not logged in — redirect to login and pass the intended URL
  // The 'returnUrl' query param lets the login page redirect back after login
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
  // ↑ returning a UrlTree redirects the user instead of just blocking them
};
```

## 12.3 `src/app/guards/admin.guard.ts`

```typescript
// src/app/guards/admin.guard.ts — full file

import { inject }                  from '@angular/core';
import { CanActivateFn, Router }   from '@angular/router';
import { AuthService }             from '../services/auth.service';

export const adminGuard: CanActivateFn = () => {
  const authService = inject(AuthService);
  const router      = inject(Router);

  // Allow only admin-role users
  if (authService.isAdmin()) {
    return true;
  }

  // Logged in but not admin → redirect to forbidden page (or home)
  return router.createUrlTree(['/home']);
};
```

## 12.4 Applying Guards to Routes

```typescript
// In app.routes.ts — guards are applied in the canActivate array
{
  path: 'admin',
  loadChildren: () => import('./pages/admin/admin.routes').then(m => m.adminRoutes),

  // Both guards run in order. If any returns false/UrlTree, navigation stops.
  canActivate: [authGuard, adminGuard],
}
```

## 12.5 CanDeactivate Guard — Unsaved Changes Warning

```typescript
// src/app/guards/unsaved-changes.guard.ts — CREATE this file

import { CanDeactivateFn } from '@angular/router';

// Step 1: Define the interface that "protected" components must implement
export interface HasUnsavedChanges {
  hasUnsavedChanges(): boolean;
}

// Step 2: The guard — asks the component if it's safe to leave
export const unsavedChangesGuard: CanDeactivateFn<HasUnsavedChanges> = (
  component: HasUnsavedChanges
) => {
  if (component.hasUnsavedChanges()) {
    return confirm('You have unsaved changes. Are you sure you want to leave?');
  }
  return true;
};
```

```typescript
// Example component that uses the CanDeactivate guard
// src/app/pages/admin/admin-users/admin-users.component.ts
// ✏️ ADD HasUnsavedChanges interface to the component

import { Component, signal } from '@angular/core';
import { HasUnsavedChanges } from '../../../guards/unsaved-changes.guard';

export class AdminUsersComponent implements HasUnsavedChanges {
  isDirty = signal(false);  // set to true when form is modified

  // The guard calls this method before allowing navigation away
  hasUnsavedChanges(): boolean {
    return this.isDirty();
  }
}
```

```typescript
// Apply CanDeactivate guard to a route:
{
  path: 'users',
  component: AdminUsersComponent,
  canDeactivate: [unsavedChangesGuard]
}
```

## 12.6 Create a Login Page Component

Since our guards redirect to `/login`, we need that page:

```typescript
// src/app/pages/login/login.component.ts — CREATE this file

import { Component, inject, signal } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';
import { AuthService } from '../../services/auth.service';

@Component({
  selector: 'app-login',
  standalone: true,
  imports: [],
  template: `
    <div class="row justify-content-center mt-5">
      <div class="col-md-4">
        <div class="card shadow">
          <div class="card-header bg-dark text-white text-center">
            <h4>🔐 Login</h4>
          </div>
          <div class="card-body p-4">

            @if (error()) {
              <div class="alert alert-danger">{{ error() }}</div>
            }

            <div class="mb-3">
              <label class="form-label">Email</label>
              <input type="email" class="form-control"
                     [value]="email()"
                     (input)="email.set($any($event.target).value)"
                     placeholder="admin@store.com or user@store.com">
            </div>

            <div class="mb-3">
              <label class="form-label">Password</label>
              <input type="password" class="form-control"
                     [value]="password()"
                     (input)="password.set($any($event.target).value)"
                     placeholder="admin123 or user123">
            </div>

            <button class="btn btn-primary w-100" (click)="login()">
              Login
            </button>

            <div class="mt-3 text-muted small text-center">
              <strong>Test credentials:</strong><br>
              admin@store.com / admin123 (Admin)<br>
              user@store.com / user123 (User)
            </div>

          </div>
        </div>
      </div>
    </div>
  `
})
export class LoginComponent {
  private authService  = inject(AuthService);
  private router       = inject(Router);
  private route        = inject(ActivatedRoute);

  email    = signal('');
  password = signal('');
  error    = signal('');

  login() {
    this.error.set('');
    const success = this.authService.login(this.email(), this.password());

    if (success) {
      // Navigate to returnUrl if provided (set by authGuard), or home
      const returnUrl = this.route.snapshot.queryParamMap.get('returnUrl') || '/home';
      this.router.navigateByUrl(returnUrl);
    } else {
      this.error.set('Invalid email or password. Try the test credentials below.');
    }
  }
}
```

Add the login route to `app.routes.ts`:

```typescript
// src/app/app.routes.ts — ✏️ ADD login route

import { LoginComponent } from './pages/login/login.component'; // ← add import

export const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home',    component: HomeComponent, title: 'Home — Product Store' },
  { path: 'login',   component: LoginComponent, title: 'Login' },  // ← ADD
  { path: 'contact', component: ContactComponent, title: 'Contact Us' },
  // ... rest of routes
];
```

---

# Part 13 — Route Resolvers

## 13.1 What is a Resolver?

A resolver **pre-fetches data** before a route is activated. The component only renders when the data is ready — no loading spinners needed inside the component.

```
Without resolver:               With resolver:
  Navigate to /products/5         Navigate to /products/5
  Component renders               Resolver runs (fetches product)
  Shows loading spinner           ← user sees spinner here (router level)
  Data arrives                    Component renders with data ready
  Shows product                   Shows product immediately
```

## 13.2 `src/app/resolvers/product-detail.resolver.ts`

```typescript
// src/app/resolvers/product-detail.resolver.ts — full file

import { inject }                    from '@angular/core';
import { ResolveFn,
         ActivatedRouteSnapshot,
         RouterStateSnapshot,
         Router }                    from '@angular/router';
import { ProductService, Product }   from '../services/product.service';

// ResolveFn<T> is a function that returns T (or Observable<T> / Promise<T>)
export const productDetailResolver: ResolveFn<Product | null> = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot
) => {
  const productService = inject(ProductService);
  const router         = inject(Router);

  // Step 1: Get the :id param from the route snapshot
  const id = Number(route.paramMap.get('id'));

  // Step 2: Try to find the product
  const product = productService.getById(id);

  if (product) {
    return product;    // resolver can return a value directly
  }

  // Step 3: Not found — redirect to products list
  router.navigate(['/products']);
  return null;
};
```

## 13.3 Attach Resolver to a Route

```typescript
// In app.routes.ts — ✏️ UPDATE product detail route

import { productDetailResolver } from './resolvers/product-detail.resolver'; // ← add

{
  path: 'products/:id',
  component: ProductDetailComponent,
  title: 'Product Details',
  resolve: {
    product: productDetailResolver  // ← key 'product' is how we access it in the component
  }
}
```

## 13.4 Access Resolver Data in Component

With `withComponentInputBinding()`, resolver data binds directly to an `input()`:

```typescript
// src/app/pages/products/product-detail/product-detail.component.ts
// ✏️ UPDATED: use resolver data instead of computing it

import { Component, input, computed } from '@angular/core';
import { RouterLink } from '@angular/router';
import { Product } from '../../../services/product.service';

@Component({
  selector: 'app-product-detail',
  standalone: true,
  imports: [RouterLink],
  templateUrl: './product-detail.component.html',
})
export class ProductDetailComponent {
  // Route param (still available)
  id = input<string>('');

  // Resolver data — key matches the resolve object key in the route definition
  // resolve: { product: productDetailResolver }
  //                ↑ must match this input name
  product = input<Product | null>(null);
}
```

```html
<!-- product-detail.component.html — UPDATED -->

@if (product(); as p) {
  <!-- render product data directly — no loading state needed! -->
  <h1>{{ p.name }}</h1>
  ...
}
```

---

# Part 14 — Route Data & Page Titles

## 14.1 Static Route Data

You can attach arbitrary static data to any route:

```typescript
// In app.routes.ts
{
  path: 'admin',
  data: {
    role: 'admin',
    breadcrumb: 'Admin Panel',
    icon: '⚙️'
  },
  // ...
}
```

```typescript
// Read it in a component or guard
const data = inject(ActivatedRoute).snapshot.data;
console.log(data['breadcrumb']); // 'Admin Panel'
```

## 14.2 Dynamic Page Titles

```typescript
// Static title (string)
{ path: 'home', component: HomeComponent, title: 'Home — Product Store' }

// Dynamic title (resolver function)
{
  path: 'products/:id',
  component: ProductDetailComponent,
  title: (route: ActivatedRouteSnapshot) => {
    // Return a dynamic title based on the route param
    return `Product #${route.paramMap.get('id')} — Product Store`;
  }
}
```

## 14.3 Custom Title Strategy

For full control over browser tab titles across the entire app:

```typescript
// src/app/services/title.strategy.ts — CREATE this file

import { Injectable } from '@angular/core';
import { RouterStateSnapshot, TitleStrategy } from '@angular/router';
import { Title } from '@angular/platform-browser';

@Injectable({ providedIn: 'root' })
export class AppTitleStrategy extends TitleStrategy {
  constructor(private title: Title) {
    super();
  }

  override updateTitle(snapshot: RouterStateSnapshot): void {
    const pageTitle = this.buildTitle(snapshot);
    if (pageTitle) {
      this.title.setTitle(`${pageTitle} | Product Store`);
    } else {
      this.title.setTitle('Product Store');
    }
  }
}
```

```typescript
// src/app/app.config.ts — ✏️ register the title strategy

import { ApplicationConfig } from '@angular/core';
import { provideRouter, withComponentInputBinding,
         withPreloading, PreloadAllModules, TitleStrategy } from '@angular/router';
import { AppTitleStrategy } from './services/title.strategy';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withComponentInputBinding(), withPreloading(PreloadAllModules)),
    { provide: TitleStrategy, useClass: AppTitleStrategy }  // ← register custom strategy
  ]
};
```

---

# Part 15 — Wildcard & 404 Routes

The wildcard route `**` catches any URL that doesn't match earlier routes.

```typescript
// Rule: wildcard MUST be the last route in the array
// Angular checks routes top to bottom, first match wins.

export const routes: Routes = [
  { path: '',         redirectTo: 'home', pathMatch: 'full' },
  { path: 'home',     component: HomeComponent },
  { path: 'products', component: ProductListComponent },
  // ... other routes ...
  
  // ✅ Last route — catches everything else
  { path: '**', component: NotFoundComponent, title: '404 — Not Found' }
];
```

> ⚠️ If you put `{ path: '**' }` before other routes, those routes will never be reached!

---

# Part 16 — Router Events

## 16.1 Listening to Navigation Events

Angular Router emits events throughout the navigation lifecycle. You can listen to them for analytics, loading indicators, etc.

```typescript
// src/app/app.component.ts — ✏️ UPDATED with router events

import { Component, inject, signal } from '@angular/core';
import { RouterOutlet, Router, NavigationStart,
         NavigationEnd, NavigationError,
         NavigationCancel, Event as RouterEvent } from '@angular/router';
import { NavbarComponent } from './shared/navbar/navbar.component';
import { filter } from 'rxjs';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, NavbarComponent],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  private router = inject(Router);

  // Signal to track loading state
  isNavigating = signal(false);

  constructor() {
    // Subscribe to router events
    this.router.events.subscribe((event: RouterEvent) => {

      if (event instanceof NavigationStart) {
        // Navigation has started
        this.isNavigating.set(true);
        console.log('Navigating to:', event.url);
      }

      if (event instanceof NavigationEnd) {
        // Navigation completed successfully
        this.isNavigating.set(false);
        console.log('Arrived at:', event.urlAfterRedirects);
        // Good place to send analytics page view
      }

      if (event instanceof NavigationCancel) {
        // Navigation was cancelled (e.g., guard returned false)
        this.isNavigating.set(false);
        console.log('Navigation cancelled:', event.reason);
      }

      if (event instanceof NavigationError) {
        // Navigation failed (e.g., lazy chunk failed to load)
        this.isNavigating.set(false);
        console.error('Navigation error:', event.error);
      }
    });
  }
}
```

```html
<!-- src/app/app.component.html — ✏️ UPDATED: add navigation indicator -->

<app-navbar></app-navbar>

<!-- Show thin progress bar during navigation -->
@if (isNavigating()) {
  <div class="progress" style="height: 3px; border-radius: 0;">
    <div class="progress-bar progress-bar-striped progress-bar-animated
                bg-primary w-100">
    </div>
  </div>
}

<main class="container mt-4">
  <router-outlet></router-outlet>
</main>

<footer class="bg-light text-center py-3 mt-5 border-top">
  <small class="text-muted">© 2025 Product Store — Built with Angular 21</small>
</footer>
```

---

# Part 17 — Complete Routes File Reference

Here is the **final, complete `app.routes.ts`** for the entire lab:

```typescript
// src/app/app.routes.ts — FINAL complete file

import { Routes } from '@angular/router';

// ── Eagerly-loaded page components ────────────────────────────────────────
import { HomeComponent }          from './pages/home/home.component';
import { ContactComponent }       from './pages/contact/contact.component';
import { NotFoundComponent }      from './pages/not-found/not-found.component';
import { LoginComponent }         from './pages/login/login.component';
import { ProductListComponent }   from './pages/products/product-list/product-list.component';
import { ProductDetailComponent } from './pages/products/product-detail/product-detail.component';
import { AboutComponent }         from './pages/about/about.component';
import { AboutReviewsComponent }  from './pages/about/about-reviews/about-reviews.component';
import { AboutTeamComponent }     from './pages/about/about-team/about-team.component';

// ── Guards ────────────────────────────────────────────────────────────────
import { authGuard }  from './guards/auth.guard';
import { adminGuard } from './guards/admin.guard';

// ── Resolvers ─────────────────────────────────────────────────────────────
import { productDetailResolver } from './resolvers/product-detail.resolver';

export const routes: Routes = [

  // ── DEFAULT REDIRECT ────────────────────────────────────────────────────
  {
    path: '',
    redirectTo: 'home',
    pathMatch: 'full'   // REQUIRED for empty path redirects
  },

  // ── AUTHENTICATION ───────────────────────────────────────────────────────
  {
    path: 'login',
    component: LoginComponent,
    title: 'Login'
  },

  // ── HOME ─────────────────────────────────────────────────────────────────
  {
    path: 'home',
    component: HomeComponent,
    title: 'Home — Product Store'
  },

  // ── PRODUCTS ─────────────────────────────────────────────────────────────
  {
    path: 'products',
    component: ProductListComponent,
    title: 'Products'
    // Note: no guards on list page — public access
  },
  {
    path: 'products/:id',          // :id is a route parameter
    component: ProductDetailComponent,
    title: 'Product Details',
    resolve: {
      product: productDetailResolver  // pre-fetches product data
    }
  },

  // ── ABOUT (with named outlet children) ──────────────────────────────────
  {
    path: 'about',
    component: AboutComponent,
    title: 'About Us',
    children: [
      {
        path: 'reviews',
        component: AboutReviewsComponent,
        outlet: 'reviews'   // renders in <router-outlet name="reviews">
      },
      {
        path: 'team',
        component: AboutTeamComponent,
        outlet: 'team'      // renders in <router-outlet name="team">
      }
    ]
  },

  // ── CONTACT ───────────────────────────────────────────────────────────────
  {
    path: 'contact',
    component: ContactComponent,
    title: 'Contact Us'
  },

  // ── ADMIN (lazy-loaded, guarded) ─────────────────────────────────────────
  {
    path: 'admin',
    loadChildren: () =>
      import('./pages/admin/admin.routes').then(m => m.adminRoutes),
    canActivate: [authGuard, adminGuard],  // must be logged in AND admin role
    title: 'Admin'
  },

  // ── WILDCARD — must be LAST ───────────────────────────────────────────────
  {
    path: '**',
    component: NotFoundComponent,
    title: '404 — Not Found'
  }
];
```

## Final `app.config.ts`

```typescript
// src/app/app.config.ts — FINAL complete file

import { ApplicationConfig } from '@angular/core';
import {
  provideRouter,
  withComponentInputBinding,
  withPreloading,
  PreloadAllModules,
  TitleStrategy
} from '@angular/router';

import { routes }           from './app.routes';
import { AppTitleStrategy } from './services/title.strategy';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withComponentInputBinding(),       // route params → input() signals
      withPreloading(PreloadAllModules)   // preload lazy chunks in background
    ),
    {
      provide: TitleStrategy,
      useClass: AppTitleStrategy         // custom title: "Page | Product Store"
    }
  ]
};
```

---

## 📌 Quick Reference — Angular 21 Routing Cheat Sheet

### Route Definition Patterns

```typescript
// Redirect
{ path: '', redirectTo: 'home', pathMatch: 'full' }

// Eager component
{ path: 'home', component: HomeComponent }

// Route parameter
{ path: 'products/:id', component: ProductDetailComponent }

// Query params (no route config needed — read in component)
// URL: /products?category=Electronics

// Lazy component
{ path: 'contact', loadComponent: () => import('./pages/contact/contact.component').then(m => m.ContactComponent) }

// Lazy child routes
{ path: 'admin', loadChildren: () => import('./pages/admin/admin.routes').then(m => m.adminRoutes) }

// With guard
{ path: 'admin', loadChildren: ..., canActivate: [authGuard, adminGuard] }

// With resolver
{ path: 'products/:id', component: ..., resolve: { product: productDetailResolver } }

// Named outlet child
{ path: 'reviews', component: ReviewsComponent, outlet: 'reviews' }

// Wildcard (always last)
{ path: '**', component: NotFoundComponent }
```

### Template Navigation

```html
<!-- Basic link -->
<a routerLink="/home">Home</a>

<!-- Dynamic link -->
<a [routerLink]="['/products', product.id]">View Product</a>

<!-- With query params -->
<a [routerLink]="['/products']" [queryParams]="{ category: 'Electronics' }">Electronics</a>

<!-- Active class -->
<a routerLink="/home" routerLinkActive="active-link" [routerLinkActiveOptions]="{ exact: true }">Home</a>

<!-- Named outlet -->
<a [routerLink]="['/about', { outlets: { reviews: ['reviews'], team: ['team'] } }]">About</a>
```

### Programmatic Navigation

```typescript
// Simple
this.router.navigate(['/products']);

// With param
this.router.navigate(['/products', id]);

// With query params
this.router.navigate(['/products'], { queryParams: { category: 'Electronics' } });

// With UrlTree (from guard)
return router.createUrlTree(['/login'], { queryParams: { returnUrl: state.url } });

// Replace URL
this.router.navigate(['/home'], { replaceUrl: true });

// String URL
this.router.navigateByUrl('/products');
```

### Functional Guard Pattern

```typescript
export const myGuard: CanActivateFn = (route, state) => {
  const service = inject(MyService);
  const router  = inject(Router);

  if (service.isAllowed()) return true;
  return router.createUrlTree(['/forbidden']);
};
```

### Signal Input Binding Pattern

```typescript
// In app.config.ts: provideRouter(routes, withComponentInputBinding())

// Route: { path: 'items/:id', ... }
export class ItemComponent {
  id = input<string>('');   // bound from :id param

  // Route: resolve: { item: itemResolver }
  item = input<Item | null>(null);  // bound from resolver
}
```

---
