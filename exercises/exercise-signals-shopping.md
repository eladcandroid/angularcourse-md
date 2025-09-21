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

<details>
<summary>💡 Hint</summary>

```typescript
currentView = signal<ViewMode>('catalog');
cartItemCount = signal(0); // Will be computed later
wishlistCount = signal(0); // Will be computed later

setView(view: ViewMode) {
  this.currentView.set(view);
}
```

</details>

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

<details>
<summary>💡 Hint</summary>

```typescript
// Input for product data
product = input.required<Product>();

// Outputs for user actions
addToCart = output<Product>();
addToWishlist = output<Product>();

onAddToCart() {
  this.addToCart.emit(this.product());
}

onAddToWishlist() {
  this.addToWishlist.emit(this.product());
}
```

</details>

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

<details>
<summary>💡 Hint</summary>

```typescript
import { computed } from '@angular/core';

selectedCategory = signal('');

// Computed signal for unique categories
categories = computed(() => {
  const cats = this.products().map(p => p.category);
  return [...new Set(cats)].sort();
});

// Computed signal for filtered products
filteredProducts = computed(() => {
  const category = this.selectedCategory();
  if (!category) return this.products();
  return this.products().filter(p => p.category === category);
});

// Outputs
productAddedToCart = output<Product>();
productAddedToWishlist = output<Product>();

onCategoryChange(event: Event) {
  const target = event.target as HTMLSelectElement;
  this.selectedCategory.set(target.value);
}

onAddToCart(product: Product) {
  this.productAddedToCart.emit(product);
}

onAddToWishlist(product: Product) {
  this.productAddedToWishlist.emit(product);
}
```

</details>

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

<details>
<summary>💡 Hint</summary>

```typescript
// Public readonly access to cart items
readonly items = this.cartItems.asReadonly();

// Computed values
readonly itemCount = computed(() =>
  this.cartItems().reduce((total, item) => total + item.quantity, 0)
);

readonly totalPrice = computed(() =>
  this.cartItems().reduce((total, item) =>
    total + (item.product.price * item.quantity), 0
  )
);

readonly isEmpty = computed(() => this.cartItems().length === 0);

// Cart management methods
addToCart(product: Product, quantity: number = 1) {
  this.cartItems.update(items => {
    const existingItem = items.find(item => item.product.id === product.id);

    if (existingItem) {
      return items.map(item =>
        item.product.id === product.id
          ? { ...item, quantity: item.quantity + quantity }
          : item
      );
    } else {
      return [...items, { product, quantity }];
    }
  });
}

removeFromCart(productId: number) {
  this.cartItems.update(items =>
    items.filter(item => item.product.id !== productId)
  );
}

updateQuantity(productId: number, quantity: number) {
  if (quantity <= 0) {
    this.removeFromCart(productId);
    return;
  }

  this.cartItems.update(items =>
    items.map(item =>
      item.product.id === productId
        ? { ...item, quantity }
        : item
    )
  );
}

clearCart() {
  this.cartItems.set([]);
}
```

</details>

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

<details>
<summary>💡 Hint</summary>

```typescript
increaseQuantity(productId: number) {
  const item = this.cartService.items().find(item => item.product.id === productId);
  if (item) {
    this.cartService.updateQuantity(productId, item.quantity + 1);
  }
}

decreaseQuantity(productId: number) {
  const item = this.cartService.items().find(item => item.product.id === productId);
  if (item && item.quantity > 1) {
    this.cartService.updateQuantity(productId, item.quantity - 1);
  }
}

onQuantityChange(productId: number, event: Event) {
  const target = event.target as HTMLInputElement;
  const quantity = parseInt(target.value, 10);
  if (!isNaN(quantity) && quantity > 0) {
    this.cartService.updateQuantity(productId, quantity);
  }
}
```

</details>

#### 3.5 Update Main App Component

Update `app.component.ts` to integrate the cart:

1. Inject the `ShoppingCartService`
2. Replace the placeholder cart count with the real count
3. Add the shopping cart component
4. Handle the cart events from the product catalog

<details>
<summary>💡 Hint</summary>

```typescript
import { ShoppingCartService } from './shopping-cart.service';
import { ProductCatalogComponent } from './product-catalog.component';
import { ShoppingCartComponent } from './shopping-cart.component';

// In the component class:
cartService = inject(ShoppingCartService);

// Replace the placeholder with:
cartItemCount = this.cartService.itemCount;

// In the template, replace the TODO sections:
@case ('catalog') {
  <app-product-catalog
    (productAddedToCart)="onAddToCart($event)"
    (productAddedToWishlist)="onAddToWishlist($event)">
  </app-product-catalog>
}
@case ('cart') {
  <app-shopping-cart></app-shopping-cart>
}

// Add methods:
onAddToCart(product: Product) {
  this.cartService.addToCart(product);
}

onAddToWishlist(product: Product) {
  // TODO: Implement in next step
  console.log('Added to wishlist:', product.name);
}
```

</details>

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

<details>
<summary>💡 Hint</summary>

```typescript
// Model inputs for two-way binding
searchQuery = model('');
priceRange = model('');
sortBy = model('name');

// Outputs for parent component
searchChanged = output<{
  query: string;
  priceRange: string;
  sortBy: string;
}>();

constructor() {
  // Emit changes whenever any filter changes
  effect(() => {
    this.searchChanged.emit({
      query: this.searchQuery(),
      priceRange: this.priceRange(),
      sortBy: this.sortBy()
    });
  });
}

clearSearch() {
  this.searchQuery.set('');
}

onPriceRangeChange(event: Event) {
  const target = event.target as HTMLSelectElement;
  this.priceRange.set(target.value);
}

onSortChange(event: Event) {
  const target = event.target as HTMLSelectElement;
  this.sortBy.set(target.value);
}
```

</details>

#### 4.3 Update Product Catalog

Update `product-catalog.component.ts` to include search functionality:

1. Add the search component
2. Create signals for search filters
3. Update the filtered products computed signal
4. Implement search handling

<details>
<summary>💡 Hint</summary>

```typescript
import { ProductSearchComponent } from './product-search.component';

// Add to imports array
imports: [CommonModule, ProductCardComponent, ProductSearchComponent]

// Add search signals
searchQuery = signal('');
priceRange = signal('');
sortBy = signal('name');

// Update the template to include search
template: `
  <div class="catalog">
    <app-product-search
      [(searchQuery)]="searchQuery"
      [(priceRange)]="priceRange"
      [(sortBy)]="sortBy">
    </app-product-search>

    <!-- rest of template -->
  `,

// Update filteredProducts computed signal
filteredProducts = computed(() => {
  let products = this.products();

  // Filter by category
  const category = this.selectedCategory();
  if (category) {
    products = products.filter(p => p.category === category);
  }

  // Filter by search query
  const query = this.searchQuery().toLowerCase();
  if (query) {
    products = products.filter(p =>
      p.name.toLowerCase().includes(query) ||
      p.description.toLowerCase().includes(query)
    );
  }

  // Filter by price range
  const priceRange = this.priceRange();
  if (priceRange) {
    products = products.filter(p => {
      const price = p.price;
      switch (priceRange) {
        case '0-50': return price < 50;
        case '50-200': return price >= 50 && price < 200;
        case '200-500': return price >= 200 && price < 500;
        case '500+': return price >= 500;
        default: return true;
      }
    });
  }

  // Sort products
  const sortBy = this.sortBy();
  return [...products].sort((a, b) => {
    switch (sortBy) {
      case 'name': return a.name.localeCompare(b.name);
      case 'price-low': return a.price - b.price;
      case 'price-high': return b.price - a.price;
      case 'rating': return b.rating - a.rating;
      default: return 0;
    }
  });
});
```

</details>

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

<details>
<summary>💡 Hint</summary>

```typescript
// Load initial preferences from localStorage
private loadPreferences(): UserPreferences {
  try {
    const stored = localStorage.getItem('userPreferences');
    if (stored) {
      return { ...this.defaultPreferences, ...JSON.parse(stored) };
    }
  } catch (error) {
    console.warn('Failed to load preferences:', error);
  }
  return this.defaultPreferences;
}

// Preference signals
private preferences = signal<UserPreferences>(this.loadPreferences());

// Public readonly access
readonly theme = computed(() => this.preferences().theme);
readonly currency = computed(() => this.preferences().currency);
readonly showOutOfStock = computed(() => this.preferences().showOutOfStock);
readonly itemsPerPage = computed(() => this.preferences().itemsPerPage);

// Exchange rates (simplified)
private exchangeRates = signal({
  USD: 1,
  EUR: 0.85,
  GBP: 0.73
});

constructor() {
  // Effect to persist preferences to localStorage
  effect(() => {
    try {
      localStorage.setItem('userPreferences', JSON.stringify(this.preferences()));
    } catch (error) {
      console.warn('Failed to save preferences:', error);
    }
  });

  // Effect to apply theme to document
  effect(() => {
    document.body.setAttribute('data-theme', this.theme());
  });
}

// Methods to update preferences
updateTheme(theme: Theme) {
  this.preferences.update(prefs => ({ ...prefs, theme }));
}

updateCurrency(currency: Currency) {
  this.preferences.update(prefs => ({ ...prefs, currency }));
}

toggleOutOfStock() {
  this.preferences.update(prefs => ({
    ...prefs,
    showOutOfStock: !prefs.showOutOfStock
  }));
}

updateItemsPerPage(itemsPerPage: number) {
  this.preferences.update(prefs => ({ ...prefs, itemsPerPage }));
}

// Currency conversion
convertPrice(price: number): number {
  const rate = this.exchangeRates()[this.currency()];
  return price * rate;
}

formatPrice(price: number): string {
  const converted = this.convertPrice(price);
  const currency = this.currency();

  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: currency
  }).format(converted);
}
```

</details>

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

Update your product card and other components to use the formatted prices:

<details>
<summary>💡 Hint</summary>

In `ProductCardComponent`, inject the preferences service and use formatted prices:

```typescript
import { UserPreferencesService } from './user-preferences.service';

export class ProductCardComponent {
  product = input.required<Product>();
  preferences = inject(UserPreferencesService);

  // In template, replace:
  // <span class="price">${{ product().price }}</span>
  // with:
  // <span class="price">{{ preferences.formatPrice(product().price) }}</span>
}
```

Do the same in `ShoppingCartComponent` and other components that display prices.
</details>

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

<details>
<summary>💡 Hint</summary>

```typescript
// Products resource that fetches data based on search params
productsResource = resource({
  request: () => this.searchParams(),

  loader: async ({ request }) => {
    // Simulate API delay
    await new Promise(resolve => setTimeout(resolve, 1000));

    // Simulate fetching data (in real app, this would be HTTP request)
    const allProducts: Product[] = [
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
      },
      {
        id: 6,
        name: 'Gaming Mouse',
        description: 'High-precision gaming mouse',
        price: 79.99,
        category: 'Electronics',
        imageUrl: 'https://images.unsplash.com/photo-1527864550417-7fd91fc51a46?w=300',
        stock: 15,
        rating: 4.6
      },
      {
        id: 7,
        name: 'Yoga Mat',
        description: 'Premium yoga mat for daily practice',
        price: 49.99,
        category: 'Sports',
        imageUrl: 'https://images.unsplash.com/photo-1544947950-fa07a98d237f?w=300',
        stock: 20,
        rating: 4.3
      },
      {
        id: 8,
        name: 'Blender',
        description: 'High-speed blender for smoothies',
        price: 149.99,
        category: 'Home & Kitchen',
        imageUrl: 'https://images.unsplash.com/photo-1570197788417-0e82375c9371?w=300',
        stock: 7,
        rating: 4.5
      }
    ];

    // Apply filters
    let filtered = allProducts;

    if (request.category) {
      filtered = filtered.filter(p => p.category === request.category);
    }

    if (request.query) {
      const query = request.query.toLowerCase();
      filtered = filtered.filter(p =>
        p.name.toLowerCase().includes(query) ||
        p.description.toLowerCase().includes(query)
      );
    }

    // Simulate pagination
    const start = (request.page - 1) * request.limit;
    const paginatedProducts = filtered.slice(start, start + request.limit);

    return {
      products: paginatedProducts,
      total: filtered.length,
      page: request.page,
      totalPages: Math.ceil(filtered.length / request.limit)
    };
  }
});

// Public methods
updateSearch(category: string = '', query: string = '', page: number = 1) {
  this.searchParams.set({
    category,
    query,
    page,
    limit: 20
  });
}

loadMore() {
  this.searchParams.update(params => ({
    ...params,
    page: params.page + 1
  }));
}

reload() {
  this.productsResource.reload();
}
```

</details>

#### 6.3 Update Product Catalog to Use Resource

Update `product-catalog.component.ts` to use the products service:

<details>
<summary>💡 Hint</summary>

```typescript
import { ProductsService } from './products.service';

export class ProductCatalogComponent {
  private productsService = inject(ProductsService);

  // Remove the static products signal and use the resource instead
  // products = signal<Product[]>([...]);

  // Access the resource
  productsResource = this.productsService.productsResource;

  // Update computed signals
  categories = computed(() => {
    const products = this.productsResource.value()?.products || [];
    const cats = products.map(p => p.category);
    return [...new Set(cats)].sort();
  });

  filteredProducts = computed(() => {
    const products = this.productsResource.value()?.products || [];

    // Apply additional filters (price range, sort)
    let filtered = [...products];

    const priceRange = this.priceRange();
    if (priceRange) {
      filtered = filtered.filter(p => {
        const price = p.price;
        switch (priceRange) {
          case '0-50': return price < 50;
          case '50-200': return price >= 50 && price < 200;
          case '200-500': return price >= 200 && price < 500;
          case '500+': return price >= 500;
          default: return true;
        }
      });
    }

    const sortBy = this.sortBy();
    return filtered.sort((a, b) => {
      switch (sortBy) {
        case 'name': return a.name.localeCompare(b.name);
        case 'price-low': return a.price - b.price;
        case 'price-high': return b.price - a.price;
        case 'rating': return b.rating - a.rating;
        default: return 0;
      }
    });
  });

  constructor() {
    // Effect to update service when search/category changes
    effect(() => {
      this.productsService.updateSearch(
        this.selectedCategory(),
        this.searchQuery()
      );
    });
  }

  // Update template to handle loading states
  template: `
    <div class="catalog">
      <app-product-search
        [(searchQuery)]="searchQuery"
        [(priceRange)]="priceRange"
        [(sortBy)]="sortBy">
      </app-product-search>

      <div class="catalog-header">
        @if (productsResource.isLoading()) {
          <h2>Loading products...</h2>
        } @else if (productsResource.value()) {
          <h2>Products ({{ productsResource.value()!.total }})</h2>
        } @else {
          <h2>Products</h2>
        }
        <!-- category filter remains the same -->
      </div>

      @if (productsResource.error()) {
        <div class="error-state">
          <p>Failed to load products: {{ productsResource.error()?.message }}</p>
          <button (click)="productsService.reload()">Retry</button>
        </div>
      } @else {
        <div class="products-grid">
          @for (product of filteredProducts(); track product.id) {
            <app-product-card
              [product]="product"
              (addToCart)="onAddToCart($event)"
              (addToWishlist)="onAddToWishlist($event)">
            </app-product-card>
          }
        </div>

        @if (productsResource.isLoading()) {
          <div class="loading-state">
            <p>Loading products...</p>
            <div class="spinner"></div>
          </div>
        }

        @if (filteredProducts().length === 0 && !productsResource.isLoading()) {
          <div class="no-products">
            <p>No products found.</p>
          </div>
        }
      }
    </div>
  `,

  // Add loading and error styles
  styles: [`
    /* existing styles */

    .loading-state {
      text-align: center;
      padding: 2rem;
    }

    .error-state {
      text-align: center;
      padding: 2rem;
      color: #dc3545;
    }

    .error-state button {
      margin-top: 1rem;
      padding: 0.5rem 1rem;
      background: #007acc;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }

    .spinner {
      border: 2px solid #f3f3f3;
      border-top: 2px solid #007acc;
      border-radius: 50%;
      width: 30px;
      height: 30px;
      animation: spin 1s linear infinite;
      margin: 1rem auto;
    }

    @keyframes spin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }
  `]
}
```

</details>

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

<details>
<summary>💡 Hint</summary>

```typescript
// Load wishlist from localStorage
private loadWishlist(): number[] {
  try {
    const stored = localStorage.getItem('wishlist');
    return stored ? JSON.parse(stored) : [];
  } catch {
    return [];
  }
}

// Initialize with stored data
private wishlistItems = signal<number[]>(this.loadWishlist());

// Public readonly access
readonly items = this.wishlistItems.asReadonly();
readonly count = computed(() => this.wishlistItems().length);
readonly isEmpty = computed(() => this.wishlistItems().length === 0);

constructor() {
  // Persist to localStorage
  effect(() => {
    localStorage.setItem('wishlist', JSON.stringify(this.wishlistItems()));
  });
}

// Wishlist management
addToWishlist(productId: number) {
  if (!this.isInWishlist(productId)) {
    this.wishlistItems.update(items => [...items, productId]);
  }
}

removeFromWishlist(productId: number) {
  this.wishlistItems.update(items =>
    items.filter(id => id !== productId)
  );
}

toggleWishlist(productId: number) {
  if (this.isInWishlist(productId)) {
    this.removeFromWishlist(productId);
  } else {
    this.addToWishlist(productId);
  }
}

isInWishlist(productId: number): boolean {
  return this.wishlistItems().includes(productId);
}

clearWishlist() {
  this.wishlistItems.set([]);
}

// Get wishlist products (requires products service)
getWishlistProducts(allProducts: Product[]): Product[] {
  const wishlistIds = this.wishlistItems();
  return allProducts.filter(product => wishlistIds.includes(product.id));
}
```

</details>

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

<details>
<summary>💡 Hint</summary>

```typescript
// Get recommended products based on cart and wishlist
getRecommendations(allProducts: Product[], limit: number = 4): Product[] {
  const cartItems = this.cartService.items();
  const wishlistIds = this.wishlistService.items();

  // Get categories from cart items
  const cartCategories = cartItems.map(item => item.product.category);
  const uniqueCartCategories = [...new Set(cartCategories)];

  // Get products already in cart or wishlist
  const excludeIds = new Set([
    ...cartItems.map(item => item.product.id),
    ...wishlistIds
  ]);

  // Find products in same categories as cart items
  let recommendations = allProducts.filter(product =>
    !excludeIds.has(product.id) &&
    uniqueCartCategories.includes(product.category)
  );

  // If not enough recommendations, add high-rated products
  if (recommendations.length < limit) {
    const additionalProducts = allProducts
      .filter(product =>
        !excludeIds.has(product.id) &&
        !recommendations.some(rec => rec.id === product.id)
      )
      .sort((a, b) => b.rating - a.rating);

    recommendations = [
      ...recommendations,
      ...additionalProducts.slice(0, limit - recommendations.length)
    ];
  }

  // Sort by rating and return limited results
  return recommendations
    .sort((a, b) => b.rating - a.rating)
    .slice(0, limit);
}

// Computed signal for recommendations
getRecommendationsSignal(allProducts: Product[]) {
  return computed(() => this.getRecommendations(allProducts, 4));
}
```

</details>

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

<details>
<summary>💡 Hint</summary>

```typescript
import { output } from '@angular/core';

// Computed signal for wishlist products
wishlistProducts = computed(() => {
  const allProducts = this.productsService.productsResource.value()?.products || [];
  return this.wishlistService.getWishlistProducts(allProducts);
});

// Outputs
productAddedToCart = output<Product>();

onAddToCart(product: Product) {
  this.productAddedToCart.emit(product);
  // Optionally remove from wishlist after adding to cart
  this.wishlistService.removeFromWishlist(product.id);
}

onRemoveFromWishlist(product: Product) {
  this.wishlistService.removeFromWishlist(product.id);
}
```

</details>

#### 7.7 Update Product Card to Show Wishlist Status

Update `product-card.component.ts` to show wishlist status:

<details>
<summary>💡 Hint</summary>

```typescript
import { WishlistService } from './wishlist.service';

export class ProductCardComponent {
  product = input.required<Product>();
  preferences = inject(UserPreferencesService);
  wishlistService = inject(WishlistService);

  // Computed signal for wishlist status
  isInWishlist = computed(() =>
    this.wishlistService.isInWishlist(this.product().id)
  );

  // Update wishlist button in template:
  // <button
  //   class="btn-secondary"
  //   [class.in-wishlist]="isInWishlist()"
  //   (click)="onAddToWishlist()">
  //   {{ isInWishlist() ? '♥' : '♡' }} Wishlist
  // </button>

  // Add styles:
  // .btn-secondary.in-wishlist {
  //   background: #dc3545;
  //   color: white;
  // }
}
```

</details>

#### 7.8 Add Recommendations Section

Update `app.component.ts` to include recommendations:

<details>
<summary>💡 Hint</summary>

Add a recommendations section to your main template that shows when viewing the catalog:

```typescript
// In the catalog case of your switch statement:
@case ('catalog') {
  <app-product-catalog
    (productAddedToCart)="onAddToCart($event)"
    (productAddedToWishlist)="onAddToWishlist($event)">
  </app-product-catalog>

  @if (!cartService.isEmpty()) {
    <div class="recommendations">
      <h3>Recommended for you</h3>
      <!-- Add recommendations display here -->
    </div>
  }
}
```

Create a small recommendations component or add the logic directly to display recommended products based on the cart contents.
</details>

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