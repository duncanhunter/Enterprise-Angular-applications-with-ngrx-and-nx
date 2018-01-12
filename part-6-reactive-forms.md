# Part 6 - Reactive Forms

[https://github.com/UltimateAngular/ngx-errors](https://github.com/UltimateAngular/ngx-errors)

```
npm i @ultimate/ngxerrors
```

save injecting the formBuilder and less junk calling it.

```ts
import { FormGroup, FormControl, Validators } from '@angular/forms';

...

loginForm = new FormGroup({
    username: new FormControl('', [Validators.required]),
    password: new FormControl('', [Validators.required])
});
```

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



```
ng generate lib data-models --nomodule
```



