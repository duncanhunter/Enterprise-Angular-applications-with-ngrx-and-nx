# Part 10 - ngrx selectors

Lets add a main menu to our customer portal to show the name of the logged in user.

1. #### Add a new layout lib to a customer-portal directory

```
ng g lib layout --directory=customer-portal
```

#### 2. Add a layout container component

```
ng g c containers/layout -a=customer-portal/layout
```

#### 3. Add Material and Router module

_**libs/customer-portal/layout/src/layout.module.ts**_

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { MaterialModule } from '@demo-app/material';
import { LayoutComponent } from './containers/layout/layout.component';
import { RouterModule } from '@angular/router';

const COMPONENTS = [LayoutComponent];
@NgModule({
  imports: [CommonModule, MaterialModule, RouterModule],
  declarations: [COMPONENTS],
  exports: [COMPONENTS]
})
export class LayoutModule {
}
```

#### 3. Add a material toolbar

_**libs/customer-portal/layout/src/containers/layout/layout.component.html**_

```ts
<mat-toolbar color="primary" fxLayout="row">
  <span>Customer Portal</span>
  <div class="right-nav">
    <span>{{(user$ | async)?.username}}</span>
  </div>
</mat-toolbar>
<ng-content></ng-content>
```

* Add styles to styles.scss

_**apps/customer-portal/src/styles.scss**_

```css
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';

body {
    margin: 0;
}
```

_**libs/customer-portal/layout/src/containers/layout/layout.component.scss**_

```css
.right-nav {
    margin-left: auto;
}
```

```ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { AuthState } from '@demo-app/auth';
import { User } from '@demo-app/data-models';
import { Observable } from 'rxjs/Observable';

@Component({
  selector: 'app-layout',
  templateUrl: './layout.component.html',
  styleUrls: ['./layout.component.scss']
})
export class LayoutComponent implements OnInit {
  user$: Observable<User>;
  constructor(private store: Store<AuthState>) { }

  ngOnInit() {
    this.user$ = this.store.select(state => state.auth.user);
  }

}
```

```ts
import { LayoutModule } from '@demo-app/customer-portal/layout';

@NgModule({
  imports: [
   ...
    LayoutModule
    ...
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

```html
<app-layout>
    <router-outlet></router-outlet>
</app-layout>
```

add index.ts

```ts
import { createSelector, createFeatureSelector } from '@ngrx/store';
import { Auth } from './auth.interfaces';

export const getAuthState = createFeatureSelector<Auth>('auth');
export const getUser = createSelector(getAuthState, state => state.user);
```

```ts
export { AuthModule , authRoutes } from './src/auth.module';
export { AuthGuard } from './src/guards/auth.guard';
export { AuthState } from './src/+state/auth.interfaces';
export * from './src/+state';
```

```ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { AuthState, getUser } from '@demo-app/auth';
import { User } from '@demo-app/data-models';
import { Observable } from 'rxjs/Observable';

@Component({
  selector: 'app-layout',
  templateUrl: './layout.component.html',
  styleUrls: ['./layout.component.scss']
})
export class LayoutComponent implements OnInit {
  user$: Observable<User>;
  constructor(private store: Store<AuthState>) { }

  ngOnInit() {
    this.user$ = this.store.select(getUser);
  }

}
```



