# Angular CLI – Comprehensive Guide

---

## 1. What is Angular CLI?

**Angular CLI (Command Line Interface)** is a tool that helps you create, build, test, and maintain Angular applications efficiently.

Instead of manually configuring Webpack, TypeScript, testing setup, and folder structure, Angular CLI does it for you.

It provides:

* Project scaffolding
* Code generation
* Development server
* Production builds
* Testing tools
* Linting and formatting support

Think of it as an automation layer around Angular development.

---

# 2. Prerequisites

Before installing Angular CLI, ensure:

* **Node.js (LTS version)** installed
* **npm** (comes with Node.js)
* A code editor (VS Code recommended)

Check versions:

```bash
node -v
npm -v
```

---

# 3. Installing Angular CLI

Install globally:

```bash
npm install -g @angular/cli
```

Verify installation:

```bash
ng version
```

If this works, Angular CLI is ready.

---

# 4. Creating Your First Angular Project

Create a new app:

```bash
ng new my-first-app
```

CLI will ask:

* Would you like to add routing? → `Yes`
* Which stylesheet format? → `SCSS` (recommended)

Move into project folder:

```bash
cd my-first-app
```

Start development server:

```bash
ng serve
```

Open browser:

```
http://localhost:4200
```

You now have a running Angular app.

---

# 5. Understanding the Project Structure

After creation, you will see:

```
my-first-app/
│
├── src/
│   ├── app/
│   ├── assets/
│   ├── environments/
│   └── main.ts
│
├── angular.json
├── package.json
├── tsconfig.json
```

### Important Files

| File               | Purpose                |
| ------------------ | ---------------------- |
| `angular.json`     | CLI configuration      |
| `package.json`     | Dependencies & scripts |
| `main.ts`          | App entry point        |
| `app.component.ts` | Root component         |
| `tsconfig.json`    | TypeScript config      |

---

# 6. Running the Application

## Development Mode

```bash
ng serve
```

Options:

```bash
ng serve --open
ng serve --port 4300
```

---

# 7. Angular CLI Core Commands

## 1. Generate Code

Angular CLI can generate boilerplate automatically.

### Generate Component

```bash
ng generate component users
```

or

```bash
ng g c users
```

This creates:

```
users/
 ├── users.component.ts
 ├── users.component.html
 ├── users.component.scss
 └── users.component.spec.ts
```

### Use the component

Open `app.component.html`:

```html
<app-users></app-users>
```

---

## 2. Generate Service

```bash
ng g s services/user
```

Creates:

```
services/user.service.ts
```

---

## 3. Generate Module

```bash
ng g m admin
```

With routing:

```bash
ng g m admin --routing
```

---

# 8. Build the Application

## Development Build

```bash
ng build
```

## Production Build

```bash
ng build --configuration production
```

Output is generated inside:

```
dist/
```

Production build:

* Minifies code
* Removes debugging
* Optimizes bundles

---

# 9. Angular CLI Configuration

## angular.json

This file controls:

* Build configurations
* Assets
* Styles
* Scripts
* File replacements

Example: Add global style

```json
"styles": [
  "src/styles.scss"
]
```

---

# 10. Environments

Inside:

```
src/environments/
```

You’ll see:

* `environment.ts`
* `environment.prod.ts`

Use environment variables:

```ts
import { environment } from '../environments/environment';

console.log(environment.production);
```

Angular CLI replaces files automatically during production build.

---

# 11. Creating a Feature Module (Incremental Practice)

Let’s build a small feature step by step.

---

## Step 1: Generate Module

```bash
ng g m features/dashboard --routing
```

---

## Step 2: Generate Component

```bash
ng g c features/dashboard/pages/home
```

---

## Step 3: Add Route

Open:

```
features/dashboard/dashboard-routing.module.ts
```

Add:

```ts
const routes: Routes = [
  { path: '', component: HomeComponent }
];
```

---

## Step 4: Lazy Load Module

Open `app-routing.module.ts`:

```ts
const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () =>
      import('./features/dashboard/dashboard.module')
        .then(m => m.DashboardModule)
  }
];
```

Visit:

```
http://localhost:4200/dashboard
```

You just implemented lazy loading using CLI-generated files.

---

# 12. Testing with Angular CLI

## Run Unit Tests

```bash
ng test
```

Uses **Karma + Jasmine**

## Run End-to-End Tests (if configured)

```bash
ng e2e
```

---

# 13. Linting

```bash
ng lint
```

Ensures code follows best practices.

---

# 14. Updating Angular CLI

Check outdated packages:

```bash
ng update
```

Update Angular:

```bash
ng update @angular/core @angular/cli
```

---

# 15. Customizing Angular CLI Behavior

### Skip spec file generation:

```bash
ng g c example --skip-tests
```

### Set default in angular.json:

```json
"schematics": {
  "@schematics/angular:component": {
    "skipTests": true
  }
}
```

---

# 16. Useful CLI Flags

| Flag           | Purpose                     |
| -------------- | --------------------------- |
| `--dry-run`    | Shows what will be created  |
| `--skip-tests` | Avoid spec files            |
| `--flat`       | No new folder               |
| `--export`     | Export component in module  |
| `--standalone` | Create standalone component |

Example:

```bash
ng g c header --standalone
```

---

# 17. Working with Standalone Components

Generate:

```bash
ng g c navbar --standalone
```

Use directly in `app.component.ts`:

```ts
import { NavbarComponent } from './navbar/navbar.component';

@Component({
  standalone: true,
  imports: [NavbarComponent],
})
```

No NgModule required.

---

# 18. Advanced: Creating a Library

```bash
ng generate library shared-ui
```

Used in monorepo setups.

---

# 19. Production Deployment Steps

1. Build production:

```bash
ng build --configuration production
```

2. Upload `dist/` folder to hosting server (Firebase, Netlify, AWS, etc.)

---

# 20. Common Errors & Fixes

### Port already in use

```bash
ng serve --port 4300
```

### Node version mismatch

Upgrade Node.js to LTS.

### Module not found

Run:

```bash
npm install
```

---

# 21. Best Practices

* Use feature-based folder structure
* Lazy load modules
* Keep shared components in shared module
* Use environment files for API URLs
* Generate using CLI (avoid manual file creation)

---

# 22. Quick Reference Cheat Sheet

```bash
ng new app-name
ng serve
ng build
ng test
ng lint
ng g c component-name
ng g s service-name
ng g m module-name
ng update
```

---

# Conclusion

Angular CLI is not just a tool for creating projects. It enforces structure, reduces configuration overhead, ensures consistency, and speeds up development.

Mastering Angular CLI means:

* Faster scaffolding
* Cleaner architecture
* Easier upgrades
* Production-ready builds

This tool is the backbone of modern Angular development.
