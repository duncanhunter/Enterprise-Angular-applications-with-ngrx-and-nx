# Part 5 - Thirdparty dependencies

[https://material.angular.io/](https://material.angular.io/)

```
npm install --save @angular/material @angular/cdk @angular/flex-layout
```

```
npm install --save @angular/animations
```

```ts
import {BrowserAnimationsModule} from '@angular/platform-browser/animations';

@NgModule({
  ...
  imports: [BrowserAnimationsModule],
  ...
})
export class AppModule { }
```

```
ng g lib material
```

```ts
import { NgModule } from '@angular/core';
import { FlexLayoutModule } from '@angular/flex-layout';

import {
  MatInputModule,
  MatCardModule,
  MatButtonModule,
  MatSidenavModule,
  MatListModule,
  MatIconModule,
  MatToolbarModule,
  MatProgressSpinnerModule,
  MatMenuModule
} from '@angular/material';

@NgModule({
  imports: [
    FlexLayoutModule,
    MatInputModule,
    MatCardModule,
    MatButtonModule,
    MatSidenavModule,
    MatListModule,
    MatIconModule,
    MatToolbarModule,
    MatProgressSpinnerModule,
    MatMenuModule,
  ],
  exports: [
    FlexLayoutModule,
    MatInputModule,
    MatCardModule,
    MatButtonModule,
    MatSidenavModule,
    MatListModule,
    MatIconModule,
    MatToolbarModule,
    MatProgressSpinnerModule,
    MatMenuModule,
  ]
})
export class MaterialModule {}
```

```ts
import { MaterialModule } from '@demo-app/material';

@NgModule({
  ...
imports: [CommonModule, RouterModule, HttpClientModule, MaterialModule],
  ...
})
export class AuthModule { }
```

```css
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';
```

```html
<mat-card>
    <mat-card-title>Login</mat-card-title>
    <mat-card-content>
        <form fxLayout="column" fxLayoutAlign="center none">
            <mat-input-container>
                <input matInput placeholder="username" type="text" #username>
            </mat-input-container>
            <mat-input-container>
                <input matInput placeholder="password" type="text" #password>
            </mat-input-container>
            <button mat-raised-button (click)="login(username.value, password.value)">login</button>
        </form>
    </mat-card-content>
</mat-card>
```

[https://tburleson-layouts-demos.firebaseapp.com/\#/docs](https://tburleson-layouts-demos.firebaseapp.com/#/docs)

