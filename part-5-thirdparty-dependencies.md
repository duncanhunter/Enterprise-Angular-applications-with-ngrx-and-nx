# Part 5 - Thirdparty dependencies

![](/assets/material-site.png)

[https://material.angular.io/](https://material.angular.io/)

#### 1.Install angular material, angular animations and angular flex layout

```
npm install --save @angular/material @angular/cdk @angular/flex-layout @angular/animations
```

* Add animations module to the main app module.

```ts
import {BrowserAnimationsModule} from '@angular/platform-browser/animations';

@NgModule({
  ...
  imports: [BrowserAnimationsModule],
  ...
})
export class AppModule { }
```

#### 2.Add a new nx lib to hold all the common material components we will use in our app

```
ng g lib material
```

* Add all the common material components and re-export them

_**libs/material/src/material.module.ts**_

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
  MatMenuModule,
  MatTableModule
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
    MatTableModule
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
    MatTableModule
  ]
})
export class MaterialModule {}
```

* Add material module to auth module

_**libs/auth/src/auth.module.ts**_

```ts
import { MaterialModule } from '@demo-app/material';

@NgModule({
  ...
imports: [CommonModule, RouterModule, HttpClientModule, MaterialModule],
  ...
})
export class AuthModule { }
```

#### 3. Add material default styles

#### _**apps/customer-portal/src/styles.scss**_

```css
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';
```

* Update the login-form to use material components

_**libs/auth/src/components/login-form/login-form.component.html**_

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

#### 4. Go and explore flex layout docs

[https://tburleson-layouts-demos.firebaseapp.com/\#/docs](https://tburleson-layouts-demos.firebaseapp.com/#/docs)

