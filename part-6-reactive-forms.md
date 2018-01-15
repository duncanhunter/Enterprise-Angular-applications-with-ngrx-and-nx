# Part 6 - Reactive Forms

#### 1.Add ngx-errors library to make it easier to display form errors

[https://github.com/UltimateAngular/ngx-errors](https://github.com/UltimateAngular/ngx-errors)

```
npm i @ultimate/ngxerrors
```

1. Add ravtice forms module to auth module imports

```ts
imports: [CommonModule, RouterModule, HttpClientModule, MaterialModule, ReactiveFormsModule],
```

#### 2. Add a reactive FormGroup to login form

Note: To save injecting the formBuilder and keeping this a presentational component with no injected dependancies we can just new up a simple FormGroup. You can read more about it here[ https://angular.io/api/forms/FormBuilder](https://angular.io/api/forms/FormBuilder).

_**libs/auth/src/components/login-form/login-form.component.ts**_

```ts
import { Component, OnInit, EventEmitter, Output } from '@angular/core';
import { Authenticate } from '@demo-app/data-models';
import { FormGroup, FormControl, Validators } from '@angular/forms';

@Component({
  selector: 'login-form',
  templateUrl: './login-form.component.html',
  styleUrls: ['./login-form.component.scss']
})
export class LoginFormComponent {
  @Output() submit = new EventEmitter<Authenticate>();

  loginForm = new FormGroup({
    username: new FormControl('', [Validators.required]),
    password: new FormControl('', [Validators.required])
  });

  login(username: string, password: string) {
    this.submit.emit({ username, password });
  }

}
```

* Add ngx-errors to the form

_**libs/auth/src/components/login-form/login-form.component.html**_

```html
<mat-card>
    <mat-card-title>Login</mat-card-title>
    <mat-card-content>
        <form fxLayout="column" fxLayoutAlign="center none" [formGroup]="loginForm">
            <mat-input-container>
                <input matInput placeholder="username" type="text" formControlName="username">
                <mat-error ngxErrors="username">
                    <div ngxError="required" when="touched">
                        Username is required
                    </div>
                </mat-error>
            </mat-input-container>
            <mat-input-container>
                <input matInput placeholder="password" type="text" formControlName="password">
                <mat-error ngxErrors="password">
                    <div ngxError="required" when="touched">
                        Password is required
                    </div>
                </mat-error>
            </mat-input-container>
            <button mat-raised-button (click)="login()">login</button>
        </form>
    </mat-card-content>
</mat-card>
```

_**libs/data-models/src/data-models.ts**_

```ts
export interface User {
    username: string;
    id: number;
    country: string;
    token: string
    role: string;
}
```

_**libs/data-models/index.ts**_

```ts
export { Authenticate, User } from './src/data-models';
```



