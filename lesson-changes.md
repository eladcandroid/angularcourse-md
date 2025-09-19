# Angular Advanced Course: Last 6 Versions Overview

This document provides a comprehensive overview of Angular versions 15-20, highlighting the major changes, features, and improvements introduced in each version for the Angular Advanced Course.

## Angular 20 (Released May 29, 2025) - Current Stable

Angular 20 represents a major milestone with stable signals and zoneless change detection, making it the most performance-focused release to date.

### <� Major Features

#### 1. Stable Signals and Complete Reactivity
All fundamental reactivity primitives graduated to stable including `signal()`, `effect()`, `linkedSignal()`, signal-based queries, and inputs.

**Example 1: Basic Signal Usage**
```typescript
import { Component, signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <div>
      <p>Count: {{ count() }}</p>
      <p>Double: {{ doubleCount() }}</p>
      <button (click)="increment()">+</button>
      <button (click)="decrement()">-</button>
    </div>
  `
})
export class CounterComponent {
  count = signal(0);
  doubleCount = computed(() => this.count() * 2);

  constructor() {
    effect(() => {
      console.log('Count changed to:', this.count());
    });
  }

  increment() {
    this.count.update(value => value + 1);
  }

  decrement() {
    this.count.update(value => value - 1);
  }
}
```

**Example 2: Signal-based Input and LinkedSignal**
```typescript
import { Component, input, linkedSignal } from '@angular/core';

@Component({
  selector: 'app-user-profile',
  template: `
    <div>
      <h2>{{ displayName() }}</h2>
      <p>Status: {{ userStatus() }}</p>
      <button (click)="toggleStatus()">Toggle Status</button>
    </div>
  `
})
export class UserProfileComponent {
  // Signal-based input (stable in v20)
  userId = input.required<string>();
  firstName = input.required<string>();
  lastName = input.required<string>();

  // LinkedSignal derives from other signals
  displayName = linkedSignal(() =>
    `${this.firstName()} ${this.lastName()}`
  );

  userStatus = signal<'online' | 'offline'>('online');

  toggleStatus() {
    this.userStatus.update(status =>
      status === 'online' ? 'offline' : 'online'
    );
  }
}
```

#### 2. Zoneless Change Detection (Stable)
Zoneless change detection became stable in Angular 20, providing better performance and reduced bundle size.

**Example 1: Zoneless Application Bootstrap**
```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { provideExperimentalZonelessChangeDetection } from '@angular/core';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    provideZonelessChangeDetection(),
    // other providers
  ]
});
```

**Example 2: Manual Change Detection with Zoneless**
```typescript
import { Component, signal, inject, ChangeDetectorRef } from '@angular/core';

@Component({
  selector: 'app-manual-detection',
  template: `
    <div>
      <p>Timer: {{ timer() }}</p>
      <button (click)="startTimer()">Start Timer</button>
      <button (click)="stopTimer()">Stop Timer</button>
    </div>
  `
})
export class ManualDetectionComponent {
  private cdr = inject(ChangeDetectorRef);
  timer = signal(0);
  private intervalId: number | null = null;

  startTimer() {
    if (this.intervalId) return;

    this.intervalId = window.setInterval(() => {
      this.timer.update(value => value + 1);
      // In zoneless mode, manual detection may be needed for some cases
      this.cdr.detectChanges();
    }, 1000);
  }

  stopTimer() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }
  }
}
```

#### 3. Template Hot Module Replacement (HMR) - Stable
Template HMR graduated to stable, improving the development experience.

**Example 1: HMR Configuration**
```typescript
// angular.json configuration for HMR
{
  "serve": {
    "builder": "@angular-devkit/build-angular:dev-server",
    "options": {
      "hmr": true
    }
  }
}
```

**Example 2: Component with HMR-friendly structure**
```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-hmr-demo',
  template: `
    <div class="container">
      <h1>{{ title() }}</h1>
      <div class="content">
        <p>This template supports HMR</p>
        <button (click)="updateTitle()">Update Title</button>
      </div>
    </div>
  `,
  styles: [`
    .container {
      padding: 20px;
      border: 1px solid #ccc;
    }
    .content {
      margin-top: 10px;
    }
  `]
})
export class HmrDemoComponent {
  title = signal('HMR Demo Component');

  updateTitle() {
    this.title.set('Updated via HMR!');
  }
}
```

---

## Angular 19 (Released Q4 2024)

Angular 19 focused on server-side rendering improvements and performance optimizations.

### <� Major Features

#### 1. Event Replay (Stable)
Event replay graduated to stable and became enabled by default for all new projects.

**Example 1: Event Replay with SSR**
```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-interactive-form',
  template: `
    <form (submit)="onSubmit($event)">
      <input
        [(ngModel)]="username"
        (input)="onUsernameChange($event)"
        placeholder="Username"
      >
      <button type="submit">Submit</button>
      <p>Typed: {{ username() }}</p>
    </form>
  `
})
export class InteractiveFormComponent {
  username = signal('');

  onUsernameChange(event: Event) {
    const target = event.target as HTMLInputElement;
    this.username.set(target.value);
    // Events are automatically replayed when hydration completes
  }

  onSubmit(event: Event) {
    event.preventDefault();
    console.log('Submitted:', this.username());
  }
}
```

**Example 2: Event Replay Configuration**
```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { provideClientHydration, withEventReplay } from '@angular/platform-browser';

bootstrapApplication(AppComponent, {
  providers: [
    provideClientHydration(withEventReplay()), // Event replay enabled
    // other providers
  ]
});
```

#### 2. Incremental Hydration (Developer Preview)
Incremental hydration powered by @defer blocks showing 40-50% LCP improvements.

**Example 1: Incremental Hydration with Defer**
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-product-page',
  template: `
    <header>
      <h1>Product Details</h1>
    </header>

    <!-- Critical content loads immediately -->
    <section class="product-info">
      <h2>{{ productName }}</h2>
      <p>{{ productPrice }}</p>
    </section>

    <!-- Non-critical content deferred -->
    @defer (on viewport) {
      <app-product-reviews />
    } @placeholder {
      <div>Loading reviews...</div>
    }

    @defer (on interaction) {
      <app-related-products />
    } @placeholder {
      <div>Click to load related products</div>
    }
  `
})
export class ProductPageComponent {
  productName = 'Amazing Product';
  productPrice = '$99.99';
}
```

**Example 2: Complex Incremental Hydration**
```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-dashboard',
  template: `
    <!-- Immediate hydration for critical UI -->
    <nav>
      <button (click)="toggleSidebar()">Menu</button>
    </nav>

    <!-- Deferred hydration for heavy components -->
    @defer (on idle) {
      <app-analytics-chart [data]="chartData()" />
    } @loading (minimum 500ms) {
      <div class="loading-spinner">Loading analytics...</div>
    }

    @defer (on timer(2s)) {
      <app-notifications />
    } @placeholder {
      <div>Notifications will load shortly...</div>
    }
  `
})
export class DashboardComponent {
  sidebarOpen = signal(false);
  chartData = signal([1, 2, 3, 4, 5]);

  toggleSidebar() {
    this.sidebarOpen.update(open => !open);
  }
}
```

#### 3. httpResource API (Experimental)
Reactive HTTP data fetching with signal-based responses.

**Example 1: Basic httpResource Usage**
```typescript
import { Component, input } from '@angular/core';
import { httpResource } from '@angular/core/http-resource';

@Component({
  selector: 'app-user-detail',
  template: `
    @if (user.hasValue()) {
      <div>
        <h2>{{ user.value().name }}</h2>
        <p>{{ user.value().email }}</p>
      </div>
    } @else if (user.error()) {
      <div>Error loading user</div>
    } @else if (user.isLoading()) {
      <div>Loading user...</div>
    }
  `
})
export class UserDetailComponent {
  userId = input.required<string>();

  user = httpResource(() => `/api/users/${this.userId()}`);
}
```

**Example 2: httpResource with Request Configuration**
```typescript
import { Component, signal } from '@angular/core';
import { httpResource } from '@angular/core/http-resource';

@Component({
  selector: 'app-search-results',
  template: `
    <input
      [(ngModel)]="searchQuery"
      placeholder="Search..."
      (input)="updateQuery($event)"
    >

    @if (searchResults.hasValue()) {
      <ul>
        @for (item of searchResults.value(); track item.id) {
          <li>{{ item.title }}</li>
        }
      </ul>
    } @else if (searchResults.isLoading()) {
      <div>Searching...</div>
    }
  `
})
export class SearchResultsComponent {
  searchQuery = signal('');

  searchResults = httpResource(() => ({
    url: '/api/search',
    method: 'GET',
    params: {
      q: this.searchQuery(),
      limit: '10'
    }
  }));

  updateQuery(event: Event) {
    const target = event.target as HTMLInputElement;
    this.searchQuery.set(target.value);
  }
}
```

---

## Angular 18 (Released Q2 2024)

Angular 18 marked the stabilization of many developer preview features from v17.

### <� Major Features

#### 1. Built-in Control Flow (Stable)
The new @if, @for, and @switch syntax graduated to stable.

**Example 1: @if and @for Control Flow**
```typescript
import { Component, signal } from '@angular/core';

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

@Component({
  selector: 'app-todo-list',
  template: `
    <div>
      <h2>Todo List ({{ todos().length }} items)</h2>

      @if (todos().length > 0) {
        <ul>
          @for (todo of todos(); track todo.id) {
            <li [class.completed]="todo.completed">
              <input
                type="checkbox"
                [checked]="todo.completed"
                (change)="toggleTodo(todo.id)"
              >
              {{ todo.text }}
              <button (click)="deleteTodo(todo.id)">Delete</button>
            </li>
          }
        </ul>
      } @else {
        <p>No todos yet. Add some!</p>
      }

      <button (click)="addTodo()">Add Todo</button>
    </div>
  `,
  styles: [`
    .completed { text-decoration: line-through; }
  `]
})
export class TodoListComponent {
  todos = signal<Todo[]>([
    { id: 1, text: 'Learn Angular 18', completed: false },
    { id: 2, text: 'Use new control flow', completed: true }
  ]);

  addTodo() {
    const newTodo: Todo = {
      id: Math.max(...this.todos().map(t => t.id)) + 1,
      text: 'New Todo',
      completed: false
    };
    this.todos.update(todos => [...todos, newTodo]);
  }

  toggleTodo(id: number) {
    this.todos.update(todos =>
      todos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }

  deleteTodo(id: number) {
    this.todos.update(todos => todos.filter(todo => todo.id !== id));
  }
}
```

**Example 2: @switch Control Flow**
```typescript
import { Component, signal } from '@angular/core';

type ViewMode = 'list' | 'grid' | 'table';

@Component({
  selector: 'app-data-view',
  template: `
    <div>
      <nav>
        <button (click)="setViewMode('list')">List</button>
        <button (click)="setViewMode('grid')">Grid</button>
        <button (click)="setViewMode('table')">Table</button>
      </nav>

      @switch (viewMode()) {
        @case ('list') {
          <div class="list-view">
            @for (item of items(); track item.id) {
              <div class="list-item">{{ item.name }}</div>
            }
          </div>
        }
        @case ('grid') {
          <div class="grid-view">
            @for (item of items(); track item.id) {
              <div class="grid-item">{{ item.name }}</div>
            }
          </div>
        }
        @case ('table') {
          <table>
            <thead>
              <tr><th>ID</th><th>Name</th></tr>
            </thead>
            <tbody>
              @for (item of items(); track item.id) {
                <tr>
                  <td>{{ item.id }}</td>
                  <td>{{ item.name }}</td>
                </tr>
              }
            </tbody>
          </table>
        }
      }
    </div>
  `,
  styles: [`
    .grid-view { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; }
    .list-item, .grid-item { padding: 10px; border: 1px solid #ccc; }
  `]
})
export class DataViewComponent {
  viewMode = signal<ViewMode>('list');
  items = signal([
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' }
  ]);

  setViewMode(mode: ViewMode) {
    this.viewMode.set(mode);
  }
}
```

#### 2. Material 3 Support (Stable)
Angular Material 3 design system support graduated to stable.

**Example 1: Material 3 Components**
```typescript
import { Component } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatIconModule } from '@angular/material/icon';

@Component({
  selector: 'app-material3-demo',
  imports: [MatButtonModule, MatCardModule, MatIconModule],
  template: `
    <mat-card appearance="outlined">
      <mat-card-header>
        <mat-card-title>Material 3 Card</mat-card-title>
        <mat-card-subtitle>With new design tokens</mat-card-subtitle>
      </mat-card-header>

      <mat-card-content>
        <p>This card uses Material 3 design principles</p>
      </mat-card-content>

      <mat-card-actions>
        <button mat-button>Basic</button>
        <button mat-raised-button color="primary">Primary</button>
        <button mat-fab color="accent">
          <mat-icon>add</mat-icon>
        </button>
      </mat-card-actions>
    </mat-card>
  `
})
export class Material3DemoComponent {}
```

**Example 2: Material 3 Theming**
```scss
// styles.scss - Material 3 theme configuration
@use '@angular/material' as mat;

// Define Material 3 color palette
$primary-palette: mat.define-palette(mat.$azure-palette);
$accent-palette: mat.define-palette(mat.$blue-palette);
$warn-palette: mat.define-palette(mat.$red-palette);

// Create Material 3 theme
$theme: mat.define-theme((
  color: (
    theme-type: light,
    primary: $primary-palette,
    tertiary: $accent-palette,
  ),
  typography: (
    brand-family: 'Roboto, sans-serif',
    plain-family: 'Roboto, sans-serif',
  ),
  density: (
    scale: 0,
  )
));

// Include Material 3 core styles
@include mat.core();
@include mat.all-component-themes($theme);
```

#### 3. Deferrable Views (Stable)
@defer blocks for lazy loading graduated to stable.

**Example 1: Defer with Multiple Triggers**
```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-defer-demo',
  template: `
    <div>
      <h2>Deferrable Views Demo</h2>

      <!-- Defer on viewport -->
      @defer (on viewport) {
        <app-heavy-chart />
      } @placeholder {
        <div class="placeholder">Chart will load when visible</div>
      } @loading (minimum 500ms) {
        <div class="loading">Loading chart...</div>
      } @error {
        <div class="error">Failed to load chart</div>
      }

      <!-- Defer on interaction -->
      @defer (on interaction) {
        <app-comment-section />
      } @placeholder {
        <button>Click to load comments</button>
      }

      <!-- Defer on timer -->
      @defer (on timer(3000ms)) {
        <app-analytics-widget />
      } @placeholder {
        <div>Analytics loading in 3 seconds...</div>
      }
    </div>
  `,
  styles: [`
    .placeholder, .loading, .error {
      padding: 20px;
      border: 2px dashed #ccc;
      text-align: center;
      margin: 10px 0;
    }
    .loading { border-color: #2196F3; }
    .error { border-color: #f44336; }
  `]
})
export class DeferDemoComponent {}
```

**Example 2: Advanced Defer with Conditions**
```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-conditional-defer',
  template: `
    <div>
      <label>
        <input
          type="checkbox"
          [checked]="showAdvanced()"
          (change)="toggleAdvanced()"
        >
        Show Advanced Features
      </label>

      @if (showAdvanced()) {
        @defer (on idle) {
          <app-advanced-settings />
        } @placeholder {
          <div>Advanced settings will load when browser is idle</div>
        }
      }

      <button (click)="loadData()" #loadBtn>Load Data</button>

      @defer (on interaction(loadBtn)) {
        <app-data-table [data]="data()" />
      } @placeholder {
        <div>Click "Load Data" to see the table</div>
      }
    </div>
  `
})
export class ConditionalDeferComponent {
  showAdvanced = signal(false);
  data = signal([
    { id: 1, name: 'Sample Data 1' },
    { id: 2, name: 'Sample Data 2' }
  ]);

  toggleAdvanced() {
    this.showAdvanced.update(show => !show);
  }

  loadData() {
    // Simulate data loading
    console.log('Loading data...');
  }
}
```

---

## Angular 17 (Released Q4 2023)

Angular 17 introduced revolutionary changes with new control flow, build system improvements, and application builder updates.

### <� Major Features

#### 1. New Control Flow (Developer Preview)
Introduction of @if, @for, @switch as developer preview.

**Example 1: Migration from *ngIf to @if**
```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-migration-example',
  template: `
    <!-- Old way with *ngIf -->
    <!--
    <div *ngIf="isLoggedIn; else loginTemplate">
      Welcome back!
    </div>
    <ng-template #loginTemplate>
      <div>Please log in</div>
    </ng-template>
    -->

    <!-- New way with @if -->
    @if (isLoggedIn()) {
      <div>Welcome back!</div>
    } @else {
      <div>Please log in</div>
    }

    <!-- Complex conditions -->
    @if (user(); as currentUser) {
      <div>
        <h2>{{ currentUser.name }}</h2>
        @if (currentUser.isAdmin) {
          <button>Admin Panel</button>
        }
      </div>
    }
  `
})
export class MigrationExampleComponent {
  isLoggedIn = signal(false);
  user = signal<{name: string, isAdmin: boolean} | null>(null);
}
```

**Example 2: Enhanced @for with Performance**
```typescript
import { Component, signal } from '@angular/core';

interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
}

@Component({
  selector: 'app-product-list',
  template: `
    <div>
      <select (change)="filterByCategory($event)">
        <option value="">All Categories</option>
        @for (category of categories(); track category) {
          <option [value]="category">{{ category }}</option>
        }
      </select>

      <!-- Enhanced @for with track and empty -->
      @for (product of filteredProducts(); track product.id) {
        <div class="product-card">
          <h3>{{ product.name }}</h3>
          <p>Price: \${{ product.price }}</p>
          <span class="category">{{ product.category }}</span>
        </div>
      } @empty {
        <div class="no-products">
          No products found in this category
        </div>
      }
    </div>
  `,
  styles: [`
    .product-card {
      border: 1px solid #ddd;
      padding: 15px;
      margin: 10px 0;
      border-radius: 5px;
    }
    .category {
      background: #e1f5fe;
      padding: 3px 8px;
      border-radius: 3px;
      font-size: 0.8em;
    }
    .no-products {
      text-align: center;
      padding: 40px;
      color: #666;
    }
  `]
})
export class ProductListComponent {
  selectedCategory = signal('');

  products = signal<Product[]>([
    { id: 1, name: 'Laptop', price: 999, category: 'Electronics' },
    { id: 2, name: 'Book', price: 29, category: 'Education' },
    { id: 3, name: 'Phone', price: 699, category: 'Electronics' }
  ]);

  categories = signal(['Electronics', 'Education', 'Clothing']);

  filteredProducts = signal<Product[]>([]);

  constructor() {
    // Update filtered products when category changes
    this.filteredProducts.set(this.products());
  }

  filterByCategory(event: Event) {
    const target = event.target as HTMLSelectElement;
    const category = target.value;
    this.selectedCategory.set(category);

    if (category) {
      this.filteredProducts.set(
        this.products().filter(p => p.category === category)
      );
    } else {
      this.filteredProducts.set(this.products());
    }
  }
}
```

#### 2. Vite/ESBuild Builder (Default)
New application builder with Vite and ESBuild became default for new projects.

**Example 1: Vite Configuration**
```typescript
// angular.json - New builder configuration
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:application",
          "options": {
            "outputPath": "dist/my-app",
            "index": "src/index.html",
            "browser": "src/main.ts",
            "polyfills": ["zone.js"],
            "tsConfig": "tsconfig.app.json",
            "assets": ["src/favicon.ico", "src/assets"],
            "styles": ["src/styles.css"],
            "scripts": []
          }
        },
        "serve": {
          "builder": "@angular-devkit/build-angular:dev-server",
          "options": {
            "buildTarget": "my-app:build"
          }
        }
      }
    }
  }
}
```

**Example 2: Performance Optimizations**
```typescript
// vite.config.ts - Custom Vite configuration (if needed)
import { defineConfig } from 'vite';

export default defineConfig({
  optimizeDeps: {
    include: ['@angular/common', '@angular/core']
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['@angular/core', '@angular/common'],
          material: ['@angular/material']
        }
      }
    }
  },
  server: {
    hmr: {
      overlay: false
    }
  }
});
```

#### 3. Standalone Components as Default
Standalone components became the default for new projects.

**Example 1: Standalone App Bootstrap**
```typescript
// main.ts - Standalone application bootstrap
import { bootstrapApplication } from '@angular/platform-browser';
import { importProvidersFrom } from '@angular/core';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';

import { AppComponent } from './app/app.component';
import { routes } from './app/app.routes';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    importProvidersFrom(BrowserAnimationsModule)
  ]
}).catch(err => console.error(err));
```

**Example 2: Standalone Component with Imports**
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { MatButtonModule } from '@angular/material/button';
import { MatInputModule } from '@angular/material/input';
import { MatFormFieldModule } from '@angular/material/form-field';

@Component({
  selector: 'app-standalone-form',
  imports: [
    CommonModule,
    FormsModule,
    MatButtonModule,
    MatInputModule,
    MatFormFieldModule
  ],
  template: `
    <form (ngSubmit)="onSubmit()">
      <mat-form-field>
        <mat-label>Name</mat-label>
        <input matInput [(ngModel)]="name" name="name">
      </mat-form-field>

      <mat-form-field>
        <mat-label>Email</mat-label>
        <input matInput [(ngModel)]="email" name="email" type="email">
      </mat-form-field>

      <button mat-raised-button color="primary" type="submit">
        Submit
      </button>
    </form>
  `
})
export class StandaloneFormComponent {
  name = '';
  email = '';

  onSubmit() {
    console.log('Form submitted:', { name: this.name, email: this.email });
  }
}
```

---

## Angular 16 (Released 2023)

Angular 16 introduced experimental signals and significant improvements to the development experience.

### <� Major Features

#### 1. Signals (Developer Preview)
First introduction of signals as a developer preview feature.

**Example 1: Basic Signals Introduction**
```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-signals-intro',
  template: `
    <div>
      <h2>Signals Introduction</h2>
      <p>First Name: {{ firstName() }}</p>
      <p>Last Name: {{ lastName() }}</p>
      <p>Full Name: {{ fullName() }}</p>
      <p>Character Count: {{ characterCount() }}</p>

      <input
        [value]="firstName()"
        (input)="updateFirstName($event)"
        placeholder="First Name"
      >
      <input
        [value]="lastName()"
        (input)="updateLastName($event)"
        placeholder="Last Name"
      >
    </div>
  `
})
export class SignalsIntroComponent {
  // Basic signals
  firstName = signal('John');
  lastName = signal('Doe');

  // Computed signals
  fullName = computed(() => `${this.firstName()} ${this.lastName()}`);
  characterCount = computed(() => this.fullName().length);

  updateFirstName(event: Event) {
    const target = event.target as HTMLInputElement;
    this.firstName.set(target.value);
  }

  updateLastName(event: Event) {
    const target = event.target as HTMLInputElement;
    this.lastName.set(target.value);
  }
}
```

**Example 2: Signals with Complex State**
```typescript
import { Component, signal, computed, effect } from '@angular/core';

interface CartItem {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

@Component({
  selector: 'app-shopping-cart',
  template: `
    <div>
      <h2>Shopping Cart</h2>

      @for (item of cartItems(); track item.id) {
        <div class="cart-item">
          <span>{{ item.name }} - \${{ item.price }}</span>
          <input
            type="number"
            [value]="item.quantity"
            (input)="updateQuantity(item.id, $event)"
            min="1"
          >
          <button (click)="removeItem(item.id)">Remove</button>
        </div>
      }

      <div class="summary">
        <p>Total Items: {{ totalItems() }}</p>
        <p>Total Price: \${{ totalPrice() }}</p>
      </div>

      <button (click)="addSampleItem()">Add Sample Item</button>
    </div>
  `,
  styles: [`
    .cart-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 10px;
      border-bottom: 1px solid #eee;
    }
    .summary {
      margin-top: 20px;
      padding: 15px;
      background: #f5f5f5;
      border-radius: 5px;
    }
  `]
})
export class ShoppingCartComponent {
  cartItems = signal<CartItem[]>([
    { id: 1, name: 'Product 1', price: 29.99, quantity: 2 },
    { id: 2, name: 'Product 2', price: 49.99, quantity: 1 }
  ]);

  totalItems = computed(() =>
    this.cartItems().reduce((sum, item) => sum + item.quantity, 0)
  );

  totalPrice = computed(() =>
    this.cartItems().reduce((sum, item) => sum + (item.price * item.quantity), 0)
  );

  constructor() {
    // Effect to log cart changes
    effect(() => {
      console.log('Cart updated:', {
        items: this.cartItems().length,
        total: this.totalPrice()
      });
    });
  }

  updateQuantity(id: number, event: Event) {
    const target = event.target as HTMLInputElement;
    const quantity = parseInt(target.value);

    this.cartItems.update(items =>
      items.map(item =>
        item.id === id ? { ...item, quantity } : item
      )
    );
  }

  removeItem(id: number) {
    this.cartItems.update(items => items.filter(item => item.id !== id));
  }

  addSampleItem() {
    const newId = Math.max(...this.cartItems().map(item => item.id)) + 1;
    this.cartItems.update(items => [
      ...items,
      { id: newId, name: `Product ${newId}`, price: 19.99, quantity: 1 }
    ]);
  }
}
```

#### 2. DestroyRef Provider
New DestroyRef provider for cleanup operations.

**Example 1: DestroyRef for Cleanup**
```typescript
import { Component, DestroyRef, inject, OnInit } from '@angular/core';
import { interval } from 'rxjs';

@Component({
  selector: 'app-destroy-ref-example',
  template: `
    <div>
      <h2>DestroyRef Example</h2>
      <p>Timer: {{ currentTime }}</p>
      <p>Data updates: {{ updateCount }}</p>
    </div>
  `
})
export class DestroyRefExampleComponent implements OnInit {
  private destroyRef = inject(DestroyRef);
  currentTime = '';
  updateCount = 0;

  ngOnInit() {
    // Set up interval that auto-cleans up
    const timeInterval = setInterval(() => {
      this.currentTime = new Date().toLocaleTimeString();
      this.updateCount++;
    }, 1000);

    // Register cleanup with DestroyRef
    this.destroyRef.onDestroy(() => {
      clearInterval(timeInterval);
      console.log('Timer cleaned up');
    });

    // Multiple cleanup operations
    const subscription = interval(5000).subscribe(() => {
      console.log('Background task running');
    });

    this.destroyRef.onDestroy(() => {
      subscription.unsubscribe();
      console.log('Subscription cleaned up');
    });
  }
}
```

**Example 2: DestroyRef with HTTP Requests**
```typescript
import { Component, DestroyRef, inject, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';

interface User {
  id: number;
  name: string;
  email: string;
}

@Component({
  selector: 'app-http-cleanup',
  template: `
    <div>
      <button (click)="loadUsers()">Load Users</button>
      <button (click)="loadUser(1)">Load User 1</button>

      @if (loading()) {
        <p>Loading...</p>
      }

      @if (users().length > 0) {
        <ul>
          @for (user of users(); track user.id) {
            <li>{{ user.name }} - {{ user.email }}</li>
          }
        </ul>
      }
    </div>
  `
})
export class HttpCleanupComponent {
  private http = inject(HttpClient);
  private destroyRef = inject(DestroyRef);

  users = signal<User[]>([]);
  loading = signal(false);

  loadUsers() {
    this.loading.set(true);

    const subscription = this.http.get<User[]>('/api/users')
      .subscribe({
        next: users => {
          this.users.set(users);
          this.loading.set(false);
        },
        error: () => {
          this.loading.set(false);
        }
      });

    // Auto-cleanup subscription
    this.destroyRef.onDestroy(() => subscription.unsubscribe());
  }

  loadUser(id: number) {
    const subscription = this.http.get<User>(`/api/users/${id}`)
      .subscribe(user => {
        this.users.update(users => [user]);
      });

    this.destroyRef.onDestroy(() => subscription.unsubscribe());
  }
}
```

#### 3. Experimental ESBuild Support
Introduction of ESBuild for faster builds.

**Example 1: ESBuild Configuration**
```json
// angular.json - ESBuild configuration
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser-esbuild",
          "options": {
            "outputPath": "dist/my-app",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": "src/polyfills.ts",
            "tsConfig": "tsconfig.app.json"
          }
        }
      }
    }
  }
}
```

**Example 2: Build Performance Comparison**
```typescript
// Performance monitoring component
import { Component, signal, OnInit } from '@angular/core';

@Component({
  selector: 'app-build-performance',
  template: `
    <div>
      <h2>Build Performance Metrics</h2>
      <div class="metrics">
        <div class="metric">
          <label>Build Time (ESBuild):</label>
          <span>{{ buildTime() }}ms</span>
        </div>
        <div class="metric">
          <label>Bundle Size:</label>
          <span>{{ bundleSize() }}KB</span>
        </div>
        <div class="metric">
          <label>Dev Server Start:</label>
          <span>{{ devServerTime() }}ms</span>
        </div>
      </div>
    </div>
  `,
  styles: [`
    .metrics { margin: 20px 0; }
    .metric {
      display: flex;
      justify-content: space-between;
      padding: 8px 0;
      border-bottom: 1px solid #eee;
    }
  `]
})
export class BuildPerformanceComponent implements OnInit {
  buildTime = signal(0);
  bundleSize = signal(0);
  devServerTime = signal(0);

  ngOnInit() {
    // Simulate performance metrics
    this.buildTime.set(1200); // ~87% faster than webpack
    this.bundleSize.set(245);
    this.devServerTime.set(800);
  }
}
```

---

## Angular 15 (Released 2022)

Angular 15 marked the stabilization of standalone APIs and significant improvements to Angular Material.

### <� Major Features

#### 1. Standalone APIs (Stable)
Standalone components, directives, and pipes graduated to stable.

**Example 1: Complete Standalone App**
```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter } from '@angular/router';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter([
      { path: '', component: HomeComponent },
      { path: 'about', component: AboutComponent }
    ])
  ]
});

// app.component.ts
import { Component } from '@angular/core';
import { RouterOutlet, RouterLink } from '@angular/router';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet, RouterLink],
  template: `
    <nav>
      <a routerLink="/">Home</a>
      <a routerLink="/about">About</a>
    </nav>
    <router-outlet></router-outlet>
  `
})
export class AppComponent {}
```

**Example 2: Standalone Directive and Pipe**
```typescript
// highlight.directive.ts
import { Directive, ElementRef, Input } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  @Input() set appHighlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color || 'yellow';
  }

  constructor(private el: ElementRef) {}
}

// reverse.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'reverse'
})
export class ReversePipe implements PipeTransform {
  transform(value: string): string {
    return value.split('').reverse().join('');
  }
}

// using-standalone.component.ts
import { Component } from '@angular/core';
import { HighlightDirective } from './highlight.directive';
import { ReversePipe } from './reverse.pipe';

@Component({
  selector: 'app-using-standalone',
  imports: [HighlightDirective, ReversePipe],
  template: `
    <div>
      <p appHighlight="lightblue">This text is highlighted</p>
      <p>{{ 'Angular' | reverse }}</p>
      <p appHighlight="lightgreen">{{ 'Standalone' | reverse }}</p>
    </div>
  `
})
export class UsingStandaloneComponent {}
```

#### 2. Image Directive with NgOptimizedImage
Angular's optimized image directive became stable.

**Example 1: Basic NgOptimizedImage Usage**
```typescript
import { Component } from '@angular/core';
import { NgOptimizedImage } from '@angular/common';

@Component({
  selector: 'app-image-gallery',
  imports: [NgOptimizedImage],
  template: `
    <div class="gallery">
      <img
        ngSrc="/assets/hero-image.jpg"
        alt="Hero image"
        width="800"
        height="400"
        priority
      >

      <div class="image-grid">
        <img
          ngSrc="/assets/product-1.jpg"
          alt="Product 1"
          width="300"
          height="200"
          placeholder="blur"
        >
        <img
          ngSrc="/assets/product-2.jpg"
          alt="Product 2"
          width="300"
          height="200"
          loading="lazy"
        >
      </div>
    </div>
  `,
  styles: [`
    .gallery { max-width: 800px; margin: 0 auto; }
    .image-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
      gap: 20px;
    }
  `]
})
export class ImageGalleryComponent {}
```

**Example 2: Advanced NgOptimizedImage with Responsive Images**
```typescript
import { Component, signal } from '@angular/core';
import { NgOptimizedImage } from '@angular/common';

interface ImageItem {
  src: string;
  alt: string;
  width: number;
  height: number;
}

@Component({
  selector: 'app-responsive-gallery',
  imports: [NgOptimizedImage],
  template: `
    <div class="responsive-gallery">
      <h2>Responsive Image Gallery</h2>

      @for (image of images(); track image.src) {
        <figure>
          <img
            [ngSrc]="image.src"
            [alt]="image.alt"
            [width]="image.width"
            [height]="image.height"
            sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
            loading="lazy"
          >
          <figcaption>{{ image.alt }}</figcaption>
        </figure>
      }
    </div>
  `,
  styles: [`
    .responsive-gallery {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
      gap: 20px;
      padding: 20px;
    }
    figure {
      margin: 0;
      border-radius: 8px;
      overflow: hidden;
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    }
    img {
      width: 100%;
      height: auto;
      display: block;
    }
    figcaption {
      padding: 10px;
      background: #f5f5f5;
      text-align: center;
    }
  `]
})
export class ResponsiveGalleryComponent {
  images = signal<ImageItem[]>([
    {
      src: '/assets/nature-1.jpg',
      alt: 'Beautiful landscape',
      width: 400,
      height: 300
    },
    {
      src: '/assets/nature-2.jpg',
      alt: 'Mountain view',
      width: 400,
      height: 300
    },
    {
      src: '/assets/nature-3.jpg',
      alt: 'Ocean sunset',
      width: 400,
      height: 300
    }
  ]);
}
```

#### 3. Material Design Components (MDC Web Integration)
Angular Material components were updated to use MDC Web.

**Example 1: Updated Material Components**
```typescript
import { Component } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatChipsModule } from '@angular/material/chips';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';

@Component({
  selector: 'app-material-showcase',
  imports: [
    MatButtonModule,
    MatCardModule,
    MatChipsModule,
    MatFormFieldModule,
    MatInputModule
  ],
  template: `
    <div class="material-showcase">
      <h2>Angular Material 15 Components</h2>

      <mat-card class="demo-card">
        <mat-card-header>
          <mat-card-title>MDC Web Integration</mat-card-title>
          <mat-card-subtitle>Enhanced Material Design</mat-card-subtitle>
        </mat-card-header>

        <mat-card-content>
          <p>Angular Material now uses MDC Web for better design compliance.</p>

          <mat-form-field appearance="outline">
            <mat-label>Email</mat-label>
            <input matInput type="email">
          </mat-form-field>

          <div class="chip-list">
            <mat-chip-listbox>
              <mat-chip-option>Angular</mat-chip-option>
              <mat-chip-option>Material</mat-chip-option>
              <mat-chip-option>MDC Web</mat-chip-option>
            </mat-chip-listbox>
          </div>
        </mat-card-content>

        <mat-card-actions>
          <button mat-button>Basic</button>
          <button mat-raised-button color="primary">Raised</button>
          <button mat-stroked-button color="accent">Stroked</button>
        </mat-card-actions>
      </mat-card>
    </div>
  `,
  styles: [`
    .material-showcase {
      max-width: 600px;
      margin: 20px auto;
    }
    .demo-card {
      margin: 20px 0;
    }
    .chip-list {
      margin: 10px 0;
    }
    mat-form-field {
      width: 100%;
    }
  `]
})
export class MaterialShowcaseComponent {}
```

**Example 2: Form with New Material Components**
```typescript
import { Component, signal } from '@angular/core';
import { FormBuilder, FormGroup, Validators, ReactiveFormsModule } from '@angular/forms';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatSelectModule } from '@angular/material/select';
import { MatCheckboxModule } from '@angular/material/checkbox';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';

@Component({
  selector: 'app-material-form',
  imports: [
    ReactiveFormsModule,
    MatFormFieldModule,
    MatInputModule,
    MatSelectModule,
    MatCheckboxModule,
    MatButtonModule,
    MatCardModule
  ],
  template: `
    <mat-card class="form-card">
      <mat-card-header>
        <mat-card-title>User Registration</mat-card-title>
      </mat-card-header>

      <mat-card-content>
        <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
          <mat-form-field appearance="outline">
            <mat-label>Full Name</mat-label>
            <input matInput formControlName="fullName">
            @if (userForm.get('fullName')?.hasError('required')) {
              <mat-error>Full name is required</mat-error>
            }
          </mat-form-field>

          <mat-form-field appearance="outline">
            <mat-label>Email</mat-label>
            <input matInput type="email" formControlName="email">
            @if (userForm.get('email')?.hasError('email')) {
              <mat-error>Please enter a valid email</mat-error>
            }
          </mat-form-field>

          <mat-form-field appearance="outline">
            <mat-label>Country</mat-label>
            <mat-select formControlName="country">
              @for (country of countries(); track country.code) {
                <mat-option [value]="country.code">
                  {{ country.name }}
                </mat-option>
              }
            </mat-select>
          </mat-form-field>

          <mat-checkbox formControlName="agreeToTerms">
            I agree to the terms and conditions
          </mat-checkbox>

          <div class="form-actions">
            <button
              mat-raised-button
              color="primary"
              type="submit"
              [disabled]="userForm.invalid"
            >
              Register
            </button>
          </div>
        </form>
      </mat-card-content>
    </mat-card>
  `,
  styles: [`
    .form-card {
      max-width: 500px;
      margin: 20px auto;
    }
    mat-form-field {
      width: 100%;
      margin-bottom: 15px;
    }
    .form-actions {
      margin-top: 20px;
      text-align: center;
    }
  `]
})
export class MaterialFormComponent {
  userForm: FormGroup;

  countries = signal([
    { code: 'US', name: 'United States' },
    { code: 'CA', name: 'Canada' },
    { code: 'UK', name: 'United Kingdom' },
    { code: 'DE', name: 'Germany' }
  ]);

  constructor(private fb: FormBuilder) {
    this.userForm = this.fb.group({
      fullName: ['', Validators.required],
      email: ['', [Validators.required, Validators.email]],
      country: ['', Validators.required],
      agreeToTerms: [false, Validators.requiredTrue]
    });
  }

  onSubmit() {
    if (this.userForm.valid) {
      console.log('Form submitted:', this.userForm.value);
    }
  }
}
```

---

## Summary and Migration Path

### Version Progression Overview
- **Angular 15**: Standalone APIs stable, NgOptimizedImage, MDC Web integration
- **Angular 16**: Signals preview, DestroyRef, experimental ESBuild
- **Angular 17**: New control flow preview, Vite/ESBuild default, standalone default
- **Angular 18**: Control flow stable, Material 3 stable, defer blocks stable
- **Angular 19**: Event replay stable, incremental hydration, httpResource experimental
- **Angular 20**: All signals stable, zoneless stable, template HMR stable

### Recommended Migration Strategy

1. **Start with Angular 15**: Migrate to standalone components
2. **Angular 16**: Introduce signals gradually, use DestroyRef for cleanup
3. **Angular 17**: Adopt new control flow syntax, leverage improved build performance
4. **Angular 18**: Stabilize control flow usage, implement defer blocks for performance
5. **Angular 19**: Enable event replay, explore incremental hydration
6. **Angular 20**: Adopt zoneless change detection, use stable signals throughout

### Key Takeaways for Advanced Developers

- **Signals** revolutionize state management with better performance and simpler mental model
- **Control Flow** (@if, @for, @switch) provides better performance than structural directives
- **Standalone Components** simplify application architecture and reduce boilerplate
- **Defer Blocks** enable sophisticated lazy loading strategies
- **Zoneless Change Detection** offers significant performance improvements
- **Modern Build Tools** (Vite/ESBuild) dramatically improve development experience

This overview provides a comprehensive foundation for understanding Angular's evolution and implementing modern Angular applications using the latest features and best practices.

### Related Resources

- Review the [Component lifecycle diagram](component-lifecycle.md) to align version upgrades with the hook sequence your components rely on.
