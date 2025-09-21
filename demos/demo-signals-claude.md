# Angular Signals Performance Demo

## Overview

This demonstration shows the performance benefits of Angular Signals compared to traditional Angular approaches. Each example includes console.log statements to visualize when components update and calculations occur.

---

## Demo 1: Simple Counter with Derived Values

### 🔴 Without Signals (Traditional Angular)

```typescript
// counter-traditional.component.ts
import { Component, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-counter-traditional',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="counter">
      <h2>Traditional Counter</h2>
      <p>Count: {{ count }}</p>
      <p>Double: {{ getDouble() }}</p>
      <p>Is Even: {{ getIsEven() }}</p>
      <p>Factorial: {{ getFactorial() }}</p>

      <button (click)="increment()">+</button>
      <button (click)="decrement()">-</button>
      <button (click)="triggerUnrelatedUpdate()">Unrelated Update</button>

      <p>Unrelated Value: {{ unrelatedValue }}</p>
    </div>
  `
})
export class CounterTraditionalComponent {
  count = 0;
  unrelatedValue = 'initial';

  constructor() {
    console.log('🔴 Traditional: Component constructed');
  }

  ngDoCheck() {
    console.log('🔴 Traditional: Change detection cycle');
  }

  increment() {
    console.log('🔴 Traditional: Incrementing count');
    this.count++;
  }

  decrement() {
    console.log('🔴 Traditional: Decrementing count');
    this.count--;
  }

  triggerUnrelatedUpdate() {
    console.log('🔴 Traditional: Updating unrelated value');
    this.unrelatedValue = 'updated at ' + new Date().toLocaleTimeString();
  }

  getDouble(): number {
    console.log('🔴 Traditional: Calculating double');
    return this.count * 2;
  }

  getIsEven(): boolean {
    console.log('🔴 Traditional: Calculating isEven');
    return this.count % 2 === 0;
  }

  getFactorial(): number {
    console.log('🔴 Traditional: Calculating factorial (expensive!)');

    // Simulate expensive calculation
    const start = performance.now();
    let result = 1;
    for (let i = 1; i <= Math.max(1, this.count); i++) {
      result *= i;
    }
    const end = performance.now();

    console.log(`🔴 Traditional: Factorial calculation took ${end - start}ms`);
    return result;
  }
}
```

### 🟢 With Signals

```typescript
// counter-signals.component.ts
import { Component, signal, computed, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-counter-signals',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="counter">
      <h2>Signals Counter</h2>
      <p>Count: {{ count() }}</p>
      <p>Double: {{ double() }}</p>
      <p>Is Even: {{ isEven() }}</p>
      <p>Factorial: {{ factorial() }}</p>

      <button (click)="increment()">+</button>
      <button (click)="decrement()">-</button>
      <button (click)="triggerUnrelatedUpdate()">Unrelated Update</button>

      <p>Unrelated Value: {{ unrelatedValue() }}</p>
    </div>
  `
})
export class CounterSignalsComponent {
  count = signal(0);
  unrelatedValue = signal('initial');

  // Computed signals - only recalculate when dependencies change
  double = computed(() => {
    console.log('🟢 Signals: Calculating double');
    return this.count() * 2;
  });

  isEven = computed(() => {
    console.log('🟢 Signals: Calculating isEven');
    return this.count() % 2 === 0;
  });

  factorial = computed(() => {
    console.log('🟢 Signals: Calculating factorial (expensive!)');

    // Simulate expensive calculation
    const start = performance.now();
    let result = 1;
    for (let i = 1; i <= Math.max(1, this.count()); i++) {
      result *= i;
    }
    const end = performance.now();

    console.log(`🟢 Signals: Factorial calculation took ${end - start}ms`);
    return result;
  });

  constructor() {
    console.log('🟢 Signals: Component constructed');
  }

  ngDoCheck() {
    console.log('🟢 Signals: Change detection cycle');
  }

  increment() {
    console.log('🟢 Signals: Incrementing count');
    this.count.update(c => c + 1);
  }

  decrement() {
    console.log('🟢 Signals: Decrementing count');
    this.count.update(c => c - 1);
  }

  triggerUnrelatedUpdate() {
    console.log('🟢 Signals: Updating unrelated value');
    this.unrelatedValue.set('updated at ' + new Date().toLocaleTimeString());
  }
}
```

### 📊 Performance Comparison

**Test this by:**
1. Open browser console
2. Click the "+", "-", and "Unrelated Update" buttons
3. Observe the console logs

**Expected Results:**

**Traditional (🔴):**
- Every button click triggers ALL method calls
- Expensive factorial calculation runs on every change detection cycle
- Unrelated updates also trigger all calculations

**Signals (🟢):**
- Only relevant computed signals recalculate
- Factorial only recalculates when count changes
- Unrelated updates don't trigger count-dependent calculations

---

## Demo 2: Parent-Child Communication

### 🔴 Without Signals (Traditional Angular)

```typescript
// parent-traditional.component.ts
@Component({
  selector: 'app-parent-traditional',
  template: `
    <div class="parent">
      <h2>Traditional Parent</h2>
      <p>Parent Counter: {{ parentCount }}</p>
      <button (click)="incrementParent()">Increment Parent</button>
      <button (click)="updateName()">Update Name</button>

      <app-child-traditional
        [count]="parentCount"
        [name]="childName">
      </app-child-traditional>
    </div>
  `
})
export class ParentTraditionalComponent {
  parentCount = 0;
  childName = 'Initial Name';

  ngDoCheck() {
    console.log('🔴 Parent Traditional: Change detection cycle');
  }

  incrementParent() {
    console.log('🔴 Parent Traditional: Incrementing parent count');
    this.parentCount++;
  }

  updateName() {
    console.log('🔴 Parent Traditional: Updating child name');
    this.childName = 'Updated at ' + new Date().toLocaleTimeString();
  }
}

// child-traditional.component.ts
@Component({
  selector: 'app-child-traditional',
  template: `
    <div class="child">
      <h3>Traditional Child</h3>
      <p>Received Count: {{ count }}</p>
      <p>Count Squared: {{ getCountSquared() }}</p>
      <p>Name: {{ name }}</p>
      <p>Name Length: {{ getNameLength() }}</p>
      <p>Child Internal: {{ internalValue }}</p>
      <button (click)="updateInternal()">Update Internal</button>
    </div>
  `
})
export class ChildTraditionalComponent {
  @Input() count = 0;
  @Input() name = '';
  internalValue = 0;

  ngDoCheck() {
    console.log('🔴 Child Traditional: Change detection cycle');
  }

  ngOnChanges(changes: SimpleChanges) {
    console.log('🔴 Child Traditional: Inputs changed', changes);
  }

  updateInternal() {
    console.log('🔴 Child Traditional: Updating internal value');
    this.internalValue++;
  }

  getCountSquared(): number {
    console.log('🔴 Child Traditional: Calculating count squared');
    return this.count * this.count;
  }

  getNameLength(): number {
    console.log('🔴 Child Traditional: Calculating name length');
    return this.name.length;
  }
}
```

### 🟢 With Signals

```typescript
// parent-signals.component.ts
@Component({
  selector: 'app-parent-signals',
  template: `
    <div class="parent">
      <h2>Signals Parent</h2>
      <p>Parent Counter: {{ parentCount() }}</p>
      <button (click)="incrementParent()">Increment Parent</button>
      <button (click)="updateName()">Update Name</button>

      <app-child-signals
        [count]="parentCount()"
        [name]="childName()">
      </app-child-signals>
    </div>
  `
})
export class ParentSignalsComponent {
  parentCount = signal(0);
  childName = signal('Initial Name');

  ngDoCheck() {
    console.log('🟢 Parent Signals: Change detection cycle');
  }

  incrementParent() {
    console.log('🟢 Parent Signals: Incrementing parent count');
    this.parentCount.update(c => c + 1);
  }

  updateName() {
    console.log('🟢 Parent Signals: Updating child name');
    this.childName.set('Updated at ' + new Date().toLocaleTimeString());
  }
}

// child-signals.component.ts
@Component({
  selector: 'app-child-signals',
  template: `
    <div class="child">
      <h3>Signals Child</h3>
      <p>Received Count: {{ count() }}</p>
      <p>Count Squared: {{ countSquared() }}</p>
      <p>Name: {{ name() }}</p>
      <p>Name Length: {{ nameLength() }}</p>
      <p>Child Internal: {{ internalValue() }}</p>
      <button (click)="updateInternal()">Update Internal</button>
    </div>
  `
})
export class ChildSignalsComponent {
  count = input<number>(0);
  name = input<string>('');
  internalValue = signal(0);

  // Computed signals - only update when their dependencies change
  countSquared = computed(() => {
    console.log('🟢 Child Signals: Calculating count squared');
    return this.count() * this.count();
  });

  nameLength = computed(() => {
    console.log('🟢 Child Signals: Calculating name length');
    return this.name().length;
  });

  ngDoCheck() {
    console.log('🟢 Child Signals: Change detection cycle');
  }

  constructor() {
    // Effect to track input changes
    effect(() => {
      console.log('🟢 Child Signals: Count input changed to', this.count());
    });

    effect(() => {
      console.log('🟢 Child Signals: Name input changed to', this.name());
    });
  }

  updateInternal() {
    console.log('🟢 Child Signals: Updating internal value');
    this.internalValue.update(v => v + 1);
  }
}
```

### 📊 Performance Comparison

**Test this by:**
1. Click "Increment Parent" - watch count-related calculations
2. Click "Update Name" - watch name-related calculations
3. Click "Update Internal" - watch internal value updates

**Expected Results:**

**Traditional (🔴):**
- Every parent update triggers child change detection
- All child methods recalculate on every change detection cycle
- No granular dependency tracking

**Signals (🟢):**
- Only affected computed signals recalculate
- Count updates only trigger count-related calculations
- Name updates only trigger name-related calculations

---

## Demo 3: List Filtering Performance

### 🔴 Without Signals (Traditional Angular)

```typescript
// list-traditional.component.ts
@Component({
  selector: 'app-list-traditional',
  template: `
    <div class="list-container">
      <h2>Traditional List</h2>

      <input
        [(ngModel)]="searchTerm"
        placeholder="Search..."
        (input)="onSearchChange()">

      <select [(ngModel)]="selectedCategory" (change)="onCategoryChange()">
        <option value="">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="books">Books</option>
        <option value="clothing">Clothing</option>
      </select>

      <p>Items: {{ getFilteredItems().length }} | Total Value: {{ getTotalValue() }}</p>

      <div class="items">
        <div *ngFor="let item of getFilteredItems(); trackBy: trackByFn" class="item">
          {{ item.name }} - ${{ item.price }} ({{ item.category }})
        </div>
      </div>

      <button (click)="addRandomItem()">Add Random Item</button>
    </div>
  `
})
export class ListTraditionalComponent {
  searchTerm = '';
  selectedCategory = '';

  items = [
    { id: 1, name: 'Laptop', price: 1000, category: 'electronics' },
    { id: 2, name: 'Book', price: 20, category: 'books' },
    { id: 3, name: 'Shirt', price: 30, category: 'clothing' },
    { id: 4, name: 'Phone', price: 800, category: 'electronics' },
    { id: 5, name: 'Novel', price: 15, category: 'books' }
  ];

  ngDoCheck() {
    console.log('🔴 List Traditional: Change detection cycle');
  }

  onSearchChange() {
    console.log('🔴 List Traditional: Search term changed to:', this.searchTerm);
  }

  onCategoryChange() {
    console.log('🔴 List Traditional: Category changed to:', this.selectedCategory);
  }

  getFilteredItems() {
    console.log('🔴 List Traditional: Filtering items (expensive operation!)');

    const start = performance.now();

    let filtered = this.items;

    if (this.searchTerm) {
      filtered = filtered.filter(item =>
        item.name.toLowerCase().includes(this.searchTerm.toLowerCase())
      );
    }

    if (this.selectedCategory) {
      filtered = filtered.filter(item => item.category === this.selectedCategory);
    }

    // Simulate expensive filtering
    for (let i = 0; i < 10000; i++) {
      Math.random();
    }

    const end = performance.now();
    console.log(`🔴 List Traditional: Filtering took ${end - start}ms`);

    return filtered;
  }

  getTotalValue(): number {
    console.log('🔴 List Traditional: Calculating total value');

    const filtered = this.getFilteredItems(); // This calls filtering again!
    return filtered.reduce((total, item) => total + item.price, 0);
  }

  trackByFn(index: number, item: any) {
    return item.id;
  }

  addRandomItem() {
    const categories = ['electronics', 'books', 'clothing'];
    const names = ['Widget', 'Gadget', 'Thing', 'Item', 'Product'];

    const newItem = {
      id: Date.now(),
      name: names[Math.floor(Math.random() * names.length)],
      price: Math.floor(Math.random() * 100) + 10,
      category: categories[Math.floor(Math.random() * categories.length)]
    };

    console.log('🔴 List Traditional: Adding item:', newItem);
    this.items.push(newItem);
  }
}
```

### 🟢 With Signals

```typescript
// list-signals.component.ts
@Component({
  selector: 'app-list-signals',
  template: `
    <div class="list-container">
      <h2>Signals List</h2>

      <input
        [value]="searchTerm()"
        (input)="updateSearchTerm($event)"
        placeholder="Search...">

      <select [value]="selectedCategory()" (change)="updateCategory($event)">
        <option value="">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="books">Books</option>
        <option value="clothing">Clothing</option>
      </select>

      <p>Items: {{ filteredItems().length }} | Total Value: {{ totalValue() }}</p>

      <div class="items">
        <div *ngFor="let item of filteredItems(); trackBy: trackByFn" class="item">
          {{ item.name }} - ${{ item.price }} ({{ item.category }})
        </div>
      </div>

      <button (click)="addRandomItem()">Add Random Item</button>
    </div>
  `
})
export class ListSignalsComponent {
  searchTerm = signal('');
  selectedCategory = signal('');

  items = signal([
    { id: 1, name: 'Laptop', price: 1000, category: 'electronics' },
    { id: 2, name: 'Book', price: 20, category: 'books' },
    { id: 3, name: 'Shirt', price: 30, category: 'clothing' },
    { id: 4, name: 'Phone', price: 800, category: 'electronics' },
    { id: 5, name: 'Novel', price: 15, category: 'books' }
  ]);

  // Computed signal - only recalculates when dependencies change
  filteredItems = computed(() => {
    console.log('🟢 List Signals: Filtering items (expensive operation!)');

    const start = performance.now();

    let filtered = this.items();
    const search = this.searchTerm();
    const category = this.selectedCategory();

    if (search) {
      filtered = filtered.filter(item =>
        item.name.toLowerCase().includes(search.toLowerCase())
      );
    }

    if (category) {
      filtered = filtered.filter(item => item.category === category);
    }

    // Simulate expensive filtering
    for (let i = 0; i < 10000; i++) {
      Math.random();
    }

    const end = performance.now();
    console.log(`🟢 List Signals: Filtering took ${end - start}ms`);

    return filtered;
  });

  // Computed signal that depends on filteredItems
  totalValue = computed(() => {
    console.log('🟢 List Signals: Calculating total value');

    // Uses cached filteredItems result!
    return this.filteredItems().reduce((total, item) => total + item.price, 0);
  });

  ngDoCheck() {
    console.log('🟢 List Signals: Change detection cycle');
  }

  updateSearchTerm(event: Event) {
    const target = event.target as HTMLInputElement;
    console.log('🟢 List Signals: Search term changed to:', target.value);
    this.searchTerm.set(target.value);
  }

  updateCategory(event: Event) {
    const target = event.target as HTMLSelectElement;
    console.log('🟢 List Signals: Category changed to:', target.value);
    this.selectedCategory.set(target.value);
  }

  trackByFn(index: number, item: any) {
    return item.id;
  }

  addRandomItem() {
    const categories = ['electronics', 'books', 'clothing'];
    const names = ['Widget', 'Gadget', 'Thing', 'Item', 'Product'];

    const newItem = {
      id: Date.now(),
      name: names[Math.floor(Math.random() * names.length)],
      price: Math.floor(Math.random() * 100) + 10,
      category: categories[Math.floor(Math.random() * categories.length)]
    };

    console.log('🟢 List Signals: Adding item:', newItem);
    this.items.update(items => [...items, newItem]);
  }
}
```

### 📊 Performance Comparison

**Test this by:**
1. Type in the search box
2. Change the category dropdown
3. Add random items
4. Watch console logs for performance differences

**Expected Results:**

**Traditional (🔴):**
- Filtering happens on EVERY change detection cycle
- Multiple filtering calls for each update (once for display, once for total)
- Adding items triggers unnecessary recalculations

**Signals (🟢):**
- Filtering only happens when search term, category, or items change
- Total value uses cached filtered result
- Much fewer recalculations overall

---

## Demo 4: Effects vs Subscriptions

### 🔴 Without Signals (Traditional Angular)

```typescript
// subscription-demo.component.ts
@Component({
  selector: 'app-subscription-demo',
  template: `
    <div class="demo">
      <h2>Traditional Subscriptions</h2>
      <p>User ID: {{ userId }}</p>
      <p>User Data: {{ userData | json }}</p>
      <p>Last Updated: {{ lastUpdated }}</p>

      <button (click)="changeUserId()">Change User ID</button>
      <button (click)="simulateUpdate()">Simulate Update</button>
    </div>
  `
})
export class SubscriptionDemoComponent implements OnInit, OnDestroy {
  userId = 1;
  userData: any = null;
  lastUpdated = '';

  private userSubject = new BehaviorSubject(this.userId);
  private destroy$ = new Subject<void>();

  constructor(private userService: UserService) {}

  ngOnInit() {
    console.log('🔴 Subscriptions: Setting up subscriptions');

    // Complex subscription chain
    this.userSubject.pipe(
      tap(id => console.log('🔴 Subscriptions: User ID changed to:', id)),
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(userId => {
        console.log('🔴 Subscriptions: Fetching user data for ID:', userId);
        return this.userService.getUser(userId);
      }),
      takeUntil(this.destroy$)
    ).subscribe({
      next: (userData) => {
        console.log('🔴 Subscriptions: Received user data:', userData);
        this.userData = userData;
        this.lastUpdated = new Date().toLocaleTimeString();
      },
      error: (error) => {
        console.error('🔴 Subscriptions: Error fetching user:', error);
      }
    });

    // Separate subscription for analytics
    this.userSubject.pipe(
      tap(id => console.log('🔴 Subscriptions: Tracking user view for ID:', id)),
      debounceTime(1000),
      takeUntil(this.destroy$)
    ).subscribe(id => {
      console.log('🔴 Subscriptions: Sending analytics for user:', id);
      // Analytics call
    });
  }

  ngOnDestroy() {
    console.log('🔴 Subscriptions: Cleaning up subscriptions');
    this.destroy$.next();
    this.destroy$.complete();
  }

  changeUserId() {
    this.userId = Math.floor(Math.random() * 10) + 1;
    console.log('🔴 Subscriptions: Changing user ID to:', this.userId);
    this.userSubject.next(this.userId);
  }

  simulateUpdate() {
    console.log('🔴 Subscriptions: Simulating unrelated update');
    // This triggers change detection but doesn't affect user data
    this.lastUpdated = 'Simulated at ' + new Date().toLocaleTimeString();
  }
}
```

### 🟢 With Signals

```typescript
// signals-demo.component.ts
@Component({
  selector: 'app-signals-demo',
  template: `
    <div class="demo">
      <h2>Signals Effects</h2>
      <p>User ID: {{ userId() }}</p>
      <p>User Data: {{ userData() | json }}</p>
      <p>Last Updated: {{ lastUpdated() }}</p>

      <button (click)="changeUserId()">Change User ID</button>
      <button (click)="simulateUpdate()">Simulate Update</button>
    </div>
  `
})
export class SignalsDemoComponent {
  userId = signal(1);
  userData = signal<any>(null);
  lastUpdated = signal('');

  constructor(private userService: UserService) {
    console.log('🟢 Signals: Setting up effects');

    // Effect for data fetching - automatically handles debouncing and cleanup
    effect(() => {
      const id = this.userId();
      console.log('🟢 Signals: User ID changed to:', id);

      // Debounce using a simple timeout
      const timeoutId = setTimeout(async () => {
        try {
          console.log('🟢 Signals: Fetching user data for ID:', id);
          const userData = await this.userService.getUser(id);
          console.log('🟢 Signals: Received user data:', userData);
          this.userData.set(userData);
          this.lastUpdated.set(new Date().toLocaleTimeString());
        } catch (error) {
          console.error('🟢 Signals: Error fetching user:', error);
        }
      }, 300);

      // Cleanup function - automatically called when effect re-runs
      return () => {
        console.log('🟢 Signals: Cleaning up previous effect');
        clearTimeout(timeoutId);
      };
    });

    // Separate effect for analytics
    effect(() => {
      const id = this.userId();
      console.log('🟢 Signals: Tracking user view for ID:', id);

      const timeoutId = setTimeout(() => {
        console.log('🟢 Signals: Sending analytics for user:', id);
        // Analytics call
      }, 1000);

      return () => clearTimeout(timeoutId);
    });
  }

  changeUserId() {
    const newId = Math.floor(Math.random() * 10) + 1;
    console.log('🟢 Signals: Changing user ID to:', newId);
    this.userId.set(newId);
  }

  simulateUpdate() {
    console.log('🟢 Signals: Simulating unrelated update');
    // This only updates the specific signal
    this.lastUpdated.set('Simulated at ' + new Date().toLocaleTimeString());
  }
}
```

### 📊 Performance Comparison

**Test this by:**
1. Click "Change User ID" rapidly
2. Click "Simulate Update"
3. Watch how each approach handles updates

**Expected Results:**

**Traditional (🔴):**
- Complex subscription setup and cleanup
- Manual memory management required
- Multiple subscription chains

**Signals (🟢):**
- Automatic cleanup and dependency tracking
- Simpler mental model
- No memory leaks by default

---

## Demo 5: Real-World Shopping Cart Example

### 🔴 Without Signals (Traditional Angular)

```typescript
// cart-traditional.service.ts
@Injectable({
  providedIn: 'root'
})
export class CartTraditionalService {
  private cartItems$ = new BehaviorSubject<CartItem[]>([]);
  private discountCode$ = new BehaviorSubject<string>('');
  private taxRate$ = new BehaviorSubject<number>(0.08);

  constructor() {
    console.log('🔴 Cart Traditional: Service initialized');
  }

  get items$() {
    return this.cartItems$.asObservable();
  }

  get itemCount$() {
    console.log('🔴 Cart Traditional: Computing item count stream');
    return this.cartItems$.pipe(
      map(items => {
        console.log('🔴 Cart Traditional: Calculating item count');
        return items.reduce((count, item) => count + item.quantity, 0);
      })
    );
  }

  get subtotal$() {
    console.log('🔴 Cart Traditional: Computing subtotal stream');
    return this.cartItems$.pipe(
      map(items => {
        console.log('🔴 Cart Traditional: Calculating subtotal');
        return items.reduce((total, item) => total + (item.price * item.quantity), 0);
      })
    );
  }

  get discount$() {
    console.log('🔴 Cart Traditional: Computing discount stream');
    return combineLatest([this.subtotal$, this.discountCode$]).pipe(
      map(([subtotal, code]) => {
        console.log('🔴 Cart Traditional: Calculating discount');
        return code === 'SAVE10' ? subtotal * 0.1 : 0;
      })
    );
  }

  get tax$() {
    console.log('🔴 Cart Traditional: Computing tax stream');
    return combineLatest([this.subtotal$, this.discount$, this.taxRate$]).pipe(
      map(([subtotal, discount, taxRate]) => {
        console.log('🔴 Cart Traditional: Calculating tax');
        return (subtotal - discount) * taxRate;
      })
    );
  }

  get total$() {
    console.log('🔴 Cart Traditional: Computing total stream');
    return combineLatest([this.subtotal$, this.discount$, this.tax$]).pipe(
      map(([subtotal, discount, tax]) => {
        console.log('🔴 Cart Traditional: Calculating total');
        return subtotal - discount + tax;
      })
    );
  }

  addItem(product: any) {
    console.log('🔴 Cart Traditional: Adding item:', product);
    const currentItems = this.cartItems$.value;
    const existingItem = currentItems.find(item => item.id === product.id);

    if (existingItem) {
      existingItem.quantity++;
    } else {
      currentItems.push({ ...product, quantity: 1 });
    }

    this.cartItems$.next([...currentItems]);
  }

  updateDiscountCode(code: string) {
    console.log('🔴 Cart Traditional: Updating discount code:', code);
    this.discountCode$.next(code);
  }

  updateTaxRate(rate: number) {
    console.log('🔴 Cart Traditional: Updating tax rate:', rate);
    this.taxRate$.next(rate);
  }
}

// cart-traditional.component.ts
@Component({
  selector: 'app-cart-traditional',
  template: `
    <div class="cart">
      <h2>Traditional Cart</h2>

      <div *ngFor="let item of items$ | async" class="cart-item">
        {{ item.name }} - ${{ item.price }} x {{ item.quantity }}
      </div>

      <div class="cart-summary">
        <p>Items: {{ itemCount$ | async }}</p>
        <p>Subtotal: ${{ (subtotal$ | async) | number:'1.2-2' }}</p>
        <p>Discount: ${{ (discount$ | async) | number:'1.2-2' }}</p>
        <p>Tax: ${{ (tax$ | async) | number:'1.2-2' }}</p>
        <p><strong>Total: ${{ (total$ | async) | number:'1.2-2' }}</strong></p>
      </div>

      <input
        [(ngModel)]="discountCode"
        (input)="updateDiscount()"
        placeholder="Discount code">

      <button (click)="addRandomItem()">Add Random Item</button>
      <button (click)="changeTaxRate()">Change Tax Rate</button>
    </div>
  `
})
export class CartTraditionalComponent implements OnInit, OnDestroy {
  items$ = this.cartService.items$;
  itemCount$ = this.cartService.itemCount$;
  subtotal$ = this.cartService.subtotal$;
  discount$ = this.cartService.discount$;
  tax$ = this.cartService.tax$;
  total$ = this.cartService.total$;

  discountCode = '';
  private destroy$ = new Subject<void>();

  constructor(private cartService: CartTraditionalService) {}

  ngOnInit() {
    // Subscribe to all streams to trigger calculations
    console.log('🔴 Cart Traditional: Setting up subscriptions');

    this.total$.pipe(takeUntil(this.destroy$)).subscribe(total => {
      console.log('🔴 Cart Traditional: Total updated to:', total);
    });
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }

  updateDiscount() {
    this.cartService.updateDiscountCode(this.discountCode);
  }

  addRandomItem() {
    const products = [
      { id: 1, name: 'Laptop', price: 1000 },
      { id: 2, name: 'Mouse', price: 25 },
      { id: 3, name: 'Keyboard', price: 75 }
    ];

    const randomProduct = products[Math.floor(Math.random() * products.length)];
    this.cartService.addItem(randomProduct);
  }

  changeTaxRate() {
    const newRate = Math.random() * 0.1 + 0.05; // 5-15%
    this.cartService.updateTaxRate(newRate);
  }
}
```

### 🟢 With Signals

```typescript
// cart-signals.service.ts
@Injectable({
  providedIn: 'root'
})
export class CartSignalsService {
  private cartItems = signal<CartItem[]>([]);
  private discountCode = signal<string>('');
  private taxRate = signal<number>(0.08);

  // Public readonly access
  readonly items = this.cartItems.asReadonly();

  // Computed signals - automatically optimized
  readonly itemCount = computed(() => {
    console.log('🟢 Cart Signals: Calculating item count');
    return this.cartItems().reduce((count, item) => count + item.quantity, 0);
  });

  readonly subtotal = computed(() => {
    console.log('🟢 Cart Signals: Calculating subtotal');
    return this.cartItems().reduce((total, item) => total + (item.price * item.quantity), 0);
  });

  readonly discount = computed(() => {
    console.log('🟢 Cart Signals: Calculating discount');
    const code = this.discountCode();
    const subtotal = this.subtotal();
    return code === 'SAVE10' ? subtotal * 0.1 : 0;
  });

  readonly tax = computed(() => {
    console.log('🟢 Cart Signals: Calculating tax');
    const subtotal = this.subtotal();
    const discount = this.discount();
    const rate = this.taxRate();
    return (subtotal - discount) * rate;
  });

  readonly total = computed(() => {
    console.log('🟢 Cart Signals: Calculating total');
    const subtotal = this.subtotal();
    const discount = this.discount();
    const tax = this.tax();
    return subtotal - discount + tax;
  });

  constructor() {
    console.log('🟢 Cart Signals: Service initialized');

    // Effect to log total changes
    effect(() => {
      console.log('🟢 Cart Signals: Total updated to:', this.total());
    });
  }

  addItem(product: any) {
    console.log('🟢 Cart Signals: Adding item:', product);
    this.cartItems.update(currentItems => {
      const existingItem = currentItems.find(item => item.id === product.id);

      if (existingItem) {
        return currentItems.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      } else {
        return [...currentItems, { ...product, quantity: 1 }];
      }
    });
  }

  updateDiscountCode(code: string) {
    console.log('🟢 Cart Signals: Updating discount code:', code);
    this.discountCode.set(code);
  }

  updateTaxRate(rate: number) {
    console.log('🟢 Cart Signals: Updating tax rate:', rate);
    this.taxRate.set(rate);
  }
}

// cart-signals.component.ts
@Component({
  selector: 'app-cart-signals',
  template: `
    <div class="cart">
      <h2>Signals Cart</h2>

      <div *ngFor="let item of cartService.items(); trackBy: trackByFn" class="cart-item">
        {{ item.name }} - ${{ item.price }} x {{ item.quantity }}
      </div>

      <div class="cart-summary">
        <p>Items: {{ cartService.itemCount() }}</p>
        <p>Subtotal: ${{ cartService.subtotal() | number:'1.2-2' }}</p>
        <p>Discount: ${{ cartService.discount() | number:'1.2-2' }}</p>
        <p>Tax: ${{ cartService.tax() | number:'1.2-2' }}</p>
        <p><strong>Total: ${{ cartService.total() | number:'1.2-2' }}</strong></p>
      </div>

      <input
        [value]="discountCode()"
        (input)="updateDiscount($event)"
        placeholder="Discount code">

      <button (click)="addRandomItem()">Add Random Item</button>
      <button (click)="changeTaxRate()">Change Tax Rate</button>
    </div>
  `
})
export class CartSignalsComponent {
  discountCode = signal('');

  constructor(public cartService: CartSignalsService) {
    console.log('🟢 Cart Signals: Component initialized');
  }

  updateDiscount(event: Event) {
    const target = event.target as HTMLInputElement;
    this.discountCode.set(target.value);
    this.cartService.updateDiscountCode(target.value);
  }

  addRandomItem() {
    const products = [
      { id: 1, name: 'Laptop', price: 1000 },
      { id: 2, name: 'Mouse', price: 25 },
      { id: 3, name: 'Keyboard', price: 75 }
    ];

    const randomProduct = products[Math.floor(Math.random() * products.length)];
    this.cartService.addItem(randomProduct);
  }

  changeTaxRate() {
    const newRate = Math.random() * 0.1 + 0.05; // 5-15%
    this.cartService.updateTaxRate(newRate);
  }

  trackByFn(index: number, item: any) {
    return item.id;
  }
}
```

### 📊 Performance Comparison

**Test this by:**
1. Add items to cart
2. Change discount code
3. Change tax rate
4. Watch console logs for calculation patterns

**Expected Results:**

**Traditional (🔴):**
- Complex stream setup with multiple observables
- Potential for duplicate calculations
- Manual subscription management

**Signals (🟢):**
- Simple, declarative computed signals
- Automatic memoization and dependency tracking
- No subscription boilerplate

---

## Summary: Key Performance Benefits

### 🟢 **Signals Advantages:**

1. **Fine-grained Reactivity**
   - Only affected computed signals recalculate
   - No unnecessary method calls

2. **Automatic Memoization**
   - Computed signals cache results
   - Only recalculate when dependencies change

3. **Simplified Mental Model**
   - Declarative dependencies
   - No subscription management

4. **Better Performance**
   - Fewer calculations
   - More predictable update patterns

5. **Automatic Cleanup**
   - No memory leaks
   - Effects handle cleanup automatically

### 🔴 **Traditional Approach Issues:**

1. **Over-computation**
   - Methods called on every change detection
   - No automatic caching

2. **Complex Subscription Management**
   - Manual setup and cleanup
   - Memory leak potential

3. **Unpredictable Updates**
   - Hard to track what triggers what
   - Multiple calculation paths

### 📊 **Performance Metrics to Watch:**

- **Calculation frequency**: How often expensive operations run
- **Dependency precision**: Only relevant updates trigger recalculations
- **Memory usage**: No subscription leaks with signals
- **Code complexity**: Simpler patterns with signals

Open your browser console and interact with both versions to see the dramatic difference in performance and update patterns!