# Angular Signals

## Topic Overview

Angular Signals is a stable reactive programming system (as of Angular 20) that provides fine-grained reactivity for managing state in Angular applications. Signals represent values that can change over time and automatically notify interested consumers when they change.

### What are Signals?

A signal is a wrapper around a value that notifies interested consumers when that value changes. Signals can contain any value, from primitives to complex data structures. You read a signal's value by calling its getter function, which allows Angular to track where the signal is used.

### Key Benefits

- **Fine-grained reactivity**: Only components that depend on changed signals are updated
- **Automatic dependency tracking**: Angular automatically tracks which signals your code reads
- **Improved performance**: More efficient change detection, especially with zoneless mode
- **Better developer experience**: Type-safe reactive state management
- **Simplified debugging**: Clear dependency graphs and predictable updates
- **Zoneless compatibility**: Works seamlessly with Angular's zoneless change detection

### Why Use Signals?

Signals enable Angular to optimize rendering updates by tracking exactly which parts of your application depend on specific pieces of data. When a signal changes, only the components and computations that actually use that signal are updated, leading to better performance and more predictable behavior.

## Basic Signals

### Creating Signals

Use the `signal()` function to create a writable signal:

```typescript
import { signal } from '@angular/core';

// Create a signal with an initial value
const count = signal(0);
const userName = signal('Guest');
const isLoading = signal(false);
```

### Reading Signal Values

Signals are functions - call them to read their current value:

```typescript
// Read the current value
console.log(`Count is: ${count()}`);
console.log(`User: ${userName()}`);
console.log(`Loading: ${isLoading()}`);
```

### Updating Signal Values

Use `.set()` to replace the value completely:

```typescript
count.set(10);
userName.set('John Doe');
isLoading.set(true);
```

Use `.update()` to compute a new value based on the current value:

```typescript
// Increment count by 1
count.update(current => current + 1);

// Capitalize username
userName.update(current => current.toUpperCase());

// Toggle loading state
isLoading.update(current => !current);
```

### Component Example

```typescript
import { Component, signal, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-counter',
  changeDetection: ChangeDetectionStrategy.OnPush, // Recommended with signals
  template: `
    <div>
      <p>Count: {{ count() }}</p>
      <button (click)="increment()">+</button>
      <button (click)="decrement()">-</button>
      <button (click)="reset()">Reset</button>
    </div>
  `
})
export class CounterComponent {
  count = signal(0);

  increment() {
    this.count.update(value => value + 1);
  }

  decrement() {
    this.count.update(value => value - 1);
  }

  reset() {
    this.count.set(0);
  }
}
```

## Computed Signals

Computed signals are read-only signals that derive their value from other signals. They automatically recalculate when their dependencies change.

### Creating Computed Signals

```typescript
import { signal, computed } from '@angular/core';

const firstName = signal('John');
const lastName = signal('Doe');

// Computed signal that combines first and last name
const fullName = computed(() => `${firstName()} ${lastName()}`);

console.log(fullName()); // "John Doe"

// When dependencies change, computed automatically updates
firstName.set('Jane');
console.log(fullName()); // "Jane Doe"
```

### Lazy Evaluation and Memoization

Computed signals are lazy (only calculated when read) and memoized (cached until dependencies change):

```typescript
const expensiveComputation = computed(() => {
  console.log('Computing...'); // Only logs when recalculation is needed
  const items = itemList();
  return items.filter(item => item.active).length;
});

// First read - computation runs
console.log(expensiveComputation()); // Logs "Computing..." then result

// Second read - uses cached value
console.log(expensiveComputation()); // Just returns cached result

// After itemList changes - computation runs again on next read
```

### Dynamic Dependencies

Computed signals only track signals that are actually read during execution:

```typescript
const showDetails = signal(false);
const userName = signal('John');
const userEmail = signal('john@example.com');

const displayText = computed(() => {
  if (showDetails()) {
    return `${userName()} (${userEmail()})`;
  } else {
    return userName();
  }
});

// Initially, only userName and showDetails are dependencies
// userEmail becomes a dependency only when showDetails is true
```

### Component Example

```typescript
import { Component, signal, computed, ChangeDetectionStrategy } from '@angular/core';

interface CartItem {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

@Component({
  selector: 'app-shopping-cart',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div>
      <h3>Shopping Cart</h3>
      <p>Items: {{ itemCount() }}</p>
      <p>Total: ${{ totalPrice() }}</p>
      <p>Average: ${{ averagePrice() }}</p>
    </div>
  `
})
export class ShoppingCartComponent {
  items = signal<CartItem[]>([
    { id: 1, name: 'Book', price: 20, quantity: 2 },
    { id: 2, name: 'Pen', price: 5, quantity: 3 }
  ]);

  itemCount = computed(() =>
    this.items().reduce((total, item) => total + item.quantity, 0)
  );

  totalPrice = computed(() =>
    this.items().reduce((total, item) => total + (item.price * item.quantity), 0)
  );

  averagePrice = computed(() => {
    const total = this.totalPrice();
    const count = this.itemCount();
    return count > 0 ? total / count : 0;
  });
}
```

## Effects

Effects are operations that run whenever one or more signal values change. They're useful for side effects like logging, local storage synchronization, or DOM manipulation.

### Creating Effects

```typescript
import { effect } from '@angular/core';

const count = signal(0);

// Effect runs whenever count changes
effect(() => {
  console.log(`Count changed to: ${count()}`);
});

// Effect runs immediately (logs "Count changed to: 0")
// Then runs again whenever count.set() or count.update() is called
```

### Effect Use Cases

#### Logging and Analytics

```typescript
effect(() => {
  const user = currentUser();
  const page = currentPage();
  analytics.track('page_view', { user: user.id, page });
});
```

#### Local Storage Synchronization

```typescript
effect(() => {
  const preferences = userPreferences();
  localStorage.setItem('preferences', JSON.stringify(preferences));
});
```

#### DOM Manipulation

```typescript
effect(() => {
  const theme = selectedTheme();
  document.body.className = `theme-${theme}`;
});
```

### Injection Context

Effects must be created within an injection context (component, directive, or service constructor):

```typescript
@Component({
  selector: 'app-user-profile'
})
export class UserProfileComponent {
  userName = signal('');

  constructor() {
    //  Correct - in constructor (injection context)
    effect(() => {
      console.log(`User: ${this.userName()}`);
    });
  }

  //  Alternative - as a field
  private loggingEffect = effect(() => {
    console.log(`User: ${this.userName()}`);
  });
}
```

### Effect Cleanup

Effects can register cleanup functions for handling subscriptions or timers:

```typescript
effect((onCleanup) => {
  const user = currentUser();

  const timer = setTimeout(() => {
    console.log(`User ${user.name} has been active for 5 seconds`);
  }, 5000);

  // Cleanup function runs before the next effect execution or when destroyed
  onCleanup(() => {
    clearTimeout(timer);
  });
});
```

### Best Practices for Effects

- Use effects sparingly - prefer computed signals for derived state
- Avoid creating infinite loops by not writing to signals that the effect reads
- Use cleanup functions for subscriptions and timers
- Keep effects simple and focused
- Effects automatically integrate with Angular's change detection (no manual detectChanges needed)
- In zoneless mode, effects work seamlessly without Zone.js

```typescript
// L Avoid - can cause infinite loops
effect(() => {
  const value = mySignal();
  mySignal.set(value + 1); // Writes to the signal it reads!
});

//  Good - effect for side effects only
effect(() => {
  const value = mySignal();
  console.log('Value changed:', value);
});
```

## Signal Inputs

Signal inputs provide a stable, modern, type-safe way to accept data from parent components using the reactive power of signals. As of Angular 20, signal inputs are the recommended approach for component inputs.

### Basic Signal Inputs

```typescript
import { Component, input } from '@angular/core';

@Component({
  selector: 'app-user-card',
  template: `
    <div class="user-card">
      <h3>{{ name() }}</h3>
      <p>Age: {{ age() }}</p>
      <p>Email: {{ email() }}</p>
    </div>
  `
})
export class UserCardComponent {
  // Input with default value - type inferred as string
  name = input('Unknown User');

  // Input with explicit type - value can be undefined
  age = input<number>();

  // Input with default value and explicit type
  email = input<string>('no-email@example.com');
}
```

### Required Inputs

```typescript
@Component({
  selector: 'app-product-card',
  template: `
    <div>
      <h3>{{ name() }}</h3>
      <p>{{ description() }}</p>
      <p>${{ price() }}</p>
    </div>
  `
})
export class ProductCardComponent {
  // Required inputs must be provided by parent
  name = input.required<string>();
  price = input.required<number>();

  // Optional input with default
  description = input('No description available');
}
```

### Using Signal Inputs

```typescript
// Parent component template
@Component({
  template: `
    <app-user-card
      [name]="userName()"
      [age]="userAge()"
      [email]="userEmail()">
    </app-user-card>

    <app-product-card
      [name]="productName()"
      [price]="productPrice()">
    </app-product-card>
  `
})
export class ParentComponent {
  userName = signal('Alice Johnson');
  userAge = signal(28);
  userEmail = signal('alice@example.com');

  productName = signal('Laptop');
  productPrice = signal(999);
}
```

### Input Transforms

Transform input values automatically:

```typescript
import { Component, input, numberAttribute, booleanAttribute } from '@angular/core';

@Component({
  selector: 'app-form-field',
  template: `
    <div class="form-field" [class.disabled]="disabled()">
      <label>{{ label() }}</label>
      <input [value]="maxLength()" [disabled]="disabled()">
    </div>
  `
})
export class FormFieldComponent {
  // String input that trims whitespace
  label = input('', {
    transform: (value: string) => value.trim()
  });

  // Transform string to number
  maxLength = input(100, {
    transform: numberAttribute
  });

  // Transform to boolean (handles "true"/"false" strings)
  disabled = input(false, {
    transform: booleanAttribute
  });
}
```

### Reading Inputs in Components

Signal inputs can be used in computed signals and effects:

```typescript
@Component({
  selector: 'app-user-profile',
  template: `
    <div>
      <h2>{{ displayName() }}</h2>
      <p>Status: {{ statusMessage() }}</p>
    </div>
  `
})
export class UserProfileComponent {
  firstName = input.required<string>();
  lastName = input.required<string>();
  isOnline = input(false);

  // Computed signal derived from inputs
  displayName = computed(() =>
    `${this.firstName()} ${this.lastName()}`
  );

  statusMessage = computed(() =>
    this.isOnline() ? 'Online' : 'Offline'
  );

  constructor() {
    // Effect that runs when inputs change
    effect(() => {
      console.log(`${this.displayName()} is ${this.statusMessage()}`);
    });
  }
}
```

## Signal Outputs

Signal outputs provide a modern, type-safe way to emit events from child components to their parents using the reactive power of signals. As of Angular 20, signal outputs are the recommended approach for component event emission.

### Basic Signal Outputs

```typescript
import { Component, output } from '@angular/core';

@Component({
  selector: 'app-button',
  template: `
    <button
      [disabled]="disabled()"
      (click)="handleClick()"
      class="custom-button">
      {{ label() }}
    </button>
  `
})
export class ButtonComponent {
  // Input for button configuration
  label = input('Click me');
  disabled = input(false);

  // Output for click events
  clicked = output<void>();

  // Output with data
  buttonPressed = output<{ timestamp: number; label: string }>();

  handleClick() {
    // Emit simple event
    this.clicked.emit();

    // Emit event with data
    this.buttonPressed.emit({
      timestamp: Date.now(),
      label: this.label()
    });
  }
}
```

### Output with Custom Data Types

```typescript
interface UserAction {
  action: 'create' | 'update' | 'delete';
  userId: number;
  data?: any;
}

@Component({
  selector: 'app-user-list',
  template: `
    <div class="user-list">
      @for (user of users(); track user.id) {
        <div class="user-item">
          <span>{{ user.name }}</span>
          <button (click)="editUser(user)">Edit</button>
          <button (click)="deleteUser(user)">Delete</button>
        </div>
      }
      <button (click)="createUser()">Add User</button>
    </div>
  `
})
export class UserListComponent {
  users = input.required<User[]>();

  // Output for user actions
  userAction = output<UserAction>();

  editUser(user: User) {
    this.userAction.emit({
      action: 'update',
      userId: user.id,
      data: user
    });
  }

  deleteUser(user: User) {
    this.userAction.emit({
      action: 'delete',
      userId: user.id
    });
  }

  createUser() {
    this.userAction.emit({
      action: 'create',
      userId: 0
    });
  }
}
```

### Using Signal Outputs in Parent Components

```typescript
@Component({
  selector: 'app-parent',
  template: `
    <div>
      <h2>Button Demo</h2>
      <app-button
        [label]="buttonLabel()"
        [disabled]="isButtonDisabled()"
        (clicked)="onButtonClicked()"
        (buttonPressed)="onButtonPressed($event)">
      </app-button>

      <h2>User Management</h2>
      <app-user-list
        [users]="users()"
        (userAction)="handleUserAction($event)">
      </app-user-list>

      <div class="status">
        <p>Last action: {{ lastAction() }}</p>
        <p>Button clicked: {{ clickCount() }} times</p>
      </div>
    </div>
  `
})
export class ParentComponent {
  // State signals
  buttonLabel = signal('Custom Button');
  isButtonDisabled = signal(false);
  clickCount = signal(0);
  lastAction = signal('None');

  users = signal<User[]>([
    { id: 1, name: 'Alice', email: 'alice@example.com' },
    { id: 2, name: 'Bob', email: 'bob@example.com' }
  ]);

  onButtonClicked() {
    this.clickCount.update(count => count + 1);
    this.lastAction.set('Button clicked');
  }

  onButtonPressed(data: { timestamp: number; label: string }) {
    console.log('Button pressed at:', new Date(data.timestamp));
    console.log('Button label:', data.label);
    this.lastAction.set(`Button "${data.label}" pressed`);
  }

  handleUserAction(action: UserAction) {
    this.lastAction.set(`User ${action.action} action`);

    switch (action.action) {
      case 'create':
        // Handle user creation
        console.log('Creating new user');
        break;
      case 'update':
        // Handle user update
        console.log('Updating user:', action.userId);
        break;
      case 'delete':
        // Handle user deletion
        this.users.update(users =>
          users.filter(user => user.id !== action.userId)
        );
        break;
    }
  }
}
```

### Output Aliases

You can provide aliases for outputs for backward compatibility or naming conventions:

```typescript
@Component({
  selector: 'app-search-box',
  template: `
    <input
      [value]="searchQuery()"
      (input)="onSearchInput($event)"
      (keyup.enter)="onSearchSubmit()"
      placeholder="Search...">
    <button (click)="clearSearch()">Clear</button>
  `
})
export class SearchBoxComponent {
  searchQuery = signal('');

  // Output with alias for backward compatibility
  searchChanged = output<string>({ alias: 'search' });
  searchSubmitted = output<string>({ alias: 'submit' });
  searchCleared = output<void>({ alias: 'clear' });

  onSearchInput(event: Event) {
    const target = event.target as HTMLInputElement;
    this.searchQuery.set(target.value);
    this.searchChanged.emit(target.value);
  }

  onSearchSubmit() {
    this.searchSubmitted.emit(this.searchQuery());
  }

  clearSearch() {
    this.searchQuery.set('');
    this.searchCleared.emit();
  }
}

// Usage in parent template:
// <app-search-box
//   (search)="onSearch($event)"
//   (submit)="onSubmit($event)"
//   (clear)="onClear()">
// </app-search-box>
```

### Advanced Output Patterns

#### Conditional Outputs

```typescript
@Component({
  selector: 'app-form-field',
  template: `
    <div class="form-field">
      <label>{{ label() }}</label>
      <input
        [value]="value()"
        [type]="inputType()"
        (input)="onValueChange($event)"
        (focus)="onFocus()"
        (blur)="onBlur()">

      @if (showValidation() && errorMessage()) {
        <span class="error">{{ errorMessage() }}</span>
      }
    </div>
  `
})
export class FormFieldComponent {
  // Inputs
  label = input('Field');
  value = input('');
  inputType = input<'text' | 'email' | 'password'>('text');
  showValidation = input(true);
  required = input(false);

  // Outputs
  valueChange = output<string>();
  validationChange = output<{ isValid: boolean; error?: string }>();
  focused = output<void>();
  blurred = output<void>();

  // Internal state
  private isTouched = signal(false);

  // Computed validation
  errorMessage = computed(() => {
    if (!this.isTouched() || !this.showValidation()) return null;

    const value = this.value();
    if (this.required() && !value.trim()) {
      return 'This field is required';
    }

    if (this.inputType() === 'email' && value) {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!emailRegex.test(value)) {
        return 'Invalid email format';
      }
    }

    return null;
  });

  isValid = computed(() => !this.errorMessage());

  constructor() {
    // Emit validation changes
    effect(() => {
      this.validationChange.emit({
        isValid: this.isValid(),
        error: this.errorMessage() || undefined
      });
    });
  }

  onValueChange(event: Event) {
    const target = event.target as HTMLInputElement;
    this.valueChange.emit(target.value);
  }

  onFocus() {
    this.focused.emit();
  }

  onBlur() {
    this.isTouched.set(true);
    this.blurred.emit();
  }
}
```

#### Output Chaining and Composition

```typescript
@Component({
  selector: 'app-wizard-step',
  template: `
    <div class="wizard-step">
      <h3>{{ title() }}</h3>
      <div class="step-content">
        <ng-content></ng-content>
      </div>

      <div class="step-actions">
        <button
          [disabled]="!canGoPrevious()"
          (click)="onPrevious()">
          Previous
        </button>
        <button
          [disabled]="!canGoNext()"
          (click)="onNext()">
          Next
        </button>
        @if (isLastStep()) {
          <button
            [disabled]="!isValid()"
            (click)="onComplete()">
            Complete
          </button>
        }
      </div>
    </div>
  `
})
export class WizardStepComponent {
  // Inputs
  title = input.required<string>();
  isValid = input(true);
  canGoPrevious = input(true);
  canGoNext = input(true);
  isLastStep = input(false);

  // Outputs
  stepChange = output<'previous' | 'next'>();
  wizardComplete = output<void>();
  stepValidation = output<boolean>();

  constructor() {
    // Emit validation changes
    effect(() => {
      this.stepValidation.emit(this.isValid());
    });
  }

  onPrevious() {
    if (this.canGoPrevious()) {
      this.stepChange.emit('previous');
    }
  }

  onNext() {
    if (this.canGoNext()) {
      this.stepChange.emit('next');
    }
  }

  onComplete() {
    if (this.isValid()) {
      this.wizardComplete.emit();
    }
  }
}

// Multi-step wizard using composed outputs
@Component({
  selector: 'app-wizard',
  template: `
    <div class="wizard">
      @for (step of steps(); track step.id; let i = $index) {
        @if (i === currentStep()) {
          <app-wizard-step
            [title]="step.title"
            [isValid]="step.isValid"
            [canGoPrevious]="i > 0"
            [canGoNext]="step.isValid && i < steps().length - 1"
            [isLastStep]="i === steps().length - 1"
            (stepChange)="onStepChange($event, i)"
            (wizardComplete)="onWizardComplete()"
            (stepValidation)="onStepValidation($event, i)">

            <!-- Dynamic step content -->
            <ng-container [ngSwitch]="step.id">
              <div *ngSwitchCase="'personal'">Personal Information Form</div>
              <div *ngSwitchCase="'address'">Address Information Form</div>
              <div *ngSwitchCase="'payment'">Payment Information Form</div>
              <div *ngSwitchCase="'review'">Review and Confirm</div>
            </ng-container>
          </app-wizard-step>
        }
      }
    </div>
  `
})
export class WizardComponent {
  currentStep = signal(0);

  steps = signal([
    { id: 'personal', title: 'Personal Info', isValid: false },
    { id: 'address', title: 'Address', isValid: false },
    { id: 'payment', title: 'Payment', isValid: false },
    { id: 'review', title: 'Review', isValid: true }
  ]);

  // Wizard outputs
  wizardCompleted = output<any>();
  stepChanged = output<{ from: number; to: number }>();

  onStepChange(direction: 'previous' | 'next', currentIndex: number) {
    const newStep = direction === 'next' ? currentIndex + 1 : currentIndex - 1;

    this.stepChanged.emit({
      from: currentIndex,
      to: newStep
    });

    this.currentStep.set(newStep);
  }

  onStepValidation(isValid: boolean, stepIndex: number) {
    this.steps.update(steps =>
      steps.map((step, index) =>
        index === stepIndex ? { ...step, isValid } : step
      )
    );
  }

  onWizardComplete() {
    const wizardData = {
      completedAt: new Date(),
      steps: this.steps()
    };

    this.wizardCompleted.emit(wizardData);
  }
}
```

### Best Practices for Signal Outputs

1. **Use descriptive names** that clearly indicate what the output represents
2. **Provide type information** for output data to ensure type safety
3. **Keep output data minimal** - only emit what the parent needs
4. **Use aliases** for backward compatibility when renaming outputs
5. **Document output behavior** - when they emit and what data they contain
6. **Consider using computed signals** to conditionally emit outputs
7. **Avoid emitting on every signal change** - use effects wisely
8. **Group related outputs** when it makes sense for the API

### When to Use Signal Outputs

Use signal outputs when:
- Creating reusable components that need to communicate with parents
- Building form controls that emit validation states
- Implementing components that trigger actions in parent components
- Creating event-driven architectures
- Building components that act as user interface controls

Examples of good use cases:
- Form field validation events
- Button click events with data
- Modal dialog results
- Search box queries
- Pagination events
- File upload progress
- Game events and actions

## Model Inputs

Model inputs enable two-way data binding between parent and child components using signals. They allow a component to both receive values from and send values back to its parent.

### Basic Model Inputs

```typescript
import { Component, model } from '@angular/core';

@Component({
  selector: 'app-custom-input',
  template: `
    <div class="custom-input">
      <label>{{ label() }}</label>
      <input
        [value]="value()"
        (input)="updateValue($event)"
        [placeholder]="placeholder()">
    </div>
  `
})
export class CustomInputComponent {
  // Model input for two-way binding
  value = model<string>('');

  // Regular inputs
  label = input('Input Field');
  placeholder = input('Enter text...');

  updateValue(event: Event) {
    const target = event.target as HTMLInputElement;
    this.value.set(target.value);
  }
}
```

### Using Model Inputs with Two-Way Binding

```typescript
@Component({
  selector: 'app-form',
  template: `
    <div>
      <app-custom-input
        [(value)]="userName"
        [label]="'Username'"
        [placeholder]="'Enter your username'">
      </app-custom-input>

      <app-custom-input
        [(value)]="email"
        [label]="'Email'"
        [placeholder]="'Enter your email'">
      </app-custom-input>

      <p>Username: {{ userName() }}</p>
      <p>Email: {{ email() }}</p>
    </div>
  `
})
export class FormComponent {
  userName = signal('');
  email = signal('');
}
```

### Advanced Model Input Example

```typescript
@Component({
  selector: 'app-counter-control',
  template: `
    <div class="counter-control">
      <button (click)="decrement()" [disabled]="value() <= min()">-</button>
      <span class="value">{{ value() }}</span>
      <button (click)="increment()" [disabled]="value() >= max()">+</button>
      <p class="range">Range: {{ min() }} - {{ max() }}</p>
    </div>
  `
})
export class CounterControlComponent {
  // Model input for the main value
  value = model(0);

  // Configuration inputs
  min = input(0);
  max = input(100);
  step = input(1);

  increment() {
    const current = this.value();
    const newValue = Math.min(current + this.step(), this.max());
    this.value.set(newValue);
  }

  decrement() {
    const current = this.value();
    const newValue = Math.max(current - this.step(), this.min());
    this.value.set(newValue);
  }
}
```

### Implicit Change Events

Model inputs automatically create change events:

```typescript
// Child component with model
@Component({
  selector: 'app-slider',
  template: `<input type="range" [value]="value()" (input)="onChange($event)">`
})
export class SliderComponent {
  // Creates automatic "valueChange" event
  value = model(50);

  onChange(event: Event) {
    const target = event.target as HTMLInputElement;
    this.value.set(Number(target.value));
  }
}

// Parent can listen to the automatic change event
@Component({
  template: `
    <app-slider
      [(value)]="sliderValue"
      (valueChange)="onSliderChange($event)">
    </app-slider>
  `
})
export class ParentComponent {
  sliderValue = signal(50);

  onSliderChange(newValue: number) {
    console.log('Slider changed to:', newValue);
  }
}
```

### When to Use Model Inputs

Use model inputs when:
- Creating custom form controls
- Building reusable components that need to modify parent state
- Implementing components that act as input/output pairs
- You need two-way data binding between parent and child

Examples of good use cases:
- Custom date pickers
- Toggle switches
- Sliders and range inputs
- Search boxes with filters
- Any component that acts as a form control

## Signal Queries

Signal queries provide a modern, reactive way to access child components, directives, and DOM elements within Angular components. As of Angular 19, signal queries are production-ready and the recommended approach for querying component children.

### View Queries

View queries retrieve results from elements in the component's template. These are the elements defined directly in the component's own template.

#### viewChild

Query for a single child element or component:

```typescript
import { Component, viewChild, ElementRef } from '@angular/core';

@Component({
  selector: 'app-parent',
  template: `
    <div>
      <button #saveBtn>Save</button>
      <app-custom-card></app-custom-card>
    </div>
  `
})
export class ParentComponent {
  // Query by template reference variable
  saveButton = viewChild<ElementRef<HTMLButtonElement>>('saveBtn');

  // Query by component type
  customCard = viewChild(CustomCardComponent);

  ngAfterViewInit() {
    // Access the elements after view initialization
    const buttonEl = this.saveButton();
    const cardComponent = this.customCard();

    if (buttonEl) {
      console.log('Save button:', buttonEl.nativeElement);
    }

    if (cardComponent) {
      console.log('Card component:', cardComponent);
    }
  }
}
```

#### viewChild.required

For elements that you know will always be present, use required queries:

```typescript
@Component({
  selector: 'app-form',
  template: `
    <form #userForm>
      <input #usernameInput type="text" required>
      <button type="submit">Submit</button>
    </form>
  `
})
export class FormComponent {
  // Required queries - guaranteed to have a value
  form = viewChild.required<ElementRef<HTMLFormElement>>('userForm');
  usernameInput = viewChild.required<ElementRef<HTMLInputElement>>('usernameInput');

  submitForm() {
    // No need to check for undefined - required queries guarantee a value
    const formEl = this.form().nativeElement;
    const inputEl = this.usernameInput().nativeElement;

    if (formEl.checkValidity()) {
      console.log('Username:', inputEl.value);
    }
  }
}
```

#### viewChildren

Query for multiple child elements:

```typescript
@Component({
  selector: 'app-tab-container',
  template: `
    <div class="tabs">
      <app-tab title="Tab 1">Content 1</app-tab>
      <app-tab title="Tab 2">Content 2</app-tab>
      <app-tab title="Tab 3">Content 3</app-tab>
    </div>
  `
})
export class TabContainerComponent {
  // Query for all tab components
  tabs = viewChildren(TabComponent);

  // Computed to get tab titles
  tabTitles = computed(() =>
    this.tabs().map(tab => tab.title)
  );

  // Computed to count tabs
  tabCount = computed(() => this.tabs().length);

  activateTab(index: number) {
    const tabs = this.tabs();
    if (tabs[index]) {
      tabs[index].activate();
    }
  }

  ngAfterViewInit() {
    // Access all tabs after view initialization
    effect(() => {
      console.log(`Found ${this.tabCount()} tabs:`, this.tabTitles());
    });
  }
}
```

### Content Queries

Content queries retrieve results from elements projected into the component through content projection (ng-content).

#### contentChild

Query for a single projected child:

```typescript
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <div class="card-header">
        <ng-content select="[slot=header]"></ng-content>
      </div>
      <div class="card-body">
        <ng-content></ng-content>
      </div>
    </div>
  `
})
export class CardComponent {
  // Query for projected header component
  header = contentChild(CardHeaderComponent);

  // Query by template reference variable
  headerElement = contentChild<ElementRef>('cardHeader');

  // Computed to check if header exists
  hasHeader = computed(() => !!this.header());

  ngAfterContentInit() {
    effect(() => {
      const header = this.header();
      if (header) {
        console.log('Header component found:', header);
      }
    });
  }
}

// Usage in parent template:
// <app-card>
//   <app-card-header slot="header">My Card Title</app-card-header>
//   <p>Card content goes here</p>
// </app-card>
```

#### contentChild.required

For projected content that must be present:

```typescript
@Component({
  selector: 'app-dialog',
  template: `
    <div class="dialog-overlay">
      <div class="dialog">
        <div class="dialog-title">
          <ng-content select="[slot=title]"></ng-content>
        </div>
        <div class="dialog-content">
          <ng-content></ng-content>
        </div>
        <div class="dialog-actions">
          <ng-content select="[slot=actions]"></ng-content>
        </div>
      </div>
    </div>
  `
})
export class DialogComponent {
  // Required projected content
  titleComponent = contentChild.required(DialogTitleComponent);

  closeDialog() {
    // Can safely access without null checking
    const title = this.titleComponent();
    console.log(`Closing dialog: ${title.text}`);
  }
}
```

#### contentChildren

Query for multiple projected children:

```typescript
@Component({
  selector: 'app-menu',
  template: `
    <nav class="menu">
      <ng-content></ng-content>
    </nav>
  `
})
export class MenuComponent {
  // Query for all projected menu items
  menuItems = contentChildren(MenuItemComponent);

  // Computed values based on content
  itemCount = computed(() => this.menuItems().length);
  hasItems = computed(() => this.itemCount() > 0);

  enabledItems = computed(() =>
    this.menuItems().filter(item => !item.disabled)
  );

  ngAfterContentInit() {
    // React to content changes
    effect(() => {
      console.log(`Menu has ${this.itemCount()} items`);

      // Set up keyboard navigation
      this.menuItems().forEach((item, index) => {
        item.tabIndex = index;
      });
    });
  }

  selectNext() {
    const items = this.enabledItems();
    // Implementation for keyboard navigation...
  }
}

// Usage:
// <app-menu>
//   <app-menu-item>Home</app-menu-item>
//   <app-menu-item>About</app-menu-item>
//   <app-menu-item [disabled]="true">Admin</app-menu-item>
// </app-menu>
```

### Query Options

Signal queries support various options to customize their behavior:

#### Reading Specific Values

Use the `read` option to extract specific values from queried elements:

```typescript
@Component({
  selector: 'app-template-example',
  template: `
    <ng-template #myTemplate>
      <p>Template content</p>
    </ng-template>

    <div #myDiv class="content">
      <span>Some content</span>
    </div>
  `
})
export class TemplateExampleComponent {
  // Read TemplateRef from template reference
  template = viewChild('myTemplate', { read: TemplateRef });

  // Read ElementRef from component (default behavior)
  divElement = viewChild<ElementRef<HTMLDivElement>>('myDiv');

  // Read ViewContainerRef from element
  viewContainer = viewChild('myDiv', { read: ViewContainerRef });

  ngAfterViewInit() {
    const template = this.template();
    const element = this.divElement();
    const container = this.viewContainer();

    if (template && container) {
      // Use template and container...
      container.createEmbeddedView(template);
    }
  }
}
```

#### Content Descendants

Content queries can control whether to traverse into descendant elements:

```typescript
@Component({
  selector: 'app-accordion',
  template: `
    <div class="accordion">
      <ng-content></ng-content>
    </div>
  `
})
export class AccordionComponent {
  // Find only direct children (default behavior for contentChildren)
  directPanels = contentChildren(AccordionPanelComponent);

  // Find all descendants including nested panels
  allPanels = contentChildren(AccordionPanelComponent, { descendants: true });

  ngAfterContentInit() {
    effect(() => {
      console.log(`Direct panels: ${this.directPanels().length}`);
      console.log(`All panels: ${this.allPanels().length}`);
    });
  }
}

// Usage with nested structure:
// <app-accordion>
//   <app-accordion-panel>Panel 1</app-accordion-panel>
//   <div class="group">
//     <app-accordion-panel>Nested Panel</app-accordion-panel>
//   </div>
// </app-accordion>
```

### Advanced Query Patterns

#### Conditional Queries

Queries automatically handle conditional rendering:

```typescript
@Component({
  selector: 'app-conditional-content',
  template: `
    <div>
      @if (showAdvanced()) {
        <app-advanced-settings #advanced></app-advanced-settings>
      }

      @if (showBasic()) {
        <app-basic-settings #basic></app-basic-settings>
      }
    </div>
  `
})
export class ConditionalContentComponent {
  showAdvanced = signal(false);
  showBasic = signal(true);

  // These queries automatically handle presence/absence
  advancedSettings = viewChild<AdvancedSettingsComponent>('advanced');
  basicSettings = viewChild<BasicSettingsComponent>('basic');

  // Computed to determine which settings are active
  activeSettings = computed(() => {
    const advanced = this.advancedSettings();
    const basic = this.basicSettings();

    if (advanced) return 'advanced';
    if (basic) return 'basic';
    return 'none';
  });

  constructor() {
    // React to settings changes
    effect(() => {
      const settings = this.activeSettings();
      console.log(`Active settings: ${settings}`);
    });
  }

  toggleAdvanced() {
    this.showAdvanced.update(show => !show);
    if (this.showAdvanced()) {
      this.showBasic.set(false);
    }
  }
}
```

#### Query with Dynamic Components

```typescript
@Component({
  selector: 'app-dynamic-host',
  template: `
    <div #dynamicHost></div>
    <button (click)="loadComponent()">Load Component</button>
  `
})
export class DynamicHostComponent {
  dynamicHost = viewChild.required<ElementRef>('dynamicHost');

  // Keep reference to dynamically created component
  private dynamicComponentRef = signal<ComponentRef<any> | null>(null);

  constructor(private viewContainer: ViewContainerRef) {}

  async loadComponent() {
    // Dynamic component loading
    const { DynamicComponent } = await import('./dynamic.component');

    this.viewContainer.clear();
    const componentRef = this.viewContainer.createComponent(DynamicComponent);

    this.dynamicComponentRef.set(componentRef);

    // You can now interact with the dynamically loaded component
    componentRef.instance.someProperty = 'Hello from host';
  }

  ngOnDestroy() {
    // Clean up dynamic component
    const componentRef = this.dynamicComponentRef();
    if (componentRef) {
      componentRef.destroy();
    }
  }
}
```

### Signal Queries vs Decorator Queries

Signal queries offer several advantages over the traditional decorator-based queries:

```typescript
// ❌ Old decorator approach
@Component({
  template: '<app-child #child></app-child>'
})
export class OldStyleComponent {
  @ViewChild('child') child!: ChildComponent;

  ngAfterViewInit() {
    // Must wait for lifecycle hook
    console.log(this.child.data);
  }
}

// ✅ New signal approach
@Component({
  template: '<app-child #child></app-child>'
})
export class ModernComponent {
  child = viewChild.required<ChildComponent>('child');

  // Can use in computed and effects
  childData = computed(() => this.child().data);

  constructor() {
    // Automatically runs when child is available
    effect(() => {
      console.log('Child data:', this.childData());
    });
  }
}
```

### Best Practices for Signal Queries

1. **Use required queries** when elements are guaranteed to exist
2. **Prefer signal queries** over decorator queries for new code
3. **Use computed signals** to derive values from query results
4. **Handle undefined values** gracefully for optional queries
5. **Combine with effects** for reactive behaviors
6. **Use specific types** for better type safety
7. **Query at the right level** - view queries for template elements, content queries for projected content

### Migration from Decorator Queries

Angular provides an automatic migration tool:

```bash
# Run the signal queries migration
ng generate @angular/core:signal-queries-migration

# With options
ng generate @angular/core:signal-queries-migration --best-effort-mode --insert-todos
```

The migration will convert:
- `@ViewChild()` → `viewChild()`
- `@ViewChildren()` → `viewChildren()`
- `@ContentChild()` → `contentChild()`
- `@ContentChildren()` → `contentChildren()`

And update all references to call the signal functions.

## Resource API

The Resource API provides a powerful, signal-based approach to managing asynchronous data fetching in Angular applications. It integrates seamlessly with signals to provide reactive data loading with built-in loading states, error handling, and streaming capabilities.

### Basic Resource Usage

```typescript
import { Component, signal, resource } from '@angular/core';

interface User {
  id: number;
  name: string;
  email: string;
}

@Component({
  selector: 'app-user-profile',
  template: `
    <div>
      @if (userResource.isLoading()) {
        <div class="loading">Loading user...</div>
      } @else if (userResource.hasValue()) {
        <div class="user-profile">
          <h2>{{ userResource.value().name }}</h2>
          <p>{{ userResource.value().email }}</p>
        </div>
      } @else if (userResource.error()) {
        <div class="error">
          <p>Failed to load user: {{ userResource.error()?.message }}</p>
          <button (click)="userResource.reload()">Retry</button>
        </div>
      }
    </div>
  `
})
export class UserProfileComponent {
  userId = signal(1);

  // Resource that fetches user data
  userResource = resource<User, number>({
    // The request is triggered when this signal changes
    params: () => this.userId(),

    // The async function that fetches data
    loader: async ({ params: userId }) => {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) {
        throw new Error(`Failed to fetch user: ${response.statusText}`);
      }
      return response.json() as User;
    }
  });

  switchUser(newUserId: number) {
    this.userId.set(newUserId);
    // Resource automatically refetches when userId changes
  }
}
```

### Resource with Default Values

```typescript
@Component({
  selector: 'app-product-list',
  template: `
    <div>
      <h2>Products</h2>
      @for (product of productsResource.value(); track product.id) {
        <div class="product">
          <h3>{{ product.name }}</h3>
          <p>{{ product.price | currency }}</p>
        </div>
      }

      @if (productsResource.isLoading()) {
        <div class="loading-overlay">Updating products...</div>
      }
    </div>
  `
})
export class ProductListComponent {
  category = signal('electronics');

  productsResource = resource({
    // Default value to show while loading initial data
    defaultValue: [] as Product[],

    params: () => this.category(),

    loader: async ({ params: category }) => {
      const response = await fetch(`/api/products?category=${category}`);
      return response.json() as Product[];
    }
  });

  changeCategory(newCategory: string) {
    this.category.set(newCategory);
  }
}
```

### Resource with Multiple Parameters

```typescript
@Component({
  selector: 'app-search-results',
  template: `
    <div>
      <input
        [value]="searchQuery()"
        (input)="updateSearchQuery($event)">

      <select [value]="sortBy()" (change)="updateSortBy($event)">
        <option value="name">Name</option>
        <option value="price">Price</option>
        <option value="rating">Rating</option>
      </select>

      @if (searchResource.isLoading()) {
        <div>Searching...</div>
      } @else {
        <div class="results">
          <p>Found {{ searchResource.value().length }} results</p>
          @for (item of searchResource.value(); track item.id) {
            <div class="result-item">{{ item.name }}</div>
          }
        </div>
      }
    </div>
  `
})
export class SearchResultsComponent {
  searchQuery = signal('');
  sortBy = signal('name');

  // Resource that depends on multiple signals
  searchResource = resource({
    defaultValue: [] as SearchResult[],

    // Combine multiple signals into params
    params: () => ({
      query: this.searchQuery(),
      sort: this.sortBy()
    }),

    loader: async ({ params }) => {
      if (!params.query.trim()) {
        return []; // Return empty results for empty query
      }

      const url = new URL('/api/search', window.location.origin);
      url.searchParams.set('q', params.query);
      url.searchParams.set('sort', params.sort);

      const response = await fetch(url);
      return response.json() as SearchResult[];
    }
  });

  updateSearchQuery(event: Event) {
    const target = event.target as HTMLInputElement;
    this.searchQuery.set(target.value);
  }

  updateSortBy(event: Event) {
    const target = event.target as HTMLSelectElement;
    this.sortBy.set(target.value);
  }
}
```

### Resource Status and State Management

```typescript
@Component({
  selector: 'app-data-manager',
  template: `
    <div>
      <div class="status-bar">
        <span class="status">Status: {{ getStatusText() }}</span>
        <button
          (click)="dataResource.reload()"
          [disabled]="dataResource.isLoading()">
          Refresh
        </button>
      </div>

      @switch (true) {
        @case (dataResource.isLoading()) {
          <div class="loading">
            <div class="spinner"></div>
            <p>Loading data...</p>
          </div>
        }
        @case (dataResource.hasValue()) {
          <div class="data-display">
            <h3>Data loaded successfully</h3>
            <pre>{{ dataResource.value() | json }}</pre>
            <p>Last updated: {{ lastUpdateTime() | date:'medium' }}</p>
          </div>
        }
        @case (dataResource.error()) {
          <div class="error-state">
            <h3>Error occurred</h3>
            <p>{{ dataResource.error()?.message }}</p>
            <button (click)="dataResource.reload()">Try Again</button>
          </div>
        }
      }
    </div>
  `
})
export class DataManagerComponent {
  refreshTrigger = signal(0);
  lastUpdateTime = signal<Date | null>(null);

  dataResource = resource({
    params: () => this.refreshTrigger(),

    loader: async () => {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 1000));

      // Randomly fail sometimes to demonstrate error handling
      if (Math.random() < 0.3) {
        throw new Error('Random API failure');
      }

      this.lastUpdateTime.set(new Date());
      return {
        timestamp: Date.now(),
        data: `Sample data ${this.refreshTrigger()}`
      };
    }
  });

  getStatusText(): string {
    if (this.dataResource.isLoading()) return 'Loading';
    if (this.dataResource.hasValue()) return 'Success';
    if (this.dataResource.error()) return 'Error';
    return 'Unknown';
  }

  forceRefresh() {
    this.refreshTrigger.update(count => count + 1);
  }
}
```

### HTTP Resource Integration

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class ApiService {
  private http = inject(HttpClient);

  // Helper method for creating HTTP-based resources
  createHttpResource<T>(url: string, options?: RequestInit) {
    return resource<T, void>({
      loader: async () => {
        return this.http.get<T>(url).toPromise();
      }
    });
  }

  // Resource factory with parameters
  createParameterizedResource<T, P>(urlFactory: (params: P) => string) {
    return (params: () => P) => resource<T, P>({
      params,
      loader: async ({ params }) => {
        const url = urlFactory(params);
        return this.http.get<T>(url).toPromise();
      }
    });
  }
}

@Component({
  selector: 'app-http-example',
  template: `
    <div>
      @if (userResource.hasValue()) {
        <h2>{{ userResource.value().name }}</h2>
      }

      @if (postsResource.hasValue()) {
        <div class="posts">
          @for (post of postsResource.value(); track post.id) {
            <div class="post">{{ post.title }}</div>
          }
        </div>
      }
    </div>
  `
})
export class HttpExampleComponent {
  private api = inject(ApiService);

  userId = signal(1);

  // Simple HTTP resource
  userResource = resource({
    params: () => this.userId(),
    loader: async ({ params: userId }) => {
      return inject(HttpClient).get<User>(`/api/users/${userId}`).toPromise();
    }
  });

  // Using service helper
  postsResource = this.api.createParameterizedResource<Post[], number>(
    (userId) => `/api/users/${userId}/posts`
  )(this.userId);
}
```

### Streaming Resources

```typescript
@Component({
  selector: 'app-streaming-example',
  template: `
    <div>
      <h2>Real-time Data Stream</h2>

      @if (streamResource.isLoading()) {
        <div>Connecting to stream...</div>
      } @else if (streamResource.hasValue()) {
        <div class="stream-data">
          <p>Current value: {{ streamResource.value() }}</p>
          <p>Updates: {{ updateCount() }}</p>
        </div>
      } @else if (streamResource.error()) {
        <div class="error">
          Stream error: {{ streamResource.error()?.message }}
          <button (click)="streamResource.reload()">Reconnect</button>
        </div>
      }
    </div>
  `
})
export class StreamingExampleComponent {
  updateCount = signal(0);

  streamResource = resource({
    stream: async () => {
      const data = signal<ResourceStreamItem<string>>({ value: 'Initial value' });

      // Simulate streaming data
      const interval = setInterval(() => {
        const newValue = `Stream update ${Date.now()}`;
        data.set({ value: newValue });
        this.updateCount.update(count => count + 1);
      }, 2000);

      // Cleanup function
      setTimeout(() => {
        clearInterval(interval);
      }, 30000);

      return data;
    }
  });
}
```

### Advanced Resource Patterns

#### Dependent Resources

```typescript
@Component({
  selector: 'app-dependent-resources',
  template: `
    <div>
      <select [value]="selectedCompanyId()" (change)="onCompanyChange($event)">
        @for (company of companiesResource.value(); track company.id) {
          <option [value]="company.id">{{ company.name }}</option>
        }
      </select>

      @if (employeesResource.hasValue()) {
        <div class="employees">
          @for (employee of employeesResource.value(); track employee.id) {
            <div class="employee">{{ employee.name }}</div>
          }
        </div>
      }
    </div>
  `
})
export class DependentResourcesComponent {
  selectedCompanyId = signal<number | null>(null);

  // First resource - companies list
  companiesResource = resource({
    defaultValue: [] as Company[],
    loader: async () => {
      const response = await fetch('/api/companies');
      return response.json() as Company[];
    }
  });

  // Second resource - depends on first resource's result
  employeesResource = resource({
    defaultValue: [] as Employee[],
    params: () => this.selectedCompanyId(),
    loader: async ({ params: companyId }) => {
      if (!companyId) return [];

      const response = await fetch(`/api/companies/${companyId}/employees`);
      return response.json() as Employee[];
    }
  });

  onCompanyChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    this.selectedCompanyId.set(Number(target.value));
  }
}
```

#### Resource with Optimistic Updates

```typescript
@Component({
  selector: 'app-optimistic-updates',
  template: `
    <div>
      <form (submit)="addTodo($event)">
        <input #todoInput placeholder="New todo...">
        <button type="submit" [disabled]="isSubmitting()">Add</button>
      </form>

      <div class="todos">
        @for (todo of allTodos(); track todo.id) {
          <div class="todo" [class.pending]="todo.pending">
            {{ todo.text }}
            @if (todo.pending) {
              <span class="status">Saving...</span>
            }
          </div>
        }
      </div>
    </div>
  `
})
export class OptimisticUpdatesComponent {
  isSubmitting = signal(false);
  pendingTodos = signal<Todo[]>([]);

  todosResource = resource({
    defaultValue: [] as Todo[],
    loader: async () => {
      const response = await fetch('/api/todos');
      return response.json() as Todo[];
    }
  });

  // Combine server todos with pending optimistic updates
  allTodos = computed(() => [
    ...this.todosResource.value(),
    ...this.pendingTodos()
  ]);

  async addTodo(event: Event) {
    event.preventDefault();
    const form = event.target as HTMLFormElement;
    const input = form.querySelector('input') as HTMLInputElement;
    const text = input.value.trim();

    if (!text) return;

    const optimisticTodo: Todo = {
      id: Date.now(), // Temporary ID
      text,
      completed: false,
      pending: true
    };

    // Add optimistically
    this.pendingTodos.update(todos => [...todos, optimisticTodo]);
    input.value = '';
    this.isSubmitting.set(true);

    try {
      // Submit to server
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text })
      });

      if (!response.ok) throw new Error('Failed to create todo');

      // Remove from pending and refresh from server
      this.pendingTodos.update(todos =>
        todos.filter(t => t.id !== optimisticTodo.id)
      );
      this.todosResource.reload();

    } catch (error) {
      // Revert optimistic update on error
      this.pendingTodos.update(todos =>
        todos.filter(t => t.id !== optimisticTodo.id)
      );
      console.error('Failed to add todo:', error);
    } finally {
      this.isSubmitting.set(false);
    }
  }
}
```

### Best Practices for Resource API

1. **Use default values** for immediate rendering while data loads
2. **Handle all states** - loading, success, and error cases in templates
3. **Combine resources with computed signals** for derived data
4. **Use streaming** for real-time data that updates frequently
5. **Implement optimistic updates** for better user experience
6. **Cache resources at appropriate levels** - component vs service
7. **Handle cleanup** properly for streaming resources
8. **Use typed interfaces** for better type safety with resource data

### Resource API vs Traditional Approaches

```typescript
// ❌ Traditional Observable approach
@Component({
  template: `
    <div>
      @if (loading) {
        <div>Loading...</div>
      } @else if (error) {
        <div>Error: {{ error }}</div>
      } @else {
        <div>{{ user?.name }}</div>
      }
    </div>
  `
})
export class TraditionalComponent implements OnInit, OnDestroy {
  user: User | null = null;
  loading = false;
  error: string | null = null;
  private destroy$ = new Subject<void>();

  ngOnInit() {
    this.loading = true;
    this.userService.getUser(1)
      .pipe(takeUntil(this.destroy$))
      .subscribe({
        next: user => {
          this.user = user;
          this.loading = false;
        },
        error: err => {
          this.error = err.message;
          this.loading = false;
        }
      });
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// ✅ Resource API approach
@Component({
  template: `
    <div>
      @if (userResource.isLoading()) {
        <div>Loading...</div>
      } @else if (userResource.error()) {
        <div>Error: {{ userResource.error()?.message }}</div>
      } @else {
        <div>{{ userResource.value()?.name }}</div>
      }
    </div>
  `
})
export class ModernComponent {
  userResource = resource({
    loader: () => this.userService.getUser(1).toPromise()
  });

  constructor(private userService: UserService) {}

  // No lifecycle management needed!
  // No manual state management needed!
  // Automatically handles cleanup!
}
```

The Resource API significantly simplifies asynchronous data management while providing better integration with Angular's reactive system through signals.

## Advanced Topics

### Signal Equality Functions

By default, signals use `Object.is()` for equality checking. You can provide custom equality functions:

```typescript
import { signal } from '@angular/core';
import { isEqual } from 'lodash-es';

// Custom equality for deep object comparison
const userSettings = signal(
  { theme: 'dark', language: 'en' },
  { equal: isEqual }
);

// This won't trigger updates because the objects are deeply equal
userSettings.set({ theme: 'dark', language: 'en' });

// Custom equality for arrays
const items = signal(
  [1, 2, 3],
  { equal: (a, b) => a.length === b.length && a.every((v, i) => v === b[i]) }
);
```

### linkedSignal

`linkedSignal` is an advanced signal API that creates a writable signal whose value is automatically reset based on a reactive computation. It's particularly useful for derived state that needs to be writable but should reset when its dependencies change.

#### Basic linkedSignal

```typescript
import { Component, signal, linkedSignal, computed } from '@angular/core';

@Component({
  selector: 'app-search-with-history',
  template: `
    <div>
      <input
        [value]="searchQuery()"
        (input)="updateSearchQuery($event)"
        placeholder="Search...">

      <div class="search-history">
        <h3>Search History</h3>
        @for (term of searchHistory(); track $index) {
          <button (click)="selectFromHistory(term)">{{ term }}</button>
        }
        <button (click)="clearHistory()">Clear History</button>
      </div>

      <div class="results">
        <p>Searching for: "{{ searchQuery() }}"</p>
        <p>History count: {{ searchHistory().length }}</p>
      </div>
    </div>
  `
})
export class SearchWithHistoryComponent {
  searchQuery = signal('');

  // linkedSignal that maintains search history
  searchHistory = linkedSignal<string[]>({
    source: () => this.searchQuery(),
    computation: (currentQuery, previous) => {
      const previousHistory = previous?.value || [];

      // Don't add empty queries or duplicates
      if (!currentQuery.trim() || previousHistory.includes(currentQuery)) {
        return previousHistory;
      }

      // Add new query to history (keep last 10)
      return [...previousHistory, currentQuery].slice(-10);
    }
  });

  updateSearchQuery(event: Event) {
    const target = event.target as HTMLInputElement;
    this.searchQuery.set(target.value);
  }

  selectFromHistory(term: string) {
    this.searchQuery.set(term);
  }

  clearHistory() {
    // Can manually override linkedSignal value
    this.searchHistory.set([]);
  }
}
```

#### linkedSignal with Previous Values

```typescript
@Component({
  selector: 'app-counter-with-delta',
  template: `
    <div>
      <p>Current Count: {{ count() }}</p>
      <p>Previous Count: {{ previousCount() }}</p>
      <p>Delta: {{ delta() }}</p>
      <p>Change History: {{ changeHistory().join(', ') }}</p>

      <button (click)="increment()">+1</button>
      <button (click)="decrement()">-1</button>
      <button (click)="reset()">Reset</button>
    </div>
  `
})
export class CounterWithDeltaComponent {
  count = signal(0);

  // Track previous count value
  previousCount = linkedSignal({
    source: () => this.count(),
    computation: (currentCount, previous) => {
      // Return the previous source value, or 0 if this is the first time
      return previous?.source ?? 0;
    }
  });

  // Calculate delta between current and previous
  delta = computed(() => this.count() - this.previousCount());

  // Track history of changes
  changeHistory = linkedSignal<number[]>({
    source: () => this.delta(),
    computation: (currentDelta, previous) => {
      const previousHistory = previous?.value || [];

      // Don't record zero deltas (initial state)
      if (currentDelta === 0) {
        return previousHistory;
      }

      // Keep last 5 changes
      return [...previousHistory, currentDelta].slice(-5);
    }
  });

  increment() {
    this.count.update(c => c + 1);
  }

  decrement() {
    this.count.update(c => c - 1);
  }

  reset() {
    this.count.set(0);
  }
}
```

#### Chat History with linkedSignal

```typescript
interface Message {
  id: string;
  text: string;
  timestamp: Date;
  sender: 'user' | 'ai';
}

@Component({
  selector: 'app-chat-interface',
  template: `
    <div class="chat-container">
      <div class="messages">
        @for (message of chatHistory(); track message.id) {
          <div class="message" [class]="message.sender">
            <span class="sender">{{ message.sender }}:</span>
            <span class="text">{{ message.text }}</span>
            <span class="time">{{ message.timestamp | date:'short' }}</span>
          </div>
        }
      </div>

      <div class="input-area">
        <input
          [value]="currentMessage()"
          (input)="updateCurrentMessage($event)"
          (keyup.enter)="sendMessage()"
          placeholder="Type a message...">
        <button (click)="sendMessage()" [disabled]="!currentMessage().trim()">
          Send
        </button>
      </div>

      <div class="chat-stats">
        <p>Total messages: {{ chatHistory().length }}</p>
        <p>User messages: {{ userMessageCount() }}</p>
        <p>AI responses: {{ aiMessageCount() }}</p>
      </div>
    </div>
  `
})
export class ChatInterfaceComponent {
  currentMessage = signal('');
  newMessage = signal<Message | null>(null);

  // Build chat history using linkedSignal
  chatHistory = linkedSignal<Message[]>({
    source: () => this.newMessage(),
    computation: (latestMessage, previous) => {
      const existingHistory = previous?.value || [];

      if (!latestMessage) {
        return existingHistory;
      }

      return [...existingHistory, latestMessage];
    }
  });

  // Computed stats
  userMessageCount = computed(() =>
    this.chatHistory().filter(msg => msg.sender === 'user').length
  );

  aiMessageCount = computed(() =>
    this.chatHistory().filter(msg => msg.sender === 'ai').length
  );

  updateCurrentMessage(event: Event) {
    const target = event.target as HTMLInputElement;
    this.currentMessage.set(target.value);
  }

  sendMessage() {
    const text = this.currentMessage().trim();
    if (!text) return;

    // Add user message
    const userMessage: Message = {
      id: `user-${Date.now()}`,
      text,
      timestamp: new Date(),
      sender: 'user'
    };

    this.newMessage.set(userMessage);
    this.currentMessage.set('');

    // Simulate AI response
    setTimeout(() => {
      const aiMessage: Message = {
        id: `ai-${Date.now()}`,
        text: `AI response to: "${text}"`,
        timestamp: new Date(),
        sender: 'ai'
      };

      this.newMessage.set(aiMessage);
    }, 1000);
  }
}
```

#### Infinite Scroll with linkedSignal

```typescript
@Component({
  selector: 'app-infinite-scroll',
  template: `
    <div class="scroll-container" (scroll)="onScroll($event)">
      @for (item of allItems(); track item.id) {
        <div class="item">{{ item.name }}</div>
      }

      @if (isLoading()) {
        <div class="loading">Loading more...</div>
      }
    </div>

    <div class="stats">
      <p>Items loaded: {{ allItems().length }}</p>
      <p>Page: {{ currentPage() }}</p>
    </div>
  `
})
export class InfiniteScrollComponent {
  currentPage = signal(1);
  isLoading = signal(false);
  newPageData = signal<Item[] | null>(null);

  // Accumulate items from multiple pages
  allItems = linkedSignal<Item[]>({
    source: () => this.newPageData(),
    computation: (newData, previous) => {
      const existingItems = previous?.value || [];

      if (!newData) {
        return existingItems;
      }

      // Append new items to existing list
      return [...existingItems, ...newData];
    }
  });

  constructor() {
    // Load initial data
    this.loadPage(1);

    // Auto-load next page when page number changes
    effect(() => {
      const page = this.currentPage();
      if (page > 1) {
        this.loadPage(page);
      }
    });
  }

  async loadPage(page: number) {
    this.isLoading.set(true);

    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 1000));

      const newItems: Item[] = Array.from({ length: 20 }, (_, i) => ({
        id: `page-${page}-item-${i}`,
        name: `Item ${(page - 1) * 20 + i + 1}`
      }));

      this.newPageData.set(newItems);
    } catch (error) {
      console.error('Failed to load page:', error);
    } finally {
      this.isLoading.set(false);
    }
  }

  onScroll(event: Event) {
    const element = event.target as HTMLElement;
    const threshold = 100; // pixels from bottom

    if (element.scrollTop + element.clientHeight >= element.scrollHeight - threshold) {
      if (!this.isLoading()) {
        this.currentPage.update(page => page + 1);
      }
    }
  }
}
```

#### linkedSignal vs computed

Understanding when to use `linkedSignal` vs `computed`:

```typescript
@Component({
  selector: 'app-comparison-example'
})
export class ComparisonExampleComponent {
  items = signal<string[]>(['apple', 'banana']);

  // ✅ Use computed for pure derived state (read-only)
  itemCount = computed(() => this.items().length);
  uppercaseItems = computed(() => this.items().map(item => item.toUpperCase()));

  // ✅ Use linkedSignal for derived state that can be manually overridden
  selectedItems = linkedSignal<string[]>({
    source: () => this.items(),
    computation: (currentItems, previous) => {
      const previousSelection = previous?.value || [];
      // Keep only items that still exist
      return previousSelection.filter(item => currentItems.includes(item));
    }
  });

  // ❌ Don't use linkedSignal for simple transformations
  // badExample = linkedSignal({
  //   source: () => this.items(),
  //   computation: (items) => items.length // Just use computed instead
  // });

  addItem(item: string) {
    this.items.update(items => [...items, item]);
  }

  toggleSelection(item: string) {
    // This is why we use linkedSignal - we can manually modify the selection
    this.selectedItems.update(selected =>
      selected.includes(item)
        ? selected.filter(i => i !== item)
        : [...selected, item]
    );
  }
}
```

#### Best Practices for linkedSignal

1. **Use for derived state that needs manual control** - when you need both automatic updates and manual overrides
2. **Ideal for accumulating data** - building lists, histories, or aggregations over time
3. **Handle edge cases in computation** - check for null/undefined previous values
4. **Keep computations pure** - avoid side effects in the computation function
5. **Consider performance** - linkedSignal creates an additional signal, use sparingly for simple cases
6. **Prefer computed for read-only derived state** - only use linkedSignal when you need writability

### Reading Without Tracking Dependencies

Use `untracked()` to read signals without creating dependencies:

```typescript
import { effect, signal, untracked } from '@angular/core';

const user = signal({ name: 'John', id: 1 });
const debugMode = signal(false);

effect(() => {
  const currentUser = user();

  // Read debugMode without making it a dependency
  if (untracked(debugMode)) {
    console.log('User changed:', currentUser);
  }

  // This effect only runs when user() changes, not when debugMode changes
});
```

### Performance Optimization

#### Why OnPush with Signals?

You might wonder: "If signals provide fine-grained reactivity, why do I still need OnPush?" This is a common question with an important answer.

**Signals Don't Eliminate Change Detection - They Optimize It**

Even with signals, Angular still runs change detection cycles. OnPush ensures your component is only checked when necessary:

```typescript
// ❌ Without OnPush - checked on every CD cycle
@Component({
  selector: 'app-counter',
  // Default change detection - component checked on every CD cycle
  template: `<p>Count: {{ count() }}</p>`
})
export class CounterComponent {
  count = signal(0); // Signal updates still trigger full CD tree traversal

  increment() {
    this.count.update(c => c + 1);
    // Even though only this signal changed, Angular might check
    // this component during unrelated change detection cycles
  }
}

// ✅ With OnPush - only checked when necessary
@Component({
  selector: 'app-counter',
  changeDetection: ChangeDetectionStrategy.OnPush, // Much more efficient
  template: `<p>Count: {{ count() }}</p>`
})
export class CounterComponent {
  count = signal(0); // Signal updates trigger targeted checks

  increment() {
    this.count.update(c => c + 1);
    // Component only checked when this signal changes or inputs change
  }
}
```

**Performance Comparison**

```typescript
// Large application scenario
@Component({
  selector: 'app-dashboard',
  changeDetection: ChangeDetectionStrategy.OnPush, // Essential for performance
  template: `
    <div>
      <!-- 100+ components with signals -->
      @for (item of items(); track item.id) {
        <app-expensive-component [data]="item" />
      }

      <!-- This signal change only affects this component -->
      <p>Status: {{ status() }}</p>
    </div>
  `
})
export class DashboardComponent {
  items = signal<Item[]>([]);
  status = signal('loading');

  updateStatus() {
    this.status.set('ready');
    // Without OnPush: Angular checks ALL 100+ components
    // With OnPush: Angular only checks THIS component
  }
}
```

**Zoneless Mode Requirement**

In Angular's zoneless mode (the future), OnPush becomes **mandatory** for proper signal integration:

```typescript
// Zoneless application bootstrap
bootstrapApplication(AppComponent, {
  providers: [
    provideZonelessChangeDetection(), // Zoneless mode
    // In zoneless mode, OnPush is required for components with signals
  ]
});

@Component({
  selector: 'app-zoneless-component',
  changeDetection: ChangeDetectionStrategy.OnPush, // Required in zoneless mode
  template: `
    <div>
      <p>Timer: {{ timer() }}</p>
      <button (click)="start()">Start</button>
    </div>
  `
})
export class ZonelessComponent {
  timer = signal(0);

  start() {
    setInterval(() => {
      this.timer.update(t => t + 1);
      // In zoneless mode, only OnPush components with signals work correctly
    }, 1000);
  }
}
```

### The Bottom Line

**OnPush is still essential**, but understanding when it helps is crucial:

- ✅ **Prevents checks**: When parent doesn't check and component has no changes
- ❌ **Cannot prevent checks**: When parent component is checking
- 🎯 **Best practice**: Use OnPush + isolated component signals for optimal performance

Check this example:
https://github.com/eladcandroid/angular-zoneless-onpush/

**Mixed Component Scenarios**

Real applications often mix signals with observables. OnPush works perfectly with both:

```typescript
@Component({
  selector: 'app-mixed-data',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div>
      <!-- Signal-based data (optimal) -->
      <p>User count: {{ userCount() }}</p>
      <p>Is loading: {{ isLoading() }}</p>

      <!-- Observable-based data (still works with OnPush) -->
      <p>Server data: {{ serverData$ | async }}</p>

      <!-- Computed from both -->
      <p>Display message: {{ displayMessage() }}</p>
    </div>
  `
})
export class MixedDataComponent {
  // Signals
  userCount = signal(0);
  isLoading = signal(false);

  // Observable
  serverData$ = this.http.get('/api/data');

  // Computed signal that can use both patterns
  displayMessage = computed(() => {
    if (this.isLoading()) return 'Loading...';
    return `Found ${this.userCount()} users`;
  });
}
```

**Key Takeaways**

1. **OnPush is required for zoneless mode** - Angular's future direction
2. **Prevents unnecessary checks** even with signals in zone.js mode
3. **No downsides** - signals work perfectly with OnPush
4. **Best practice** for all modern Angular components
5. **Performance critical** in large applications with many components

**The Bottom Line**: Signals make change detection **smarter** (fine-grained reactivity), but OnPush makes it **more selective** (fewer unnecessary checks). Together, they provide optimal performance.

#### OnPush Change Detection and Zoneless Compatibility

Here's how OnPush and signals work together in practice:

```typescript
import { Component, ChangeDetectionStrategy, signal, computed } from '@angular/core';

@Component({
  selector: 'app-optimized',
  changeDetection: ChangeDetectionStrategy.OnPush, // Required for optimal signal performance
  template: `
    <div>
      <p>Count: {{ count() }}</p>
      <p>Double: {{ doubleCount() }}</p>
    </div>
  `
})
export class OptimizedComponent {
  count = signal(0);
  doubleCount = computed(() => this.count() * 2);

  increment() {
    this.count.update(c => c + 1);
    // Angular automatically marks component for check when signals change
    // Works seamlessly in both zone.js and zoneless modes
  }
}
```

#### Avoiding Unnecessary Computations

```typescript
//  Good - computed signals are memoized
const expensiveValue = computed(() => {
  return expensiveCalculation(data());
});

// L Avoid - recalculates on every template evaluation
getExpensiveValue() {
  return expensiveCalculation(this.data());
}
```

### RxJS Interoperability

Angular provides several utilities to seamlessly integrate signals with RxJS observables, allowing you to migrate gradually and use both reactive systems together.

#### toSignal - Convert Observable to Signal

```typescript
import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { toSignal } from '@angular/core/rxjs-interop';
import { interval, map, startWith } from 'rxjs';

@Component({
  selector: 'app-signal-conversion',
  template: `
    <div>
      <h3>User Data</h3>
      @if (userData()) {
        <p>Name: {{ userData()?.name }}</p>
        <p>Email: {{ userData()?.email }}</p>
      } @else {
        <p>Loading...</p>
      }

      <h3>Timer</h3>
      <p>Current time: {{ currentTime() }}</p>

      <h3>Counter</h3>
      <p>Count: {{ counter() }}</p>
    </div>
  `
})
export class SignalConversionComponent {
  private http = inject(HttpClient);

  // Convert HTTP observable to signal with initial value
  userData = toSignal(
    this.http.get<User>('/api/user'),
    { initialValue: null }
  );

  // Convert timer observable to signal
  currentTime = toSignal(
    interval(1000).pipe(
      map(() => new Date().toLocaleTimeString())
    ),
    { initialValue: new Date().toLocaleTimeString() }
  );

  // Convert counter with initial value using startWith
  counter = toSignal(
    interval(1000).pipe(
      startWith(0),
      map(count => count + 1)
    )
  );

  // Handle errors with toSignal
  weatherData = toSignal(
    this.http.get('/api/weather').pipe(
      catchError(error => {
        console.error('Weather API error:', error);
        return of({ temperature: 'N/A', condition: 'Unknown' });
      })
    ),
    { initialValue: { temperature: 'Loading...', condition: 'Loading...' } }
  );
}
```

#### toObservable - Convert Signal to Observable

```typescript
import { Component, signal } from '@angular/core';
import { toObservable } from '@angular/core/rxjs-interop';
import { debounceTime, distinctUntilChanged, switchMap } from 'rxjs';

@Component({
  selector: 'app-observable-conversion',
  template: `
    <div>
      <input
        [value]="searchTerm()"
        (input)="updateSearchTerm($event)"
        placeholder="Search users...">

      <div class="results">
        @for (user of searchResults(); track user.id) {
          <div class="user">{{ user.name }}</div>
        }
      </div>

      <div class="analytics">
        <p>Search performed {{ searchCount() }} times</p>
        <p>Last search: {{ lastSearchTime() | date:'medium' }}</p>
      </div>
    </div>
  `
})
export class ObservableConversionComponent {
  searchTerm = signal('');
  searchCount = signal(0);
  lastSearchTime = signal<Date | null>(null);

  // Convert signal to observable for complex RxJS operations
  searchTerm$ = toObservable(this.searchTerm);

  // Use RxJS operators on the signal-derived observable
  searchResults = toSignal(
    this.searchTerm$.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(term => {
        if (!term.trim()) {
          return of([]);
        }

        this.searchCount.update(count => count + 1);
        this.lastSearchTime.set(new Date());

        return this.http.get<User[]>(`/api/users/search?q=${term}`);
      })
    ),
    { initialValue: [] as User[] }
  );

  updateSearchTerm(event: Event) {
    const target = event.target as HTMLInputElement;
    this.searchTerm.set(target.value);
  }

  constructor(private http: HttpClient) {}
}
```

#### outputFromObservable - Convert Observable to Output

```typescript
import { Component, outputFromObservable } from '@angular/core';
import { Subject, fromEvent, throttleTime, map } from 'rxjs';

@Component({
  selector: 'app-scroll-detector',
  template: `
    <div
      #scrollArea
      class="scroll-area"
      (scroll)="onScroll($event)">
      <div class="content">
        <!-- Long content that requires scrolling -->
        @for (item of items; track $index) {
          <div class="item">Item {{ item }}</div>
        }
      </div>
    </div>
  `
})
export class ScrollDetectorComponent implements AfterViewInit {
  @ViewChild('scrollArea') scrollArea!: ElementRef<HTMLDivElement>;

  items = Array.from({ length: 100 }, (_, i) => i + 1);

  private scrollSubject = new Subject<Event>();

  // Convert observable to output for parent components
  scrollPosition = outputFromObservable(
    this.scrollSubject.pipe(
      throttleTime(100),
      map(event => {
        const element = event.target as HTMLElement;
        return {
          scrollTop: element.scrollTop,
          scrollHeight: element.scrollHeight,
          clientHeight: element.clientHeight,
          percentage: (element.scrollTop / (element.scrollHeight - element.clientHeight)) * 100
        };
      })
    )
  );

  // Another example - mouse movement tracking
  mousePosition = outputFromObservable(
    fromEvent<MouseEvent>(document, 'mousemove').pipe(
      throttleTime(50),
      map(event => ({ x: event.clientX, y: event.clientY }))
    )
  );

  onScroll(event: Event) {
    this.scrollSubject.next(event);
  }

  ngAfterViewInit() {
    // Can also create outputs from DOM events directly
    const resizeOutput = outputFromObservable(
      fromEvent(window, 'resize').pipe(
        throttleTime(250),
        map(() => ({
          width: window.innerWidth,
          height: window.innerHeight
        }))
      )
    );
  }
}

// Usage in parent component:
@Component({
  template: `
    <app-scroll-detector
      (scrollPosition)="onScrollUpdate($event)"
      (mousePosition)="onMouseMove($event)">
    </app-scroll-detector>

    <div class="status">
      <p>Scroll: {{ scrollInfo?.percentage.toFixed(1) }}%</p>
      <p>Mouse: {{ mouseInfo?.x }}, {{ mouseInfo?.y }}</p>
    </div>
  `
})
export class ParentComponent {
  scrollInfo: any = null;
  mouseInfo: any = null;

  onScrollUpdate(info: any) {
    this.scrollInfo = info;
  }

  onMouseMove(position: any) {
    this.mouseInfo = position;
  }
}
```

#### outputToObservable - Convert Output to Observable

```typescript
import { Component, ViewChild, AfterViewInit } from '@angular/core';
import { outputToObservable } from '@angular/core/rxjs-interop';
import { debounceTime, distinctUntilChanged } from 'rxjs';

@Component({
  selector: 'app-form-observer',
  template: `
    <div>
      <app-custom-input
        #customInput
        [label]="'Username'"
        [placeholder]="'Enter username'"
        (valueChange)="onValueChange($event)"
        (validationChange)="onValidationChange($event)">
      </app-custom-input>

      <div class="feedback">
        <p>Real-time: {{ realtimeValue }}</p>
        <p>Debounced: {{ debouncedValue }}</p>
        <p>Valid: {{ isValid }}</p>
      </div>
    </div>
  `
})
export class FormObserverComponent implements AfterViewInit {
  @ViewChild('customInput') customInput!: CustomInputComponent;

  realtimeValue = '';
  debouncedValue = '';
  isValid = true;

  ngAfterViewInit() {
    // Convert output to observable for advanced RxJS operations
    const valueChange$ = outputToObservable(this.customInput.valueChange);
    const validationChange$ = outputToObservable(this.customInput.validationChange);

    // Debounce value changes
    valueChange$.pipe(
      debounceTime(500),
      distinctUntilChanged()
    ).subscribe(value => {
      this.debouncedValue = value;
      console.log('Debounced value:', value);
    });

    // React to validation changes
    validationChange$.subscribe(validation => {
      this.isValid = validation.isValid;
    });

    // Combine multiple outputs
    combineLatest([valueChange$, validationChange$]).subscribe(
      ([value, validation]) => {
        console.log('Combined update:', { value, validation });
      }
    );
  }

  onValueChange(value: string) {
    this.realtimeValue = value;
  }

  onValidationChange(validation: any) {
    // Handle immediate validation feedback
  }
}
```

#### Advanced Interoperability Patterns

```typescript
@Component({
  selector: 'app-hybrid-data-flow',
  template: `
    <div>
      <button (click)="refresh()">Refresh</button>

      @if (isLoading()) {
        <div>Loading...</div>
      } @else {
        <div class="data-grid">
          @for (item of processedData(); track item.id) {
            <div class="item">{{ item.displayName }}</div>
          }
        </div>
      }

      <div class="stats">
        <p>Refresh count: {{ refreshCount() }}</p>
        <p>Last update: {{ lastUpdate() | date:'medium' }}</p>
      </div>
    </div>
  `
})
export class HybridDataFlowComponent implements OnInit {
  // Signals for simple state
  refreshCount = signal(0);
  lastUpdate = signal<Date | null>(null);
  isLoading = signal(false);

  // Manual refresh trigger
  private refreshTrigger = new Subject<void>();

  // Complex data fetching using observables
  private data$ = this.refreshTrigger.pipe(
    startWith(null), // Initial load
    tap(() => this.isLoading.set(true)),
    switchMap(() =>
      this.http.get<RawData[]>('/api/data').pipe(
        tap(() => {
          this.refreshCount.update(count => count + 1);
          this.lastUpdate.set(new Date());
          this.isLoading.set(false);
        }),
        catchError(error => {
          this.isLoading.set(false);
          console.error('Data fetch error:', error);
          return of([]);
        })
      )
    ),
    shareReplay(1)
  );

  // Convert observable back to signal for template use
  rawData = toSignal(this.data$, { initialValue: [] as RawData[] });

  // Use computed signal for data transformation
  processedData = computed(() =>
    this.rawData().map(item => ({
      ...item,
      displayName: `${item.name} (${item.category})`
    }))
  );

  // Convert signals back to observables for side effects
  refreshCount$ = toObservable(this.refreshCount);

  ngOnInit() {
    // Use RxJS for complex side effects
    this.refreshCount$.pipe(
      filter(count => count > 0),
      debounceTime(1000)
    ).subscribe(count => {
      console.log(`Data refreshed ${count} times`);
      // Analytics or logging
    });
  }

  refresh() {
    this.refreshTrigger.next();
  }

  constructor(private http: HttpClient) {}
}
```

#### Migration Strategies

```typescript
// Step 1: Start with observables, gradually convert to signals
@Component({})
export class MigrationExampleComponent {
  // Legacy: Pure observable approach
  legacyData$ = this.http.get('/api/legacy');

  // Hybrid: Observable converted to signal
  hybridData = toSignal(this.http.get('/api/hybrid'), { initialValue: null });

  // Modern: Pure signal approach with resource
  modernData = resource({
    loader: () => this.http.get('/api/modern').toPromise()
  });

  // Gradual migration of computed values
  // Old: Observable-based computed
  legacyCount$ = this.legacyData$.pipe(
    map(data => data?.items?.length || 0)
  );

  // New: Signal-based computed
  hybridCount = computed(() => this.hybridData()?.items?.length || 0);
  modernCount = computed(() => this.modernData.value()?.items?.length || 0);
}
```

#### Best Practices for RxJS Integration

1. **Use toSignal for converting observables** that you primarily read in templates
2. **Use toObservable for complex RxJS operations** on signal data
3. **Use outputFromObservable** for creating reactive outputs from observables
4. **Use outputToObservable** when you need RxJS operators on component outputs
5. **Gradual migration** - start with critical paths, migrate incrementally
6. **Prefer signals for simple state** - use observables for complex async flows
7. **Always provide initialValue** with toSignal to avoid undefined states
8. **Combine both approaches** - signals for state, observables for streams

### Best Practices Summary

1. **Always use OnPush change detection** with signal-based components (see "Why OnPush with Signals?" section for detailed explanation)
2. **Prefer computed over methods** for derived values
3. **Use effects sparingly** - only for side effects
4. **Keep signals focused** - one responsibility per signal
5. **Use model inputs** for two-way binding components
6. **Leverage TypeScript** for type safety
7. **Avoid complex equality functions** unless necessary
8. **Use untracked()** when reading signals without dependencies
9. **Enable zoneless mode** for maximum performance with signals
10. **Never use manual change detection** with signals - they handle reactivity automatically
11. **Understand that signals optimize change detection, they don't eliminate it** - OnPush is still essential
12. **Isolate component signals** - avoid sharing signals across parent/child when possible for maximum OnPush benefits
13. **Remember OnPush limitations** - child components with OnPush still check when parent checks

## Practical Examples

### Todo List with Statistics

```typescript
import { Component, signal, computed, ChangeDetectionStrategy } from '@angular/core';

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

@Component({
  selector: 'app-todo-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="todo-app">
      <h2>Todo List</h2>

      <div class="stats">
        <p>Total: {{ totalCount() }}</p>
        <p>Completed: {{ completedCount() }}</p>
        <p>Remaining: {{ remainingCount() }}</p>
        <p>Progress: {{ progressPercentage() }}%</p>
      </div>

      <div class="add-todo">
        <input
          [(ngModel)]="newTodoText"
          (keyup.enter)="addTodo()"
          placeholder="Add new todo">
        <button (click)="addTodo()">Add</button>
      </div>

      <div class="todo-list">
        @for (todo of todos(); track todo.id) {
          <div class="todo-item">
            <input
              type="checkbox"
              [checked]="todo.completed"
              (change)="toggleTodo(todo.id)">
            <span [class.completed]="todo.completed">{{ todo.text }}</span>
            <button (click)="removeTodo(todo.id)">Delete</button>
          </div>
        }
      </div>

      <div class="filters">
        <button
          [class.active]="filter() === 'all'"
          (click)="setFilter('all')">All</button>
        <button
          [class.active]="filter() === 'active'"
          (click)="setFilter('active')">Active</button>
        <button
          [class.active]="filter() === 'completed'"
          (click)="setFilter('completed')">Completed</button>
      </div>

      <div class="filtered-todos">
        @for (todo of filteredTodos(); track todo.id) {
          <div class="todo-item">
            <span [class.completed]="todo.completed">{{ todo.text }}</span>
          </div>
        }
      </div>
    </div>
  `
})
export class TodoListComponent {
  // State signals
  todos = signal<Todo[]>([
    { id: 1, text: 'Learn Angular Signals', completed: false },
    { id: 2, text: 'Build a todo app', completed: true },
    { id: 3, text: 'Write tests', completed: false }
  ]);

  newTodoText = signal('');
  filter = signal<'all' | 'active' | 'completed'>('all');

  // Computed statistics
  totalCount = computed(() => this.todos().length);

  completedCount = computed(() =>
    this.todos().filter(todo => todo.completed).length
  );

  remainingCount = computed(() =>
    this.todos().filter(todo => !todo.completed).length
  );

  progressPercentage = computed(() => {
    const total = this.totalCount();
    const completed = this.completedCount();
    return total === 0 ? 0 : Math.round((completed / total) * 100);
  });

  // Filtered todos based on current filter
  filteredTodos = computed(() => {
    const currentFilter = this.filter();
    const allTodos = this.todos();

    switch (currentFilter) {
      case 'active':
        return allTodos.filter(todo => !todo.completed);
      case 'completed':
        return allTodos.filter(todo => todo.completed);
      default:
        return allTodos;
    }
  });

  // Actions
  addTodo() {
    const text = this.newTodoText().trim();
    if (text) {
      this.todos.update(todos => [
        ...todos,
        {
          id: Date.now(),
          text,
          completed: false
        }
      ]);
      this.newTodoText.set('');
    }
  }

  toggleTodo(id: number) {
    this.todos.update(todos =>
      todos.map(todo =>
        todo.id === id
          ? { ...todo, completed: !todo.completed }
          : todo
      )
    );
  }

  removeTodo(id: number) {
    this.todos.update(todos =>
      todos.filter(todo => todo.id !== id)
    );
  }

  setFilter(newFilter: 'all' | 'active' | 'completed') {
    this.filter.set(newFilter);
  }
}
```

### User Profile Form with Validation

```typescript
import { Component, signal, computed, effect } from '@angular/core';

interface User {
  firstName: string;
  lastName: string;
  email: string;
  age: number;
}

@Component({
  selector: 'app-user-form',
  template: `
    <form class="user-form">
      <h2>User Profile</h2>

      <div class="form-group">
        <label>First Name</label>
        <input
          [value]="firstName()"
          (input)="updateFirstName($event)"
          [class.error]="firstNameError()">
        @if (firstNameError()) {
          <span class="error-message">{{ firstNameError() }}</span>
        }
      </div>

      <div class="form-group">
        <label>Last Name</label>
        <input
          [value]="lastName()"
          (input)="updateLastName($event)"
          [class.error]="lastNameError()">
        @if (lastNameError()) {
          <span class="error-message">{{ lastNameError() }}</span>
        }
      </div>

      <div class="form-group">
        <label>Email</label>
        <input
          type="email"
          [value]="email()"
          (input)="updateEmail($event)"
          [class.error]="emailError()">
        @if (emailError()) {
          <span class="error-message">{{ emailError() }}</span>
        }
      </div>

      <div class="form-group">
        <label>Age</label>
        <input
          type="number"
          [value]="age()"
          (input)="updateAge($event)"
          [class.error]="ageError()">
        @if (ageError()) {
          <span class="error-message">{{ ageError() }}</span>
        }
      </div>

      <div class="form-summary">
        <p>Full Name: {{ fullName() }}</p>
        <p>Form Valid: {{ isFormValid() ? 'Yes' : 'No' }}</p>
        <p>Errors: {{ errorCount() }}</p>
      </div>

      <button
        [disabled]="!isFormValid()"
        (click)="saveUser()">
        Save User
      </button>
    </form>
  `
})
export class UserFormComponent {
  // Form field signals
  firstName = signal('');
  lastName = signal('');
  email = signal('');
  age = signal(0);

  // Validation computed signals
  firstNameError = computed(() => {
    const name = this.firstName().trim();
    if (!name) return 'First name is required';
    if (name.length < 2) return 'First name must be at least 2 characters';
    return null;
  });

  lastNameError = computed(() => {
    const name = this.lastName().trim();
    if (!name) return 'Last name is required';
    if (name.length < 2) return 'Last name must be at least 2 characters';
    return null;
  });

  emailError = computed(() => {
    const email = this.email().trim();
    if (!email) return 'Email is required';

    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) return 'Invalid email format';

    return null;
  });

  ageError = computed(() => {
    const age = this.age();
    if (age < 1) return 'Age must be at least 1';
    if (age > 120) return 'Age must be less than 120';
    return null;
  });

  // Computed derived values
  fullName = computed(() =>
    `${this.firstName().trim()} ${this.lastName().trim()}`.trim()
  );

  errorCount = computed(() => {
    const errors = [
      this.firstNameError(),
      this.lastNameError(),
      this.emailError(),
      this.ageError()
    ].filter(error => error !== null);

    return errors.length;
  });

  isFormValid = computed(() => this.errorCount() === 0);

  constructor() {
    // Effect for auto-save to localStorage
    effect(() => {
      const user: User = {
        firstName: this.firstName(),
        lastName: this.lastName(),
        email: this.email(),
        age: this.age()
      };

      localStorage.setItem('userFormDraft', JSON.stringify(user));
    });

    // Load from localStorage on init
    this.loadFromStorage();
  }

  updateFirstName(event: Event) {
    const target = event.target as HTMLInputElement;
    this.firstName.set(target.value);
  }

  updateLastName(event: Event) {
    const target = event.target as HTMLInputElement;
    this.lastName.set(target.value);
  }

  updateEmail(event: Event) {
    const target = event.target as HTMLInputElement;
    this.email.set(target.value);
  }

  updateAge(event: Event) {
    const target = event.target as HTMLInputElement;
    this.age.set(Number(target.value));
  }

  saveUser() {
    if (this.isFormValid()) {
      const user: User = {
        firstName: this.firstName(),
        lastName: this.lastName(),
        email: this.email(),
        age: this.age()
      };

      console.log('Saving user:', user);
      // Here you would typically call a service to save the user

      // Clear the form
      this.clearForm();
    }
  }

  clearForm() {
    this.firstName.set('');
    this.lastName.set('');
    this.email.set('');
    this.age.set(0);
  }

  loadFromStorage() {
    const saved = localStorage.getItem('userFormDraft');
    if (saved) {
      try {
        const user: User = JSON.parse(saved);
        this.firstName.set(user.firstName || '');
        this.lastName.set(user.lastName || '');
        this.email.set(user.email || '');
        this.age.set(user.age || 0);
      } catch (error) {
        console.error('Error loading from storage:', error);
      }
    }
  }
}
```

### Shopping Cart with Local Storage

```typescript
import { Component, signal, computed, effect } from '@angular/core';

interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
}

interface CartItem extends Product {
  quantity: number;
}

@Component({
  selector: 'app-shopping-cart',
  template: `
    <div class="shopping-app">
      <header class="cart-header">
        <h1>Shopping Cart</h1>
        <div class="cart-summary">
          <span>Items: {{ totalItems() }}</span>
          <span>Total: ${{ totalPrice() }}</span>
          <button (click)="toggleCart()">
            {{ showCart() ? 'Hide' : 'Show' }} Cart
          </button>
        </div>
      </header>

      <div class="products-grid">
        <h2>Available Products</h2>
        @for (product of availableProducts(); track product.id) {
          <div class="product-card">
            <img [src]="product.image" [alt]="product.name">
            <h3>{{ product.name }}</h3>
            <p>${{ product.price }}</p>
            <button (click)="addToCart(product)">Add to Cart</button>
          </div>
        }
      </div>

      @if (showCart()) {
        <div class="cart-panel">
          <h2>Shopping Cart</h2>

          @if (cartItems().length === 0) {
            <p>Your cart is empty</p>
          } @else {
            @for (item of cartItems(); track item.id) {
              <div class="cart-item">
                <img [src]="item.image" [alt]="item.name">
                <div class="item-details">
                  <h4>{{ item.name }}</h4>
                  <p>${{ item.price }} each</p>
                </div>
                <div class="quantity-controls">
                  <button (click)="decreaseQuantity(item.id)">-</button>
                  <span>{{ item.quantity }}</span>
                  <button (click)="increaseQuantity(item.id)">+</button>
                </div>
                <div class="item-total">
                  ${{ getItemTotal(item) }}
                </div>
                <button (click)="removeFromCart(item.id)" class="remove">�</button>
              </div>
            }

            <div class="cart-totals">
              <p>Subtotal: ${{ subtotal() }}</p>
              <p>Tax (8%): ${{ tax() }}</p>
              <p>Shipping: ${{ shipping() }}</p>
              <h3>Total: ${{ totalPrice() }}</h3>
            </div>

            <div class="cart-actions">
              <button (click)="clearCart()" class="clear">Clear Cart</button>
              <button
                [disabled]="cartItems().length === 0"
                (click)="checkout()"
                class="checkout">
                Checkout
              </button>
            </div>
          }
        </div>
      }
    </div>
  `
})
export class ShoppingCartComponent {
  // Available products
  availableProducts = signal<Product[]>([
    { id: 1, name: 'Laptop', price: 999, image: '/assets/laptop.jpg' },
    { id: 2, name: 'Mouse', price: 25, image: '/assets/mouse.jpg' },
    { id: 3, name: 'Keyboard', price: 75, image: '/assets/keyboard.jpg' },
    { id: 4, name: 'Monitor', price: 200, image: '/assets/monitor.jpg' }
  ]);

  // Cart state
  cartItems = signal<CartItem[]>([]);
  showCart = signal(false);

  // Computed values
  totalItems = computed(() =>
    this.cartItems().reduce((total, item) => total + item.quantity, 0)
  );

  subtotal = computed(() =>
    this.cartItems().reduce((total, item) => total + (item.price * item.quantity), 0)
  );

  tax = computed(() =>
    Math.round(this.subtotal() * 0.08 * 100) / 100
  );

  shipping = computed(() => {
    const subtotal = this.subtotal();
    if (subtotal === 0) return 0;
    if (subtotal > 100) return 0; // Free shipping over $100
    return 10;
  });

  totalPrice = computed(() =>
    Math.round((this.subtotal() + this.tax() + this.shipping()) * 100) / 100
  );

  constructor() {
    // Auto-save cart to localStorage
    effect(() => {
      const items = this.cartItems();
      localStorage.setItem('shopping-cart', JSON.stringify(items));
    });

    // Load cart from localStorage on init
    this.loadCartFromStorage();
  }

  addToCart(product: Product) {
    this.cartItems.update(items => {
      const existingItem = items.find(item => item.id === product.id);

      if (existingItem) {
        return items.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      } else {
        return [...items, { ...product, quantity: 1 }];
      }
    });
  }

  removeFromCart(productId: number) {
    this.cartItems.update(items =>
      items.filter(item => item.id !== productId)
    );
  }

  increaseQuantity(productId: number) {
    this.cartItems.update(items =>
      items.map(item =>
        item.id === productId
          ? { ...item, quantity: item.quantity + 1 }
          : item
      )
    );
  }

  decreaseQuantity(productId: number) {
    this.cartItems.update(items =>
      items.map(item =>
        item.id === productId
          ? { ...item, quantity: Math.max(1, item.quantity - 1) }
          : item
      )
    );
  }

  getItemTotal(item: CartItem): number {
    return Math.round(item.price * item.quantity * 100) / 100;
  }

  clearCart() {
    this.cartItems.set([]);
  }

  toggleCart() {
    this.showCart.update(show => !show);
  }

  checkout() {
    const items = this.cartItems();
    const total = this.totalPrice();

    console.log('Processing checkout:', { items, total });

    // Simulate checkout process
    alert(`Checkout complete! Total: $${total}`);
    this.clearCart();
    this.showCart.set(false);
  }

  loadCartFromStorage() {
    const saved = localStorage.getItem('shopping-cart');
    if (saved) {
      try {
        const items: CartItem[] = JSON.parse(saved);
        this.cartItems.set(items);
      } catch (error) {
        console.error('Error loading cart from storage:', error);
      }
    }
  }
}
```

These examples demonstrate real-world usage of Angular Signals, showing how they can simplify state management, improve performance, and create more maintainable reactive applications.

---

## Summary

Angular Signals provide a stable, powerful reactive approach to state management that offers:

- **Better Performance**: Fine-grained reactivity that only updates what actually changed
- **Improved Developer Experience**: Type-safe, predictable state management
- **Simplified Code**: Less boilerplate compared to traditional reactive patterns
- **Automatic Optimization**: Built-in memoization and lazy evaluation
- **Zoneless Compatibility**: Essential for Angular's zoneless change detection mode

Key takeaways:
1. Use `signal()` for basic reactive state
2. Use `computed()` for derived values
3. Use `effect()` sparingly for side effects
4. Use `input()` and `model()` for component communication
5. Always enable OnPush change detection with signals
6. Keep signals focused and avoid complex computations in effects
7. Signals are stable as of Angular 20 and ready for production use
8. Perfect foundation for zoneless Angular applications

Signals represent the present and future of reactive programming in Angular and provide a solid foundation for building modern, performant web applications with optimal change detection performance.