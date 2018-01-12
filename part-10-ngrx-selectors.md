# Part 10 - ngrx selectors

```
ng g lib layout --directory=customer-portal
```

```html
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

```
ng g c containers/layout -a=customer-portal/layout
```

```ts
<mat-toolbar color="primary" fxLayout="row">
  <span>Customer Portal</span>
  <div class="right-nav">
    <span>{{(user$ | async)?.username}}</span>
  </div>
</mat-toolbar>
<ng-content></ng-content>
```

```css
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';

body {
    margin: 0;
}
```

```css
.right-nav {
    margin-left: auto;
}
```



