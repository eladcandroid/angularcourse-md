# Topic Overview
Feature Modules are a recommended approach for organizing code in Angular that allows dividing the application into independent modules by features or areas of responsibility.
 Each module contains all the components, services, guards, and routes related to a specific feature. 
This approach improves scalability, enables efficient lazy loading, and facilitates maintenance and team collaboration. The structure creates clear separation between application parts and prevents unnecessary dependencies.

src/app/
├── core/                 # Global services, Guards, Interceptors
├── shared/               # Shared components and services
├── features/
│   ├── products/
│   │   ├── products.module.ts
│   │   ├── products-routing.module.ts
│   │   ├── components/
│   │   ├── services/
│   │   └── models/
│   └── users/
│       ├── users.module.ts
│       ├── users-routing.module.ts
│       ├── components/
│       └── services/
└── app.module.ts

```ts
// products.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ProductsRoutingModule } from './products-routing.module';
import { SharedModule } from '@app/shared/shared.module';

// Components
import { ProductListComponent } from './components/product-list/product-list.component';
import { ProductDetailComponent } from './components/product-detail/product-detail.component';

// Services
import { ProductService } from './services/product.service';

@NgModule({
  declarations: [
    ProductListComponent,
    ProductDetailComponent
  ],
  imports: [
    CommonModule,
    SharedModule,
    ProductsRoutingModule
  ],
  providers: [
    ProductService
  ]
})
export class ProductsModule { }
```

```ts
// products-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { ProductListComponent } from './components/product-list/product-list.component';
import { ProductDetailComponent } from './components/product-detail/product-detail.component';

const routes: Routes = [
  {
    path: '',
    component: ProductListComponent
  },
  {
    path: ':id',
    component: ProductDetailComponent
  }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class ProductsRoutingModule { }
```

```ts
// app-routing.module.ts
const routes: Routes = [
  {
    path: 'products',
    loadChildren: () => import('./features/products/products.module')
      .then(m => m.ProductsModule)
  },
  {
    path: 'users',
    loadChildren: () => import('./features/users/users.module')
      .then(m => m.UsersModule)
  },
  {
    path: '',
    redirectTo: '/products',
    pathMatch: 'full'
  }
];
```
