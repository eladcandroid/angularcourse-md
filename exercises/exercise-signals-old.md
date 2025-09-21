# Angular Signals Practice

Build on the concepts from `lesson-signals.md` by completing a focused series of signal-powered components. Work inside an Angular CLI sandbox (`npx -p @angular/cli ng new signals-sandbox`) so you can experiment safely while you iterate.

## Exercise Overview
- Create and update writable signals to manage local state.
- Derive aggregate values with `computed` signals and side effects with `effect`.
- Compose components that communicate via signal inputs, `model`, and `output` APIs.

## Task 1 – Writable Signals Warm Up
You will author a standalone `TaskCounterComponent` that keeps track of opened and completed tasks. Use writable signals and bind their values directly in the template.

### Starter Component
```ts
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-task-counter',
  standalone: true,
  templateUrl: './task-counter.component.html'
})
export class TaskCounterComponent {
  openCount = signal(0);
  completedCount = signal(0);

  addTask() {}
  completeTask() {}
  resetCounts() {}
}
```

```html
<section>
  <p>Open: {{ openCount() }}</p>
  <p>Completed: {{ completedCount() }}</p>
  <button (click)="addTask()">Add Task</button>
  <button (click)="completeTask()">Complete Task</button>
  <button (click)="resetCounts()">Reset</button>
</section>
```

### Steps
1. Update the methods to call `.update()` for incremental changes and `.set()` for resets.
2. Prevent the completed count from exceeding the open count.
3. Style the section lightly to keep counters readable when embedded in other lessons.

### Expected Outcome
- Buttons update the rendered counts immediately.
- Reset returns both counters to zero using `.set()`.
- Logic guards against negative totals.

### Self-Check
- Do you read signal values with `()` everywhere in the template?
- Can you explain the difference between `.update()` and `.set()` in your implementation?
- Does the component remain SSR-safe (no direct DOM access)?

## Task 2 – Derived State and Effects
Extend the concept by building a `CartTotalsComponent` that aggregates shopping cart data with `computed` signals and synchronizes to storage with an `effect`.

### Starter Component
```ts
import { Component, computed, effect, signal } from '@angular/core';

interface CartItem {
  id: number;
  title: string;
  price: number;
  quantity: number;
}

@Component({
  selector: 'app-cart-totals',
  standalone: true,
  templateUrl: './cart-totals.component.html'
})
export class CartTotalsComponent {
  items = signal<CartItem[]>([]);
  totalItems = computed(() => 0);
  totalCost = computed(() => 0);

  constructor() {
    effect(() => {
      const snapshot = this.items();
      localStorage.setItem('cart', JSON.stringify(snapshot));
    }, { allowSignalWrites: false });
  }

  addItem(item: CartItem) {}
  updateQuantity(id: number, delta: number) {}
}
```

### Steps
1. Replace the placeholder computed implementations so they derive totals from `items()`.
2. Enhance the effect to hydrate initial data from `localStorage` and guard against JSON parse errors.
3. Render totals in the template with Angular’s `@for` control flow to show each line item and the aggregated footer.

### Expected Outcome
- Totals recalculate only when `items()` changes.
- The effect persists the cart and avoids writes during hydration.
- Quantity updates never mutate arrays outside `signal.update`.

### Self-Check
- Do your computed signals remain pure and synchronous?
- Does the effect avoid writing to the signal it reads, preventing loops?
- Can you reload the page and observe the cart rehydrating from storage?

## Task 3 – Component Collaboration
Compose a small task board using signal-based component APIs. Create a parent `TaskBoardComponent` plus child components that communicate through signal inputs, `model`, and `output` helpers.

### Requirements
1. Parent holds `tasks = signal<Task[]>(seedTasks)` and a `selectedFilter = signal<'all' | 'open' | 'done'>('all')`.
2. `TaskListComponent` accepts filtered tasks via `input.required<Task[]>('visibleTasks')` and renders them with `@for`.
3. `TaskComposerComponent` exposes `taskDraft = model<TaskDraft>('draft')` for two-way binding of a form, emitting a completed draft back to the parent.
4. `TaskListFooterComponent` raises `filterChanged = output<'all' | 'open' | 'done'>()` when users select a different filter.
5. Parent wires child outputs to update its own signals and derives visible tasks with a `computed`.

### Self-Check
- Are all cross-component handoffs implemented with the signal-friendly APIs (`input`, `model`, `output`)?
- Do you avoid injecting `ChangeDetectorRef` because signals drive updates automatically?
- Can you articulate which computed signals depend on which writable signals?

## Stretch Goals (Optional)
- Add a `computed` that returns heatmap data (tasks completed per weekday) for future visualization.
- Introduce a debounced `effect` that syncs batched updates to a mock API service.
- Add signal-based form validation that exposes derived error messages via `computed` and clears them with `onCleanup` inside an effect.

## Supporting Snippets
- Seed data you can paste into your sandbox:
```ts
export const seedTasks = [
  { id: 1, title: 'Draft backlog', status: 'open' as const },
  { id: 2, title: 'Review signals lesson', status: 'done' as const }
];

export const seedItems: CartItem[] = [
  { id: 101, title: 'Angular Handbook', price: 39, quantity: 1 },
  { id: 102, title: 'Component Patterns', price: 29, quantity: 2 }
];
```
- Reuse the temporary sandbox command whenever you need a fresh workspace: `npx -p @angular/cli ng new signals-sandbox`.
