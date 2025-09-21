# Signals Practice Exercises

Use these exercises to turn the concepts from `lesson-signals.md` into working Angular features. Spin up a throwaway Angular workspace with `npx -p @angular/cli ng new tmp-signals --skip-git --skip-tests` so you can iterate quickly, then delete it when you're done.

## Exercise 1: Counter With Writable Signals

Build a standalone `CounterPanelComponent` that demonstrates signal creation and updates.

- Generate a component inside `src/app/counter-panel.component.ts` and enable `ChangeDetectionStrategy.OnPush`.
- Add a `count = signal(0)` field and expose `increment`, `decrement`, and `reset` methods that call `update` or `set`.
- Render the current count plus three buttons. Disable the decrement button when the count would drop below zero.
- Style the buttons with inline `class` bindings instead of `ngClass` so active and disabled states read clearly.

**Self-check**
- You never call `count = 0` directly; all mutations go through `set` or `update`.
- The template uses `@if` to show a "Great job" badge when the count is above 10.
- The component compiles in your sandbox and clicking the buttons updates the view instantly.

## Exercise 2: Derive Totals With Computed Signals

Model a mini cart summary that showcases `computed()`.

- Create a `CartSummaryComponent` with an `items = signal<CartItem[]>([])` field where `CartItem` contains `name`, `price`, and `quantity`.
- Provide an `addItem(item: CartItem)` method that uses `update` to append new entries immutably.
- Add `computed` signals for `itemCount`, `subtotal`, and `averagePrice`. Guard against division by zero in the average.
- Display a table with `@for` that lists each item plus the derived totals under it. Format currency with the `currency` pipe.

**Self-check**
- `itemCount`, `subtotal`, and `averagePrice` are read-only computed signals.
- Calling `addItem` triggers a recalculation without manual change detection.
- `@for` includes a `track` clause to avoid rendering warnings.

## Exercise 3: Persist Preferences With Effects

Create a settings panel that saves a theme preference using `effect()`.

- Define a `ThemeSettingsComponent` with a `selectedTheme = signal<'light' | 'dark' | 'contrast'>('light')`.
- Build a template with radio buttons bound to `selectedTheme()` and an `update` call inside the change handler.
- Register an effect that writes `selectedTheme()` to `localStorage` and another effect that toggles a body class via `document.body.classList`.
- Use the cleanup callback to remove any event listeners or revert DOM changes if you add them later.

**Self-check**
- Neither effect writes back to `selectedTheme`, preventing loops.
- The local storage key is namespaced (for example, `angularcourse.theme`).
- Reloading the sandbox app replays the stored value during `ngOnInit` before the template renders.

## Exercise 4: Communicate With Signal Inputs and Outputs

Connect a child list component to a parent container using signal inputs and outputs.

- Scaffold a `TodoListComponent` that exposes `todos = input.required<Todo[]>();` and `filter = input<'all' | 'active' | 'done'>('all');`.
- Provide computed signals for `visibleTodos` and `remainingCount`. Render them with `@for` and show an empty state using `@if`.
- Add outputs `todoToggled = output<number>();` and `todoCleared = output<void>();` that emit when a checkbox changes or a "Clear completed" button is pressed.
- In `TodoDashboardComponent`, manage the `Todo[]` signal state, respond to outputs, and forward a filter selection menu that updates the child input via `set`.

**Self-check**
- Parent and child communicate only through signals—no template reference variables or `ViewChild`.
- Output payloads are strongly typed and minimal (`number` for IDs, no entire arrays).
- `visibleTodos` recomputes instantly when the filter changes without manual subscriptions.

## Recap and Next Steps

You now have hands-on practice with writable signals, computed signals, effects, and signal-based component inputs and outputs. Keep experimenting by combining these patterns with routing or form controls, and review `lesson-signals.md` whenever you need a refresher.
