# Angular Signals Shopping App Exercise

## Overview

This exercise will guide you through building a complete shopping application using Angular Signals. You'll implement all the key concepts from the signals lesson, including basic signals, computed signals, effects, signal inputs/outputs, and the Resource API.

## Learning Objectives

By completing this exercise, you will:
- Master basic signal operations (create, read, update)
- Implement computed signals for derived state
- Use effects for side effects and persistence
- Create reusable components with signal inputs and outputs
- Handle asynchronous data with the Resource API
- Build a complete, reactive shopping application

## Prerequisites

- Angular 20+ with signals support
- Basic TypeScript and Angular knowledge
- Understanding of the concepts covered in the Angular Signals lesson

## Exercise Structure

The exercise is divided into progressive steps, each building upon the previous one:

1. **Setup & Basic Structure** - Create the app foundation
2. **Product Catalog** - Display products with basic signals
3. **Shopping Cart** - Add/remove items with computed totals
4. **Search & Filtering** - Reactive search functionality
5. **User Preferences** - Theme and settings with effects
6. **Async Data Loading** - Use Resource API for product data
7. **Advanced Features** - Wishlist and recommendations

---

## Step 1: Setup & Basic Structure

### Objective
Create the basic application structure with navigation and layout components.

### Tasks

#### 1.1 Create the Main App Component

Create `app.component.ts` with basic navigation:

```typescript
import { Component, signal } from '@angular/core';
import { CommonModule } from '@angular/common';

type ViewMode = 'catalog' | 'cart' | 'wishlist';

@Component({
  selector: 'app-root',
  imports: [CommonModule],
  template: `
    <div class="app">
      <header class="header">
        <h1>🛒 Signals Shopping</h1>
        <nav class="nav">
          <button
            [class.active]="currentView() === 'catalog'"
            (click)="setView('catalog')">
            Catalog
          </button>
          <button
            [class.active]="currentView() === 'cart'"
            (click)="setView('cart')">
            Cart ({{ cartItemCount() }})
          </button>
          <button
            [class.active]="currentView() === 'wishlist'"
            (click)="setView('wishlist')">
            Wishlist ({{ wishlistCount() }})
          </button>
        </nav>
      </header>

      <main class="main">
        @switch (currentView()) {
          @case ('catalog') {
            <div>Product Catalog - TODO</div>
          }
          @case ('cart') {
            <div>Shopping Cart - TODO</div>
          }
          @case ('wishlist') {
            <div>Wishlist - TODO</div>
          }
        }
      </main>
    </div>
  `,
  styles: [`
    .app {
      max-width: 1200px;
      margin: 0 auto;
      padding: 1rem;
    }

    .header {
      border-bottom: 2px solid #e0e0e0;
      padding-bottom: 1rem;
      margin-bottom: 2rem;
    }

    .nav {
      display: flex;
      gap: 1rem;
      margin-top: 1rem;
    }

    .nav button {
      padding: 0.5rem 1rem;
      border: 1px solid #ddd;
      background: white;
      cursor: pointer;
      border-radius: 4px;
    }

    .nav button.active {
      background: #007acc;
      color: white;
    }

    .main {
      min-height: 400px;
    }
  `]
})
export class AppComponent {
  // TODO: Implement signals for view management and counters
}
```

#### 1.2 Your Implementation

Implement the missing signals in `AppComponent`:

1. Create a signal for the current view mode
2. Create a method to change the view
3. Create placeholder signals for cart and wishlist counts that return 0 for now

---

## Step 2: Product Catalog with Basic Signals

### Objective
Create a product catalog component that displays products and allows adding them to cart.

### Tasks

#### 2.1 Define Product Interface

First, create the product data structure:

```typescript
export interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
  category: string;
  imageUrl: string;
  stock: number;
  rating: number;
}

export interface CartItem {
  product: Product;
  quantity: number;
}
```

#### 2.2 Create Product Card Component

Create `product-card.component.ts`:

```typescript
import { Component, input, output } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-product-card',
  imports: [CommonModule],
  template: `
    <div class="product-card">
      <img [src]="product().imageUrl" [alt]="product().name" class="product-image">

      <div class="product-info">
        <h3 class="product-name">{{ product().name }}</h3>
        <p class="product-description">{{ product().description }}</p>

        <div class="product-details">
          <span class="price">\${{ product().price }}</span>
          <span class="rating">⭐ {{ product().rating }}/5</span>
        </div>

        <div class="product-stock">
          @if (product().stock > 0) {
            <span class="in-stock">{{ product().stock }} in stock</span>
          } @else {
            <span class="out-of-stock">Out of stock</span>
          }
        </div>
      </div>

      <div class="product-actions">
        <button
          class="btn-primary"
          [disabled]="product().stock === 0"
          (click)="onAddToCart()">
          Add to Cart
        </button>
        <button
          class="btn-secondary"
          (click)="onAddToWishlist()">
          ♡ Wishlist
        </button>
      </div>
    </div>
  `,
  styles: [`
    .product-card {
      border: 1px solid #ddd;
      border-radius: 8px;
      overflow: hidden;
      transition: transform 0.2s;
    }

    .product-card:hover {
      transform: translateY(-2px);
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    }

    .product-image {
      width: 100%;
      height: 200px;
      object-fit: cover;
    }

    .product-info {
      padding: 1rem;
    }

    .product-name {
      margin: 0 0 0.5rem 0;
      font-size: 1.2rem;
    }

    .product-description {
      color: #666;
      font-size: 0.9rem;
      margin-bottom: 1rem;
    }

    .product-details {
      display: flex;
      justify-content: space-between;
      margin-bottom: 0.5rem;
    }

    .price {
      font-weight: bold;
      color: #007acc;
      font-size: 1.1rem;
    }

    .product-stock {
      margin-bottom: 1rem;
    }

    .in-stock {
      color: #28a745;
      font-size: 0.9rem;
    }

    .out-of-stock {
      color: #dc3545;
      font-size: 0.9rem;
    }

    .product-actions {
      padding: 1rem;
      display: flex;
      gap: 0.5rem;
    }

    .btn-primary, .btn-secondary {
      flex: 1;
      padding: 0.5rem;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }

    .btn-primary {
      background: #007acc;
      color: white;
    }

    .btn-primary:disabled {
      background: #ccc;
      cursor: not-allowed;
    }

    .btn-secondary {
      background: white;
      border: 1px solid #ddd;
    }
  `]
})
export class ProductCardComponent {
  // TODO: Implement inputs and outputs
}
```

#### 2.3 Your Implementation

1. Add the required input for the product
2. Add outputs for add to cart and add to wishlist events
3. Implement the event handler methods

#### 2.4 Create Product Catalog Component

Create `product-catalog.component.ts`:

```typescript
import { Component, signal, output } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ProductCardComponent } from './product-card.component';

@Component({
  selector: 'app-product-catalog',
  imports: [CommonModule, ProductCardComponent],
  template: `
    <div class="catalog">
      <div class="catalog-header">
        <h2>Products ({{ products().length }})</h2>
        <div class="category-filter">
          <label>Category:</label>
          <select [value]="selectedCategory()" (change)="onCategoryChange($event)">
            <option value="">All Categories</option>
            @for (category of categories(); track category) {
              <option [value]="category">{{ category }}</option>
            }
          </select>
        </div>
      </div>

      <div class="products-grid">
        @for (product of filteredProducts(); track product.id) {
          <app-product-card
            [product]="product"
            (addToCart)="onAddToCart($event)"
            (addToWishlist)="onAddToWishlist($event)">
          </app-product-card>
        }
      </div>

      @if (filteredProducts().length === 0) {
        <div class="no-products">
          <p>No products found in this category.</p>
        </div>
      }
    </div>
  `,
  styles: [`
    .catalog {
      padding: 1rem;
    }

    .catalog-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 2rem;
    }

    .category-filter {
      display: flex;
      align-items: center;
      gap: 0.5rem;
    }

    .category-filter select {
      padding: 0.5rem;
      border: 1px solid #ddd;
      border-radius: 4px;
    }

    .products-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
      gap: 1.5rem;
    }

    .no-products {
      text-align: center;
      padding: 3rem;
      color: #666;
    }
  `]
})
export class ProductCatalogComponent {
  // Sample products data
  products = signal<Product[]>([
    {
      id: 1,
      name: 'Laptop Pro',
      description: 'High-performance laptop for professionals',
      price: 1299.99,
      category: 'Electronics',
      imageUrl: 'https://images.unsplash.com/photo-1496181133206-80ce9b88a853?w=300',
      stock: 5,
      rating: 4.8
    },
    {
      id: 2,
      name: 'Coffee Maker',
      description: 'Premium coffee maker for perfect brew',
      price: 89.99,
      category: 'Home & Kitchen',
      imageUrl: 'https://images.unsplash.com/photo-1495474472287-4d71bcdd2085?w=300',
      stock: 12,
      rating: 4.5
    },
    {
      id: 3,
      name: 'Running Shoes',
      description: 'Comfortable running shoes for daily exercise',
      price: 129.99,
      category: 'Sports',
      imageUrl: 'https://images.unsplash.com/photo-1542291026-7eec264c27ff?w=300',
      stock: 0,
      rating: 4.7
    },
    {
      id: 4,
      name: 'Smartphone',
      description: 'Latest smartphone with advanced features',
      price: 799.99,
      category: 'Electronics',
      imageUrl: 'https://images.unsplash.com/photo-1511707171634-5f897ff02aa9?w=300',
      stock: 8,
      rating: 4.6
    },
    {
      id: 5,
      name: 'Cooking Set',
      description: 'Complete cooking set for home chefs',
      price: 159.99,
      category: 'Home & Kitchen',
      imageUrl: 'https://images.unsplash.com/photo-1556909114-f6e7ad7d3136?w=300',
      stock: 3,
      rating: 4.4
    }
  ]);

  // TODO: Implement category filtering and event handling
}
```

#### 2.5 Your Implementation

Implement the missing functionality:

1. Create a signal for selected category
2. Create a computed signal for unique categories
3. Create a computed signal for filtered products
4. Implement category change handler
5. Add outputs for cart and wishlist events
6. Implement event handlers

---

## Step 3: Shopping Cart with Computed Signals

### Objective
Create a shopping cart that manages items, quantities, and calculates totals using computed signals.

### Tasks

#### 3.1 Create Shopping Cart Service

Create `shopping-cart.service.ts`:

```typescript
import { Injectable, signal, computed } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class ShoppingCartService {
  // Cart items state
  private cartItems = signal<CartItem[]>([]);

  // TODO: Implement cart management methods and computed values
}
```

#### 3.2 Your Implementation

Implement the complete shopping cart service:

1. Create computed signals for cart summary
2. Implement methods to add, remove, and update items
3. Add methods to clear cart and get specific items

#### 3.3 Create Shopping Cart Component

Create `shopping-cart.component.ts`:

```typescript
import { Component, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ShoppingCartService } from './shopping-cart.service';

@Component({
  selector: 'app-shopping-cart',
  imports: [CommonModule],
  template: `
    <div class="cart">
      <div class="cart-header">
        <h2>Shopping Cart</h2>
        @if (!cartService.isEmpty()) {
          <button class="btn-clear" (click)="cartService.clearCart()">
            Clear Cart
          </button>
        }
      </div>

      @if (cartService.isEmpty()) {
        <div class="empty-cart">
          <p>Your cart is empty</p>
          <p>Add some products to get started!</p>
        </div>
      } @else {
        <div class="cart-items">
          @for (item of cartService.items(); track item.product.id) {
            <div class="cart-item">
              <img [src]="item.product.imageUrl" [alt]="item.product.name" class="item-image">

              <div class="item-details">
                <h3>{{ item.product.name }}</h3>
                <p>{{ item.product.description }}</p>
                <span class="item-price">\${{ item.product.price }}</span>
              </div>

              <div class="item-controls">
                <div class="quantity-controls">
                  <button
                    (click)="decreaseQuantity(item.product.id)"
                    [disabled]="item.quantity <= 1">
                    -
                  </button>
                  <input
                    type="number"
                    [value]="item.quantity"
                    (change)="onQuantityChange(item.product.id, $event)"
                    min="1">
                  <button (click)="increaseQuantity(item.product.id)">
                    +
                  </button>
                </div>

                <div class="item-total">
                  \${{ (item.product.price * item.quantity).toFixed(2) }}
                </div>

                <button
                  class="btn-remove"
                  (click)="cartService.removeFromCart(item.product.id)">
                  Remove
                </button>
              </div>
            </div>
          }
        </div>

        <div class="cart-summary">
          <div class="summary-row">
            <span>Items: {{ cartService.itemCount() }}</span>
            <span>Total: \${{ cartService.totalPrice().toFixed(2) }}</span>
          </div>
          <button class="btn-checkout">
            Proceed to Checkout
          </button>
        </div>
      }
    </div>
  `,
  styles: [`
    .cart {
      padding: 1rem;
      max-width: 800px;
      margin: 0 auto;
    }

    .cart-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 2rem;
    }

    .btn-clear {
      padding: 0.5rem 1rem;
      background: #dc3545;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }

    .empty-cart {
      text-align: center;
      padding: 3rem;
      color: #666;
    }

    .cart-item {
      display: flex;
      gap: 1rem;
      padding: 1rem;
      border: 1px solid #ddd;
      border-radius: 8px;
      margin-bottom: 1rem;
    }

    .item-image {
      width: 80px;
      height: 80px;
      object-fit: cover;
      border-radius: 4px;
    }

    .item-details {
      flex: 1;
    }

    .item-details h3 {
      margin: 0 0 0.5rem 0;
    }

    .item-details p {
      color: #666;
      margin: 0 0 0.5rem 0;
    }

    .item-price {
      font-weight: bold;
      color: #007acc;
    }

    .item-controls {
      display: flex;
      flex-direction: column;
      gap: 0.5rem;
      align-items: flex-end;
    }

    .quantity-controls {
      display: flex;
      align-items: center;
      gap: 0.5rem;
    }

    .quantity-controls button {
      width: 30px;
      height: 30px;
      border: 1px solid #ddd;
      background: white;
      cursor: pointer;
    }

    .quantity-controls input {
      width: 60px;
      text-align: center;
      border: 1px solid #ddd;
      padding: 0.25rem;
    }

    .item-total {
      font-weight: bold;
      font-size: 1.1rem;
    }

    .btn-remove {
      padding: 0.25rem 0.5rem;
      background: #dc3545;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      font-size: 0.9rem;
    }

    .cart-summary {
      border-top: 2px solid #e0e0e0;
      padding-top: 1rem;
      margin-top: 1rem;
    }

    .summary-row {
      display: flex;
      justify-content: space-between;
      font-size: 1.2rem;
      font-weight: bold;
      margin-bottom: 1rem;
    }

    .btn-checkout {
      width: 100%;
      padding: 1rem;
      background: #28a745;
      color: white;
      border: none;
      border-radius: 4px;
      font-size: 1.1rem;
      cursor: pointer;
    }
  `]
})
export class ShoppingCartComponent {
  cartService = inject(ShoppingCartService);

  // TODO: Implement quantity management methods
}
```

#### 3.4 Your Implementation

Implement the quantity management methods:

#### 3.5 Update Main App Component

Update `app.component.ts` to integrate the cart:

1. Inject the `ShoppingCartService`
2. Replace the placeholder cart count with the real count
3. Add the shopping cart component
4. Handle the cart events from the product catalog

---

## Step 4: Search & Filtering with Reactive Signals

### Objective
Add a search component that reactively filters products based on user input.

### Tasks

#### 4.1 Create Search Component

Create `product-search.component.ts`:

```typescript
import { Component, signal, output, model } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-product-search',
  imports: [CommonModule],
  template: `
    <div class="search-container">
      <div class="search-bar">
        <input
          type="text"
          [(value)]="searchQuery"
          placeholder="Search products..."
          class="search-input">
        <button
          class="clear-search"
          (click)="clearSearch()"
          [style.display]="searchQuery() ? 'block' : 'none'">
          ✕
        </button>
      </div>

      <div class="search-filters">
        <div class="price-range">
          <label>Price Range:</label>
          <select [value]="priceRange()" (change)="onPriceRangeChange($event)">
            <option value="">Any Price</option>
            <option value="0-50">Under $50</option>
            <option value="50-200">$50 - $200</option>
            <option value="200-500">$200 - $500</option>
            <option value="500+">$500+</option>
          </select>
        </div>

        <div class="sort-options">
          <label>Sort by:</label>
          <select [value]="sortBy()" (change)="onSortChange($event)">
            <option value="name">Name</option>
            <option value="price-low">Price: Low to High</option>
            <option value="price-high">Price: High to Low</option>
            <option value="rating">Rating</option>
          </select>
        </div>
      </div>

      @if (searchQuery()) {
        <div class="search-info">
          Searching for: "{{ searchQuery() }}"
        </div>
      }
    </div>
  `,
  styles: [`
    .search-container {
      background: #f8f9fa;
      padding: 1rem;
      border-radius: 8px;
      margin-bottom: 1rem;
    }

    .search-bar {
      position: relative;
      margin-bottom: 1rem;
    }

    .search-input {
      width: 100%;
      padding: 0.75rem;
      border: 1px solid #ddd;
      border-radius: 4px;
      font-size: 1rem;
    }

    .clear-search {
      position: absolute;
      right: 8px;
      top: 50%;
      transform: translateY(-50%);
      background: none;
      border: none;
      cursor: pointer;
      color: #666;
      padding: 0.25rem;
    }

    .search-filters {
      display: flex;
      gap: 2rem;
      flex-wrap: wrap;
    }

    .price-range, .sort-options {
      display: flex;
      align-items: center;
      gap: 0.5rem;
    }

    .search-filters select {
      padding: 0.5rem;
      border: 1px solid #ddd;
      border-radius: 4px;
    }

    .search-info {
      margin-top: 1rem;
      color: #666;
      font-style: italic;
    }
  `]
})
export class ProductSearchComponent {
  // TODO: Implement search signals and methods
}
```

#### 4.2 Your Implementation

Implement the search functionality:

1. Create model inputs for two-way binding
2. Create outputs for filter changes
3. Implement event handlers

#### 4.3 Update Product Catalog

Update `product-catalog.component.ts` to include search functionality:

1. Add the search component
2. Create signals for search filters
3. Update the filtered products computed signal
4. Implement search handling

---

## Step 5: User Preferences with Effects

### Objective
Add user preferences (theme, currency) that persist to localStorage using effects.

### Tasks

#### 5.1 Create User Preferences Service

Create `user-preferences.service.ts`:

```typescript
import { Injectable, signal, effect } from '@angular/core';

export type Theme = 'light' | 'dark';
export type Currency = 'USD' | 'EUR' | 'GBP';

export interface UserPreferences {
  theme: Theme;
  currency: Currency;
  showOutOfStock: boolean;
  itemsPerPage: number;
}

@Injectable({
  providedIn: 'root'
})
export class UserPreferencesService {
  // Default preferences
  private readonly defaultPreferences: UserPreferences = {
    theme: 'light',
    currency: 'USD',
    showOutOfStock: true,
    itemsPerPage: 12
  };

  // TODO: Implement preferences signals and effects
}
```

#### 5.2 Your Implementation

Implement the complete preferences service:

#### 5.3 Create Settings Component

Create `settings.component.ts`:

```typescript
import { Component, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { UserPreferencesService } from './user-preferences.service';

@Component({
  selector: 'app-settings',
  imports: [CommonModule],
  template: `
    <div class="settings">
      <h2>Settings</h2>

      <div class="setting-group">
        <label>Theme</label>
        <div class="theme-options">
          <button
            [class.active]="preferences.theme() === 'light'"
            (click)="preferences.updateTheme('light')">
            ☀️ Light
          </button>
          <button
            [class.active]="preferences.theme() === 'dark'"
            (click)="preferences.updateTheme('dark')">
            🌙 Dark
          </button>
        </div>
      </div>

      <div class="setting-group">
        <label for="currency">Currency</label>
        <select
          id="currency"
          [value]="preferences.currency()"
          (change)="onCurrencyChange($event)">
          <option value="USD">USD ($)</option>
          <option value="EUR">EUR (€)</option>
          <option value="GBP">GBP (£)</option>
        </select>
      </div>

      <div class="setting-group">
        <label class="checkbox-label">
          <input
            type="checkbox"
            [checked]="preferences.showOutOfStock()"
            (change)="preferences.toggleOutOfStock()">
          Show out of stock items
        </label>
      </div>

      <div class="setting-group">
        <label for="itemsPerPage">Items per page</label>
        <select
          id="itemsPerPage"
          [value]="preferences.itemsPerPage()"
          (change)="onItemsPerPageChange($event)">
          <option value="6">6</option>
          <option value="12">12</option>
          <option value="24">24</option>
          <option value="48">48</option>
        </select>
      </div>
    </div>
  `,
  styles: [`
    .settings {
      max-width: 600px;
      margin: 0 auto;
      padding: 2rem;
    }

    .setting-group {
      margin-bottom: 2rem;
    }

    .setting-group label {
      display: block;
      margin-bottom: 0.5rem;
      font-weight: bold;
    }

    .theme-options {
      display: flex;
      gap: 1rem;
    }

    .theme-options button {
      padding: 0.5rem 1rem;
      border: 1px solid #ddd;
      background: white;
      cursor: pointer;
      border-radius: 4px;
    }

    .theme-options button.active {
      background: #007acc;
      color: white;
    }

    select {
      padding: 0.5rem;
      border: 1px solid #ddd;
      border-radius: 4px;
      min-width: 120px;
    }

    .checkbox-label {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      cursor: pointer;
    }
  `]
})
export class SettingsComponent {
  preferences = inject(UserPreferencesService);

  onCurrencyChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    this.preferences.updateCurrency(target.value as any);
  }

  onItemsPerPageChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    this.preferences.updateItemsPerPage(Number(target.value));
  }
}
```

#### 5.4 Add Global Styles for Themes

Add these styles to your global CSS or in the main component:

```css
/* Light theme (default) */
[data-theme="light"] {
  --bg-color: #ffffff;
  --text-color: #333333;
  --border-color: #dddddd;
  --card-bg: #ffffff;
}

/* Dark theme */
[data-theme="dark"] {
  --bg-color: #1a1a1a;
  --text-color: #ffffff;
  --border-color: #444444;
  --card-bg: #2d2d2d;
}

body {
  background-color: var(--bg-color);
  color: var(--text-color);
  transition: background-color 0.3s, color 0.3s;
}

.product-card, .cart-item {
  background-color: var(--card-bg);
  border-color: var(--border-color);
}
```

#### 5.5 Update Components to Use Preferences

Update your product card and other components to use the formatted prices.

---

## Step 6: Async Data Loading with Resource API

### Objective
Replace static product data with async data loading using the Resource API.

### Tasks

#### 6.1 Create Products Service

Create `products.service.ts`:

```typescript
import { Injectable, signal, resource } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class ProductsService {
  // API endpoint (simulated)
  private readonly apiUrl = 'https://jsonplaceholder.typicode.com/photos';

  // Search parameters
  private searchParams = signal({
    category: '',
    query: '',
    page: 1,
    limit: 20
  });

  // TODO: Implement resource for products loading
}
```

#### 6.2 Your Implementation

Implement the products resource:

#### 6.3 Update Product Catalog to Use Resource

Update `product-catalog.component.ts` to use the products service:

---

## Step 7: Advanced Features - Wishlist and Recommendations

### Objective
Implement wishlist functionality and product recommendations using advanced signal patterns.

### Tasks

#### 7.1 Create Wishlist Service

Create `wishlist.service.ts`:

```typescript
import { Injectable, signal, computed, effect } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class WishlistService {
  // Wishlist items (just product IDs for simplicity)
  private wishlistItems = signal<number[]>([]);

  // TODO: Implement wishlist management
}
```

#### 7.2 Your Implementation

Implement the complete wishlist service:

#### 7.3 Create Recommendations Service

Create `recommendations.service.ts`:

```typescript
import { Injectable, signal, computed, inject } from '@angular/core';
import { ShoppingCartService } from './shopping-cart.service';
import { WishlistService } from './wishlist.service';

@Injectable({
  providedIn: 'root'
})
export class RecommendationsService {
  private cartService = inject(ShoppingCartService);
  private wishlistService = inject(WishlistService);

  // TODO: Implement recommendation logic
}
```

#### 7.4 Your Implementation

Implement the recommendations service:

#### 7.5 Create Wishlist Component

Create `wishlist.component.ts`:

```typescript
import { Component, inject, computed } from '@angular/core';
import { CommonModule } from '@angular/common';
import { WishlistService } from './wishlist.service';
import { ProductsService } from './products.service';
import { ProductCardComponent } from './product-card.component';

@Component({
  selector: 'app-wishlist',
  imports: [CommonModule, ProductCardComponent],
  template: `
    <div class="wishlist">
      <div class="wishlist-header">
        <h2>My Wishlist</h2>
        @if (!wishlistService.isEmpty()) {
          <button class="btn-clear" (click)="wishlistService.clearWishlist()">
            Clear Wishlist
          </button>
        }
      </div>

      @if (wishlistService.isEmpty()) {
        <div class="empty-wishlist">
          <p>Your wishlist is empty</p>
          <p>Add some products you'd like to buy later!</p>
        </div>
      } @else if (productsService.productsResource.isLoading()) {
        <div class="loading">Loading wishlist items...</div>
      } @else {
        <div class="wishlist-grid">
          @for (product of wishlistProducts(); track product.id) {
            <app-product-card
              [product]="product"
              (addToCart)="onAddToCart($event)"
              (addToWishlist)="onRemoveFromWishlist($event)">
            </app-product-card>
          }
        </div>
      }
    </div>
  `,
  styles: [`
    .wishlist {
      padding: 1rem;
    }

    .wishlist-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 2rem;
    }

    .btn-clear {
      padding: 0.5rem 1rem;
      background: #dc3545;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }

    .empty-wishlist {
      text-align: center;
      padding: 3rem;
      color: #666;
    }

    .loading {
      text-align: center;
      padding: 3rem;
    }

    .wishlist-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
      gap: 1.5rem;
    }
  `]
})
export class WishlistComponent {
  wishlistService = inject(WishlistService);
  productsService = inject(ProductsService);

  // TODO: Implement wishlist products computation and event handlers
}
```

#### 7.6 Your Implementation

Complete the wishlist component:

#### 7.7 Update Product Card to Show Wishlist Status

Update `product-card.component.ts` to show wishlist status:

#### 7.8 Add Recommendations Section

Update `app.component.ts` to include recommendations:

---

## Bonus Challenges

### Challenge 1: Product Reviews
Add a review system with ratings and comments using signals.

### Challenge 2: Order History
Implement order history using the Resource API to fetch past orders.

### Challenge 3: Real-time Inventory
Use effects to simulate real-time inventory updates.

### Challenge 4: Advanced Search
Add faceted search with multiple filters and auto-suggestions.

### Challenge 5: Progressive Web App
Add service worker functionality for offline cart management.

---

## Testing Your Implementation

### Manual Testing Checklist

1. **Basic Navigation**
   - [ ] Can switch between Catalog, Cart, and Wishlist views
   - [ ] Navigation shows correct item counts

2. **Product Catalog**
   - [ ] Products display correctly with images, prices, and details
   - [ ] Category filtering works
   - [ ] Search functionality filters products
   - [ ] Price range filtering works
   - [ ] Sorting by different criteria works
   - [ ] Loading states display properly

3. **Shopping Cart**
   - [ ] Can add products to cart
   - [ ] Can update quantities
   - [ ] Can remove items
   - [ ] Cart totals calculate correctly
   - [ ] Empty cart state displays properly

4. **Wishlist**
   - [ ] Can add/remove products from wishlist
   - [ ] Wishlist status persists between sessions
   - [ ] Can move items from wishlist to cart

5. **User Preferences**
   - [ ] Theme switching works (light/dark)
   - [ ] Currency conversion displays correctly
   - [ ] Settings persist between sessions
   - [ ] Out of stock toggle affects display

6. **Data Loading**
   - [ ] Loading states display during data fetch
   - [ ] Error states handle failures gracefully
   - [ ] Retry functionality works

### Code Quality Checklist

1. **Signals Usage**
   - [ ] Used `signal()` for state management
   - [ ] Used `computed()` for derived state
   - [ ] Used `effect()` for side effects
   - [ ] Used signal inputs/outputs for component communication

2. **Angular Best Practices**
   - [ ] Used standalone components
   - [ ] Used `ChangeDetectionStrategy.OnPush`
   - [ ] Used modern control flow (`@if`, `@for`, `@switch`)
   - [ ] Used `inject()` for dependency injection

3. **TypeScript**
   - [ ] Proper type definitions for interfaces
   - [ ] No use of `any` type
   - [ ] Consistent naming conventions

---

## Summary

Congratulations! You've built a complete shopping application using Angular Signals. You've learned and implemented:

1. **Basic Signals** - Managing state with `signal()`, `set()`, and `update()`
2. **Computed Signals** - Deriving state with automatic dependency tracking
3. **Effects** - Handling side effects and persistence
4. **Signal Inputs/Outputs** - Component communication with reactive patterns
5. **Model Inputs** - Two-way data binding
6. **Resource API** - Async data loading with built-in loading states
7. **Advanced Patterns** - Complex state management and reactive UIs

This exercise demonstrates how Angular Signals provide a powerful, reactive foundation for building modern web applications with better performance and developer experience.

### Next Steps

- Explore more advanced signal patterns
- Add unit tests for your components and services
- Implement additional features like user authentication
- Consider migrating existing Angular applications to use signals
- Learn about signal-based forms and other Angular features