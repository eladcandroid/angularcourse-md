# Demo: Signals for Lean Change Detection

You will walk through two mini components to see how Angular signals cut down redundant work. Paste the snippets into an Angular CLI sandbox (`npx -p @angular/cli ng new tmp-signals-demo --standalone --style css --routing false`) and replace the default `app.component.ts` with each version in turn. Open DevTools so you can watch the `console.log` output.

## Baseline Without Signals

This legacy-style component leans on the default change detection strategy and an inline getter. Every time any parent binding updates, Angular re-enters the getter, even when the `total` has not changed.

```ts
import { ChangeDetectionStrategy, Component, Input } from '@angular/core';

@Component({
  selector: 'app-sales-card-legacy',
  template: `
    <article class="card">
      <h2>{{ title }}</h2>
      <p>Confirmed sales: {{ total }}</p>
      <p>Projected sales: {{ projectedTotal }}</p>
    </article>
  `,
})
export class SalesCardLegacyComponent {
  @Input() title = 'Legacy total';
  @Input() total = 0;

  get projectedTotal(): number {
    console.log('[legacy] recalculating projected total…');
    return Math.round(this.total * 1.2);
  }
}
```

Drop the component below into `app.component.ts` so each click causes a layout change that does *not* alter the `total` input. Notice how the console still fires on every button press.

```ts
import { Component } from '@angular/core';
import { SalesCardLegacyComponent } from './sales-card-legacy.component';

@Component({
  selector: 'app-root',
  imports: [SalesCardLegacyComponent],
  template: `
    <section class="controls">
      <button type="button" (click)="recordSale()">Record sale (+100)</button>
      <button type="button" (click)="toggleFilter()">Toggle filter label</button>
      <p>Filter: {{ activeFilter }}</p>
    </section>

    <app-sales-card-legacy [total]="totalSales" [title]="'Legacy total'"></app-sales-card-legacy>
  `,
})
export class AppComponent {
  totalSales = 0;
  activeFilter = 'all regions';

  recordSale(): void {
    this.totalSales += 100;
  }

  toggleFilter(): void {
    this.activeFilter = this.activeFilter === 'all regions' ? 'northern region' : 'all regions';
  }
}
```

Click `Toggle filter label` a few times without recording sales. The log keeps spamming `[legacy] recalculating projected total…`, proving that the getter executes even with unchanged data.

## Modern Signals Version

Here you swap the getter for a `computed()` signal. The console log now lives inside an `effect()`, so Angular only runs it when the underlying `total` signal changes. You also switch to `ChangeDetectionStrategy.OnPush` to benefit from signal-driven reactivity.

```ts
import { ChangeDetectionStrategy, Component, computed, effect, input } from '@angular/core';

@Component({
  selector: 'app-sales-card-signals',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <article class="card">
      <h2>{{ title() }}</h2>
      <p>Confirmed sales: {{ total() }}</p>
      <p>Projected sales: {{ projectedTotal() }}</p>
    </article>
  `,
})
export class SalesCardSignalsComponent {
  // `input()` returns a signal so you can read it with total().
  readonly title = input('Signals total');
  readonly total = input.required<number>();

  private readonly multiplier = 1.2;

  readonly projectedTotal = computed(() => Math.round(this.total() * this.multiplier));

  constructor() {
    effect(() => {
      console.log('[signals] projected total recomputed:', this.projectedTotal());
    });
  }
}
```

Update `app.component.ts` to inject the signal-powered card. The parent now stores its state in signals too, so Angular can schedule focused updates.

```ts
import { Component, signal } from '@angular/core';
import { SalesCardSignalsComponent } from './sales-card-signals.component';

@Component({
  selector: 'app-root',
  imports: [SalesCardSignalsComponent],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <section class="controls">
      <button type="button" (click)="recordSale()">Record sale (+100)</button>
      <button type="button" (click)="toggleFilter()">Toggle filter label</button>
      <p>Filter: {{ activeFilter() }}</p>
    </section>

    <app-sales-card-signals [total]="totalSales()" [title]="'Signals total'"></app-sales-card-signals>
  `,
})
export class AppComponent {
  readonly totalSales = signal(0);
  readonly activeFilter = signal('all regions');

  recordSale(): void {
    this.totalSales.update((value) => value + 100);
  }

  toggleFilter(): void {
    this.activeFilter.update((value) => (value === 'all regions' ? 'northern region' : 'all regions'));
  }
}
```

Now press `Toggle filter label` repeatedly. The console stays quiet. Only when you click `Record sale` (which changes the `total` signal) does `[signals] projected total recomputed:` appear, confirming that the expensive calculation runs strictly on demand.

## What You Should Notice

- The getter-based component recalculates on *every* change detection pass, even if nothing relevant changed.
- Signals cache the derived value, so Angular skips the heavy work until its dependencies shift.
- `effect()` gives you a clean, declarative place to add logging without polluting the template.
- Pairing signals with `ChangeDetectionStrategy.OnPush` produces predictable updates you can reason about.
